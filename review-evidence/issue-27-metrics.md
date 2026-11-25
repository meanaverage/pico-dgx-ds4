# Pico metrics review evidence

The Pico `dsfront` candidate at `b8fd37a` is ready for upstream submission. All 25 review passes completed across four revisions. The final five-pass review had no verified actionable findings. This records the sparkDash metrics work requested in issue #27. [Pico issue #27](https://github.com/Picom-code/pico-dgx-ds4/issues/27) and [Pico PR #12](https://github.com/Picom-code/pico-dgx-ds4/pull/12)

The patch targets Pico `main` at `922dddb`. Its only new HTTP route is `GET /metrics`. The exporter retains the `tensorfold:` names consumed by sparkDash. Its frontend gate gauges describe requests past or waiting at the gate.

## Final follow-up on b8fd37a

| Reviewer | Reported result | Findings and disposition |
| --- | --- | --- |
| DeepSeek V4.1 Flash, test oracle | LGTM | Confirmed the independent wire values and the per-run cold-cache prefix. Suggested optional additional HTTP assertions. |
| DeepSeek V4.1 Flash, skeptical | LGTM | No blocking findings or optional changes. |
| Gemini 4 Neon XHigh | LGTM | Confirmed the narrowed fixtures. Suggested extra malformed-frame combinations that the isolated checks exercised. |
| Claude Opus XHigh, skeptical | LGTM | Confirmed the per-run prefix, exact timing fields, wire prompt length and zero initial generation. The optional log check has a pre-existing second-run limitation. |
| Claude Opus XHigh, test oracle | 2 P2 reported | Suggested deliberately conflicting DONE fields for cancelled token counts and the cached-prefix clamp. The existing socket test rejects counting at DONE. Independent wire checks verified both conflict cases. |

The final review closed the repeat-run, wire-field and initial-counter findings. The Opus test-oracle suggestions cover conflicting fields from an out-of-contract peer. Separate wire checks confirmed the actual exporter in both cases.

## Round three on f5f63bd

| Reviewer | Reported result | Findings and disposition |
| --- | --- | --- |
| DeepSeek V4.1 Flash, skeptical | LGTM | Confirmed rank-zero framing, completion exclusions, token accounting after cancellation, and held-request timing. Suggested clearer test invocation and diagnostics. |
| DeepSeek V4.1 Flash, test oracle | 2 P1 and 4 P2 reported | Raised DONE-versus-TOKENS accounting, disconnect causality, source fields, and TTFT coverage. The socket regression rejects generation-at-DONE. Forced repetition cancellation fails the real disconnect check. The follow-up separates SUBMIT, queue, decode, and prefill values. |
| Claude Opus XHigh, skeptical | LGTM | Confirmed the frame layouts, exact metrics, ordering after cancellation, and test gate teardown. Suggested optional test diagnostics. |
| Claude Opus XHigh, test oracle | 3 P2 reported | Flagged neighbouring wire fields, TTFT bounds, and the existing API on malformed cache counts. The next commit separates the fields. The documented held smoke test catches a late TTFT timestamp. |
| Gemini 4 Neon XHigh | LGTM | Confirmed exact wire accounting and the held-gate checks. Suggested distinct timing fields and an absolute first-token count, added in the follow-up. |
| GPT-6.2 Sol XHigh, skeptical | 3 P2 reported | Flagged a repeat-run failure that independent testing reproduced and zero queue time. The follow-up fixes both. Ephemeral-port contention remains a fixture startup failure. |
| GPT-6.2 Sol XHigh, test oracle | 4 P2 reported | Flagged EOS-only completions, live-stream cohort exclusion, suppressed IDs, and the initial generation baseline. The follow-up pins the baseline. Independent wire checks verified the other cases. |

The test-only round added exact prefill and post-disconnect checks. Independent verification reproduced a cold-cache smoke failure on a second run and showed that neighbouring wire fields and the initial counter needed independent expected values. The final follow-up resolves these three findings.

## Round two on aee483b

