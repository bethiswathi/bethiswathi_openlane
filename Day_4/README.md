# 🎯 Pre-Layout Timing Analysis and Importance of Clock Tree
To run previous flow, add tag to prep design:
```
prep -design picorv32a -tag [date]
```
## ✅ Lab 1: Extracting the LEF file
- Place-and-Route (PnR) tools do not use the full Magic .mag layout (which contains layers, shapes, and transistor details).
- Instead, they only need abstract information that allows placement and routing.
- This abstract view is provided by the LEF (Library Exchange Format) file.

## ✅ What PnR Needs From a Standard Cell (LEF Contains):
- Cell boundary (width and height)
- Power/Ground terminals (VDD, VSS rails)
- Input/Output pins with:
- Pin shapes
- Pin positions
- Routing layer information
- PnR tools do not need:
- Transistor geometry
- Device connectivity
- Diffusion / poly / well layers
(these remain in the GDS, not the LEF)


## ✅ Guidelines for Standard-Cell Layout for PnR Compatibility

To ensure the PnR tool can route to the cells correctly, the layout must follow these rules:

1. Pin Alignment
- Input and output pins must lie at track intersections.
   → ensures routers can reach the pins easily.

2. Cell Width and Height Must Align to Routing Grid
- Width must be an odd multiple of the horizontal track pitch.
- Height must be an odd multiple of the vertical track pitch.

This ensures:
- Cells align properly in rows.
- Power/ground rails line up across all cells.
- Routing tracks line up from cell to cell

To check these guidelines, we need to change the grid of Magic to match the actual metal tracks. The pdks/sky130A/libs.tech/openlane/sky130_fd_sc_hd/tracks.info contains those metal informations.

1. Use grid command inside the tkon terminal to match the tracks informations:

<img width="803" height="459" alt="image" src="https://github.com/user-attachments/assets/c30a08df-79f1-4945-a089-af92af63afc9" />

The grids show where the routing for the local-interconnet layer can only happen, the distance of the grid lines are the required pitch of the wire. Below, we can see that the guidelines are satisfied:

<img width="593" height="358" alt="image" src="https://github.com/user-attachments/assets/8843e5c8-74ca-4a99-aa56-6fe844794663" />

2. Next, we will extract the LEF file. The LEF file contains the cell size, port definitions, and properties which aid the placer and router tool. With that, the ports definition, port class, and port use must be set first. The instructions to set these definitions via Magic are on the <a href="https://github.com/nickson-jose/vsdstdcelldesign#create-port-definition">vsdstdcelldesign repo</a>.

3. Next, save the mag file with a new filename save sky130_myinverter.mag. Then type lef write on the tcon terminal. It will generate a LEF file with same name as the magfile sky130_myinverter.lef. Inside that LEF file is: 

<img width="940" height="441" alt="image" src="https://github.com/user-attachments/assets/ef9418a3-44b6-4f8c-91f7-31d4db30a4e0" />


## ✅ Lab 2: Plug-in the customized inverter cell into OpenLane
- The SKY130 PDK provides Liberty <a href="https://teamvlsi.com/2020/05/lib-and-lef-file-in-asic-design.html">(.lib) timing files</a> in sky130_fd_sc_hd/lib/, containing timing and power data for each standard cell.
- These files exist for different <a href="https://chipedge.com/resources/what-are-pvt-corners-in-vlsi/">PVT corners</a> (process–voltage–temperature), such as slow, typical, fast and voltages like 1.65 V, 1.80 V, 1.95 V.
- A file like sky130_fd_sc_hd__ss_025C_1v80.lib represents the slow–slow corner, 25 °C, at 1.8 V, giving worst-case delays.
- The timing/power values inside a liberty file come from detailed simulations of each cell across multiple conditions.
- During synthesis, ABC uses these liberty files to map generic logical gates to actual standard-cell implementations.
- The library contains multiple sizes/flavors of the same gate (e.g., inverter size-1 vs size-4), where larger cells are built from multiple smaller ones to provide higher drive strength.

<img width="759" height="396" alt="image" src="https://github.com/user-attachments/assets/68aaa5d6-8dbe-4c75-8acb-1f0645bcb016" />


Provided inside the cloned vsdstdcelldesign are the liberty files containing the customized inverter cell.

1. Copy the extracted lef file sky130_myinverter.lef and the liberty files sky130*.lib from /openlane/vsdstdcelldesign/libs to the src directory of picorv32a. Open each liberty files then change the cell name sky130_vsdinv to sky130_myinverter to match the new LEF file cell name.

