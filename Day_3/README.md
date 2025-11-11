# Design Library cell Using Magic Layout and ngspice Characterization

Configurations on OpenLANE can be changed on the flight. For example, to change IO_mode to be not equidistant, use % set ::env(FP_IO_MODE) 2; on OpenLANE. The IO pins will not be equidistant on mode 2 (default of 1). Run floorplan again via % run_floorplan and view the def layout on magic. However, changing the configuration on the fly will not change the runs/config.tcl, the configuration will only be available on the current session. To echo current value of variable: echo $::env(FP_IO_MODE)

## Designing a Library Cell:
- SPICE deck = component connectivity (basically a netlist) of the CMOS inverter.
- SPICE deck values = value for W/L (0.375u/0.25u means width is 375nm and lengthis 250nm). PMOS should be wider in width(2x or 3x) than NMOS. The gate and supply voltages are normally a multiple of length (in the example, gate voltage can be 2.5V)
- Add nodes to surround each component and name it. This will be used in SPICE to identify a component.

### Notes:
- Width is the length of source and drain. Length is the distance between source and drain
- PMOS' hole carrier is slower than NMOS' electron carrier mobility, so to match the rise and fall time PMOS must be thicker (less resistance thus higher mobility) than NMOS
- A good refresher on MOSFETS and CMOS is this video and this site.

## SPICE Deck Netlist Description
<img width="940" height="384" alt="image" src="https://github.com/user-attachments/assets/d0bc0993-9cd0-41c9-8b27-a14120f2bfbc" />

### Notes:
- Syntax for the PMOS and NMOS descriptiom:
[component name] [drain] [gate] [source] [substrate] [transistor type] W=[width] L=[length]
- All components are described based on nodes and its values
- .op is the start of SPICE simulation operation where Vin will be sweep from 0 to 2.5 with 0.5 steps
- tsmc_025um_model.mod is the model file containing the technological parameters for the 0.25um NMOS and PMOS.

Build basic CMOS inverter netlist spice deck file using ngspice and perform dc and transient analysis. Understanding basic terminologies of CMOS inverter like static and dynamic characteristics.

## ✅ CMOS Inverter Characteristics

| **Category**                | **Parameter**                 | **Meaning / Definition**                                                                    |
| --------------------------- | ----------------------------- | ------------------------------------------------------------------------------------------- |
| **Static Characteristics**  | **VIL** (Input Low Voltage)   | Maximum input voltage recognized as logic **LOW** (before inverter output starts dropping). |
|                             | **VIH** (Input High Voltage)  | Minimum input voltage recognized as logic **HIGH** (before inverter output starts rising).  |
|                             | **VOL** (Output Low Voltage)  | Output voltage when input is HIGH and NMOS is ON (ideally ≈ 0 V).                           |
|                             | **VOH** (Output High Voltage) | Output voltage when input is LOW and PMOS is ON (ideally ≈ VDD).                            |
|                             | **Switching Threshold (VM)**  | Vin value where **Vout = Vin** on the VTC curve — inverter switching point.                 |
|                             | **Noise Margin Low (NML)**    | NML = VIL – VOL → amount of allowable LOW-level noise.                                      |
|                             | **Noise Margin High (NMH)**   | NMH = VOH – VIH → amount of allowable HIGH-level noise.                                     |
| **Dynamic Characteristics** | **tpHL** (High-to-Low delay)  | Delay when output transitions from **HIGH → LOW** after an input rising edge.               |
|                             | **tpLH** (Low-to-High delay)  | Delay when output transitions from **LOW → HIGH** after an input falling edge.              |
|                             | **Rise Time (tr)**            | Time for output to rise from **10% → 90%** of VDD.                                          |
|                             | **Fall Time (tf)**            | Time for output to fall from **90% → 10%** of VDD.                                          |


- The following are the steps to simulate in SPICE:

```
source [filename].cir
run
setplot 
dc1 
plot out vs in 
```

## SPICE Analysis For Switching Threshold and Propogation Delay

CMOS robustness depends on:

