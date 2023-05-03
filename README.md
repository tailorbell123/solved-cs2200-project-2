Download Link: https://assignmentchef.com/product/solved-cs2200-project-2
<br>
<strong>1       Why Pipelining?</strong>

The datapath design that we implemented for Project 1 was, in fact, grossly inefficient. By focusing on increasing throughput, a pipelined processor can get more instructions done per clock cycle. In the real world, that means higher performance, lower power draw, and most importantly, happy customers!

<h1>2       Project Requirements</h1>

In this project, you will make a pipelined processor that implements the Conte-200 ISA. There will be five stages in your pipeline:

<ol>

 <li><strong>IF </strong>– Instruction Fetch</li>

 <li><strong>ID/RR </strong>– Instruction Decode/Register Read</li>

 <li><strong>EX </strong>– Execute (ALU operations)</li>

 <li><strong>MEM </strong>– Memory (both reads and writes with memory)</li>

 <li><strong>WB </strong>– Writeback (writing to registers)</li>

</ol>

Before you move on, read Appendix A: Conte-200 Instruction Set Architecture to understand the ISA that you will be implementing. We provide you with a Brandonsim file with the some of the structure laid out.

<h1>3       Building the Pipeline</h1>

First, you will have to build the hardware to support all of your instructions. You will have to make each stage such that it can accommodate the actions of all instructions passing through it. Use the book (Ch. 5) to get an idea of what the pipeline looks like and to understand the function of each stage before you start building your circuits.

<h2>1. IF Stage</h2>

The IF stage is responsible for:

<ul>

 <li>Getting the instruction from I-MEM at location PC</li>

 <li>Updating the PC</li>

</ul>

For normal sequential execution, we would update the PC by incrementing it by 1. Notice, however, that this may not be the case when executing a SKP, CALL, RET, or GOTO instruction. Hence, you will likely need to multiplex which value is used to update the PC.

<h2>2. ID/RR Stage</h2>

The ID/RR stage is responsible for:

<ul>

 <li>Decoding the instruction</li>

 <li>Reading the appropriate registers</li>

 <li>Resolving any CALL, RET, SKP, or GOTO instructions</li>

</ul>

Please look at Appendix A: Conte-200 Instruction Set Architecture in order to understand the instruction formats! You will have a dual ported register file (DPRF), which allows you to read from two registers and write one register all at the same time. As you will notice, the TAs have been very kind in making the DPRF and providing it to you.

Some of the instructions require both inputs into to the ALU to be values pulled from the DPRF. However, other instructions contain a value within the instruction, such as an immval20, offset20, or PCAddr24 field. You may either pass all of these possible values to the next stage (requires bigger buffer registers), or condense them into just the values needed to execute the instruction in the following cycles (requires more logic, but buffer size can be optimized).

<h2>3. EX Stage</h2>

The EX stage is responsible for:

<ul>

 <li>Performing all necessary arithmetic and logic calculations</li>

</ul>

In the Execute (EX) stage, you will perform any arithmetic computations required by the instruction. This stage should host a complete ALU to perform the actual adding or NANDing as required by the instruction. For memory access instructions, this stage will perform the Base + Offset computation required to determine the memory address to access.

<h2>4. MEM Stage</h2>

The MEM stage is responsible for:

<ul>

 <li>Reading from or writing a result to memory</li>

</ul>

All you need to do is to use the value calculated in the EX stage as the address for the RAM. <strong>Note that you must use the maximum address length for the RAM block – this is 24 bits. </strong>To accomplish this, simply take the lower 24 bits of the calculated address. Depending on the instruction, this stage will need to pass either the value read from memory or the value computed in EX to the WB stage.

<h2>5. WB Stage</h2>

The WB stage is responsible for:

<ul>

 <li>Writing results back to the DPRF (dual-ported register file)</li>

</ul>

