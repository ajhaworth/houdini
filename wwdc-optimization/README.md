# Immersive Optimization Toolkit

**A modular toolkit for real-time asset optimization in immersive environments — built entirely in Houdini.**

Based on [Optimize your 3D assets for spatial computing](https://developer.apple.com/videos/play/wwdc2025/305/) (WWDC25)

This toolkit provides a suite of **procedural Houdini Digital Assets (HDAs)** designed to automate and streamline the preparation of 3D assets for **real-time immersive experiences**. The toolkit enables technical artists and developers to optimize geometry and UVs for real-time performance while preserving the fidelity of offline-quality renders.

The tools are purpose-built for the **immersive boundary** of Vision Pro — a **1.5-meter** radius traversable space that a person can move within. Assets within this space are optimized for maximum visual quality, while those outside are aggressively simplified to minimize rendering overhead. Techniques include triangle reduction, billboard proxies, UV projection, geometry partition, and visibility culling.

Each tool is highly customizable and designed to be dropped into any Houdini network. You can use the provided sample scene to explore the pipeline or integrate individual HDAs into your own workflows.

---

### 📦 Dependencies

Built and tested with the following versions.  
Earlier or later versions may also be compatible.

- **Houdini** 20.5.487  
- **Houdini Labs** 20.0.669

⚠️ **License Notice**  
`.hip` and `.hda` files were created with a **Houdini Production license**.  
Opening in **Indie**, **Apprentice**, or **Education** versions may limit saving and asset editing.  
[See license comparison →](https://www.sidefx.com/products/compare/)

---

### 🚀 First-Time Setup: Running the Sample Scene

To get started, open the `optimize.hip` project file.  
Inside the `/obj/` context, you’ll find three main networks that structure the pipeline:

#### 1️⃣ **`source`** – High-Resolution Input

This network contains the raw, high-poly geometry.  
It’s built entirely from procedural terrain and rock setups and differs from what is shown in the demonstration video.  

#### 2️⃣ **`optimize`** – Optimization Workflow

This is the heart of the toolkit.  It processes the source geometry using a chain of custom HDAs to:

- Reduce polygon count 
- Cull hidden or back-facing geometry
- Partition and project UVs
- Prepare optimized assets for export

#### 3️⃣ **`usd`** – Scene Export

This network converts the optimized SOP geometry into a structured USD hierarchy.  
It uses primitive attributes created during the optimization stage (`name`, `class`, etc.) to organize content into transforms and `GeomSubsets` in Solaris.

- `name` — String attribute used to define the name of each USD prim  
- `groups` — Attribute that defines the `GeomSubset` membership for organizing geometry 

#### ✅ Quickstart: Cook the Optimization Graph

Some optimization steps can be computationally expensive, so a custom HDA is used to file-cache each major stage of the graph. This creates checkpoints to avoid re-cooking during iteration.

1. Open the file: `optimize.hip`  
2. Navigate to: `/obj/optimized/cook_book`  
3. Click **Cook All** to cache all major operations to disk  
4. Wait approximately **5 to 10+** minutes, depending on your system’s performance
5. You can now click through the filecache nodes to see each optimization step

#### 📂 Where to Find the Tools

All custom HDAs are installed and accessible from the tab menu under the **Optimize** category.  
You can drag and drop them into your networks as needed to build or modify your own optimization pipeline.

### 🧠 Optimization Tips & Best Practices

- **File cache expensive steps.** Save to disk any parts of the graph that take a long time to compute to avoid repeated processing during iteration.  
- **Order of operations matters.** Perform poly reduction before culling to avoid generating excess open edges that need to be preserved.  
- **Begin with broad optimizations and refine where necessary.** Begin optimization at a high level and let the tools handle the bulk of the work. Only make manual adjustments to specific sections when necessary.  
- **Check sample point placement.** Ensure sample points are positioned above the geometry within the immersive boundary to avoid incorrect visibility results. 
- **Minimize UV partitions.** Fewer partitions result in fewer UV islands, which reduces space lost to texture padding.  
- **Adjust for content type.** The sample scene represents an outdoor environment and would use a separate texture for the sky. For interior scenes, additional surfaces must be included in the UV atlas, which increases texture size requirements.  
- **Consider billboards for complex geometry.** Consider reducing high-complexity assets to billboards if they're far from view or have minimal visual impact.  

## ⚠️ Content-Specific Considerations

Tool behavior can vary depending on the complexity, topology, and structure of the input geometry. While each tool is built to be flexible, results may differ depending on the specific content. You may need to update parameters, adjust the content — or even modify the tool itself — to achieve the intended outcome.

---

### 🛠️ HDAs

#### 🔧 Utility
Tools that support boundary definition, sample generation, and scene caching.
#### `boundary_cam.hda`
Visualizes the immersive boundary and provides camera presets for evaluation.

- `Height` — Sets the vertical extent of the immersive boundary  
- `Radius` — Sets the horizontal extent (radius) of the immersive boundary  
- `Boom` — Vertically offsets the camera position  
- `Tracking` — Horizontally offsets the camera position  
- `Tilt` — Adjusts vertical camera rotation  
- `Custom Camera` — Accepts an Alembic camera with a custom animation path 

#### `boundary_samples.hda`
Generates sample points within the immersive boundary for various HDA calculations.

- `Translate` — Defines the origin of the sample grid  
- `Remove Center` — Removes sample points along the Y-axis at the center  
- `Radial Divisions` — Number of radial divisions (columns)  
- `Height Separation` — Vertical distance between sample points  
- `Floor Offset` — Sets the minimum height to begin sampling  
- `Rows` — Number of inner rows of sample points  

#### `cook_book.hda`
Automates file caching across the optimization graph by executing nodes in sequence.

- `Cook all` — Runs all nodes defined in the HDA  
- `Nodes +` — Specifies nodes to cook, based on the `exec` parameter  
- `Cook from` — Begins cooking from the selected node downward  

#### `vis_sphere.hda`
Ray casts geometry onto a spherical surface and measures polygon area.  
Useful for simulating how objects and triangle density appear in screen space.

- `Center` — Origin of the sphere to ray cast to  
- `Uniform Scale` — Sets the size of the sphere  
- `Blend` — Interpolates between ray-cast and original geometry positions  
- `Use Connectivity` — Measures area per connected island instead of individual polygons  

---

### 🪨 Geometry Optimization
Assets for reducing complexity, culling hidden geometry, and generating billboard LODs.

#### `vista_billboard.hda`
Converts geometry beyond a certain distance into a wraparound billboard mesh.

- `Origin` — Target position for billboard projection  
- `Vista Distance` — Distance at which geometry transitions to billboard  
- `Collision Mesh` — Billboard shape (cylindrical or spherical)  
- `Remesh Size` — Controls inner triangle density of the billboard  
- `Unrefine Tolerance` — Removes unimportant edge points using the Refine SOP’s unrefine setting 
- `Bottom Offset` - Offsets the bottom edge of the billboard 
- `Blend` - Blends the bottom edge to the min bounding box height

#### `disk_project.hda` 
Creates a radial mesh with decreasing triangle density from the origin, snapped to the closest geometry surface.

- `Disk Size` — Sets the diameter of the disk. Triangles will be largest at this point  
- `Start Distance`  — Triangles stay at their _minimum_ size until this distance
- `Min Size` — Defines the smallest triangle edge length
- `Max Size` — Defines the largest triangle edge length
- `Remap` — Adjusts how quickly triangle size scales between the min and max distances

#### `adaptive_reduce.hda`
Uses PolyReduce and sample points to retain or reduce detail based on view importance.  

- `Prims to Keep` — The number of polygons to keep  
- `Sample Points` — Points to iterate over to calculate silhouette and distance  
- `Visualize Area` — Sets the visualization width of a polygon’s area in world space  
- `Boundaries` — Increases preservation along open edges  
- `Front-Facing` — Reduces polygons facing away from the viewer  
- `Silhouette` — Preserves polygons that contribute to the silhouette  
- `Distance` — Reduces polygons based on distance  
- `Min Distance` — Sets the start point of the distance-based falloff  
- `Distance Ramp` — Curve to control distance-based reduction strength 

#### `remove_backfaces.hda`
Deletes or groups back-facing geometry based on sample-point dot product.  Displays removed primitives below the HDA.


- `Sample Points` — Used to determine back-facing polygons  
- `Just Group` — Option to group backfaces instead of deleting them  

#### `occlusion_culling.hda`
Removes occluded geometry using radial ray casts from sample points.  Displays removed primitives below the HDA.


- `Sample Points` — Locations used to cast visibility rays  
- `Ray Samples` — Number of rays cast per point (higher = more accurate)  
- `Min Hits` — Minimum hits required for a polygon to be considered visible  
- `Object Check` — Additional objects considered for occlusion testing  
- `Just Group` — Groups hidden geometry instead of deleting  
- `Single Pass` — Isolates individual sample points for debugging 

#### `portal_culling.hda`
Culls geometry by ray casting from each vertex toward sample points. If a ray intersects portal geometry, the vertex is marked as visible; all others are removed.

- `Sample Points` — Positions used to cast visibility rays at  
- `Portal` — Geometry used to determine valid lines of sight
- `Padding Steps` — Expands the visible region by growing the group a few steps beyond the initial result

#### `frustum_partition.hda`
Applies class attributes for hierarchical frustum-based partitioning.

- `Group` — Input primitive group to affect  
- `Translate` — Sets the origin for partitioning  
- `Start Size` — Size of the smallest tiles  
- `Number of Rows` — Number of expansion rings, each doubling in size  
- `Bisect Origin` — Adds X and Z bisecting partitions  
- `Prefix` — Label prefix for each output class  

---

### 🗺️ UV Layout
Partitioning and projecting UVs to support screen-space mapping.
#### `boundary_partition.hda`
Partitions and UVs geometry based on immersive boundary limits.

- `Translate` — Sets the origin point of the immersive boundary  
- `Immersive Boundary` — Defines the main traversable area
	  - `Boolean` — Enables strict cutting at the defined radius  
	  - `Radius` — Horizontal boundary distance  
	  - `Height` — Vertical range for checking intersecting geometry  
	  - `Exclude` — Groups to exclude from partitioning  
	  - `Include` — Groups to include in partitioning  
	  - `Expand Group` — Expands the specified group for more flexible inclusion  
- `Buffer Region` — Adds a margin beyond the boundary to reduce UV distortion near the cutoff  

#### `multi_uv_partition.hda`
Assigns a class attribute based on dot product comparison between surface normals and sample vectors.  Used to group primitives for best-fit projection.

- `Sample Points` — Positions used for directional clustering  
- `Main Projection` — Preferred projection direction  
- `Main Tolerance` — Biases clustering toward the main projection direction  
- `Vector Blurring` — Blends nearby vectors for smoother clustering  
- `Name Prefix` — Prefix for class attribute names  

#### `multi_uv_projection.hda`
Performs UV projection using the optimal position per partitioned class.

- `Type` — Spherical or camera-based projection  
- `Make Square` — Converts spherical UVs from 2:1 to 1:1 aspect ratio  
- `Sample Points` — Candidate positions for projection  
- `Piece Attribute` — Attribute used to isolate each primitive group  
- `Flatten` — Normalizes UV scale within each island  
- `Blend Projection` — Interpolates between flattened and raw projections  
- `Single Pass` — Isolates specific positions or partitions for debugging 