# BC16-V1
## What is it?
It's an attempt to design a basic 16-bit CPU by using digital circuits simulated by the old [Logisim](http://www.cburch.com/logisim/) program.
Its name is very telling - BC16-V1 - basic CPU, 16 bits, version 1. 
The '1' actually indicates it has a monocycle organization, that is, this CPU executes a single instruction a clock tick.
A hypothetical version 2 would have to come into life equipped with pipelining.

![high_level_view](https://github.com/adfcf/bc16-v1/blob/main/high_level_view.png)

## Outcome
It was a successful attempt; its organization is pretty cohesive and it has the three distinct structures a CPU (usually) has: a register file, ALU and CU.
Furthermore, it can access a fictitious memory, get an instruction, interpret it and execute it. That means, BC16-V1 has a little architecture of its own, as demonstrated below with the usage of symbolic language.

![symbolic_language_example](https://github.com/adfcf/bc16-v1/blob/main/symbolic_language_example.png)

## Report
I plan on publishing a report to explain some details about this project.