Depending on the instruction, you may need to write a value back to a register. To do this, your WB stage will attach to the data in and write enable inputs of the DPRF in ID/RR. Remember that the DPRF can write <strong>and </strong>read different registers <strong>in the same clock cycle</strong>, which is why WB and ID/RR can share the same register file. For instructions that do not write a register, your WB stage may not do anything at all.

<h1>4       General Advice</h1>

<h2>Subcircuits</h2>

For this project, we highly encourage using modular design and creating subcircuits when necessary. We <strong>strongly </strong>recommend using subcircuits when building your pipeline buffers as well as your forwarding unit.

<h2>Pipeline Buffers</h2>

For deciding what to pass through buffers, remember that we need to support the requirements of every possible instruction. Think of what each instruction needs to fulfill its duty, and pass a union of all those requirements. (By union we mean the mathematical union, for example say I1 needs PC and Rx, while I2 needs Rx and Ry, then you should pass PC, Rx and Ry through the buffer). You can also feel free to implement your hardware such that you re-use space in the buffer for different purposes depending on the instruction, but this is not required.

<h2>Control Signals</h2>

In the Project 1 datapath, recall that we had one main ROM that was the single source of all the control signals on the datapath. Now that we are spreading out our work across different stages of the pipeline, you have a choice of how to implement your signals!

There are two options:

<ol>

 <li>You can either have a single large main ROM in ID/RR which calculates all the control signals for every stage.</li>

</ol>

<h3>OR</h3>

<ol start="2">

 <li>you can have a small(er) ROM in each stage which takes in the opcode and assert the proper signals for that operation.</li>

</ol>

Note that if you choose the first method, you will need to pass all the signals needed for later stages through the earlier stages, and in the second method, you will need to pass the instruction opcode though all the stages so that you know which signals to assert during that stage.

<h2>Stalling the Pipeline</h2>

One must stall the pipeline when an instruction cannot proceed to the next stage because a value is not yet available to an instruction. This usually happens because of a data hazard. For example, consider two instructions in the following program:

<ol>

 <li>LW $t0, 5($t1)</li>

 <li>ADDI $t0, $t0, 1</li>

</ol>

Without stalling the ADDI instruction in the ID/RR stage, it will get an out of date value for $t0 from the regfile, as the correct value for $t0 isn’t known the LW reaches the MEM stage! Therefore, we must stall. Consult the textbook (or your notes) for more information on data hazards. It is also important to note that through data forwarding, stalls can be lessened in penalty or in some cases avoided entirely. Data forwarding is discussed in the next section

To stall the pipeline, the stages preceding the stalled stage should disable writes into their buffers, i.e. they should continue to output the previous value into the next stage. The stalled stage itself will output NOOP (example, ADD $zero, $zero, $zero) instructions down the pipeline until the cause of the stall finishes.

<h2>Data Forwarding</h2>

If you really liked the busy-bit/read-pending signal forwarding described in lecture and in your book, feel free to use that. We present an alternate way to do forwarding in this section.

Forwarding is one way to increase the performance of the pipeline. This allows us to get values computed in stages beyond ID/RR back to ID/RR so that we do not have to stall the instruction. I would strongly recommend against using the busy bit/read pending bit strategy suggested in the book – this has some very nasty edge cases and requires much more logic than necessary.

I would recommend that you make a forwarding unit that implements various stock rules. The forwarding unit should take in the two register values you are reading, the output value from the EX stage, the output value from the MEM stage, and the output value from the WB stage. To forward a value from a future stage back to ID/RR, you must check to see if the destination register number from a particular stage is equal to your source register numbers in the ID/RR stage. If so, you must forward the value from that stage to your ID/RR stage.

You shouldn’t update the value of the register when you forward the value back – writes to the register file should only occur in the WB stage. Of course, forwarding cannot save you from one situation: when the destination register of a LW instruction is the source register of an instruction immediately after it. In this case, you must stall the instruction in the ID/RR stage. I will leave it to you to flesh out all of the stall rules.

<strong>Keep in mind: </strong>the zero register can never change, therefore it should not be considered for forwarding and stalling situations.

<h2>Flushing the Pipeline</h2>