<img width="940" height="212" alt="image" src="https://github.com/user-attachments/assets/fddabf0d-344b-4427-96f8-cde30ba40611" />

2. Add the folowing to config.tcl inside the picorv32a
```
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```
This sets the liberty file that will be used for ABC mapping of synthesis (LIB_SYNTH) and for STA (_FASTEST,_SLOWEST,_TYPICAL) and also the extra LEF files (EXTRA_LEFS) for the customized inverter cell. The whole config.tcl then is:

<img width="840" height="290" alt="image" src="https://github.com/user-attachments/assets/caf34e86-bbf3-4296-bbea-16834e39b457" />

3. Run docker and prepare the design picorv32a. Plug the new lef file to the OpenLANE flow via
```
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
```
4. Next run_synthesis. Below is the synthesis statistics report runs/[date]/reports/synthesis/1-synthesis.AREA_0.stat.rpt after the run, and as we can see sky130_myinverter cell is successfully included in the design!

<img width="753" height="214" alt="image" src="https://github.com/user-attachments/assets/ab9a9975-1a12-4acc-b94c-b50d1c04259d" />

HOWEVER, looking at the STA log runs/[date]/logs/synthesis/sta.log, we are not meeting timing. Our neext goal is to solve this negative slack:

<img width="840" height="209" alt="image" src="https://github.com/user-attachments/assets/5726b717-0063-4343-9b08-fc99983523fb" />


## ✅ Need for Delay Tables
- Cell delay depends on input slew and output load, not a single fixed number.
- Delay varies across PVT corners (process, voltage, temperature).
- CMOS cells exhibit non-linear timing behavior, which must be modeled accurately.
- STA and synthesis tools require precise timing data to meet design constraints.Delay Table

## ✅ Benefits of Delay Tables
- Accurate timing analysis: Provides realistic delay/slew values for all operating conditions.
- Better optimization: Tools can choose proper cell sizes, buffers, and gate replacements.
- Improved timing closure: Helps prevent setup/hold violations in later stages.
- Reliable silicon performance: Reduces the risk of timing failures after fabrication.

In order to avoid large skew between endpoints of a clock tree (signal arrives at different point in time):

- Buffers on the same level must have same capacitive load to ensure same timing delay or latency on the same level.
- Buffers on the same level must also be the same size (different buffer sizes -> different W/L ratio -> different resistance -> different RC constant -> different delay).

<img width="740" height="380" alt="image" src="https://github.com/user-attachments/assets/f715f4c4-979c-46a1-acab-bfed9f18a99d" /></br>

<img width="844" height="453" alt="image" src="https://github.com/user-attachments/assets/2aabf2f2-7f4b-4db8-80ed-9ea1f54cb2b8" />


## Lab 3: Fix Negative Slack
1. Let us change some variables to minimize the negative slack. We will now change the variables "on the flight". Use echo $::env(SYNTH_STRATEGY) to view the current value of the variables before changing it:
```
% echo $::env(SYNTH_STRATEGY)
AREA 0
% set ::env(SYNTH_STRATEGY) "DELAY 0"
% echo $::env(SYNTH_BUFFERING)
1
% echo $::env(SYNTH_SIZING)
0
% set ::env(SYNTH_SIZING) 1
% echo $::env(SYNTH_DRIVING_CELL)
sky130_fd_sc_hd__inv_2
```

With SYNTH_STRATEGY of Delay 0, the tool will focus more on optimizing/minimizing the delay, index can be 0 to 3 where 3 is the most optimized for timing (sacrificing more area). SYNTH_BUFFERING of 1 ensures cell buffer will be used on high fanout cells to reduce delay due to high capacitance load. SYNTH_SIZING of 1 will enable cell sizing where cell will be upsize or downsized as needed to meet timing. SYNTH_DRIVING_CELL is the cell used to drive the input ports and is vital for cells with a lot of fan-outs since it needs higher drive strength (larger driving cell needed).

2. Below is the log report for slack and area. The area becomes bigger (from 98492 to 103364) but no negative slack anymore (from -1.2ns to +0.35ns)!

<img width="678" height="372" alt="image" src="https://github.com/user-attachments/assets/dfc7cc9c-311c-48cd-abca-d6e91feeb4f2" /></br>

<img width="633" height="102" alt="image" src="https://github.com/user-attachments/assets/7e9af8d2-8c8b-46ef-955b-c3ff4ccf8ff6" />

3. Next, we do run_floorplan HOWEVER:

<img width="913" height="584" alt="image" src="https://github.com/user-attachments/assets/f057d1de-cdf6-49c8-a833-99e0a9533906" />

