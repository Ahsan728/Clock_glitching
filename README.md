# Fault Injection
The process of purposefully creating flaws or mistakes in hardware components to get around security features like password authentication or debug protection is known as 
fault injection, and it is a technique used to evaluate a device's security. The purpose of these injections is to corrupt memory transactions or skip instructions; they 
should occur at a predetermined time and for a certain amount of time. Hardware gadgets are one way to accomplish that.
- Software techniques.
- Hardware techniques.
- an approach of software and hardware.

In recent years, this type of assault has become more common in sensitive areas like content protection and payment cards, and it has also become an exploitable technique.
Clock glitching is one of them.

# Bypassing pin clock glitch

Clock glitching is widely adapted technique for fault injection. Clock glitching insert the short glitches in device clock that causes the timing violation inside any system. 
In sequential circuits, clock signals are primarily used to control the device output, nevertheless, the period of the clock depends on the critical path delay.

![image](https://github.com/Ahsan728/Clock_glitching/assets/34878134/27079f85-28f6-4a23-99b1-b9f57de81afd)

Figure shows the structure of a simple digital circuit where there exists combinational circuit between two sequential elements. Data is transmitted from the source register 
at the rising edge of the clock signal, processed by the combination part, and finally stored in the destination register on the next rising edge.

As a result, the period of the clock signal must be larger than the sum of the propagation delay time (critical path) and the setup time to guarantee the correct operation 
of the processor. Otherwise, the processor will not have enough time to perform its tasks. Therefore, for a digital system to work correctly, the following relationship must 
always be true:
- 𝑇𝑐𝑙𝑜𝑐𝑘 > 𝑇𝑆𝑒𝑡𝑢𝑝 𝑇𝑖𝑚𝑒 +𝑇𝐶𝑟𝑖𝑡𝑖𝑐𝑎𝑙 𝑝𝑎𝑡ℎ…………….(1.1)

Now, in this attack, the attacker is trying to violate the time limits for the correct operation of the system by reducing the clock period, and as a result, the system cannot 
continue its normal operation during the periods when the timing limit is violated.For example, if the processor makes a request to read from the memory and this time the attacker 
violates the time limits, then the processor will not have enough time to receive data from the bus, resulting in incorrect reading from the memory. Also, in cases where the clock 
period is reduced before the end of the current instruction, the processor jumps to the next instruction before the end of the current instruction, thus losing the result of the 
current instruction.

![image](https://github.com/Ahsan728/Clock_glitching/assets/34878134/15dca8c6-6267-4b50-bdab-4166c016d416) ![image](https://github.com/Ahsan728/Clock_glitching/assets/34878134/a491ca42-e448-4715-abbf-aefab4841b5f)

This Figure shows the normal clock signal for a pipeline-based processor, as can be seen, this processor performs a new operation on each rising edge of the clock signal, and 
the clock period is set in such a way that the entire operation has enough time to be done, now if the injection of a fault on the clock signal can cause a new edge or in 
other words to shorten the period, then the instruction currently being executed in the pipeline will not have enough time to complete.
