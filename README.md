# Instant 0-tick CPU in Minecraft

[Download the world](https://github.com/HironTez/0-tick-cpu-minecraft/releases)

After an unreasonable amount of time, I've built a functional CPU in Minecraft that operates entirely on 0-tick mechanics — meaning all logical operations complete within a single redstone tick.

## Architecture Overview
The CPU is built exclusively from redstone dust and pistons. No repeaters, no comparators, no torches — components that typically introduce delay or state complexity are entirely absent. Also no command blocks or mods. All timing behavior is derived from 0-tick piston glitches, which allow state transitions to propagate and resolve within one tick.
The design follows a strict 1-cycle principle: there are no internal subcycles. An operation initiated on a rising clock edge produces its result before the cycle closes. This required careful signal path analysis to ensure no intermediate states persist across cycle boundaries.

## ALU
The ALU supports five operations: AND, OR, XOR, Add, Subtract. Scope was intentionally limited to demonstrate architectural viability rather than operational completeness.
XOR gates present a known hazard in 0-tick contexts: if two inputs transition from 0,0 to 1,1 but arrive at different times, an intermediate 1,0 state produces a spurious output pulse that breaks downstream circuits. To address this, the ALU sits behind a synchronization gateway. The gate is held closed until a dedicated presence signal arrives. This signal is routed to guarantee it arrives only after all data inputs have fully settled — eliminating the hazard window.

## Registers
The CPU has 4 registers. Each register is implemented as two cascaded D flip-flops to guarantee read-cycle integrity. The first DFF updates on write. The second DFF holds the previous value and updates only on the falling clock edge. This allows a register to be both source and destination in the same instruction — the read value remains stable throughout the cycle regardless of the write operation occurring in parallel.

## Decay Sequencing
Clock shutdown follows a strict ordered sequence to prevent state corruption:
Register input gate closes — prevents a decaying ALU result from being latched.
Presence signal deactivates — closes the ALU synchronization gate, protecting XOR inputs from asynchronous fade (the 1,1 → 0,0 intermediate state problem, symmetric to the setup hazard).
Register output safely transitions to the new value.
Clock receives a feedback signal confirming the cycle has fully terminated and the next cycle may begin.

## Clock Rate
The 0-tick mechanism requires a mechanical reset period of approximately 1 second. Effective clock rate is ~1 Hz.

## What's Missing
There are no memory operations. The CPU has no load/store instructions and no addressable memory. This is a proof-of-concept for the 0-tick single-cycle execution model — implementing memory would require a substantial piston array that is disproportionate to the demonstrative value of the project at this stage. The architecture supports future memory integration in principle.

## Performance
Surprisingly, the performance is not awful. It was all built on an old laptop and can perform one operation per second without serious lag.

## Summary
This demonstrates that a functional single-cycle CPU architecture is achievable within 0-tick constraints using only redstone and pistons. The primary contributions are the synchronization gateway pattern for XOR hazard elimination, the dual-DFF register design for same-register read/write integrity, and the decay sequence for clean cycle termination.

PS. there's a prototype development ground on coordinates (0, 0) where you can inspect each part separately. The fully assembled CPU is on (0, -300).

<img width="1920" height="1048" alt="Screenshot From 2026-03-03 16-28-29" src="https://github.com/user-attachments/assets/263c1270-d581-4712-9055-8df237c97b1f" />
<img width="1920" height="1048" alt="Screenshot From 2026-03-03 18-03-53" src="https://github.com/user-attachments/assets/9d52b02e-b11c-4340-9108-56cb137f7072" />
<img width="1920" height="1048" alt="Screenshot From 2026-03-03 16-28-47" src="https://github.com/user-attachments/assets/fe1d5622-98b3-49f9-a8cd-6782b65161c8" />
<img width="1920" height="1048" alt="Screenshot From 2026-03-03 16-28-59" src="https://github.com/user-attachments/assets/dcbea164-2034-4789-94d6-486178570107" />
<img width="1920" height="1048" alt="Screenshot From 2026-03-03 18-59-23" src="https://github.com/user-attachments/assets/71ca526c-0adf-4bb5-83f7-b2c6f695fcac" />
<img width="1920" height="1048" alt="Screenshot From 2026-03-03 18-59-49" src="https://github.com/user-attachments/assets/55e1c0f2-e4b2-41ca-8ccc-a417b3e28d81" />