The solution for this error is found on this issue thread. basic_macro_placement command is failing since EXTRA_LEFS variable inside config.tcl is assumed as a macro which is not. The temporary solution is to comment call on basic_macro_placement inside the OpenLane/scripts/tcl_commands/floorplan.tcl (this is okay since we are not adding any macro to the design).

4. After that run_placement, another error will occur relating to remove_buffers, the solution is to comment the call to remove_buffers_from_nets in OpenLane/scripts/tcl_commands/placement.tcl. After successfully running placement, runs/[date]/results/placement/picorv32.def will be created.


## 🧩 Lab 4: Locating the Inverter cell in Layout
1. Search for instance of cell sky130_myinverter inside the DEF file after placement stage: cat picorv32.def | grep sky130_myinverter:
2. Open the def file via magic:
```
magic -T /home/angelo/Desktop/OpenLane/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.nom.lef def read picorv32.def &
```
Select a single sky130_myinverter cell instance from the list dumped by grep (e.g. _07237_). On tkcon, command % select cell _07237_ then ctrl+z to zoom into that cell. As shown below, our customized inverter cell sky130_myinverter is sucessfully placed on the core. Use expand on tkon to show the footprint of the cell and notice how the power and ground of sky130_myinverter overlaps the power and ground pins of its adjacent cells.

<img width="940" height="276" alt="image" src="https://github.com/user-attachments/assets/3e4f0759-575c-4963-9dad-f6968e437177" />


## Lab 5: Pre-Layout STA with OpenSTA
- Single-corner STA uses only the typical liberty file (LIB_TYPICAL) and is mainly used in pre-layout or post-synthesis timing checks.
- Multi-corner STA uses multiple liberty files to capture worst-case behavior across PVT variations.
- Setup analysis uses the slowest corner (LIB_SLOWEST: high temp, low voltage), while hold analysis uses the fastest corner (LIB_FASTEST: low temp, high voltage).

1. Run STA engine using OpenROAD (which in turn calls OpenSTA): run OpenROAD first then source /openlane/scripts/openroad/sta.tcl which contains the OpenROAD commands for single corner STA. This file also contains the path to the SDC file which specifies the actual timing constraints of the design.

<img width="875" height="122" alt="image" src="https://github.com/user-attachments/assets/9e16b2e0-710b-4fc7-b687-c5ebce81ba9a" />

The result of running STA in OpenROAD will be exactly the same as the log result of STA after running run_synthesis inside OpenLane. Observe the delay:

<img width="940" height="341" alt="image" src="https://github.com/user-attachments/assets/787e9be2-a1d0-4cb9-bc95-99e4b778a28e" />

2. To reduce negative slack, focus on large delays. Notice how net _02682_ has big fanout of 5. Use report_net -connections _02682_ to display connections. First thing we can do is to go back to OpenLane and reduce fanouts by set ::env(SYNTH_MAX_FANOUT) 4 then run_synthesis again. As shown below, wns is reduced from -1.35ns to -0.82ns.

<img width="940" height="216" alt="image" src="https://github.com/user-attachments/assets/d104fb80-990b-462a-91c2-d9a048c32fe1" />

3. To further reduce the negative slack, we can also try upsizing the cell with high fanout so bigger driver will be used. High fanout results in high load cap which then results in high delay. But since we cannot change the load cap, we can just change the cell size to better drive that large cap load for less delay. As shown below, cell _41882_ has a high cap load of 0.04nF and this causes a large delay due to buf_1 not having enough drive strength to drive that high cap load. We can try upsizing the buf_1 to buf_4 (listed on the used liberty files are all cells which you can choose) inside OpenSTA: replace_cell _41882_ sky130_fd_sc_hd__buf_4
   
<img width="940" height="299" alt="image" src="https://github.com/user-attachments/assets/2e09d034-7755-4468-80d8-2ca389c855e7" />

This can be done iteratively until desired slack is reached, this is called timing ECO (Engineering Change Order). To extract the modified verilog netlist: write_verilog designs/picorv32a/runs/RUN_2022.09.14_05.18.35/results/synthesis/picorv32.v. Beware that upsizing the cell will naturally increase core size.