For the CALL/RET/SKP/GOTO instructions, we calculate the target in the ID/RR stage of the pipeline. However, the next instruction the IF stage fetches while ID/RR is computing the target may not be the next instruction we want to execute. When this happens, we must have a hardware mechanism to “cancel” or “flush” the incorrectly-fetched instructions after we realize they are incorrect.

In implementing your flushing mechanism, we <strong>highly recommend </strong>avoiding the asynchronous clear feature of registers in Brandonsim, as this may cause timing issues. Instead, we suggest using a multiplexer to selectively send a NOOP into the buffer input.

<h2>Skip Prediction</h2>

When you encounter a SKP instruction, you should predict that the SKP is not taken. This means there should be no stalling, Fetch should simply go on and retrieve the next instruction at PC + 1.

Upon resolving the branch, the pipeline should continue normally in the case of a correct prediction, or flush the instruction following the SKP in the case of an incorrect prediction.

<h1>5      Testing</h1>

When you have constructed your pipeline, you should test it instruction by instruction to see if you have all the necessary components to ensure proper execution.

Be careful to only use the instructions listed in the appendix – there are some subtle points in having a separate instruction and data memory. Load the assembled program into both the instruction memory and the data memory and let your processor execute it. Any writes to memory will only affect the data memory.

<h1>6      Deliverables</h1>

Please submit all of the following files in a <strong>.tar.gz </strong>archive. You must turn in:

<ul>

 <li>Brandonsim Datapath File (Conte-200-pipeline.circ)</li>

</ul>

If you are running on a Linux or Unix-based machine, run make submit to automatically package your project for submission.

<strong>Always re-download your assignment from T-Square after submitting to ensure that all necessary files were properly uploaded. If what we download does not work, you will get a 0 regardless of what is on your machine.</strong>

<strong>This project will be demoed. </strong>In order to receive full credit, you must sign up for a demo slot and complete the demo. We will announce when demo times are released.

<h1>7           Appendix A: Conte-200 Instruction Set Architecture</h1>

The Conte-200 is a simple, yet capable computer architecture.

The Conte-200 is a <strong>word-addressable</strong>, <strong>32-bit </strong>computer. <strong>All addresses refer to words</strong>, i.e. the first word (four bytes) in memory occupies address 0x0, the second word, 0x1, etc.

All memory addresses are truncated to 24 bits on access, discarding the 8 most significant bits if the address was stored in a 32-bit register. This provides roughly 67 MB of addressable memory.

<h2>7.1     Registers</h2>

The Conte-200 has 16 general-purpose registers. While there are no hardware-enforced restraints on the uses of these registers, your code is expected to follow the conventions outlined below.

Table 1: Registers and their Uses

<table width="427">

 <tbody>

  <tr>

   <td width="115">Register Number</td>

   <td width="50">Name</td>

   <td width="175">Use</td>

   <td width="88">Callee Save?</td>

  </tr>

  <tr>

   <td width="115">0</td>

   <td width="50">$zero</td>

   <td width="175">Always Zero</td>

   <td width="88">NA</td>

  </tr>

  <tr>

   <td width="115">1</td>

   <td width="50">$at</td>

   <td width="175">Reserved for the Assembler</td>

   <td width="88">NA</td>

  </tr>

  <tr>

   <td width="115">2</td>

   <td width="50">$v0</td>

   <td width="175">Return Value</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">3</td>

   <td width="50">$a0</td>

   <td width="175">Argument 1</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">4</td>

   <td width="50">$a1</td>

   <td width="175">Argument 2</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">5</td>

   <td width="50">$a2</td>

   <td width="175">Argument 3</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">6</td>

   <td width="50">$t0</td>

   <td width="175">Temporary Variable</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">7</td>

   <td width="50">$t1</td>

   <td width="175">Temporary Variable</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">8</td>

   <td width="50">$t2</td>

   <td width="175">Temporary Variable</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">9</td>

   <td width="50">$s0</td>

   <td width="175">Saved Register</td>

   <td width="88">Yes</td>

  </tr>

  <tr>

   <td width="115">10</td>

   <td width="50">$s1</td>

   <td width="175">Saved Register</td>

   <td width="88">Yes</td>

  </tr>

  <tr>

   <td width="115">11</td>

   <td width="50">$s2</td>

   <td width="175">Saved Register</td>

   <td width="88">Yes</td>

  </tr>

  <tr>

   <td width="115">12</td>

   <td width="50">$k0</td>

   <td width="175">Reserved for OS and Traps</td>

   <td width="88">NA</td>

  </tr>

  <tr>

   <td width="115">13</td>

   <td width="50">$sp</td>

   <td width="175">Stack Pointer</td>

   <td width="88">No</td>

  </tr>

  <tr>

   <td width="115">14</td>

   <td width="50">$fp</td>

   <td width="175">Frame Pointer</td>

   <td width="88">Yes</td>

  </tr>

  <tr>

   <td width="115">15</td>

   <td width="50">$ra</td>

   <td width="175">Return Address</td>

   <td width="88">No</td>

  </tr>

 </tbody>

