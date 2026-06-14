---
layout: layouts/post.html
title: "LOD Architect Devlog #1: Geometry Nodes"
date: 2026-06-14
summary: "An in-depth look at building a procedural Level of Detail tool inside Blender using Geometry Nodes and Python."
tags: post
---

Welcome to the first devlog for **LOD Architect**. In this post, I want to break down the core approach I took to build a procedural Level of Detail (LOD) generator in Blender using Geometry Nodes. 

For real-time rendering, optimization is key. LODs help us maintain high frame rates in games by swapping out high-resolution models for lower-poly counts as they move further from the camera.

## Why Geometry Nodes?

Traditionally, making LODs is a destructive and manual process—you either decimate the mesh manually or run standard decimate scripts that ruin the topology. 

By building a custom generator in **Blender 4.x Geometry Nodes**, we get:
1. **Procedural Control:** Tweak parameters (decimation percentage, welding distances) in real-time.
2. **Non-Destructive Workflows:** Keep the high-res mesh intact while generating low-poly variants dynamically.
3. **Automated UV-Mapping:** Transfer UV coordinates and paint properties seamlessly down to the lower LOD levels.

---

## The Node Network Graph

Here is the general layout of how the Geometry Nodes modifier processes the input mesh:

* **Stage 1 (Welding):** Merges duplicate vertices based on a threshold to simplify flat regions.
* **Stage 2 (Decimation):** Reduces polycount using edge-collapse algorithms.
* **Stage 3 (UV Transfer):** Re-projects UV islands from the original high-poly source.

Below is a demonstration of the decimation algorithm running inside Blender:

![LOD Architect in Action](/blog/images/lod_demo.gif)

> Note: If you want to use the tool in Unreal Engine or Unity, make sure to export all generated LOD levels inside a single `.fbx` container.

---

## Automating Exports with Python

To make this tool seamless for game pipelines, I wrote a helper script that automates the generation and exporting of LOD0, LOD1, and LOD2. 

Here is a snippet of the exporter code:

```python
import bpy

def export_lods(target_object, export_path):
    # Ensure the object has the LOD modifier
    modifier = target_object.modifiers.get("LOD_Architect")
    if not modifier:
        print("LOD Architect modifier not found!")
        return

    # Loop through each LOD setting
    for lod_level in [0, 1, 2]:
        modifier["Input_Decimation"] = 1.0 - (lod_level * 0.35)
        bpy.context.view_layer.update()
        
        # Export individual FBX
        filename = f"{target_object.name}_LOD{lod_level}.fbx"
        bpy.ops.export_scene.fbx(
            filepath=export_path + filename,
            use_selection=True
        )
        print(f"Exported LOD {lod_level} to {filename}")
```

In the next log, I'll detail how I handled baking normals from high-poly models down to LOD2. Stay tuned!
