# Final Reflection

## What I would do differently

If I restarted this project, I would design the measurement plan at the same time as the task architecture. At first, I focused mainly on making the GPIO interrupt and FreeRTOS signaling work. Later, I realized that a real-time project also needs clear definitions for what is being measured. The GPIO 18 to GPIO 19 logic-analyzer measurement represents interrupt response, while the serial latency values represent the larger ISR-to-bottom-half path. I would define those two measurements before writing the final code and would add a dedicated monitoring task earlier. I would also avoid waking two equal-priority tasks from the same interrupt in a production design because the comparison task and its serial logging influence the timing of the real command handler.

## What was harder than expected

The hardest part was understanding why a fast signaling primitive could still produce a large observed latency. The direct notification frequently woke in approximately 20 to 31 microseconds, but some runs showed values near two milliseconds. The API itself was not necessarily taking two milliseconds. The result also included scheduler decisions, a higher-priority load task, another priority-12 responder, and serial logging. The fault-injection result was also less dramatic than expected. Removing `portYIELD_FROM_ISR()` did not stop delivery and did not consistently create a much larger maximum in Wokwi. That forced me to report what the evidence actually showed instead of claiming the expected result.

## Most valuable thing learned

The most valuable lesson was that correct output is only one part of a real-time system. The design also needs a timing contract, instrumentation, a task and priority model, a failure story, and evidence that supports the conclusions. I learned why an ISR should perform only short, ISR-safe operations and defer the real work to a task. I also learned how background load and priority relationships affect response time even when the interrupt itself remains short. This project gave me a better way to explain an embedded system in an interview: the architecture, the timing evidence, the tradeoffs, and the limitations of the experiment are all part of the final result.