</table>

<ol>

 <li><strong>Register 0 </strong>is always read as zero. Any values written to it are discarded. <strong>Note: </strong>for the purposes of this project, you must implement the zero register. Regardless of what is written to this register, it should always output zero.</li>

 <li><strong>Register 1 </strong>is a general purpose register. You should not use it because the assembler will use it in processing pseudo-instructions.</li>

 <li><strong>Register 2 </strong>is where you should store any returned value from a subroutine call.</li>

 <li><strong>Registers 3 – 5 </strong>are used to store function/subroutine arguments. <strong>Note: </strong>registers 2 through 8 should be placed on the stack if the caller wants to retain those values. These registers are fair game for the callee (subroutine) to trash.</li>

 <li><strong>Registers 6 – 8 </strong>are designated for temporary variables. The caller must save these registers if they want these values to be retained.</li>

 <li><strong>Registers 9 – 11 </strong>are saved registers. The caller may assume that these registers are never tampered with by the subroutine. If the subroutine needs these registers, then it should place them on the stack and restore them before they jump back to the caller.</li>

 <li><strong>Register 12 </strong>is reserved for handling interrupts. While it should be implemented, it otherwise will not have any special use on this assignment.</li>

 <li><strong>Register 13 </strong>is your anchor on the stack. It keeps track of the top of the activation record for a subroutine.</li>

 <li><strong>Register 14 </strong>is used to point to the first address on the activation record for the currently executing process. Don’t worry about using this register.</li>

 <li><strong>Register 15 </strong>is used to store the address a subroutine should return to when it is finished executing. It is automatically used for this purpose by the CALL and RET instructions.</li>

</ol>

<h2>7.2      Instruction Overview</h2>

The Conte-200 supports a variety of instruction forms, only a few of which we will use for this project. The instructions we will implement in this project are summarized below.