| Reviewer | Result | Main findings |
| --- | --- | --- |
| DeepSeek V4.1 Flash, test oracle | 2 P1 and 3 P2 reported | Flagged coverage for independent prefill and prompt counts, unusual completions, generation after disconnect, and TTFT on thinking streams. Independent verification found coverage gaps and confirmed the current runtime behavior. |
| DeepSeek V4.1 Flash, skeptical | LGTM | Found no blocking issue. Flagged a possible test race after SSE completion and confirmed the documented unauthenticated metrics exposure. |
| Claude Fable 5.2 High | LGTM | Confirmed token accounting across disconnects and completion exclusions by inspection. Suggested test hardening and a note about the exported TTFT interval. |
| Gemini 4 Neon XHigh | LGTM | Suggested an additional guard against a test race after SSE completion and more direct coverage of completion guards. |
| Claude Opus XHigh, skeptical | LGTM | Confirmed socket accounting and the consistent completion snapshot. Suggested clearer wording for exported TTFT and the log. |
| Claude Opus XHigh, test oracle | 2 P2 reported | Flagged the optional admin-socket check and missing committed coverage for unusual engine replies. |
| GPT-6.2 Astra High | 1 P3 | Reproduced a smoke-test gap that accepted TTFT substituted for engine prefill time. All 48 Rust tests and 60 smoke checks passed on the unmodified candidate. |
| Claude Opus XHigh, simplification | LGTM | Found the ownership and locking straightforward. Suggested optional test setup cleanup. |

The socket reader now counts each decoded token before request delivery. This preserves generation totals when an HTTP client slows down or disconnects. The revised tests cover active and held requests, in-flight accounting, exposition types, and the TTFT interval.

Round two had no verified P0 or P1 findings. One P2 and five P3 findings concerned the committed test coverage or smoke reliability. The test-only rounds above address the exact prefill oracle and other verified gaps.

## Round one on 0d5fe19

| Reviewer | Reported result | Main findings |
| --- | --- | --- |
| DeepSeek V4.1 Flash, skeptical | 1 P2 | Slow HTTP consumers could delay generation accounting. Dropping a receiver could lose buffered tokens from the count. |
| DeepSeek V4.1 Flash, test oracle | 1 P1 and 2 P2 | Idle-only gauge checks accepted swapped or zero gauges. The suite also accepted completion-only generation and incorrect timing. |
| Gemini 4 Neon XHigh | 2 P2 | Nonzero gauges, exposition metadata, and accounting on unsuccessful or abandoned requests needed better coverage. |
| Claude Opus XHigh, skeptical | 2 P2 | The endpoint was reachable on each configured listener without a key. The route inventory and exposure guidance needed to say so. |
| Claude Opus XHigh, test oracle | 1 P1 and 4 P2 | Missing coverage for nonzero gauges, in-flight tokens, timing, cancellations, and Prometheus types. |

Independent verification found six distinct P2 items and no P0 or P1 defects. The reported P1s both concerned the same test gap. The actual nonzero gauges were correct when exercised over HTTP. The generation defect was real. The initial build could lose decoded tokens when an HTTP stream stopped consuming them.

## Verification

- **Rust and smoke.** All 48 Rust tests passed. The smoke suite passed 68 checks on macOS ARM64 and Linux ARM64. Two macOS runs on one fixture passed, and a fresh run with the optional plan log passed 69 checks.

- **Exact wire responses.** The real HTTP route reported 70,000 μs of engine prefill independently of the queue and decode fields, counted four IDs after observed client cancellation, and preserved the completed cohort across errors, empty frames and zero-token completions.

- **Regression checks.** Thirteen deliberate mutations failed their expected assertions. A further socket check rejected generation accounting at DONE. The previous smoke test failed under forced SSE teardown delay, and the revised test passed.

- **sparkDash.** The actual collector read 530 prompt tokens, 265 cached tokens and 14 generated tokens across two requests. It reported both prefill rates.

- **Compatibility.** The release candidate and current `main` matched across 11 existing HTTP cases on Linux ARM64 after normalizing generated IDs and timestamps. The candidate returned metrics where the baseline returned 404.

The candidate ran against isolated CPU fixtures. Its four-Spark CUDA ring run remains outstanding.

From the Pico recipe directory, run the exact wire check after building `dsfront` using the recipe's tokenizer file.

```sh
cargo build --locked --release --manifest-path frontend/Cargo.toml
python3 frontend/tools/metrics_test.py frontend/target/release/dsfront /path/to/tokenizer.json
```
