# VSDBabySoC

VSDBabySoC is a small SoC including PLL, DAC, and a RISCV-based processor named RVMYTH.



# VSDBabySoC Modeling

Here we are going to model and simulate the VSDBabySoC using `iverilog`, then we will show the results using `gtkwave` tool. Some initial input signals will be fed into `vsdbabysoc` module that make the pll start generating the proper `CLK` for the circuit. The clock signal will make the `rvmyth` to execute instructions in its `imem`. As a result the register `r17` will be filled with some values cycle by cycle. These values are used by dac core to provide the final output signal named `OUT`. So we have 3 main elements (IP cores) and a wrapper as an SoC and of-course there would be also a testbench module out there.


## Step by step modeling walkthrough

In this section we will walk through the whole process of modeling the VSDBabySoC in details. We will increase/decrease the digital output value and feed it to the DAC model so we can watch the changes on the SoC output. Ubuntu(18.04 LTS)

  1. Installing the dependencies in the system:

  ```
  $ sudo apt install make python python3 python3-pip git iverilog gtkwave docker.io
  $ sudo chmod 666 /var/run/docker.sock
  $ cd ~
  $ pip3 install pyyaml click sandpiper-saas
  ```

  2. Cloning the VSDBabySoC repository:

  ```
  $ cd ~
  $ git clone https://github.com/manili/VSDBabySoC.git
  ```

  3. Synthesis:

  ```
  $ cd VSDBabySoC
  $ make pre_synth_sim
  ```
  
  The result of the simulation (i.e. `pre_synth_sim.vcd`) will be stored in the `output/pre_synth_sim` directory.

  4. Waveform Simulation:

  ```
  $ gtkwave output/pre_synth_sim/pre_synth_sim.vcd
  ```
  
  Two most important signals are `CLK` and `OUT`. The `CLK` signal is provided by the PLL and the `OUT` is the output of the DAC model. Here is the final result of the modeling process:
  
  <img src="https://github.com/Lakshay-Kaushik-2025/lakshay-vsd/blob/main/Week_1/Day_1/images/testbench_components.png" alt="Design & Testbench Overview" width="70%">

In this picture we can see the following signals:

  * **CLK:** This is the `input CLK` signal of the `RVMYTH` core. This signal comes from the PLL, originally.
  * **reset:** This is the `input reset` signal of the `RVMYTH` core. This signal comes from an external source, originally.
  * **OUT:** This is the `output OUT` signal of the `VSDBabySoC` module. This signal comes from the DAC (due to simulation restrictions it behaves like a digital signal which is incorrect), originally.
  * **RV_TO_DAC[9:0]:** This is the 10-bit `output [9:0] OUT` port of the `RVMYTH` core. This port comes from the RVMYTH register #17, originally.
  * **OUT:** This is a `real` datatype wire which can simulate analog values. It is the `output wire real OUT` signal of the `DAC` module. This signal comes from the DAC, originally.

**PLEASE NOTE** that the sythesis process does not support `real` variables, so we must use the simple `wire` datatype for the `\vsdbabysoc.OUT` instead. The `iverilog` simulator always behaves `wire` as a digital signal. As a result we can not see the analog output via `\vsdbabysoc.OUT` port and we need to use `\dac.OUT` (which is a `real` datatype) instead.

# OpenLANE