## Summary of OpenSTA Commands
```
report_net -connections _02682_
replace_cell _41882_ sky130_fd_sc_hd__buf_4`
report_checks -fields {cap slew nets} -digits 4
report_checks -from _18671_ -to _18739_ -fields {cap slew nets} -digits 4
report_wns
report_tns
report_worst_slack -max
write_verilog designs/picorv32a/runs/RUN_2022.09.14_05.18.35/results/synthesis/picorv32.v
```

## Clock Tree Synthesis Stage
- Clock Skew: A clock tree equalizes wire lengths and delays so all clock endpoints receive the signal with minimal timing differences.
- Clock Slew: Wire RC causes waveform degradation at endpoints; dedicated clock buffers with equal rise/fall times restore clean transitions.
- Crosstalk: Clock shielding (using grounded/VDD wires beside the clock net) reduces coupling capacitance and prevents noise from nearby aggressor nets.

<img width="940" height="442" alt="image" src="https://github.com/user-attachments/assets/cb730cca-48ef-415d-94b1-13752f3a2ddd" />

## Timing Analysis with Real Clocks
- Setup analysis requires that data arrives before the next clock edge; violations occur when the path is too slow, influenced by combinational delay, clock buffer delay, clock period, setup time, and setup uncertainty.
- Hold analysis ensures data is held stable after the same clock edge; violations occur when the path is too fast, influenced by combinational delay, clock buffer delays, and hold time (clock period and uncertainty do not matter).
- The objective in STA is to achieve positive slack for both setup and hold checks, ensuring safe and reliable timing.

<img width="840" height="228" alt="image" src="https://github.com/user-attachments/assets/0a665466-2a92-4bcc-a8f8-fa3323bcca2a" />

STA report for hold analysis (min path):

<img width="840" height="681" alt="image" src="https://github.com/user-attachments/assets/4335ee42-656a-4c05-ae48-4f755fe3e1bc" />

STA report for setup analysis (max path):

<img width="840" height="363" alt="image" src="https://github.com/user-attachments/assets/71fff769-e114-4923-aa62-28c83ed78553" />


## Lab 6: Multi-corner STA for Post-CTS
- write_db/read_db create and load a timing database from LEF and the latest DEF before running STA.
- Multi-corner STA requires both min (hold) and max (setup) libraries, unlike single-corner STA which uses only the typical library.
- The same SDC is used for both single- and multi-corner STA.
- set_propagated_clock is applied post-CTS to include real clock latency instead of ideal clocks used pre-layout.
- OpenROAD provides an automated script, sta_multi_corner.tcl, which runs multi-corner STA with more complete settings.

<img width="940" height="153" alt="image" src="https://github.com/user-attachments/assets/84d5a60a-8f97-40cb-8349-f0863cc4f5fe" />

Conclusion:
- The design fails both setup and hold checks; setup can be fixed by lowering clock frequency, but hold is harder since it is independent of the clock period.
- Routing often helps hold violations because added wire RC increases data-path delay, reducing negative slack.
- The large hold failure occurs because TritonCTS builds the clock tree only for the typical corner, not for min/max corners.

Therefore, running multi-corner STA here is inappropriate; switching back to single-corner STA (using only the typical library) provides the correct analysis.

## Lab 7: Replacing the Clock Buffer 
- TritonCTS builds the clock tree by trying buffers from smallest to largest in the list $::env(CTS_CLK_BUFFER_LIST) until the target skew (200 ps) is met.
- Current STA shows that the largest buffer (clkbuf_8) is mostly used.
- By modifying the buffer list to include smaller buffers, we can rebuild the clock tree and observe how this change affects timing (skew, slack) and chip area.

1. Use tcl lreplace command to modify $::env(CTS_CLK_BUFFER_LIST) so that only sky130_fd_sc_hd__clkbuf_2 will remain:

<img width="840" height="139" alt="image" src="https://github.com/user-attachments/assets/2582751d-930b-4619-ad52-ebdcf79b20ec" />

2. If you do run_cts now, the result will be wrong (the removed clock buffers will still appear) since we already run CTS before. The reason being is that the $::env(CURRENT_DEF) used by CTS is the DEF file result of the previously run CTS too. What DEF file we want for CTS is the placement's DEF file. So just change the $::env(CURRENT_DEF) to point to placement DEF file then run_cts:

<img width="840" height="236" alt="image" src="https://github.com/user-attachments/assets/cb559a24-9132-4fa5-906a-6fd932b1cdd6" />

3. Observe the resulting post-CTS STA compared to before we modify the clock buffer. Only buf_2 clock buffer is used now compared to buf_8 used in previous run. The WNS is worse now since we used smaller clock buffers thus larger clock path delay, however the area is now smaller since we used smaller clock buffer.

<img width="840" height="360" alt="image" src="https://github.com/user-attachments/assets/e9fa9016-0858-45cc-9dac-9e4320068c88" />















































