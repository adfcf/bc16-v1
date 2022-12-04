# Report
The following report defines BC16-V1 architecture and explains many details of the project.
## Introduction
### Central Processing Unit (CPU)
A central processing unit (CPU), also known as a processor, is a complex digital circuit that is able to execute logical-arithmetic operations as well as control its
ownstate and the state of components which communicate with it. The main characteristic of a CPU is the programmability; it can receive logical symbols denominated
machine instructions, interpret them and have its behavior altered by those instructions. From this, the CPU moves digital data throughout the system and processes it
when necessary. A processor is not sufficient for a computer to work though: a memory subsystem is essential, because the CPU has to commute data with it frequently. 

### RISC vs. CISC
There are two ideologies regarding computer organization: Reduced Instruction Set Computer (RISC) and Complex Instruction Set Computer (CISC). Each has advantages and
disadvantages, so the choice to be made when designing a CPU depends on several factors. Fundamentally, CISC principles state that CPUs should have complex circuitry
and a great number of instructions available for use. Such design provides further ease for the users and makes programs written for it smaller, although its
complexity renders a more expensive product. In opposition, RISC principles are fond of simplicity. Smaller circuits might operate faster and obviously are cheaper to
manufacture. Because of that, RISC CPUs do not offer a wide variety of instructions for its low-level user. 

### RISC characteristics
In a more technical sense, these are typical RISC characteristics, which have been conveniently adopted for this work:
- Reduced number of instructions.
- Simple and primary instructions.
- Monocycle (one clock cycle per instruction).
- Restricted memory access (only two instructions perform main memory operations).

### Software Logisim
Logisim is a program that allows the description and implementation of digital circuits. It can also simulate their operation. Logisim has been used to construct the
CPU presented here. Despite not being maintained anymore, Logisim is still a satisfactory tool for this type of project.  
<img alt='logisim' src='https://github.com/adfcf/bc16-v1/blob/main/Images/logisim.png' width='420' height='250'/>

## Instruction Set Architecture (ISA)
### Intruction Set
One of the most important elements of the architecture of a CPU is its instruction set, the catalog of the operations it can perform. The CPU designed for this project
provides 16 distinct instructions and these are listed below:
| Instruction  | Opcode    | Description | Operands | Symbolic function |
|--------------|-----------|-------------|----------|-------------------|
| halt | 0x0      | It stops the operation of the CPU. | ∅ | ∅ |
| lw | 0x1      | It loads a word from the main memory. | $base, shift | [$acc] = mem[[$base] + shift] |
| sw | 0x2      | It stores a word into the main memory. | $base, shift | mem[[$base] + shift] = [$acc] |
| bc | 0x3      | It branches the exectuion flow on a certain condition. | shift | if [$conde] != 0, then [$pc] = [$pc] + shift |
| j | 0x4      | It disrupts the execution flow unconditionally. | jmp | [$pc] = jmp |
| add | 0x5      | It adds two numbers and keeps the sum. | $d, $s0, $s1 | [$d] = [$s0] + [$s1] |
| addi | 0x6      | It adds two numbers and keeps the sum. One of the summands is embedded in the instruction | $d, $s0, s1 | [$d] = [$s0] + s1 |
| mult | 0x7      | It multiplies two numbers and keeps the product. | $d, $f0, $f1 | [$d] = [$f0] * [$f1] |
| multi | 0x8      | It multiplies two numbers and keeps the product. One of the factors is embedded in the instruction | $d, $f0, f1 | [$d] = [$f0] * f1 |
| sub | 0x9      | It subtracts two numbers and keeps the difference. | $d, $m, $s | [$d] = [$m] - [$s] |
| subi | 0xA      | It subtracts two numbers and keeps the difference. The subtrahend is embeded in the instruction | $d, $m, s | [$d] = [$m] - s |
| div | 0xB      | It divides two numbers and keeps the quotient. | $d, $d0, $d1 | [$d] = [$d0] / [$d1] |
| cmp | 0xC      | It compares two numbers and keeps the result. | $d, $s0, $s1 | if [$s0] == [$s1], then [$d] = [$eq]; elif [$s0] > [$s1], then [$d] = [$gt]; else [$d] = [$lt] |
| and | 0xD      | It performs a logical AND on two values and keeps the result. | $d, $s0, $s1 | [$d] = [$s0] & [$s1] |
| or | 0xE      | It performs a logical OR on two values and keeps the result. | $d, $s0, $s1 | [$d] = [$s0] \| [$s1] |
| not | 0xF      | It performs a logical NOT on two values and keeps the result. | $d, $s0 |[$d] = ~[$s0] |

