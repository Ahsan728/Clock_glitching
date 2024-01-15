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
- 
We need to tune these three parameters accordingly.

```python
scope.glitch.clk_src = "clkgen"
scope.glitch.output = "clock_xor"
scope.glitch.trigger_src = "ext_single"
scope.io.hs2 = "glitch"
scope.adc.timeout = 0.1
print(scope.glitch)
```
