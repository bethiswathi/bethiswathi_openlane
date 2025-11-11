# Pre-Layout Timing Analysis and Importance of Clock Tree
To run previous flow, add tag to prep design:
```
prep -design picorv32a -tag [date]
```
## Lab 1: Extracting the LEF file
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
- Input and output pins must lie at track intersections
   → ensures routers can reach the pins easily.

2. Cell Width and Height Must Align to Routing Grid
- Width must be an odd multiple of the horizontal track pitch
- Height must be an odd multiple of the vertical track pitch

This ensures:
- Cells align properly in rows
- Power/ground rails line up across all cells
- Routing tracks line up from cell to cell

To check these guidelines, we need to change the grid of Magic to match the actual metal tracks. The pdks/sky130A/libs.tech/openlane/sky130_fd_sc_hd/tracks.info contains those metal informations.

1. Use grid command inside the tkon terminal to match the tracks informations:

<img width="803" height="459" alt="image" src="https://github.com/user-attachments/assets/c30a08df-79f1-4945-a089-af92af63afc9" />

The grids show where the routing for the local-interconnet layer can only happen, the distance of the grid lines are the required pitch of the wire. Below, we can see that the guidelines are satisfied:

<img width="593" height="358" alt="image" src="https://github.com/user-attachments/assets/8843e5c8-74ca-4a99-aa56-6fe844794663" />

2. Next, we will extract the LEF file. The LEF file contains the cell size, port definitions, and properties which aid the placer and router tool. With that, the ports definition, port class, and port use must be set first. The instructions to set these definitions via Magic are on the <a href="https://github.com/nickson-jose/vsdstdcelldesign#create-port-definition">vsdstdcelldesign repo</a>.

3. Next, save the mag file with a new filename save sky130_myinverter.mag. Then type lef write on the tcon terminal. It will generate a LEF file with same name as the magfile sky130_myinverter.lef. Inside that LEF file is: 

<img width="940" height="441" alt="image" src="https://github.com/user-attachments/assets/ef9418a3-44b6-4f8c-91f7-31d4db30a4e0" />


## Lab 2: Plug-in the customized inverter cell into OpenLane
- The SKY130 PDK provides Liberty <a href="https://teamvlsi.com/2020/05/lib-and-lef-file-in-asic-design.html">(.lib) timing files</a> in sky130_fd_sc_hd/lib/, containing timing and power data for each standard cell.
- These files exist for different <a href="https://chipedge.com/resources/what-are-pvt-corners-in-vlsi/">PVT corners</a> (process–voltage–temperature), such as slow, typical, fast and voltages like 1.65 V, 1.80 V, 1.95 V.
- A file like sky130_fd_sc_hd__ss_025C_1v80.lib represents the slow–slow corner, 25 °C, at 1.8 V, giving worst-case delays.
- The timing/power values inside a liberty file come from detailed simulations of each cell across multiple conditions.
- During synthesis, ABC uses these liberty files to map generic logical gates to actual standard-cell implementations.
- The library contains multiple sizes/flavors of the same gate (e.g., inverter size-1 vs size-4), where larger cells are built from multiple smaller ones to provide higher drive strength.






































