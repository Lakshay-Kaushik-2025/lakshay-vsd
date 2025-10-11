# GLS OF BABYSOC
## POST-SYNTHESIS SIMULATION

### Purpose of GLS:
Gate-Level Simulation is used to verify the functionality of a design after the synthesis process. Unlike behavioral or RTL (Register Transfer Level) simulations, which are performed at a higher level of abstraction, GLS works on the netlist generated post-synthesis. This netlist includes the actual gates and connections used to implement the design.

### Key Aspects of GLS for BabySoC:
1. **Verification with Timing Information:**
   - GLS is performed using Standard Delay Format (SDF) files to ensure timing correctness.
   - This checks if the SoC behaves as expected under real-world timing constraints.

2. **Design Validation Post-Synthesis:**
   - Confirms that the design's logical behavior remains correct after mapping it to the gate-level representation.
   - Ensures that the design is free from issues like metastability or glitches.

3. **Simulation Tools:**
   - Tools like Icarus Verilog or a similar simulator can be used for compiling and running the gate-level netlist.
   - Waveforms are typically analyzed using GTKWave.

4. **Importance for BabySoC:**
   - BabySoC consists of multiple modules like the RISC-V processor, PLL, and DAC. GLS ensures that these modules interact correctly and meet the timing requirements in the synthesized design.


Here is the step-by-step execution plan for running the  commands manually:
---
### **Run the following commands**
```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog -I /home/lakshay/vsdflow/VSDBabySoC/src/include /home/lakshay/vsdflow/VSDBabySoC/src/module/clk_gate.v
read_verilog -I /home/lakshay/vsdflow/VSDBabySoC/src/include /home/lakshay/vsdflow/VSDBabySoC/src/module/out/rvmyth.v
synth -top rvmyth
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}
flatten
setundef -zero
clean -purge
rename -enumerate
stat
write_verilog -noattr rvmyth_synth.v

```

<img src="https://github.com/Lakshay-Kaushik-2025/lakshay-vsd/blob/main/Week_3/Task1/images/synth_stat.png" alt="Design & Testbench Overview" width="100%">




## POST_SYNTHESIS SIMULATION AND WAVEFORMS
---

### **Step 1: Compile the Testbench**
Run the following `iverilog` command to compile the testbench:
```bash
iverilog -o /home/ananya123/VSDBabySoCC/VSDBabySoC/output/post_synth_sim/post_synth_sim.out -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 -I /home/ananya123/VSDBabySoCC/VSDBabySoC/src/include -I /home/ananya123/VSDBabySoCC/VSDBabySoC/src/module /home/ananya123/VSDBabySoCC/VSDBabySoC/src/module/testbench.v
```
---
### **Step 2: Navigate to the Post-Synthesis Simulation Output Directory**
```bash
cd output/post_synth_sim/
```
---
### **Step 3: Run the Simulation**

```bash
./post_synth_sim.out
```
---
### **Step 4: View the Waveforms in GTKWave**

```bash
gtkwave post_synth_sim.vcd
```
---

![WhatsApp Image 2024-11-25 at 9 07 01 PM](https://github.com/user-attachments/assets/9d79b832-7315-46ed-b028-e2dd5d14d27a)

![WhatsApp Image 2024-11-25 at 9 07 01 PM (2)](https://github.com/user-attachments/assets/0a6d272d-1aae-45b5-b6aa-914a0087df84)

![WhatsApp Image 2024-11-25 at 9 07 01 PM (1)](https://github.com/user-attachments/assets/239557c2-2447-4cd1-a18f-fb1966feebf2)

