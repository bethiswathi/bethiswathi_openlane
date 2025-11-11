# Final Steps for RTL2GDS using TritonRoute and OpenSTA
## Maze Routing
- Maze Routing (Lee’s algorithm) finds a shortest path by expanding a wavefront with incremental steps.
- Multiple shortest paths may exist, but the preferred one has fewer bends.
- Routes must avoid diagonal movement and obstacles like macros.
- Although accurate, the algorithm is slow and memory-intensive, so modern routers use optimized variants based on the same principles.

<img width="940" height="212" alt="image" src="https://github.com/user-attachments/assets/660b4da3-9dbd-424f-bc99-6b85741704f5" />

## DRC Cleaning
- DRC cleaning ensures routed layouts meet fabrication rules and can be accurately printed on silicon.
- Many violations arise from photolithography limits, such as minimum wire width, wire pitch, and wire spacing.
- Shorts between signals can be fixed by rerouting or switching layers using vias, which introduce additional via-related DRCs.
- Higher metal layers require larger widths, creating further constraints to satisfy during cleanup.

<img width="386" height="308" alt="image" src="https://github.com/user-attachments/assets/3faa68d4-53a2-4244-8ca1-478493b5b215" />


## Power Distribution Network 
This is just a review on PDN. The power and ground rails has a pitch of 2.72um thus the reason why the customized inverter cell has a height of 2.72 or else the power and ground rails will not be able to power up the cell. Looking at the LEF file runs/[date]/tmp/merged.nom.lef, you will notice that all cells are of height 2.72um and only width differs.

As shown below, power and ground flows from power/ground pads -> power/ground ring-> power/ground straps -> power/ground rails.

<img width="747" height="415" alt="image" src="https://github.com/user-attachments/assets/3bcfb6f9-b0d3-4eac-a81a-e1a5bc4c6cd2" />


## Routing Stage and TritonRoute
OpenLane routing stage consists of two stages:
- Global Routing - Form routing guides that can route all the nets. The tool used is FastRoute
- Detailed Routing - Uses the global routing's guide to actually connect the pins with least amount of wire and bends. The tool used is TritonRoute.


## Triton Route
- Detailed routing follows the global route guides and uses a MILP-based panel routing approach.
- Routing is parallel within a layer and sequential across layers, progressing from lower to higher metals.
- Each metal layer’s preferred routing direction (e.g., M1 horizontal, M2 vertical) is enforced.
- Alternating layer directions help prevent wire overlap and reduce interlayer capacitance, improving signal integrity.

<img width="840" height="468" alt="image" src="https://github.com/user-attachments/assets/f70e9b99-e3fd-4f6c-8e16-9e39cefcea8d" />

Best reference for this the <a href="https://vlsicad.ucsd.edu/Publications/Conferences/363/c363.pdf">Triton Route paper</a>.

## Lab 1: Routing Stage
- Running run_routing performs both global and detailed routing with repeated optimization passes to eliminate DRC violations.
- The initial (0th) iteration had 27,426 violations, and all were resolved by the 8th iteration.
- The full routing process took 1 hour 10 minutes on a 2-core Linux machine.
- Despite a small die size (~584 µm × 595 µm), the total routed wirelength reached ~0.5 meters, showing how dense on-chip interconnects can be.

<img width="840" height="431" alt="image" src="https://github.com/user-attachments/assets/93036834-d117-43eb-9187-5287ecdd9fc6" />

<img width="840" height="444" alt="image" src="https://github.com/user-attachments/assets/71c28dd8-71d1-4e2d-8b1f-f8f00bb1545a" />

A DEF file will be formed runs/[date]/results/routing/picorv32.def Open the DEF file output of routing stage in Magic:
```
magic -T /home/angelo/Desktop/OpenLane/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.nom.lef def read picorv32.def
```
Similar to what we did when we plugged in the custom inverter cell, look for sky130_myinverter at the DEF file then search that cell instance in magic:

<img width="840" height="396" alt="image" src="https://github.com/user-attachments/assets/b4b1e642-f72a-45d1-9486-42cd022c6f55" />


## Lab 2: SPEF Extraction and GDSII Streaming
- **run_parasitics_sta** performs SPEF extraction to capture real resistance and capacitance of routed wires.
- It then runs **multi-corner STA** (min, max, nominal) using these extracted parasitics.
- Real-world parasitic delays typically **reduce setup and hold slack**, and timing ECOs may be needed to fix violations.
- SPEF files are stored in the routing results folder; STA logs go to the signoff logs folder.
- Finally, **run_magic** generates the **GDSII file**, which is the fabrication-ready layout and can be opened in Magic.

























