# Static Timing Analysis with OpenSTA
OpenSTA (Open Static Timing Analyzer) is a versatile tool used for timing analysis in digital circuits. To install OpenSTA, ensure your system is set up with the necessary build tools like GCC, Make, and Tcl/Tk development libraries. The installation process typically involves cloning the OpenSTA GitHub repository, building the source code, and adding the compiled binary to your system's PATH for easy access.


## Static timing analysis using OpenSTA

#### VSDBabySoC basic timing analysis

      sta
      read_liberty -min sky130_fd_sc_hd__tt_025C_1v80.lib
      read_liberty -max sky130_fd_sc_hd__tt_025C_1v80.lib
      read_liberty -min avsdpll.lib
      read_liberty -max avsdpll.lib
      read_liberty -min avsddac.lib
      read_liberty -max avsddac.lib
      read_verilog /home/lakshay/vsdflow/VSDBabySoC/output/synth/vsdbabysoc_new.synth.v
      link_design vsdbabysoc
      read_sdc /home/lakshay/vsdflow/VSDBabySoC/src/sdc/synth.sdc
      report_checks
     
     
<img src="https://github.com/Lakshay-Kaushik-2025/lakshay-vsd/blob/main/Week_3/Task3/images/sta_analysis.png" alt="Design & Testbench Overview" width="100%">

     
### **VSDBabySoC PVT Corner Analysis (Post-Synthesis Timing)**  
STA is performed across all PVT corners to validate that the design meets timing requirements.

The worst max path (Setup-critical) corners in sub-40nm nodes are generally:  
- **ss_LowTemp_LowVolt**  
- **ss_HighTemp_LowVolt** *(Slowest corners)*  

The worst min path (Hold-critical) corners are:  
- **ff_LowTemp_HighVolt**  
- **ff_HighTemp_HighVolt** *(Fastest corners)*  



#### the TCL file is 
     set list_of_lib_files(1)  "sky130_fd_sc_hd__tt_025C_1v80.lib"
     set list_of_lib_files(2)  "sky130_fd_sc_hd__ff_100C_1v65.lib"
     set list_of_lib_files(3)  "sky130_fd_sc_hd__ff_100C_1v95.lib"
     set list_of_lib_files(4)  "sky130_fd_sc_hd__ff_n40C_1v56.lib"
     set list_of_lib_files(5)  "sky130_fd_sc_hd__ff_n40C_1v65.lib"
     set list_of_lib_files(6)  "sky130_fd_sc_hd__ff_n40C_1v76.lib"
     set list_of_lib_files(7)  "sky130_fd_sc_hd__ss_100C_1v40.lib"
     set list_of_lib_files(8)  "sky130_fd_sc_hd__ss_100C_1v60.lib"
     set list_of_lib_files(9)  "sky130_fd_sc_hd__ss_n40C_1v28.lib"
     set list_of_lib_files(10) "sky130_fd_sc_hd__ss_n40C_1v35.lib"
     set list_of_lib_files(11) "sky130_fd_sc_hd__ss_n40C_1v40.lib"
     set list_of_lib_files(12) "sky130_fd_sc_hd__ss_n40C_1v44.lib"
     set list_of_lib_files(13) "sky130_fd_sc_hd__ss_n40C_1v76.lib"

     # Paths
     set lib_path   "/home/lakshay/vsdflow/VSDBabySoC/output/sta_output/skywater-pdk-libs-sky130_fd_sc_hd/timing"
     set design_v   "/home/lakshay/sky130RTLDesignAndSynthesisWorkshop/verilog_files/rvmyth_synth.v"
     set pll_lib    "/home/lakshay/vsdflow/VSDBabySoC/src/lib/avsdpll.lib"
     set dac_lib    "/home/lakshay/vsdflow/VSDBabySoC/src/lib/avsddac.lib"
     set sdc_file   "/home/lakshay/vsdflow/VSDBabySoC/src/sdc/synth.sdc"
     set report_dir "/home/lakshay/vsdflow/VSDBabySoC/output/sta_output"

     # -------------------------------------------------------------
     # Read Design and Common Libraries (done once)
     # -------------------------------------------------------------

     read_liberty /home/lakshay/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
     read_verilog $design_v
     link_design rvmyth
     current_design rvmyth

     # Read constraint file
     read_sdc $sdc_file

     # Read custom block libraries
     read_liberty $pll_lib
     read_liberty $dac_lib

     # -------------------------------------------------------------
     # Loop over each PVT corner
     # -------------------------------------------------------------
     for {set i 1} {$i <= [array size list_of_lib_files]} {incr i} {

    # Clear previously loaded timing libs (for per-corner STA)
   

    # Read standard cell library for this corner
    set this_lib "$lib_path/$list_of_lib_files($i)"
    puts "\n=== Running STA for $this_lib ==="
    read_liberty $this_lib

    # Run setup/hold checks
    check_setup -verbose

    # Generate reports
    report_checks -path_delay min_max -fields {nets cap slew input_pins fanout} -digits 4 \
        > "$report_dir/min_max_$list_of_lib_files($i).txt"

    # Worst max slack
    exec echo "\n$list_of_lib_files($i)" >> "$report_dir/sta_worst_max_slack.txt"
    report_worst_slack -max -digits 4 >> "$report_dir/sta_worst_max_slack.txt"

    # Worst min slack
    exec echo "\n$list_of_lib_files($i)" >> "$report_dir/sta_worst_min_slack.txt"
    report_worst_slack -min -digits 4 >> "$report_dir/sta_worst_min_slack.txt"

    # Total negative slack
    exec echo "\n$list_of_lib_files($i)" >> "$report_dir/sta_tns.txt"
    report_tns -digits 4 >> "$report_dir/sta_tns.txt"

    # Worst negative slack
    exec echo "\n$list_of_lib_files($i)" >> "$report_dir/sta_wns.txt"
    report_wns -digits 4 >> "$report_dir/sta_wns.txt"
     }

    


