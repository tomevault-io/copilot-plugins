## troubleshooting

> Fixing problems, recovering from failures


When troubleshooting, writing new code is the final step, not the first. Do not randomly try new code.

Instead, think of up to five root causes of a failure, then narrow down to the most likely one or two.

Only after providing a detailed plan for remediation will you write new code.

Do not invent functions or properties on external libraries that don't actually exist, since this won't actually solve the problem.

Switching away from Arduino framework is never a viable strategy.

If USB operations falter, this command can reset the whole USB subsystem:

sudo pkill -f "usb|serial|uart"; sudo pkill -f tty; sudo killall -STOP usbd; sleep 2; sudo killall -CONT usbd;

---
> Source: [PostHog/DeskHog](https://github.com/PostHog/DeskHog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
