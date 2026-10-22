# T3 Code: follow listeners miss a late timeline mount

This is a field test of MStack's `reproduce-bug` workflow, investigated on October 22, 2026. It adds evidence to T3 Code's existing [#20366](https://github.com/pingdotgg/t3code/issues/20366). It establishes a specific failure path, not the cause of every report in that issue.

The tested checkout matches the installed nightly `0.1.2-nightly.20261021.3499`: [b5a226e](https://github.com/pingdotgg/t3code/commit/b5a226e49c52ecab4ddc2d678a7012d97e651a6a). The client ran inside T3 Code's native Browser panel on Windows, Chromium 152.0.7977.167 / Electron 44.9.3. Source verification against a later main commit is not a runtime test of that commit.

## Reproduce

1. Start the matching checkout using `vp run dev --no-seed`, with its own ignored `.t3` state. Pair the Browser panel using the startup link. Never use the live user database.
2. Prepare a neutral transcript long enough to scroll. The recorded transcript contains a completed response listing 1 through 500, separated by blank lines. No private conversation is used.
3. Emulate a slow history load by adding `yield* Effect.sleep("3 seconds");` at the start of `threadBoundedSnapshot` in `apps/server/src/orchestration-v2/http.ts`. This delays the real response; it does not replace its payload.
4. Clear only the `thread` object store in the isolated dev origin's `t3code:connection-runtime` IndexedDB database, then reload the transcript. Wait for the history to appear. A cached detail snapshot can bypass the intended delay.
5. Focus `.messages-timeline-scroll`, press native Page Up, and wait for scrolling to settle. The automated test made the existing element focusable with `tabIndex = 0` and called `focus({preventScroll:true})`; it did not assign `scrollTop`.
6. Change the viewport from 1240×800 to 1240×780. The baseline returns to the end. Repeat with the same delay and a diagnostic-only change from `attach(12)` to `attach(600)` in `useTimelineFollow.ts`: the chosen position holds and the scroll-to-end button appears.

The recorded input targets ordinary numbered text in the main timeline. It does not target an HTML frame. `preview_scroll` was deliberately excluded from the keyboard reproduction because its implementation uses `scrollBy`, not a wheel gesture.

## Evidence

| Signal | Original 12-retry baseline | Diagnostic 600-retry control |
| --- | ---: | ---: |
| Listener retries exhausted | 1,855 ms | Never |
| List first available | 4,660 ms | 4,635 ms |
| Listeners attached | Never | 4,802 ms |
| Scroll top before Page Up | 21,449 | 21,449 |
| Scroll top after Page Up | 20,795 | 20,795 |
| Scroll-to-end button after Page Up | Absent | Present |
| Scroll top after resize | 21,469, at end | 20,795, retained |

The content height was 22,197 px in both recorded trials. The visible timeline height changed from 748 to 728 px. [Machine-readable measurements](trace.json), [baseline video](baseline.mp4), and [diagnostic control video](control.mp4) are retained here. The delayed-load baseline snapped in three trials; the extended-retry control held in one trial. These are a small controlled experiment, not a field failure-rate estimate.

Instrumentation recorded `performance.now()` when listener retries ran out, the list reference became available, and listener attachment succeeded. The two recordings use the same transcript, delay, keyboard input, and viewport change. All instrumentation, artificial latency, and the diagnostic retry change were removed afterward. The T3 source worktree is clean, and no T3 fix or PR was created.

## Mechanism and limits

The [follow effect](https://github.com/pingdotgg/t3code/blob/b5a226e49c52ecab4ddc2d678a7012d97e651a6a/apps/web/src/components/chat/useTimelineFollow.ts#L278) tries to obtain a scroll node for a finite number of animation frames. Its dependencies do not include a reactive notification that a late list reference became available. If history arrives after that window, its wheel, touch, pointer, key, and upward-scroll listeners remain absent. The [empty/loading branch](https://github.com/pingdotgg/t3code/blob/b5a226e49c52ecab4ddc2d678a7012d97e651a6a/apps/web/src/components/chat/MessagesTimeline.tsx#L1039) can render before a list exists. The normal at-end callback also suppresses the away-from-end transition while the live-follow generation is still current.

The diagnostic retry experiment supports the missing-listener mechanism; increasing a timeout is not proposed as a durable fix. A fix should follow the actual scroll-node lifecycle, including replacement and cleanup.

This experiment reproduced an idle timeline snap on a layout change. It did not directly reproduce a mouse-wheel gesture or an active tool-start/finish snap. A separate healthy-load trial retained its position during streaming growth. Do not turn those distinctions into an unsupported claim that all scrolling failures are fixed or share this cause.
