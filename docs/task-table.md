# Tasks and Timing Evidence

## Task table

| Task / context | Core | Priority | Period or trigger | Deadline / goal | Evidence |
|---|---:|---:|---|---|---|
| GPIO 18 ISR | Interrupt | Hardware | Button falling edge | Keep ISR bounded | GPIO 18 falling edge to GPIO 19 rising edge: approximately **15.35 µs** |
| `btn_notif` | 1 | 12 | Direct notification from ISR | Respond as soon as scheduled | Idle max **1983 µs**; loaded max **2198 µs**; loaded fault max **2204 µs** |
| `btn_sem` | 1 | 12 | Binary semaphore from ISR | Comparison path | Idle max **2383 µs**; loaded max **2488 µs**; loaded fault max **2367 µs** |
| `load_a` | 1 | 15 | 10 ms | 10 ms | Firmware records `wcet_a_max_us` |
| `load_b` | 1 | 10 | 20 ms | 20 ms | Firmware records `wcet_b_max_us` |
| `load_c` | 1 | 5 | 50 ms | 50 ms | Firmware records `wcet_c_max_us` |
| `load_d` | 1 | 2 | 100 ms | 100 ms | Firmware records `wcet_d_max_us` |
| `metrics` | 0 | 1 | 5 s | Soft monitoring | Prints WCET maxima and heartbeats without competing on Core 1 |

## Recorded latency experiments

| Configuration | Presses | Notification maximum | Semaphore maximum |
|---|---:|---:|---:|
| Idle normal (`WITH_LOAD=0`, `FAULT_MODE=0`) | 50 | 1983 µs | 2383 µs |
| Loaded normal (`WITH_LOAD=1`, `FAULT_MODE=0`) | 57 | 2198 µs | 2488 µs |
| Loaded fault (`WITH_LOAD=1`, `FAULT_MODE=1`) | 48 | 2204 µs | 2367 µs |

The loaded normal test increased the observed notification maximum by 215 µs and the semaphore maximum by 105 µs compared with the idle normal test. The fault-mode maxima were similar to normal loaded operation, so this project does not claim a large deterministic latency increase from removing `portYIELD_FROM_ISR()`. The supported conclusion is that delivery continued while immediate rescheduling was no longer explicitly requested.

## WCET collection

The four periodic tasks use the `MEASURE_WCET` macro. The final firmware adds a Core-0 metrics task that prints this line every five seconds:

```text
[wcet] A=<us> B=<us> C=<us> D=<us> | hb=<A>/<B>/<C>/<D>
```

These numbers are simulator-specific and should be copied from the final Wokwi run when presenting exact periodic-task utilization. The repository does not invent WCET values that were not recorded.
