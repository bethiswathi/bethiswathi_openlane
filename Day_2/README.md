# Chip Floor Plan Considerations

Floorplanning is the initial physical design step where the overall structure and layout of a chip are defined. It determines how effectively the design can be placed and routed later. The key tasks involved are:

Ex:- Consider a netlist with 2 Flip-Flops with an And gate and a OR gate. </br>

<img width="600" height="400" alt="image" src="https://github.com/user-attachments/assets/2187bef7-151a-4add-89da-dfa7464558ab" /></br>
Assume the dimensions of standard cells and flip-flops as 1 unit ie. the area is 1 sq.unit. If we club all of them we can findout the rough calculation of  width and height</br>

<img width="600" height="250" alt="image" src="https://github.com/user-attachments/assets/d9dcce6e-f743-4bd2-a1d2-ed8b1315002c" /></br>
If we realize all the cells on the core area, we can observe it is occupying complete core area.</br>
<img width="600" height="326" alt="image" src="https://github.com/user-attachments/assets/1d93aeca-1893-4bb0-867b-138dc925ab1b" /></br>

### 1. Die and Core Definition
- The die area and core area are set based on design size and routing needs.
- This establishes the physical boundary within which all logic will be placed.

### 2. Aspect Ratio
- Defines the shape of the core using the ratio:
- Aspect Ratio = Core Height / Core Width
- Aspect ratio 1 → square, otherwise rectangular.
- Impacts routing congestion and timing.
- The aspect ratio for the above example is 1
  
<img width="548" height="52" alt="image" src="https://github.com/user-attachments/assets/af1f19a4-d846-400e-8120-63b5948004b7" />

### 3. Utilization Factor
- Indicates how much of the core area is used by standard cells.
- Defined as:
Utilization = (Area of netlist) / (Core area)
- Typically kept between 0.5 to 0.7 to leave whitespace for routing.
- The Utilization factor for above example is 100%
  
<img width="500" height="144" alt="image" src="https://github.com/user-attachments/assets/2c1a2a86-9227-44cf-b1f5-e2210196284e" />

### 4. Preplaced Cells (Macros)
- Large blocks (e.g., memories, comparators, clock gating cells) with fixed locations.
- These IP's/blocks have user-defined locations and hence they are placed before automated placement and routing.
- They are placed before standard cells and influence overall routing and congestion.
- May be used multiple times in the design.

### 5. Decoupling Capacitors (Decaps)
- Placed near macros or high-switching regions.
- Help reduce voltage drop, maintain noise margin, and provide local charge during fast switching.
- Act as local power reservoirs.

### 6. Power Planning (PDN)
- Creation of power distribution networks (VDD and GND rails, straps, rings).
- Ensures uniform power delivery and mitigates IR drop and ground bounce.
- In OpenLANE, PDN is usually done before routing.

### 7. Pin Placement
- Input/output pins are assigned around the core periphery.
- Cell placement blockages are added to avoid routing congestion near boundaries.

### 8. Final Floorplanning
- Combines all the above: macro placement, I/O pins, power grid, aspect ratio, utilization.
- Ensures the design is ready for placement and routing.





### Steps To Run Floorplan Using OpenLane
### Review Floorplan Files and Steps to view floorplan
### Review Floorplan Layout in Magic

## Library Binding and Placement
### Netlist Binding and initial place design
### Optimize Placement Using Estimated wire length and capacitance
### Final Placement Optimization
### Need for Libraries and Characterizarization
### Congestion Aware Placement Using Replace

## Cell Design and Charaterization Flows
### Inputs for Cell Design Flow
### Circuit Design Step
### Layout Design Step
### Typical Characterization Flow

## General Timing Parameters
### Timing Threshold Definitions
### Propogation Delay and Transition Time