- Switching threshold = Vin is equal to Vout. This the point where both PMOS and NMOS is in saturation or kind of turned on, and leakage current is high. If PMOS is thicker than NMOS, the CMOS will have higher switching threshold (1.2V vs 1V) while threshold will be lower when NMOS becomes thicker.
- Propagation delay = rise or fall delay

DC transfer analysis is used for finding switching threshold. SPICE DC analysis below uses DC input of 2.5V. Simulation operation is DC sweep from 0V to 2.5V by 0.05V steps:

```
Vin in 0 2.5
*** Simulation Command ***
.op
.dc Vin 0 2.5 0.05
```
Below is the result of SPICE simulation for DC analysis, the line intersection is the switching threshold:

<img width="700" height="349" alt="image" src="https://github.com/user-attachments/assets/6373443a-6d63-472c-a41f-336aabb98729" />

Meanwhile, transient analysis is used for finding propagation delay. SPICE transient analysis uses pulse input:

1. starts at 0V
2. ends at 2.5V
3. starts at time 0
4. rise time of 10ps
5. fall time of 10ps
6. pulse-width of 1ns
7. period of 2ns

<img width="331" height="194" alt="image" src="https://github.com/user-attachments/assets/ccfd5d53-d675-4245-a81e-028fe2f30fff" />

The simulation operation has 10ps step and ends at 4ns:
```
Vin in 0 0 pulse 0 2.5 0 10p 10p 1n 2n 
*** Simulation Command ***
.op
.tran 10p 4n
```
Below is the result of SPICE simulation for transient analysis:

<img width="645" height="361" alt="image" src="https://github.com/user-attachments/assets/5d83e256-fe7c-46b5-8f0c-32fa0c519aa9" />

## Layout and Metal Layers
When polysilicon crosses N-diffusion/P-diffusion (diffusion is also called implantation), then an NMOS/PMOS is created. Explained 
<a href="https://electronics.stackexchange.com/questions/223973/why-diffusions-in-cmos-cad-tool-magic-is-continuous">here</a> is the reason why the diffusion layer of source and drain "seems" to be connected under the polysilicon (diffusion layer for source and drain supposedly be separated).

The first layer is local-interconnect layer or local-i then metal 1 to 5. <a href="https://skywater-pdk.readthedocs.io/en/main/rules/assumptions.html">Here is the process stack diagram</a> of sky130nm PDK. Metal 1 is for Power and Ground lines. Nsubstratecontact connects the N-well to locali. licon connects the locali to metal1.Locali is for local connections of cells.

The layer hierarchy for NMOS is: Psubstrate -> Psubstrate Diffusion (psd) -> Psubstrate Contact (psc) -> Local-interconnect (li) -> Mcon -> Metal1. For poly: Poly -> Polycontact -> Locali. P-substrate diffusion an N-substrate diffusion is also referred to as P-tap and N-tap.

The output of the layout is the LEF file. <a href="https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html">LEF (Library Exchange Format)</a> is used by the router tool in PnR design to get the location of standard cells pins to route them properly. So it is basically the abstract form of layout of a standard cell. picorv32a/runs/[DATE]/tmp contains the merged lef files (cell LEF and tech LEF). Notice how metal layer directon (horizontal or vertical) is alternating. Also, metal layer width and thickness is increasing.


## Magic Commands
<a href="https://www.youtube.com/watch?v=RPppaGdjbj0">Here is a video guide</a> on layout using Magic. And <a href="http://opencircuitdesign.com/magic/">here is the Magic website</a> with tutorials.

- Right click = upper-right corner of box
- "z" = zoom in, "Z" = zoom out, "ctrl + z" = zoom into the box
- Middle click on empty area will turn the box into empty (similar to erasing it)
- "s" three times will select all geometries electrically connected to each other
- :box = display parameters of selected box
- :grid 0.5um 0.5um = turn on/off and set grid
- :snap user = snap based on current grid
- :help snap = display help for command
- :drc style drc(full) = use all DRC when doing DRC checking
- :paint poly = paint "poly" to current box
- :drc why = show drc violation inside selected area (white dots are DRC violations )
- :erase poly = delete poly inside the box
- :select area = select all geometries inside the box
- :copy n 30 = copy selected geometries to North by 30 grid steps
- :move n 1 = move selected geometries to North by 1 step ("." to move more, "u" to undo)
- : select cell _08555_ = select a particular cell instance (e.g. cell _08555_ which can be searched in the DEF file)
- :cellname allcells = list all cells in the layout
- :cellname exists sky130_fd_sc_hd__xor3_4 = check if a cell exists
- :drc why = show DRC violation and also the DRC name which can be referenced from Sky130 PDK Periphery Rules.

