
![Copertina YT](https://github.com/user-attachments/assets/63ec7a39-e102-4e61-9f4e-6bd9a86ad1aa)

## Video tutorials available on YouTube
Italian with english subtitles
[![Grid_PRO Tutorials]([https://img.youtube.com/vi/VWgaY-i0iHQ&list=PLpUJHu84xzlo3qFtnYqij7nG_C9A3Nrkc/0.jpg](https://www.youtube.com/watch?v=VWgaY-i0iHQ&list=PLpUJHu84xzlo3qFtnYqij7nG_C9A3Nrkc))]

# Introduction to Grid_PRO

**Grid_PRO** is a comprehensive package of Grasshopper definitions developed for Rhinoceros, designed to assist users through the entire workflow of designing, analyzing, and generating fabrication-ready drawings for **active-bending timber gridshells**.

The process begins with a  NURBS surface and culminates in the automatic generation of executive drawings. These drawings detail the precise length of each rod, the exact position of drill holes, and the relative placement of all components, making them ready for manufacturing.

The package is composed of three distinct modules that are used sequentially:

1. **Grid Maker:** Transforms the initial surface into a regular, planar, square mesh.
2. **Grid Shell Form Finding Tool (GFFT):** Simulates the physical deformation (active-bending) of the planar mesh, allowing for the analysis of the final 3D geometry and its curvatures.
3. **Officina:** Subdivides the final grid into smaller, manageable sub-modules and generates all the necessary executive drawings for fabrication.

### 1️⃣ Part 1: Grid Maker

The first module, **Grid Maker**, is responsible for generating a regular, square grid on a complex target surface using the principles of **Chebyshev nets**.

When opened, the definition displays a control panel in the top-left of the Grasshopper canvas. The workflow is as follows:

- **Input Surface:** The user first sets a target surface (e.g., created with `Network Surface` or `Patch`) into the "Target Surface" container. The surface is then automatically moved to the 0,0,0 origin.
- **Grid Generation:** Grid Maker immediately draws a grid based on default parameters. This process is driven by two generative **geodesic axes**. Nodes are placed along these axes at a distance defined by the "Grid Span" parameter.
- **Iterative Process:** The grid is built iteratively. Starting from the axis nodes, the definition generates spheres with a radius equal to the grid span. The intersection of these spheres creates a circle, which is then intersected with the target surface. The algorithm chooses the intersection point furthest from the origin to create the first "square" of the mesh.
- **Grid Properties:** This method ensures two key properties:
    1. All segments in the flat grid have the **same length** (the "Grid Span").
    2. The angles at the nodes, which are 90° in the flat planar state, will **deform** during the form-finding. This deviation is most pronounced in areas of high surface curvature and is a critical factor in the optimization process.

### Optimization with Galapagos

The core of Grid Maker is the optimization process, which uses the **Galapagos** plugin to find the best possible grid configuration. Galapagos automatically manipulates four key sliders:

- **X/Y Coordinate:** The origin point of the grid on the surface.
- **Axis Angle:** The relative rotation between the two generative axes.
- **Axis Rotation:** The rotation of the entire axis system relative to the XY plane.

The goal is to find the grid with the highest **fitness value**, which is a combined score based on two main objectives:

1. **Rod Curvature (to be optimized):** The analysis seeks to "compress" the domain of curvatures. It tries to **maximize the difference between the minimum curvature and zero** (to avoid flat, unstable points) and simultaneously **maximize the difference between the maximum curvature and the breaking/plasticizing limit** of the timber.
2. **Angle Deviation (to be minimized):** The process analyzes the deviation of node angles from the initial 90°. The goal is to find a grid where this deviation is as low as possible.

The user sets Galapagos to **"Maximize"** the evolutionary solver and starts the process. After the algorithm converges (which can take 15-20 minutes or more), the user can select the best-performing solution from the list and "reinstate" it.

Finally, the user **bakes** the necessary geometries into Rhinoceros layers. These outputs are crucial for the next step and include the `Plan Grid/Trim Geometry`, the `Curve Constrain` (ground attachment profiles), and the `Ground Point Selection` (curves used to select the ground nodes).

### 2️⃣ Part 2: Gridshell Form Finding Tool (GFFT)

The second module, **GFFT**, simulates the physical form-finding process of the gridshell. It takes the flat, optimized grid from Grid Maker and digitally "builds" it, deforming it into its final 3D shape.

This module requires a specific plugin version: **Kangaroo 0.0.099**.

- **Input Method:** The user can choose between the `Grid Maker` method (to use the previously generated grid) or a `Custom` method for an arbitrary perimeter.
- **Workflow (Grid Maker Method):**
    1. **Reference Geometries:** The user references the baked `Grid Maker Grid Trim Geometry` (which GFFT automatically populates with diagonals) and the constraint curves.
    2. **Reference Constraints:** This is a critical step. The user must reference the `Curve Constrain` (the final 3D target position for the ground attachments) and the `Curve for Ground Point Selection` (the 2D curves that select which nodes on the flat grid are ground points). **The selection order must be identical** for both sets of curves to ensure the correct nodes are pulled to the correct targets.
- **Simulation (Kangaroo):**
    1. The user first activates a small **"Pullup"** force to give the simulation an initial direction.
    2. By toggling **"Simulation On,"** the Kangaroo solver starts. The selected ground points are pulled toward their respective 3D constraint curves, and the rest of the grid deforms realistically under the forces of active-bending.
    3. The user should let the simulation run for a high number of iterations (e.g., at least 100,000) to ensure it reaches a stable static equilibrium.
- **3D Model & Analysis:**
    
    > Note: The 3D model and curvature analysis are computationally expensive. They should be kept off during the simulation and only activated once the simulation is paused and stable.
    > 
    1. **3D Model:** Toggling "3D Curvature" on generates a full 3D model of the gridshell, including X/Y layers and diagonals. The user can specify `Section Properties` (width, height, shear block length) and `Material Selection` (e.g., C24, C30) which defines the structural limits.
    2. **Curvature Analysis:** This is the most important verification step. The tool checks if any rod exceeds the maximum allowable bending, which is based on the selected material and section.
        - **Bending Types:** The analysis can be run for `Out-of-plane bending` (minor axis) and `In-plane bending` (major axis).
        - **Visualization:** A pass/fail view shows rods **in blue (pass)** and **in red (fail/exceeds limit)**. A gradient view shows curvature from green (low) to red (high).
    3. **Troubleshooting:** If any rods are red, the design is not feasible. The user must either: use a thinner (more flexible) section, go back to Grid Maker and find a new grid, or modify the boundary constraints.

Once satisfied, the user can **Bake** the final 3D model (X-rods, Y-rods, and diagonals) into Rhinoceros for rendering or further use.

### 3️⃣ Part 3: Officina

The final module, **Officina** (Workshop), is a fabrication tool. It takes the *flat grid geometry* from Grid Maker and subdivides it into smaller, buildable sub-modules, generating all the necessary fabrication drawings.

- **Input:** The user references the flat `Grid Maker Geometry`.
- **Subdivision:** Officina overlays a cutting grid to dice the main grid into modules. The user has several key parameters in `Module Properties`:
    - **Number of Rods:** Sets how many rods (e.g., 3x3, 4x4) are in each module, which controls the module's size.
    - **Grid Shift (X/Y):** Allows the user to move the cutting grid. This is a powerful optimization tool to ensure modules are regular and avoid tiny, impractical rod segments at the edges.
    - **Overlap:** A crucial parameter that adds a specified length to rods on the edge of a module. This overlap is necessary to connect and fasten one module to the next during assembly.
- **Real-Time Drawing Generation:** As parameters are adjusted, Officina generates and updates three sets of drawings in the Rhino viewport:
    1. **The cutting grid** on the main mesh.
    2. **Assembly Layout:** An exploded view of all modules, color-coded by layer (e.g., cyan and red) and labeled with a clear nomenclature (e.g., A1, A2, B1...).
    3. **Fabrication Layout:** The most detailed view. For *each* module, it shows an assembly diagram and then lays out *each* individual rod (for both X and Y layers) complete with dimensions for total length and the precise position of drill holes.
- **Export (Bake):**
The user can bake the layouts into Rhinoceros.
    - **Assembly Layout:** Bakes as clean, closed curves, ready for export as DXF or DWG.
    - **Fabrication Layout:** Bakes all dimensioned drawings.
    
    > Note on Dimensions: Due to a quirk in Grasshopper, baked dimensions (annotations) may appear illegible. The user must go into the Annotation Styles panel in Rhinoceros (Drafting tab) and adjust the Model Space Scale to make the dimension text and arrows the correct size.
    >