#### Legend
- mem[x]: word stored in the main memory at address x.
- [$x]: word stored in the register x.
- $x: address of the register x.
- x: immediate value.

### Registers
Registers are memory elements that can retain a sequence of bits. The processor has a few
registers to temporarily store some data it is processing at the moment as well as to keep its
current state. The registers are usually grouped into a structure called register file. In
such a structure, each register gets an address for it to be selected and have its content
retrieved or overwritten. A register file of 16 registers was chosen for this project and all
of its registers have a capacity of 16 bits. 

#### User-visible registers
|Register|Address|Value|
|--------|-------|-----|
|zero    |0x0|0x0000 (read-only) |
|acc     |0x1|Variable|
|conde   |0x2|Variable|
|mod     |0x3|The remainder of the last calculated division.|
|eq      |0x4|0x0001 (read-only)|
|gt      |0x5|0x0002 (read-only)|
|lt      |0x6|0x0004 (read-only)|
|shift   |0x7|0x0fff (read-only)|
|r0 to r1|0x8 to 0xf|Variable|

#### User-hidden registers
|Register|Value|
|--------|-----|
|PC|The memory address of the next instruction to be executed.|
|IR|The instruction that is being executed.|

## Circuits
### Overview
![cpu_overview](https://github.com/adfcf/bc16-v1/blob/main/Images/cpu_overview.png)

This is the general circuit of the BC16-V1. The Control Unit, Arithmetic and Logic Unit, and
the Register File are present. BC16-V1 is chained in the instruction cycle until it finds a
halt and suspends its operation permanently. The instruction cycle has the exact same period
of the clock and is composed of the fetch cycle and the execution cycle. The fetch cycle
occurs when the clock signal is down (the CPU is waiting for the next instruction),
conversely, the execution cycle occurs when the clock signal is high (the CPU is executing the
current instruction). In order to keep the data synchronized, any writing operations only
happen during the fetch cycle. 

### Datapath
The CPU’s datapath consists of: an arithmetic-logic unit (ALU), a set of registers and the
network of the conducting wires. 
#### 16-Register
![register_16](https://github.com/adfcf/bc16-v1/blob/main/Images/register_16.png)

This circuit is straightforward. There are 16 D-type flip-flops and each one stores one bit.
The register is updated with the value data_in when the enable_writing bit becomes high. Its
output is data_out.
#### Register File
The register file circuit groups most of the CPU’s registers and gives each a 4-bit address.
It is divided into two sections: the set of registers and the output selector.
##### Set of Registers
![register_set_16](https://github.com/adfcf/bc16-v1/blob/main/Images/register_set_16.png)

This circuit has 16 registers. It receives a data_in word, a value that is supposed to
overwrite the registers’ contents. To avoid data loss, the writing process is guarded by
writing_reg_address that, after being decoded, selects which register is going to have its
value updated. Furthermore, the writing will only occur after the control unit sends
writing_enabled signal. Note that a few registers are immune to overwriting since they are
read-only. Finally, the register mod has a privileged writing mechanism and it can be
overwritten independently from the control unit; in fact, the ALU circuit is responsible for
updating this register when necessary. 
##### Output Selector
![output_selector](https://github.com/adfcf/bc16-v1/blob/main/Images/output_selector.png)

The output selector is simple. It consists of two multiplexers which decode two inputs
respectively (reading_0_reg_address and reading_1_reg_address) and select the appropriate
register value.
#### Arithmetic and Logic Unit
The ALU is a wide circuit that manipulates data; most of its operation is about performing
arithmetic and logic computing.
![alu](https://github.com/adfcf/bc16-v1/blob/main/Images/alu.png)

ALU receives two 16-bit operands op0, op1, and a 3-bit func. The operands can be added,
subtracted, multiplied, divided (includes the calculation of the remainder) and compared. AND,
OR and NOT operations are also available. The respective circuits of each of these operations
are abstracted away (AND, OR and NOT are trivial). As indicated by the representation above,
all of the operations are performed. Yet, a multiplexer is used to select which result is
required, then it is sent as the output of the ALU.
Besides that, mod is an additional output of the ALU in case of a division; it is the
remainder of said operation. ALU also sends a div_op signal when performing a division in
order to store mod value into the mod register.
Note that the comparison outputs gt, eq and lt are all transformed into comparison following
this conversion table:
|Comparison result|comparison value|
|--------|-----|
|greater than|0x1|
|equal|0x2|
|less than|0x4|

### Control Unit
The function of the control unit (CU) in a CPU is to acquire instructions, interpret them and
then, coordinate the datapath by sending a variety of signals. It is a comprehensive circuit
and is divided into several parts: instruction decoder, multiplexing region, program counter,
additional signals generator, halting mechanism, instruction classification and ALU function
encoder.

#### Intruction Decoder Wrapper
![instruction_decoder_wrapper](https://github.com/adfcf/bc16-v1/blob/main/Images/instruction_decoder_wrapper.png)

It is a higher level view of the instruction decoder. It receives an instruction and generates
some signals.

#### Multiplexing Region
![multiplexing_region](https://github.com/adfcf/bc16-v1/blob/main/Images/multiplexing_region.png)

This circuit does a lot of data selections depending on the instruction type.

#### Additional Signals Generator
![additional_signals_generator](https://github.com/adfcf/bc16-v1/blob/main/Images/additional_signals_generator.png)

This region of the CU generates three very important signals: main memory writing authorization
(enable_mem_writing), which only occurs during sw instructions; register file writing negation,
which only occurs during sw, j or bc instructions; and branching authorization, which only
occurs when [$conde] ≠ 0 and bc is being executed. 

#### Halting Mechanism
![halting_mechanism](https://github.com/adfcf/bc16-v1/blob/main/Images/halting_mechanism.png)

When halt instruction happens, this D-type flip-flop will be set, which will result in a
permanent suspension state in the CPU.

#### Instruction Classification
![instruction_classification](https://github.com/adfcf/bc16-v1/blob/main/Images/instruction_classification.png)

This circuit generates instruction classification signals.

#### ALU function encoder.
![alu_function_encoder](https://github.com/adfcf/bc16-v1/blob/main/Images/alu_function_encoder.png)

ALU circuit receives a 3-bit signal that controls its operation mode. Said 3-bit signal is encoded depending on the current instruction.
|Operation|ALU Code|
|---------|--------|
|addition|0x00|
|multiplication|0x01|
|subtraction|0x02|
|division|0x03|
|comparison|0x04|
|and|0x05|
|or|0x06|
|not|0x07|

#### Instruction Decoder
![instruction_decoder](https://github.com/adfcf/bc16-v1/blob/main/Images/instruction_decoder.png)

This circuit selects the first four bits of the instruction (MSBs) and sends a different signal
for each different instruction.

#### Program Counter
![program_counter](https://github.com/adfcf/bc16-v1/blob/main/Images/program_counter.png)

This circuit contains the PC register; its value is incremented by one in every execution cycle.
Branches make the PC be incremented by arbitrary amounts (maximum of 4096). Jumps overwrite the
last 12 bits of the PC (LSBs). All of this process is implemented by multiplexing and
feedbacking into the PC.


#### Field Selector
![field_selector](https://github.com/adfcf/bc16-v1/blob/main/Images/field_selector.png)

This circuit divides the fields of the current instruction. It can be divided either into three
quartets or into one quartet and one octet.

### Basic Computer
BC16-V1, as a whole, represented below as “CPU”, is connected with a memory device. This completes the computer’s circuit, which now can be considered functional. The
CPU selects a memory position by sending a 16-bit address (mem_address) to the memory device. Then, the CPU both receives the 16-bit word located at mem_address and,
if the enable_mem_writing bit is set, writes mem_writing_data to mem_address location. Also, the CPU is connected to a clock circuit, and it sends a suspension state
signal to a symbolic LED. 
![basic_computer](https://github.com/adfcf/bc16-v1/blob/main/Images/basic_computer.png)

## Simulation
Three programs have been created to demonstrate BC16-V1 functionality. As stated above, Logisim can simulate the circuits implemented in it. In other words, it is
possible to load the aforementioned programs into Logisim’s virtual main memory device and activate the clock signal. This will make the CPU start computing what it
has been ordered to.

### Uninspired Writer
This program loads two numbers from the main memory. The first one searched for at 0x0010 is expected to be a counting step; the second one searched for at 0x0011 is
expected to be an upper bound for the counting. The program proceeds to add the “step” to the accumulator register until it becomes greater than the upper bound.
Additionally, it is written each partial value of the accumulator at the main memory successively, from position 0x0012 onwards. When it finishes, it puts a numerical
constant at the end of the writing.
#### Legend
- r0: step.
- r1: upper bound.
- r2 shift.

#### Symbolic Language
```
00 add $r2 $zero $zero
01 lw $zero 16
02 add $r0 $zero $acc
03 lw $zero 17
04 add $r1 $zero $acc
05 add $acc $zero $zero
06 add $acc $acc $r0
07 cmp $conde $acc $r1
08 and $conde $conde $gt
09 bc 3
10 sw $r2 18
11 addi $r2 $r2 1
12 j 6
13 add $acc $zero $shift
14 sw $r2 18
15 halt
```

#### Machine Language
```
00 0101101000000000 (0x5a00)
01 0001000000010000 (0x1010)
02 0101100000000001 (0x5801)
03 0001000000010001 (0x1011)
04 0101100100000001 (0x5901)
05 0101000100000000 (0x5100)
06 0101000100011000 (0x5118)
07 1100001000011001 (0xc219)
08 1101001000100101 (0xd225)
09 0011000000000011 (0x3003)
10 0010101000010010 (0x2a12)
11 0110101010100001 (0x6aa1)
12 0100000000000110 (0x4006)
13 0101000100000111 (0x5107)
14 0010101000010010 (0x2a12)
15 0000000000000000 (0x0000)
```

#### Memory
![uninspired_writer](https://github.com/adfcf/bc16-v1/blob/main/Images/uninspired_writer.png)

### Dynamic Average
This program sequentially loads numbers from an array located at 0x0010; its size is unknown a
priori. All the read values are added to the accumulator register, and a null value is
expected to indicate the end of the array. The integer average of the collected numbers is
inserted in place of the null value.

#### Legend
- r0: summation.
- r1: index.

#### Symbolic Language
```
00 add $r0 $zero $zero
01 add $r1 $zero $zero
02 lw $r1 16
03 add $r0 $r0 $acc
04 addi $r1 $r1 1
05 cmp $conde $acc $zero
06 and $conde $conde $eq
07 bc 1
08 j 2
09 subi $r1 $r1 1
10 div $acc $r0 $r1
11 sw $r1 16
12 halt
```
#### Machine Language
```
00 0101100000000000 (0x5800)
01 0101100100000000 (0x5900)
02 0001100100010000 (0x1910)
03 0101100010000001 (0x5881)
04 0110100110010001 (0x6991)
05 1100001000010000 (0xc210)
06 1101001000100100 (0xd224)
07 0011000000000001 (0x3001)
08 0100000000000010 (0x4002)
09 1010100110010001 (0xa991)
10 1011000110001001 (0xb189)
11 0010100100010000 (0x2910) 
12 0000000000000000 (0x0000)
```
#### Memory
![dynamic_average](https://github.com/adfcf/bc16-v1/blob/main/Images/dynamic_average.png)

### Hurried Calculator
This program loads two numbers from the main memory. The first number is expected to be at the
position 0x0010, and the second one at 0x0011. The program will immediately compute all the
basic arithmetic operations with those numbers. [0x0010] is treated as the first operand, and
[0x0011] is treated as the second one. Finally, the results (sum, difference, product,
quotient and remainder) are recorded at [0x0012 - 0x0016] retrospectively.

#### Legend
- r0: first operand.
- r1: second operand.

#### Symbolic Language
```
00 lw $zero 16
01 add $r0 $acc $zero
02 lw $zero 17
03 add $r1 $acc $zero
04 add $acc $r0 $r1
05 sw $zero 18
06 sub $acc $r0 $r1
07 sw $zero 19
08 mult $acc $r0 $r1
09 sw $zero 20
10 div $acc $r0 $r1
11 sw $zero 21
12 add $acc $mod $zero
13 sw $zero 22
14 halt
```
#### Machine Language
```
00 0001000000010000 (0x1010)
01 0101100000010000 (0x5810)
02 0001000000010001 (0x1011)
03 0101100100010000 (0x5910)
04 0101000110001001 (0x5189)
05 0010000000010010 (0x2012)
06 1001000110001001 (0x9189)
07 0010000000010011 (0x2013)
08 0111000110001001 (0x7189)
09 0010000000010100 (0x2014)
10 1011000110001001 (0xb189)
11 0010000000010101 (0x2015)
12 0101000100110000 (0x5130)
13 0010000000010110 (0x2016)
14 0000000000000000 (0x0000)
```
#### Memory
![hurried_calculator](https://github.com/adfcf/bc16-v1/blob/main/Images/hurried_calculator.png)

## Bibliography
Patterson, David A.; Hennessy, John L. Computer Organization and Design MIPS Edition: The Hardware/Software Interface. Morgan Kaufmann. December 2013.