OpenLANE is an automated RTL to GDSII flow based on several components including OpenROAD, Yosys, Magic, Netgen, Fault, SPEF-Extractor and custom methodology scripts for design exploration and optimization. The main usage of OpenLANE in this project is for [VSDBabySoC Physical Design](#vsdbabysoc-physical-design). However, we need OpenLANE for the synthesis and STA process in the [Post-synthesis simulation](#post-synthesis-simulation) section. So we'll talk about its installation process here and let the details be until the [VSDBabySoC Physical Design](#vsdbabysoc-physical-design) section.

## OpenLANE installation

The OpenLANE and sky130 installation can be done by following the steps in this repository `https://github.com/nickson-jose/openlane_build_script`.

* More information on OpenLANE can be found in the following repositories:

  * `https://github.com/The-OpenROAD-Project/OpenLane`
  * `https://github.com/efabless/openlane`

To summerize the installation processes:

  ```
  $ git clone https://github.com/The-OpenROAD-Project/OpenLane.git
  $ cd OpenLane/
  $ make openlane
  $ make pdk
  $ make test
  ```

For more info please refer to the GitHub repositories.

**PLEASE NOTE** that currently we are using commit version `8580c248a995b575f7734813b80bb6c4aa82d4f2` for the OpenLANE and our docker image version is `2021.09.09_03.00.48`.

# Post-synthesis simulation

First step in the design flow is to synthesize the generated RTL code and after that we will simulate the result. This way we can find more about our code and its bugs. So in this section we are going to synthesize our code then do a post-synthesis simulation to look for any issues. The post and pre (modeling section) synthesis results should be identical.

## Synthesizing using Yosys

* In OpenLANE the RTL synthesis is performed by `yosys`.
* The technology mapping is performed by `abc`.
* Finally, the timing reports for the synthesized netlist are generated by `OpenSTA`.

## How to synthesize the design

To perform the synthesis process do the following:

  ```
  $ cd ~/VSDBabySoC
  $ make synth
  ```

The heavy job will be done by the script. When the process has been done, we can see the result in the `output/synth/vsdbabysoc.synth.v` file.

## Post-synthesis simulation (GLS)

There is an issue for post-synthesis simulation (Gate-Level Simulation) which can be tracked [here](https://github.com/google/skywater-pdk/issues/310). However, we hacked the source-code by the following instructions and we managed to workaround the issue for now ([here is the reference](https://github.com/The-OpenROAD-Project/OpenLane/issues/518)):

  1. In `$YOUR_PDK_PATH/sky130A/libs.ref/sky130_fd_sc_hd/verilog/sky130_fd_sc_hd.v` file we should manually correct `endif SKY130_FD_SC_HD__LPFLOW_BLEEDER_FUNCTIONAL_V` to `endif //SKY130_FD_SC_HD__LPFLOW_BLEEDER_FUNCTIONAL_V`.
  2. We can simulate with the functional models by passing the `FUNCTIONAL` define to `iverilog`. Also we need to set `UNIT_DELAY` macro to some value. As a result we'll have `iverilog -DFUNCTIONAL -DUNIT_DELAY=#1 <THE SOURCE-CODEs TO BE COMPILED>`.

User could bypass these confusing steps by using our provided Makefile:

  ```
  $ cd ~/VSDBabySoC
  $ make post_synth_sim
  ```
The result of the simulation (i.e. `post_synth_sim.vcd`) will be stored in the `output/post_synth_sim` directory and the waveform could be seen by the following command:

  ```
  $ gtkwave output/post_synth_sim/post_synth_sim.vcd
  ```
Here is the final result:

  ![post_synth_sim](images/post_synth_sim.png)

In this picture we can see the following signals:

  * **\core.CLK:** This is the `input CLK` signal of the `RVMYTH` core. This signal comes from the PLL, originally.
  * **reset:** This is the `input reset` signal of the `RVMYTH` core. This signal comes from an external source, originally.
  * **OUT:** This is the `output OUT` signal of the `VSDBabySoC` module. This signal comes from the DAC (due to simulation restrictions it behaves like a digital signal which is incorrect), originally.
  * **\core.OUT[9:0]:** This is the 10-bit `output [9:0] OUT` port of the `RVMYTH` core. This port comes from the RVMYTH register #17, originally.
  * **OUT:** This is a `real` datatype wire which can simulate analog values. It is the `output wire real OUT` signal of the `DAC` module. This signal comes from the DAC, originally.

**PLEASE NOTE** that the sythesis process does not support `real` variables, so we must use the simple `wire` datatype for the `\vsdbabysoc.OUT` instead. The `iverilog` simulator always behaves `wire` as a digital signal. As a result we can not see the analog output via `\vsdbabysoc.OUT` port and we need to use `\dac.OUT` (which is a `real` datatype) instead.

## Yosys final report

  ```
  === vsdbabysoc ===

   Number of wires:               5559
   Number of wire bits:           5559
   Number of public wires:        1323
   Number of public wire bits:    1323
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:               5552
     avsddac                         1
     avsdpll1v8                      1
     sky130_fd_sc_hd__a211o_2        1
     sky130_fd_sc_hd__a21o_2         4
     sky130_fd_sc_hd__a21oi_2       19
     sky130_fd_sc_hd__a221o_2       56
     sky130_fd_sc_hd__a22o_2        32
     sky130_fd_sc_hd__a2bb2o_2      14
     sky130_fd_sc_hd__a2bb2oi_2     12
     sky130_fd_sc_hd__a311o_2        1
     sky130_fd_sc_hd__a31o_2         7
     sky130_fd_sc_hd__a31oi_2        3
     sky130_fd_sc_hd__a32o_2         8
     sky130_fd_sc_hd__a41o_2         1
     sky130_fd_sc_hd__and2_2        38
     sky130_fd_sc_hd__and3_2         5
     sky130_fd_sc_hd__and4b_2        1
     sky130_fd_sc_hd__buf_1        885
     sky130_fd_sc_hd__conb_1         6
     sky130_fd_sc_hd__dfxtp_2     1144
     sky130_fd_sc_hd__inv_2       1026
     sky130_fd_sc_hd__mux2_1       513
     sky130_fd_sc_hd__nand2_2        3
     sky130_fd_sc_hd__nand4_2       32
     sky130_fd_sc_hd__nor2_2        61
     sky130_fd_sc_hd__nor2b_2        1
     sky130_fd_sc_hd__nor4_2         2
     sky130_fd_sc_hd__o2111a_2       1
     sky130_fd_sc_hd__o2111ai_2     65
     sky130_fd_sc_hd__o211a_2        4
     sky130_fd_sc_hd__o21a_2         6
     sky130_fd_sc_hd__o21ai_2        9
     sky130_fd_sc_hd__o221a_2      955
     sky130_fd_sc_hd__o221ai_2       2
     sky130_fd_sc_hd__o22a_2       427
     sky130_fd_sc_hd__o2bb2a_2      23
     sky130_fd_sc_hd__o2bb2ai_2      2
     sky130_fd_sc_hd__o311a_2        2
     sky130_fd_sc_hd__o31a_2        10
     sky130_fd_sc_hd__o32a_2        15
     sky130_fd_sc_hd__or2_2         48
     sky130_fd_sc_hd__or2b_2        32
     sky130_fd_sc_hd__or3_2         35
     sky130_fd_sc_hd__or4_2         36
     sky130_fd_sc_hd__or4b_2         3

   Area for cell type \avsddac is unknown!
   Area for cell type \avsdpll1v8 is unknown!

   Chip area for module '\vsdbabysoc': 58173.292800
  ```