Table 2: Conte-200 Instruction Set

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0000</td>

   <td width="47">DR</td>

   <td width="47">SR1</td>

   <td width="187">unused</td>

   <td width="47">SR2</td>

  </tr>

  <tr>

   <td width="47">0001</td>

   <td width="47">DR</td>

   <td width="47">SR1</td>

   <td width="187">immval20</td>

   <td width="47"> </td>

  </tr>

  <tr>

   <td width="47">0010</td>

   <td width="47">DR</td>

   <td width="47">SR1</td>

   <td width="187">unused</td>

   <td width="47">SR2</td>

  </tr>

  <tr>

   <td width="47">0011</td>

   <td width="47">mode</td>

   <td width="47">SR1</td>

   <td width="187">unused</td>

   <td width="47">SR2</td>

  </tr>

  <tr>

   <td width="47">0100</td>

   <td width="47">0000</td>

   <td width="47"> </td>

   <td width="187">PCaddr24</td>

   <td width="47"> </td>

  </tr>

  <tr>

   <td width="47">0101</td>

   <td width="47">DR</td>

   <td width="47"> </td>

   <td width="187">PCaddr24</td>

   <td width="47"> </td>

  </tr>

  <tr>

   <td width="47">1000</td>

   <td width="47">DR</td>

   <td width="47">BaseR</td>

   <td width="187">offset20</td>

   <td width="47"> </td>

  </tr>

  <tr>

   <td width="47">1001</td>

   <td width="47">SR</td>

   <td width="47">BaseR</td>

   <td width="187">offset20</td>

   <td width="47"> </td>

  </tr>

  <tr>

   <td width="47">1100</td>

   <td width="47">TR</td>

   <td width="47"> </td>

   <td width="187">unused</td>

   <td width="47"> </td>

  </tr>

  <tr>

   <td width="47">1101</td>

   <td width="47"> </td>

   <td width="47"> </td>

   <td width="187">unused</td>

   <td width="47"> </td>

  </tr>

  <tr>

   <td width="47">1111</td>

   <td width="47"> </td>

   <td width="47"> </td>

   <td width="187">unused</td>

   <td width="47"> </td>

  </tr>

 </tbody>

</table>

ADD

ADDI

NAND

SKP

GOTO

LEA

LW

SW

CALL

RET

HALT

<h3>7.2.1         Conditional Branching</h3>

Conditional branching in the Conte-200 ISA is provided via two instructions: the SKP (“skip”) instruction and the GOTO (“unconditional branch”) instruction.

The SKP instruction compares two registers and skips the immediately following instruction if the comparison evaluates to true. If the action to be conditionally executed is only a single instruction, it can be placed immediately following the SKP instruction. Otherwise a GOTO can be placed following the SKP instruction to branch over to a longer sequence of instructions to be conditionally executed.

<h2>7.3       Detailed Instruction Reference</h2>

<strong>7.3.1        ADD</strong>

<strong>Assembler Syntax</strong>

ADD             DR, SR1, SR2

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0000</td>

   <td width="47">DR</td>

   <td width="47">SR1</td>

   <td width="187">unused</td>

   <td width="47">SR2</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

DR = SR1 + SR2;

<h3>Description</h3>

The ADD instruction obtains the first source operand from the SR1 register. The second source operand is obtained from the SR2 register. The second operand is added to the first source operand, and the result is stored in DR.

<strong>7.3.2        ADDI</strong>

<strong>Assembler Syntax</strong>

ADDI            DR, SR1, immval20

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0001</td>

   <td width="47">DR</td>

   <td width="47">SR1</td>

   <td width="234">immval20</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

DR = SR1 + SEXT(immval20);

<h3>Description</h3>

The ADDI instruction obtains the first source operand from the SR1 register. The second source operand is obtained by sign-extending the immval20 field to 32 bits. The resulting operand is added to the first source operand, and the result is stored in DR.

<strong>7.3.3         NAND</strong>

<strong>Assembler Syntax</strong>

NAND         DR, SR1, SR2

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0010</td>

   <td width="47">DR</td>

   <td width="47">SR1</td>

   <td width="187">unused</td>

   <td width="47">SR2</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

DR = ~(SR1 &amp; SR2);

<h3>Description</h3>

The NAND instruction performs a logical NAND (AND NOT) on the source operands obtained from SR1 and SR2. The result is stored in DR.

<table width="623">

 <tbody>

  <tr>

   <td width="623"><strong>HINT: </strong>A logical NOT can be achieved by performing a NAND with both source operands the same.For instance,NAND DR, SR1, SR1…achieves the following logical operation: <em>DR</em>←<em>SR</em>1.</td>

  </tr>

 </tbody>

</table>

<strong>7.3.4        SKP</strong>

<h3>Assembler Syntax</h3>

SKPNE          SR1, SR2

SKPLE           SR1, SR2

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0011</td>

   <td width="47">mode</td>

   <td width="47">SR1</td>

   <td width="187">unused</td>

   <td width="47">SR2</td>

  </tr>

 </tbody>

</table>

