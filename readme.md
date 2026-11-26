# Procedural Creature Generator and Rigging

**Unity · C# · Procedural Content Generation · Runtime Mesh Generation · Procedural Rigging · Inverse Kinematics**

This project generates 3D creatures from a random seed, builds their meshes at runtime, and creates a skeleton to pose them. Creatures can have one to three heads, animal or insect legs, or tentacles, along with randomized eyes, antennae, tails, and procedural colors. Animal and insect legs also receive IK controls, so moving a foot target bends the leg toward it.

## Source code availability

I’m keeping the source code private in accordance with my Georgia Tech course policy. I’ll upload it after the semester ends.

## Screenshots

https://github.com/user-attachments/assets/c1ff56c9-f435-42c7-9a3a-74151f95dbd5

Save the images in `screenshots/` and uncomment the corresponding Markdown lines to display them on GitHub. A short recording of a foot target being moved would also demonstrate the IK.

## Getting started

Once the source code is available, follow these steps to run the generator in Unity.

1. Open the project in **Unity 6.3 LTS**, using **6000.3.23f1** to match the project version. The project uses the Universal Render Pipeline and **Animation Rigging 1.4.1**.
2. Open `Assets/Scenes/SampleScene.unity`. The scene contains five configured creature objects with different seeds, materials, and prefab references.
3. Enter **Play Mode**. Each creature generates its surface, texture, skeleton, and supported leg IK controls in `Start()`.
4. Open the **Scene** view and enable **Gizmos** to see the skeletons and IK markers. Expand a creature’s `Skeleton` hierarchy to select individual bones.
5. Exit Play Mode, change **Seed** in the Creature Generator component, and enter Play Mode again to generate another variation. Keep the seed and settings unchanged to reproduce a variation.

To set up a new creature object, add a **Mesh Renderer** with a body material and attach **Creature Generator**. Assign at least three entries in **Antenna Prefabs** and a **Feather Prefab** with a renderer and material. The current initialization still accesses the feather material even when wing generation is disabled. **Procedural Creature Rig** is attached automatically, and jointed legs receive **Creature Leg IK**, an Animator, and a Rig Builder at runtime.

For a first preview, use the sample scene’s generation settings: **Grid Dimensions = (200, 200, 200)**, **Cube Size = 0.5**, and **Iso Value = 0.2**. The smaller defaults in the script do not cover the full creature. Larger grids increase generation time substantially.

## Inspector controls

| Component / control | What it changes |
| --- | --- |
| Creature Generator — Seed | Controls the random choices used to generate a creature. |
| Creature Generator — Grid Dimensions | Sets the number of marching-cubes cells along each axis and the extent of the sampled volume. |
| Creature Generator — Cube Size | Sets the spacing between grid samples. Smaller values provide finer sampling at a given volume size, but require more cells to cover that volume. |
| Creature Generator — Iso Value | Sets the density threshold used to extract the creature’s surface. |
| Creature Generator — Antenna Prefabs | Supplies the antenna shapes selected for each head. The current code selects from the first three entries. |
| Creature Generator — Feather Prefab | Supplies the feather geometry and material used by the wing-generation code. |
| Procedural Creature Rig — Bounds Padding | Expands the skinned renderer’s bounds to accommodate posed limbs. |
| Bone Renderer — Shape, Bone Size, and Color | Adjust the skeleton’s appearance in the Scene view. |
| Bone Renderer — Tripod Size checkbox and value | Show joint axes and adjust their size. |
| Creature Leg IK — Draw Controls and Control Size | Toggle the foot-target and knee-hint markers and adjust their size. |
| Rig on LegIK — Weight | Blends the leg IK layer from no influence at `0` to full influence at `1`. |

Head count, leg type, and most anatomical proportions are currently chosen by the generation code rather than exposed as separate Inspector controls.

## Trying the leg IK

1. Enter **Play Mode** and select a creature with animal or insect legs.
2. Expand `LegIK → [leg name]_IK` in the Hierarchy.
3. Select **FootTarget**, press **W**, and move it in the Scene view. The upper and lower leg rotate to bring the foot toward the target. Rotating the target also changes the foot’s orientation.
4. Select **KneeHint** and move it to change the direction the knee bends.

Foot targets appear as **green markers**, and knee hints appear as **orange markers** when Gizmos are enabled. Targets beyond a leg’s reach cause it to extend toward them while preserving its bone lengths. Play Mode changes reset when Play Mode ends.

The IK currently supports manual posing. Automatic walking, step timing, and terrain-based foot placement are not implemented.

## How it works

1. **Define the creature’s shape.** The generator creates a Catmull–Rom curve through the head, neck, and torso regions, then samples spheres along it. Sphere radii vary by region, and the head region can split into one, two, or three heads.
2. **Generate appendages.** Animal legs use paired tapered sphere chains with a small knee bend. Insect legs follow mirrored Catmull–Rom curves, while tentacles use sinusoidal paths. A tapered, oscillating tail contributes additional spheres to the same body surface.
3. **Build an implicit surface.** Each sphere contributes to a scalar density field using a smooth falloff. Marching cubes samples this field, interpolates surface vertices at the chosen iso value, and creates a triangle mesh with UVs and recalculated normals. The body, heads, legs, and tail share this mesh.
4. **Add details and color.** Separate objects provide eyes, irises, and antennae. The body receives a generated 256 × 256 texture that blends light and dark shades of a random hue using Perlin noise. Eye pivots use the `Wobble` component for motion.
5. **Build and bind the skeleton.** The rig creates a pelvis, spine, shared neck, separate head bones, and jointed leg chains. Sphere contributions are accumulated into vertex influences, reduced to the strongest four, and normalized. Bind poses and weights connect the mesh to a `SkinnedMeshRenderer`. Eyes and antennae follow their corresponding head bones.
6. **Create the IK controls.** Animal and insect legs receive Unity’s Two Bone IK constraints, foot targets, and knee hints. A generated generic Avatar and Rig Builder bind the procedural hierarchy to Unity’s animation system. Targets live outside the skeleton so the solved bones do not move their own controls.

The source also contains a wing generator based on bicubic Bézier patches, with randomized proportions and feather placement. Its `GenerateArms()` call is currently commented out in `Start()`, so the current preview generates creatures without wings.

## Main scripts

| Script | Purpose |
| --- | --- |
| `CreatureGenerator.cs` | Generates anatomy, the scalar field, mesh, textures, and detail objects. |
| `MarchingCubes.cs` | Stores the lookup tables and grid-edge definitions used for mesh extraction. |
| `ProceduralCreatureRig.cs` | Builds the skeleton, calculates skin weights and bind poses, and connects Bone Renderer and leg IK. |
| `CreatureLegIK.cs` | Creates the jointed-leg IK constraints, targets, hints, and Scene view markers. |
| `Wobble.cs` | Applies a sinusoidal rotation around a configurable axis. |

## Current scope

- Head, neck, torso, animal legs, and insect legs have procedural bones and skinning.
- Tentacles and the tail are weighted to the pelvis without separate articulated chains or IK.
- Skeleton visualization uses Unity’s Bone Renderer in the Editor’s Scene view.
- Meshes and rigs are generated during Play Mode; there is no editor bake or export workflow yet.
- Large grids can take time to generate because the density field evaluates sphere contributions across the sampling volume.

For standalone player builds, keep the submission-packaging utility `SubmissionPrep.cs` in an `Editor` folder because it uses Unity Editor APIs.
