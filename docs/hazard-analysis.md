# Hazard Analysis and Industry Mapping

This is an educational Wokwi prototype. The mappings below are conceptual and do not claim formal compliance.

| Hazard | Possible cause | Effect | Detection | Mitigation |
|---|---|---|---|---|
| Delayed ground-command response | Higher-priority Core-1 load or missing immediate ISR yield | Command is handled late | Maximum latency counters and logic-analyzer trace | Keep ISR short, use direct notification, retain `portYIELD_FROM_ISR()`, review priorities |
| Repeated commands are coalesced | Binary semaphore stores only one pending state | Comparison path may not represent every rapid press | Compare `presses_observed` and handler counts | Use the direct notification count as the real command path |
| Long ISR execution | Logging or processing moved into ISR | Increased interrupt blocking and jitter | GPIO 19 ISR pulse width | ISR only timestamps, signals tasks, and exits |
| Stale or misleading timing result | Both priority-12 tasks wake together and log serial output | Measured value includes scheduling and logging overhead | Compare individual traces and document measurement scope | Describe results as observed ISR-to-task latency, not pure API cost |
| Core-1 overload | Periodic task WCET exceeds its budget | Missed response or large jitter | WCET maxima and task heartbeats | Keep load deterministic, verify `C < T`, lower noncritical workload |
| Fault mode accidentally left enabled | Build configuration error | No explicit immediate yield from ISR | Startup warning prints `FAULT MODE` | Use `FAULT_MODE=0` for final normal build and review startup log |

## Conceptual standard mapping

- **DO-178C concepts:** requirements traceability, bounded interrupt behavior, verification evidence, and documented failure behavior.
- **ISO 26262 concepts:** hazard analysis, freedom from interference between critical and noncritical work, and explicit safety mechanisms.
- **MISRA C concepts:** bounded control flow, clear data ownership, fixed-width types where appropriate, and reviewable ISR behavior.

The system is not certified to any of these standards. They are included to show how the engineering evidence relates to production practices.