mode is defined to be 0x0 for SKPNE, and 0x1 for SKPLE.

<h3>Operation</h3>

if (MODE == 0x0) { if (SR1 != SR2) PC = PC + 1;

} else if (MODE == 0x1) { if (SR1 &lt;= SR2) PC = PC + 1;

}

<h3>Description</h3>

The SKP instruction compares the source operands SR1 and SR2 according to the rule specified by the mode field. For mode 0x0, the comparison succeeds if SR1 does NOT equal SR2. For mode 0x1, the comparison succeeds if SR1 is less than or equal to SR2.

If the comparison succeeds, the incremented PC (address of instruction + 1) is incremented again, for a resulting PC of (address of instruction + 2). <strong>This effectively “skips” the immediately following instruction. </strong>If the comparison fails, the program continues execution as normal.

<strong>7.3.5         GOTO</strong>

<strong>Assembler Syntax</strong>

GOTO       LABEL

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0100</td>

   <td width="47">0000</td>

   <td width="281">PCaddr24</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

PC = ZEXT(PCaddr24);

<h3>Description</h3>

The program unconditionally branches to the location specified by the zero-extended bits [23:0]. <strong>This instruction is not PC-relative. It goes exactly to the address specified in the PCaddr24 field.</strong>

<strong>7.3.6        LEA</strong>

<strong>Assembler Syntax</strong>

LEA              DR, label

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">0101</td>

   <td width="47">DR</td>

   <td width="281">PCaddr24</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

DR = ZEXT(PCaddr24);

<h3>Description</h3>

An address is computed by zero-extending bits [23:0] to 32 bits and storing this result in DR. This instruction effectively performs the same computation as the GOTO instruction, but rather than performing an unconditional branch, merely stores the computed address into register DR.

<strong>7.3.7       LW</strong>

<strong>Assembler Syntax</strong>

LW               DR, offset20(BaseR)

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1000</td>

   <td width="47">DR</td>

   <td width="47">BaseR</td>

   <td width="234">offset20</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

DR = MEM[BaseR + SEXT(offset20)];

<h3>Description</h3>

An address is computed by sign-extending bits [19:0] to 32 bits and then adding this result to the contents of the register specified by bits [23:20]. The 32-bit word at this address is loaded into DR.

<strong>7.3.8        SW</strong>

<strong>Assembler Syntax</strong>

SW              SR, offset20(BaseR)

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1001</td>

   <td width="47">SR</td>

   <td width="47">BaseR</td>

   <td width="234">offset20</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

MEM[BaseR + SEXT(offset20)] = SR;

<h3>Description</h3>

An address is computed by sign-extending bits [19:0] to 32 bits and then adding this result to the contents of the register specified by bits [23:20]. The 32-bit word obtained from register SR is then stored at this address.

<strong>7.3.9         CALL</strong>

<strong>Assembler Syntax</strong>

CALL         TR

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1100</td>

   <td width="47">TR</td>

   <td width="281">unused</td>

  </tr>

 </tbody>

</table>

<h3>Operation</h3>

$ra = PC;

PC = TR;

<strong>Description          </strong>First, the incremented PC (address of the instruction + 1) is stored into the $ra register. Next, the PC is loaded with the value of register TR, and the computer resumes execution at the new PC.

<strong>7.3.10        RET</strong>

<strong>Assembler Syntax</strong>

RET

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1101</td>

   <td width="328">unused</td>

  </tr>

 </tbody>

</table>

<strong>Operation</strong>

PC = $ra;

<strong>Description</strong>

The PC is loaded with the value of the $ra register, and the computer resumes execution at the new PC.

<strong>7.3.11         HALT</strong>

<strong>Assembler Syntax</strong>

HALT

<h3>Encoding</h3>

31302928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0

<table width="375">

 <tbody>

  <tr>

   <td width="47">1111</td>

   <td width="328">unused</td>

  </tr>

 </tbody>

</table>

<h3>Description</h3>

The machine is brought to a halt and executes no further instructions.