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

Now, in this attack, the attacker is trying to violate the time limits for the correct operation of the system by reducing the clock period, and as a result, the system cannot continue its normal operation during the periods when the timing limit is violated.For example, if the processor makes a request to read from the memory and this time the attacker violates the time limits, then the processor will not have enough time to receive data from the bus, resulting in incorrect reading from the memory. Also, in cases where the clock period is reduced before the end of the current instruction, the processor jumps to the next instruction before the end of the current instruction, thus losing the result of the current instruction.

![image](https://github.com/Ahsan728/Clock_glitching/assets/34878134/15dca8c6-6267-4b50-bdab-4166c016d416) ![image](https://github.com/Ahsan728/Clock_glitching/assets/34878134/a491ca42-e448-4715-abbf-aefab4841b5f)

This Figure shows the normal clock signal for a pipeline-based processor, as can be seen, this processor performs a new operation on each rising edge of the clock signal, and the clock period is set in such a way that the entire operation has enough time to be done, now if the injection of a fault on the clock signal can cause a new edge or in other words to shorten the period, then the instruction currently being executed in the pipeline will not have enough time to complete.

## Clock glitching parameters
In this work a simple password verification software is run on the ChipWhisperer chip and clock glitching will be used in order to fault the processor into accepting a password that is actually incorrect. This will require skipping the correct instruction during execution, and so in order to achieve a skip, the glitch must be placed correctly. This poses a problem because there are three independent variables to control for a glitch:

- Width: the length of the positive clock, a percentage of the original clock cycle.
- Delay: between the trigger signal and the rising edge of the targeted clock cycle.
- Shift: length from the rising edge of the targeted clock cycle and the start of the clock glitch.

![image](https://github.com/Ahsan728/Clock_glitching/assets/34878134/fc31bac6-40a8-437b-bb76-585c84875140)

## Preliminary steps
After installing the chipwhisper, we must introduce the setup path:
```python
SCOPETYPE = 'OPENADC'
PLATFORM = 'CWLITEARM'
#other platforms: CW308_STM32F1, CW308_STM32F0, CW308_STM32F3, CW308_STM32F4 or CWNANO
CW_PATH = '/home/vagrant/work/projects/chipwhisperer/'
```
### Connecting to target board
The next step is to connect to the chipwhisperer board. To ensure communication, we must receive: Found ChipWhisperer

![image](https://github.com/Ahsan728/Clock_glitching/assets/34878134/1c074339-8317-4e1c-9dfa-0dd00f7141fe)
```python
%run $CW_PATH"jupyter/Setup_Scripts/Setup_Generic.ipynb"
```
### Programming and initializing the target
After connecting the board, we need to program its flash memory:

```python
import chipwhisperer as cw
fw_path = "{}/hardware/victims/firmware/simpleserial-glitch/simpleserial-glitch-{}.hex".format(CW_PATH, PLATFORM)
cw.program_target(scope, prog, fw_path)
scope.io.nrst = False
time.sleep(0.05)
scope.io.nrst = "high_z"
time.sleep(0.05)
target.flush()
```

## Testing nominal behavior
The next step is to ensure the correct operation of the program implemented on the board, for this purpose we first 
enter a wrong password and then a correct password, it can be seen that first an "FAILED" message and then a "SUCCESS" 
message are displayed, so the program correctly Works.

### Wrong password

What happens if a wrong password in inserted? The system should indeed deny the authentication. We use the variable pw to define, in ASCII encoding, the password we want to submit. The communication with the target board is provided through a serial link. We can use the function simpleserial_write to send the command p (send a password) together with a random value, for instance "aeiou". We need then to read the answer from the board with the function simpleserial_read_witherrors and the parameter r (reading). The result is stored in the val variable, which has a complex structure. The interesting field is the rv (return value), storing the vlue of the answer to the passwrod that was sent. As we sent the wrong password, we expect to get rv=0. We check that this is the case by verifying that FAILED is returned.

```python
pw = "aeiou".encode('ascii')
target.simpleserial_write('p', pw)
val = target.simpleserial_read_witherrors('r', 1, glitch_timeout=10)
passok = int.from_bytes(val["payload"], byteorder='big', signed=False)
print(val)
print(passok)
if not(passok):
    print("FAILED")
else:
    print("SUCCESS")
```
### Correct password
We can use the same code to send now the correct password correct ("touch"), as shown below. The returned value should be now rv=1, as it can be checked through the appearanec of the message SUCCESS.

```python
pw = "touch".encode('ascii')
target.simpleserial_write('p', pw)
val = target.simpleserial_read_witherrors('r', 1, glitch_timeout=10)
passok = int.from_bytes(val["payload"], byteorder='big', signed=False)
print(val)
print(passok)
if not(passok):
    print("FAILED")
else:
    print("SUCCESS")

```

## Clock glitch
After validating the expected behaviour of the code, wa aim at changing it through a clock glitch. The following cell defines the parameters needed for the fault injection. Using the function print shows the values of all parameters of the glitch module. We can find here the three parameters described earlier:

- width,
- offset (shift),
- ext_offset (delay).
  
We need to tune these three parameters accordingly.

```python
scope.glitch.clk_src = "clkgen"
scope.glitch.output = "clock_xor"
scope.glitch.trigger_src = "ext_single"
scope.io.hs2 = "glitch"
scope.adc.timeout = 0.1
print(scope.glitch)
```
## Characterization code

There are three parameters to define: the width, the offset (shift), and the delay. At first, there is no easy way to guess the best correct values that will allow to inject an exploitable fault. In principle, we should therefore explore the full search space in order to find the optimal combination of values, leading to a successful attack (being authorized with the wrong password).

If we were to test all possible values, this would mean:

- every possible width, between 1 and 49, by steps of 1: 49 possible values,
- every possible offset, between -49 et +49, by steps of 1: 99 possible values,
- every possible clock cycle (delay) between 1 and 200, by steps of 1: 200 possible values.

The exhaustive search for the above space would lead to:  49×99×200≃106  possible combinations. Even if we were able to try 100 triples per second (which is already an upper bound!), this process would require  104  seconds, i.e., a bit less than 3 hours. How much time do you still have today, before the end of the session?

Even if exhaustive search of the parameter space may be a viable (and somteimes the only) approach, we are going to speed things up a little. In fact, this method is time consuming as the three parameters and independents of each other and all combinations should be tested. We will see how to reduce the degrees of freedom and thus the number of dimensions to explore.

In particular, the delay is mostly dependent on the application we are targeting, whereas the width and the offset depend on the actual physical properties of the platform. We will resort to a characterisation algorithm, where faults will be easily observable. This code is made by two nested for loops, each made of 50 iterations, where a global counter is incremented. The value of this counter is read at the end. Normally, the expected result would be 50×50=2500. On the other hand, one for loop can be easily faulted: two nested loops even more easily.

The goal is to find the values for width and offset that lead to an actual (potentially exploitable) error. In order to relax the constraint on the delay value (needed to target a specific instruction), we will try to inject several consecutive clock glitches. To achieve this effect, we are going to set the parameter repeat to a large value (for instance, 20), to inject several consecutive glitches.

```python
scope.glitch.repeat = 20
```
In fact, we do not care in this case about the value of the output. The goal is to find the optimal values for width and offset (target dependent) leading to an error. This will largely reduce the search space.

## Reboot function
It is possible that the error induced by the fault injection will not allow the CPU to continue its normal execution. In this case, the system will require a reset. The function here below allows to reset the board in case it stops responding.

```python
def reboot_flush():            
    scope.io.nrst = False
    time.sleep(0.05)
    scope.io.nrst = "high_z"
    time.sleep(0.05)
    #Flush garbage too
    target.flush()
```

On the terminal standard output, there will also be a lot of warning messages that might make the analysis of the output too hard. The following cell filters this warnings, limiting the output messages to more serious errors.

```python
import logging
logging.basicConfig()
logging.getLogger().setLevel(logging.ERROR)
```
## Characterization loop

Let us do the space exploration to find the good pairs of values for the width and the offset. YOU have to choose the boundaires for the values to test. Just remember:

width : possible values are in the range ]0; 50[. Values too small will create glitches that will not even be detected by the system; a value too large will allow the instruction to complete normally, thus giving no effects. offset : possible values are in the range ]-50; 50[. We may constrain the search to negative values, in order to define a glitch occurring before a rising edge. Likely, smaller values may go unnoticed by the system, thus useless, whereas excessive values will let the instruction to execute regularly.
You need to complete yourselves these value in the cell below. The step for delay parameter should be of 20 units, as we are also injecting 20 consecutive glitches each time.

Each parameter set is tested three times, as it is possible that some fault injections may fail.


```python
reboot_flush()

resets   = [] # List to save those parameters leading to a reset
glitches = [] # List to save those parameters leading to an actual (useful) glitch

largeurs  = []  # TO BE COMPLETED
decalages = []  # TO BE COMPLETED
delais    = []  # TO BE COMPLETED

for scope.glitch.width in largeurs:
    print("Width : {}".format(scope.glitch.width))
    for scope.glitch.offset in decalage:
        for scope.glitch.ext_offset in delais:
            for repetition in range(3):
                scope.arm()

                target.write("g\n")
                ret = scope.capture()
                val = target.simpleserial_read_witherrors('r', 4, glitch_timeout=10)

                if ret: #here the trigger never went high - sometimes the target is stil crashed from a previous glitch
                    resets.append((scope.glitch.width, scope.glitch.offset))
                    reboot_flush()
                elif val["payload"]:
                    loop_counter = int.from_bytes(val["payload"], byteorder='little')
                    if loop_counter != 2500:
                        glitches.append((scope.glitch.width, scope.glitch.offset))
```
To identify those parameters that lead to a successful fault injection, we can trace a graph from the collected data where:

- on X axis : the width of the pulse,
- on Y axis : its offset.
This can be achieved by executing the next cell, using the glitches and resets vectors that we just filled in.
```python
import matplotlib.pyplot as plt
%matplotlib inline

plt.xlabel("Width")
plt.ylabel("Offset")
plt.scatter(x=[width for (width, offset) in resets],
            y=[offset for (width, offset) in resets],
            marker="x",
            s=100,
            color="red",
            label="reset")
plt.scatter(x=[width for (width, offset) in glitches],
            y=[offset for (width, offset) in glitches],
            marker="o",
            color="blue",
            label="fault")
plt.legend()
plt.show()
```
By using these values, it can be seen that we have been able to perform successful attacks, which means bypassing the password check, and thus the attacker can enter the system:

![image](https://github.com/Ahsan728/Clock_glitching/assets/34878134/fd1d3996-ab27-44d6-ae41-da0b96324949) 

As shown in Figure, the blue dots represent the successful glitch parameters, which means that the attacker was able to enter the system, and the red dots represent the states where the system is reset after the fault injection. Now, if we want to look from an attacker's point of view, the parameters that cause the attack to succeed but do not cause the system to restart are the best, Because the system could not detect this attack and reset itself !




