---
layout: page
title: 3D Animated Scene
description: Project for Ecole Polytechnique Computer Graphics Course
img: assets/img/INF443_Scene_Image.png
importance: 3
category: CS & ML
giscus_comments: false
repository: adriengoldszal/3D-Animated-Scene
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ha-long-bay-in-vietnam.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/INF443_Scene_Image.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The real Ha-Long-Bay (left) and our rendered scene (right).
</div>

This project, part of Ecole Polytechnique's Computer Graphics class, leverages C++ and the OpenGL API to render a 3D animated scene. It takes advantage of the [CGP Library](https://graphicscomputing.fr/cgp/documentation/01_general/index.html) for most of the low level implementation. 

Together with [Emilie Liaud](https://www.linkedin.com/in/emilie-liaud-425058212/?originalSubdomain=fr), we created an interactive 3D Scene depicting the Ha-Long Bay in Vietnam, where the user can control a boat across the landscape. A few key elements are described in the following paragraphs, the full code can be found [here](https://github.com/adriengoldszal/3D-Animated-Scene)

## I - Elements of the scene

### Background and Skybox
- The applied skybox is slightly animated to give the impression of time passing.  
- Additionally, a shader is used to gradually change the skybox's color (blend) over time, creating the illusion of a day-night cycle (with red tones transitioning to black). These color changes are also applied to the ocean and rocks to create a cohesive effect across the entire scene.  
- Finally, fog is added to all shaders. To preserve the skybox's visibility, a gradient is applied to the skybox based on height, allowing for a smooth transition between the water's fog and the clearer sky.

### Terrain

To enable infinite navigation, a system with 9 different meshes (including water, seabed, and rocks with varying placements) has been implemented. These meshes are managed using a 3x3 mesh grid that moves along with the boat. When a column of meshes is shifted, the column behind it moves to the front to ensure the boat always remains at the center of the 9 meshes.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/columns.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Boat movement to the right (left image), boat movement upwards (right image).
</div>

### The Ocean 

The ocean is a simple square mesh with shader modifications:
- Perlin noise applied in the vertex shader to simulate waves.  
- Implementation of reflection and refraction of the skybox on the water in the fragment shader, using the Fresnel effect.

### The Rocks
We generated 4 rock objects with different shapes using the `marching_cubes_dynamic` fimplementation from CGP. The rocks were created by merging two deformed spheres with Perlin noise and exporting them as .obj files.

However, the Marching Cubes program does not support UV coordinates for .obj files, which are essential for applying textures to the rocks. We addressed this by using Blender. After applying the UV-unwrap method to the rocks in Blender to establish vertex connectivity, we exported .obj files with UV coordinates.

Next, we aimed to apply textures with greenery on the rocks to simulate trees growing on the rocks in a bay. Applying the rock texture was straightforward, but adding greenery was more challenging. Initially, we considered using "grass" billboards, as done in a previous exercise from the course, to populate the rocks. However, since Blender generated vertex connections in an uncontrolled manner, accurately placing each billboard on the rock object was difficult, resulting in an unappealing final render. Moreover, generating thousands of "grass" billboards for different rocks quickly became computationally expensive.

Ultimately, we switched to generating the texture in Blender by drawing 2D textures for each rock type, which we could then directly apply to the rocks.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/rocks.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Final rock formation (left image), 2D Texture applied to the rock (right image).
</div>

### The Villages

The 3D models of fishing houses were sourced from an opensource sketchfab model. Around certain rocks, several houses were added with varying positions and orientations (depending on the number of rocks per terrain, with a maximum of 3, to simulate fishing villages.

## II - Animated elements

### The junk boat 
The open-source model used can be found [here](https://sketchfab.com/3d-models/chinese-junk-ship-35b340bce9fb4e0680bc0116cebc35c9)

- User interaction was implemented; the user can move and rotate the boat through the scene using the WASD or ZQSD keys.  
- Simple Collision Detection : Collision is detected using bounding boxes around the rocks and houses.  
- Lighting : We decided to attach the scene's light source to the front of the boat to simulate realistic nighttime lighting (e.g., illuminating the rocks at night).

### Flying fish

Flying fish are characteristic of tropical oceans, such as those near Vietnam. An open-source 3D model was used [link here](https://sketchfab.com/3d-models/flying-fish-tobiuo-77e1a00a725148a1b4601b7482e60e30).

- Animation: The fish are animated using key positions and time points, interpolated with cardinal spline interpolation.  
- Dynamic Movement: Their positions are updated by a timer and move in sync with the boat's position.

{% if page.repository %}
<div class="repository-card">
  <h2>Associated Repository</h2>
  {% include repository/repo.liquid repository=page.repository %}
</div>
{% endif %}