## Lab1: Slew rate and Propogation Delay Characteristics
The task is to characterize a sample inverter cell by its slew rate and propagation delay.

1. Clone vsdstdcelldesign. Copy the techfile sky130A.tech from pdks/sky130A/libs.tech/magic/ to directory of the cloned repo. Below are the contents of vsdstdcelldesign/libs/:

<img width="940" height="338" alt="image" src="https://github.com/user-attachments/assets/8490efc0-eb96-411e-ade4-1d49a6c4ff21" /></br>
<img width="940" height="66" alt="image" src="https://github.com/user-attachments/assets/1dbc6e96-11a1-4408-8ddd-9ef8367f18b9" /></br>

2.View the mag file using magic
```
magic -T ./libs/sky130A.tech sky130_inv.mag & 
```

<img width="940" height="573" alt="image" src="https://github.com/user-attachments/assets/ae011a66-601f-4b74-b638-a8710e273bc7" /></br>

3. Make an extract file .ext by typing the following command in the tkon terminal.
```
   extract all 
```
4. Extract the .spice file from this ext file by typing the following command in the tcon terminal.
```
   ext2spice cthresh 0 rthresh 0 then ext2spice 
```
<img width="940" height="471" alt="image" src="https://github.com/user-attachments/assets/86846d26-f844-4479-84b5-922fea8445df" /></br>
<img width="940" height="315" alt="image" src="https://github.com/user-attachments/assets/c0c5196c-e5e6-463c-9f15-b3448d3ee7b7" /></br>

Below is the initial spice file 
<img width="940" height="379" alt="image" src="https://github.com/user-attachments/assets/bf9b3e81-21d1-4f6e-8115-542d9b02b5fb" />

Few modifications needs to be done in spice deck file
- Scale needs to be aligned with the layout grid size and check the model name from pshort.lib and nshort.lib
- Specify power supply
- Apply stimulus
- Perform transient analysis

We then modify the spice file to be able to plot a transient response:
```
* SPICE3 file created from sky130_inv.ext - technology: sky130A

.option scale=0.01u
.include ./libs/pshort.lib
.include ./libs/nshort.lib

* .subckt sky130_inv A Y VPWR VGND
M0 Y A VGND VGND nshort_model.0 ad=1435 pd=152 as=1365 ps=148 w=35 l=23
M1 Y A VPWR VPWR pshort_model.0 ad=1443 pd=152 as=1517 ps=156 w=37 l=23
C0 A VPWR 0.07fF
C1 Y VPWR 0.11fF
C2 A Y 0.05fF
C3 Y VGND 0.2fF
C4 VPWR VGND 0.59fF
* .ends

* Power supply 
VDD VPWR 0 3.3V 
VSS VGND 0 0V 

* Input Signal
Va A VGND PULSE(0V 3.3V 0 0.1ns 0.1ns 2ns 4ns)

* Simulation Control
.tran 1n 20n
.control
run
.endc
.end
```
Open the spice file by typing ngspice sky130A_inv.spice. 

<img width="840" height="249" alt="image" src="https://github.com/user-attachments/assets/59e21d66-ae7f-440f-8f40-1db46d910c24" />

To plot transient analysis output, where y - output node and a - input node
```
plot y vs time a
```

<img width="840" height="492" alt="image" src="https://github.com/user-attachments/assets/f5e79c98-88cd-4beb-bfc5-b936be05556f" />

<img width="840" height="444" alt="image" src="https://github.com/user-attachments/assets/84a0ed9a-f3ad-4776-93ab-8ba188929a2c" />






















