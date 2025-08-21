<div align="center">

# Kirara - Pure Python Ray Tracer

![Python](https://img.shields.io/badge/Python-3.6+-blue.svg)
![PPM](https://img.shields.io/badge/Output-PPM-brightgreen.svg)
![Multiprocessing](https://img.shields.io/badge/Multiprocessing-✓-orange.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

*A pure Python implementation of a ray tracer featuring Phong illumination, reflections, and parallel rendering*

</div>

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Implementation Process](#implementation-process)
  - [Milestone 1 - Implementation of Vector](#milestone-1---implementation-of-vector)
  - [Milestone 2 - Color](#milestone-2---color)
  - [Milestone 3 - Image Production](#milestone-3---image-production)
  - [Milestone 4 - Ray Tracing](#milestone-4-raytracing)
  - [Milestone 5 - Lighting](#milestone-5-lighting)
  - [Milestone 6 - Reflection](#milestone-6-reflection)
  - [Milestone 7 - Parallel Processing](#milestone-7-firing-all-cores---parallize-the-raytracer)
- [File Structure and Implementation Details](#file-structure-and-implementation-details)
  - [Core Components](#core-components)
  - [Scene Elements](#scene-elements)
  - [Rendering Engine](#rendering-engine)
- [Examples](#examples)
- [Usage Guide](#usage-guide)
  - [Running the Ray Tracer](#running-the-ray-tracer)
  - [Creating Custom Scenes](#creating-custom-scenes)
- [Performance Optimizations](#performance-optimizations)
  - [Multiprocessing Implementation](#multiprocessing-implementation)
  - [Ray Tracing Optimizations](#ray-tracing-optimizations)

## Overview

Puray is a pure Python implementation of a ray tracer that can render 3D scenes with realistic lighting, shadows, and reflections. The project demonstrates the fundamentals of computer graphics and ray tracing while maintaining clean, modular code structure.

## Features

### Core Rendering Features
- **Pure Python Implementation**
  - No external dependencies required
  - Written in standard Python 3
  - Easy to understand and modify

- **Ray Tracing Algorithm**
  - Recursive ray tracing for realistic reflections
  - Configurable maximum reflection depth
  - Accurate shadow calculations
  - Precise ray-object intersection calculations

- **Advanced Lighting Model**
  - Phong illumination model implementation:
    - Ambient lighting for base illumination
    - Diffuse reflection for matte surfaces
    - Specular highlights for shiny surfaces
  - Multiple light source support
  - Realistic light attenuation

### Material System
- **Material Properties**
  - Customizable ambient, diffuse, and specular coefficients
  - Adjustable reflection intensity
  - Support for solid colors and patterns

- **Pattern Support**
  - Checkered material implementation
  - Configurable pattern colors and scale
  - Extensible pattern system

### Performance Features
- **Multiprocessing Support**
  - Parallel rendering across multiple CPU cores
  - Automatic CPU core detection
  - Configurable process count
  - Progress tracking during rendering

### Output
- **PPM Image Format**
  - Both ASCII and raw binary PPM support
  - Wide color range support (24-bit color)
  - Compatible with most image viewers
  - Easy conversion to other formats


## Implementation Process
### Milestone 1 - Implementation of Vector
In `vector.py`, implemented
- Basic vector addition, subtraction, multiplication and division operators

<blockquote>
<b><i>Learning point:</i></b>
<code>__div__()</code> vs <code>__truediv__()</code>
<br/>
Python 2 u
</blockquote>

- `magnitude()` and `normalize()` functions

### Milestone 2 - Color
The `color.py` is basically treated as a vector with its `x` as `r`, `y` as `g` and `z` as `b`, whose values range from 0 to 1.

The `from_hex()` class method takes a hex string and returns a `Color` vector.

<blockquote>
<b><i>Learning point:</i></b>
<code>@classmethod</code> and <code>@staticmethod</code> in Python
<br/>
Python 2 u
</blockquote>

### Milestone 3 - Image production
The `image.py` contains the code for production of image.
- `set_pixel(x,y,color)` sets the pixel at position `(x,y)` with color `color` (instance of `Color`). These pixels are stored in 2D array `pixels` of size `width`x`height`
<blockquote>
<b><i>Learning point:</i></b>
<br/>
<code>pixels[][]</code> contains all the pixels on the screen. The <code>x</code> of <code>set_pixel</code> (x-coordinate of pixel) actually refers to the column of <code>pixels[][]</code>. Similarly, <code>y</code> refers to the row. <br/>
Thus pixel at <code>x,y</code> is set at <code>pixels[y][x]</code>
</blockquote>

- `write_ppm(file)` writes the `pixels` array into the opened file where the function is being called. The contents `pixels` array is scaled to 255.
<blockquote>
<b><i>Learning point:</i></b>
PPM File format
<br/>
The only human readable image file format. <br/>
PPM file format:
<pre>
P3 (width) (height)
(max color)
(space delimited triplets of x,y,color)
</pre>
Sample PPM File:
<pre>
P3 100 100
255
255 0 0 255 0 0
</pre>
</blockquote>

Until this milestone, we can generate decent `.ppm` image file with visible color pixels.

### Milestone 4: RayTracing
**Requirement** - Render a 3D ball into an image _without shading_.
#### Ray
Equation of ray:
```math
\hat{ray}(t) = \hat{ray}_{origin} + \hat{ray}_{direction} * t
```

where `ray(t)` represents the origin of the ray, and `ray_direction` is the ray direction (usually normalized). Changing the value of `t` allows defining any position along the ray. When 
`t` is greater than 0, the point is located in front of the ray's origin (looking down the ray's direction) when 
`t` is equal to 0, the point coincides with the ray's origin, and when 
`t` is negative, the point is located behind its origin.

The **`ray.py`** implements a ray, with `origin` and `direction` vectors. (`direction` being a unit vector through `Vector.normalize()`) 
 

Now, we will be studying raytracing with **spheres**.

<br/>

#### Simplified Algorithm:
1. Shoot a ray towards each pixel
2. Find nearest object hit by the ray in the scene
3. If hit, then find the color of the object.

<br/>

#### Ray-Sphere intersection cases:
1. Intersection with 2 points of sphere, i.e. _ray went through the sphere_
2. Intersection at only one point, i.e., _ray goes tangentially to the sphere_
3. No intersection


Now, we define a vector from ray origin to the center of sphere 

```math
sphere\_to\_ray = ray_{origin} - sphere_{center}
```


   where the vector `ray_origin` is the origin point of ray and vector `sphere_center` is the center point of the sphere.



<br/>

#### Ray-Sphere Interesection Formula
[Derivation](#derivation)
```math
a = 1 \\
b = 2*ray_{dir} \cdot \hat{sphere\_to\_ray} \\
c = \hat{sphere\_to\_ray} \cdot \hat{sphere\_to\_ray} - (radius)^2 \\
discriminant = b^2 - 4ac
```

$discriminant > 0$ Intersects at 2 points

$discriminant = 0$ Intersects at 1 point

$discriminant < 0$ No intersection

Distance to the point of intersection from origin of ray ($ray_{origin}$):
```math
dist = \frac{-b \pm \sqrt{discriminant}}{2a}
```

:warning: We ignore the negative distance. So we replace $\pm$ to $-$, since our camera will be at **negative z**, looking at **positive z**. If the distance turns out to be negative, we will conclude that the ray did not hit the sphere, i.e., _either we are in the sphere_ or _the sphere is in negative z axis_.

<br/>

The **`sphere.py`** implements a sphere which takes `center:Vector`, radius and material.

The `intersects()` method takes a `ray:Ray` and returns the `dist` if the `ray` intersects the shphere using the above mathematical formulae, or `None` if the `ray` doesn't intersect.
<br/>

#### Hit Position
To determine where the ray hit (_hit position_), we put the calculated $dist$ into the equation of ray as $t$, i.e.,
```math
\hat{hit}_{pos} = \hat{ray}_{origin} + \hat{ray}_{direction} * dist
```

<br/>

#### Color determination at Hit Position
Since we are not dealing with the shading(_the process of altering the color of an object/surface/polygon in the 3D scene, based on the surface's angle to lights, its distance from lights, its angle to the camera and material properties,etc._), we will use plain color for now. 

<br/>

#### Aspect Ratio
Sometimes our rendered image might look squashed or stretched, because the aspect ratio was strongly calculated. Because it might not always be the case that the dimensions of the logical space which we are looking in will be the same as dimensiom of our screen. 

If aspect ratio is not taken into consideration, the rendered image might get **distorted**, i.e., if the rendering window is wider than it is tall, the rendered image will be stretched horizontally. Also, it might happen that the rays cast through the pixel might not correctly correspond to the intended **ray direction**. Objects might appear stretched or squished due to unproportionate widow and viewport size (**Non-uniform scaling**)

For example, if the original image (which acts as a screen/viewport to the rendered objects), is of ratio 16:9 and if we change the size of our image (again, viewport) to 4:3 ratio, the objects might appear distorted because the rays cast through the pixels now correspond to a different proportion of width to height, and the pixels of 16:9 ratio will be forced to fit in the frame of 4:3. This mismatch leads to incorrect rendering of the scene's proportions. To counter this, we can make the image remain in same aspect ratio and either
- Scale the image (pillarboxing or letterboxing)
- Crop the image

In fact, logical (_normalized_) dimensions of the scene is $x_{min}=-1$ to $x_{max}=+1$ 

Suppose our screen is of size 300x200, i.e., rectangular. However, the pixels are square in shape.

```math
aspectRatio = \frac{width}{height} = \frac{320}{200}=1.6
```
So, 
```math
y_{max} = \frac{1}{aspectRatio} = \frac{1}{1.6}=+0.625
```
becomes bottom edge (maximum y), and `-0.625` becomes top edge.


<br/>
<br/>
<div style="display:flex;width:100%;justify-content:center;">
<svg height="300" width="300" style="display:flex;">
   <rect width="300" height="300" fill="white" fill-opacity="0.5"/>
   <rect width="300" height="200" y="45" fill="black"/>
   <circle r="50" cx="50%" cy="50%" fill="red" />
   <text style="font-family:Consolas" x="75.5%" y="15" fill="white">y:-1.000</text>
   <text style="font-family:Consolas" x="75.5%" y="40" fill="white">y:-0.625</text>
   <text style="font-family:Consolas" x="75.5%" y="260" fill="white">y:+0.625</text>
   <text style="font-family:Consolas" x="75.5%" y="297" fill="white">y:+1.000</text>
   <text style="font-family:Consolas" x="0%" y="50%" fill="white">x:+1.000</text>
   <text style="font-family:Consolas" x="75.5%" y="50%" fill="white">x:+1.000</text>
   <text style="font-family:Consolas" x="49%" y="51%" fill="white">0</text>
</svg>
</div>
<br/>
<br/>

Thus, $y$ cannot range from $+1$ to $-1$. It has to be adjusted by the aspect ratio to prevent image distortion.


#### Scene
We now render a sphere into a 320x200 image.
Camera position `(0,0,-1)`. Sphere `(0,0,0,r=0.5,#000000)`

The `scene.py` implements a scene of size `width`x`height`, takes an array `objects` and a `camera`.
<br/>
<br/>

#### The `render()` method implementation
The `render()` method in `engine.py` takes a `Scene` and returns rendered `Image`. Sequentially, we
- calculate `aspect_ratio` of the `scene`
- calculate normalized `x_step` and `y_step` according to the aspect ratio
- initialize empty `Image` object where the rendered image will be stored
- do for each row `i`
  - initialize `y` as marker at current row, skipping previous `i` rows by `y_steps`
  - do for each pixel in the row `j`
    - initialize `x` as marker at current col, skipping previous `j` cols by `x_steps`
    - throw a `ray` from camera position towards **each** normalized coordinate `(x,y)`
    - find the `color` by `ray_trace()`ing the `ray` in the `scene`
    - `set_pixel()` the current ***pixel*** `(i,j)` with the `color`
- return the 2D `pixels` array (`Image`)
<blockquote> 
<b><i>Learning point:</i></b>
Why throw rays per <code>(x,y)</code> and not <code>(i,j)</code>
<br/>
null
</blockquote>


#### The `ray_trace()` method implementation
The `ray_trace()` method in `engine.py` takes a `Ray` to be ray traced in the `Scene` and returns the `Color` at the point of hit (if any). Sequentially, we
- we find the object which the ray hits and the scalar distance to the hit position by looping over all the objects in the `Scene` and testing the intersection (using the mathematical formulae) with each (handled by `find_nearest()` method)
- assume a black `color` and retur it in case the ray doesn't hit anyone
- make a vector `hit_pos` from the origin of the ray, till the hit position, by multiplying the distance till hit position
- return the color of object at this hit position in the scene

#### The `color_at()` method implementation
The `color_at()` method in `engine.py` simply returns the color of `material` of the object as we are not considering any lighting yet.

By now we are able to create a basic image of a red sphere which looks like a circle now because of uniform color and no lighting.

### Milestone 5: Lighting
Lights can bounce, reflect, refract and interact with different objects differently. We can improve our ray tracing model significantly, by just focusing on three components how light interact.
- Ambient reflectivity _is the overall light in the scene. Thus shadows are not pure black_
- Diffuse reflectivity _is when the light reflects on rough surfaces like wood and scatters uniformly / diffuses in every direction. All materials exhibit this property unless it is a perfectly reflective surface like mirror._
- Specular reflectivity _is that shiny spot of light that appears when light reflects off from a smooth surface. This is shown by materials like marbles, metals._

This is commonly known as the Phong Model, which is the most common model used in rendering in game engines.

Let's implement Phong's Model by applying shading algorithms.

#### Diffuse Shading - Reflections from a matte surface
The light in not shining back to us, but being diffused by that material. Many shading models for diffuse shading, most popular is **Lambert Shading Model**.

Lambert's Cosine Law - when the light falls obliquely on a surface the illumination of the surface is directly proportional to the cosine of the angle $\theta$ between the direction of the incident light and the surface normal.

When the light rays fall directly on the surface (making $0\deg$ angle) the illumination is highest. When the light grazes through the surface, none of the rays interact with the surface, making no illumination ($90\deg$ angle)

**Remark:** The dot product of two _normalized_ vector gives the angle between them.
e.g. $cos\theta = \text{L . N}$ where $\text{L}$ is the incident light vector and $\text{N}$ is the surface normal vector at hit position (of ray and the object), both normalized. Thus, in our implementation, we will use `Vector.normalize()` and `Vector.dot_product()` methods to find the angle. The dot product must not be negative.

Thus, our Lambert Shading Model now looks like, <br/>
<small style="font-size:14px;">[detailed derivation](https://www.scratchapixel.com/lessons/3d-basic-rendering/introduction-to-shading/diffuse-lambertian-shading.html)   </small>
```math
diffuse = \text{L . N} M_{diffuse} \\
\space \\
Color_{surface_{diffusion}} = diffuse * C_{object} \\
=\text{L . N} M_{diffuse} C_{object}
```
where $M_{diffuse}$ is the material's diffusion coefficient (how much the material is absorbing light) and $C_{object}$ is the color of the object.

If $M_{diffuse}$ is 0, the material is rough ($Color_{surface}$ approaches to `(0,0,0)`) and when $M_{diffuse}$ is 1, material is smooth ($Color_{surface}$ approaches to the original value of the surface color).

#### Specular Shading - Reflections from shiny plastics
Simple Phong Shading model states that the light reflected varies based on the angle between the difference between the view direction and the direction of the perfect reflection.

Let $L$ be the incident light vector on the surface, making $\theta$ angle to the $N$ normal. $V$ is where the viewer is (ray towards viewer). $R$ represents the unit vector directed towards the ideal specular reflection,i.e. gad the viewer be at $R$, he would have seen a sharp reflection in his eyes.
<div style="display:flex;width:100%;justify-content:center;">
<svg height="200" width="300">
   <rect x="20" y="150" fill="white" width="250" height="5" />
   <line x1="20" y1="20" x2="140" y2="150" stroke="white" />
   <line x1="260" y1="20" x2="140" y2="150" stroke="white" />
   <line x1="210" y1="20" x2="140" y2="150" stroke="white" />
   <line x1="140" y1="20" x2="140" y2="150" stroke="white" />
   <text style="font-family:Consolas" x="39%" y="51%" fill="white">θ</text>
   <text style="font-family:Consolas" x="20%" y="20%" fill="white">L</text>
   <text style="font-family:Consolas" x="42%" y="20%" fill="white">N</text>
   <text style="font-family:Consolas" x="60%" y="20%" fill="white">V</text>
   <text style="font-family:Consolas" x="90%" y="20%" fill="white">R</text>
</svg>
</div>

Note - Though $L$ depicts the incident light ray vector, its direction is reversed as we only consider the light’s interaction with the surface at the point of contact, and we’re interested in the angle it makes with the normal.

We define $$Phong Term = (V.R)^S$$
where $S$ is used to adjust how strongly the reflection is happening (material's specular coefficient; shininess).

**Issue**: The dot product in the above must not be negative. But angle between $\bold V$ **and** $\bold R$ can be larger than $90 deg$, i.e. the viewer is free to move, and the dot product becomes negative.

Hence, the **Phong-Blinn Modification** calculates a halfway vector (normalized) $H$, which is half way between the **viewer $\bold V$ and the light source $\bold L$**, i.e. the angle between the viewer and the light source is cut into half by $H$. We then replace $V$ with $H$.
<div style="display:flex;width:100%;justify-content:center;">
<svg height="200" width="300">
   <rect x="20" y="150" fill="white" width="250" height="5" />
   <line x1="20" y1="20" x2="140" y2="150" stroke="white" />
   <line x1="260" y1="20" x2="140" y2="150" stroke="white" />
   <line x1="120" y1="20" x2="140" y2="150" stroke="white" />
   <line x1="210" y1="20" x2="140" y2="150" stroke="white" />
   <line x1="140" y1="20" x2="140" y2="150" stroke="white" />
   <text style="font-family:Consolas" x="39%" y="51%" fill="white">θ</text>
   <text style="font-family:Consolas" x="20%" y="20%" fill="white">L</text>
   <text style="font-family:Consolas" x="42%" y="20%" fill="white">N</text>
   <text style="font-family:Consolas" x="36%" y="35%" fill="white">H</text>
   <text style="font-family:Consolas" x="60%" y="20%" fill="white">V</text>
   <text style="font-family:Consolas" x="90%" y="20%" fill="white">R</text>
</svg>
</div>


```math
H = L+V \\
H = norm(H)
```
Thus,
```math
Blinn Term = (H.R)^S
```
And,
```math
Color_{surface_{spec}} = BlinnTerm * C_{object} \\
=\text{L . N} M_{diffuse} C_{object}
```

So, according to the Phong Model, the final surface color is determined as,
```math
Color_{surface} = Color_{surface_{diff}} + Color_{surface_{spec}}
```
#### `Material` Implementation
So far, we used `material` of `Sphere` as an instance of `Color`. To implement the above discussed Diffused and Specular reflections, we must define data structues for `Material`. We'll consider
- Color _what is the base color?_
- Ambient _what color it reflects in ambient light?_
- Diffuse _what color it reflects in diffuse light?_
- Specular _what color it reflects in specular highlight?_

We also implement light source `Light` for which we consider
- Position
- Color of the light source

and finish `material.py`.

#### The `color_at()` method implementation
Now we will add the functionality to render diffusion and specular lighting in `engine.py`. To so do, we will make use of `hit_pos` and `hit_normal`.



    def color_at(self, obj_hit:Sphere, hit_pos:Vector, hit_normal:Vector, scene:Scene):
        material = obj_hit.material
        obj_color = material.color_at(hit_pos)

        to_cam_ = scene.camera - hit_pos

        color = material.ambient * Color.from_hex("#000000")
        specular_S = 50
        # Light Calculations
        for light in scene.lights:
            to_light = Ray(hit_pos, light.position - hit_pos)

            #Diffuse Shading (Lambert)
            color += obj_color * material.diffuse * max(hit_normal.dot_product(to_light.direction), 0)

            #Specular Shading (Blinn-Phong)
            half_vector = (to_light.direction + to_cam_).normalize()

            color += light.color * material.specular * max(hit_normal.dot_product(half_vector), 0) ** specular_S

        return color

### Milestone 6: Reflection

For reflection, we will find the reflected ray and recursively call `RenderEngine.ray_trace(reflected_ray)` method, until maximum depth (3 or 5 rays) is reached.

#### Finding Reflected Ray - **R**
```math
R_{dir} = L - 2 (L\space .\space N) N \\
R_{pos} = H_{pos} + N*\delta
```
where $L$ is the light vector which is originally hitting the object, $N$ is the normal at the point, at which the light hits the object (hit position normal)

The position of the reflected ray $R_{pos}$ is same as the hit position $H_{pos}$ in theory, but we are adding the term  number $N*\delta$ (where $\delta$ is a very small number). The reason why we do this is because if we start from the same hit position for the reflected ray, it is very likely that the algorithm will get confused that it again hit the object where we stared from. To prevent the algorithm from calculate it as the hit position, we displace the ray along the direction of normal by a very small amount. It is practically same as hit position but a little bit displaced. 

#### Creating Floor
To reuse the `Sphere`, we exploit the fact that the Earth is just a giant sphere, and so our floor will be.

To make the ground chequered pattern, we will create new subclass of `Material` class and manipulate the `color_at` method of this class to create the pattern.

Pattern - Every black square both X and Y are even or odd at the same time

### Milestone 7: Firing all Cores - Parallize the Raytracer
Sync problems; locks;
Python uses memory management model called reference countingwhich is not tread safe.

If one python process can run on one core, why not we use other cores to run more processes?
`multiprocessing` module enables us to do this. Instead of threads, new processes are made.
Still we might encounter a race conditions because we are sharing progress variable.
We can counter this using the value class of the multiprocessing module.


Processor bound? PyPy
IO bound? AsyncIO
### Derivations
#### Ray-Sphere intersection formula derivation


Equation of ray:
```math
R(t) = O + tD
```
Equation of sphere:
```math
x^2+y^2+z^2 = r^2
```
Let C(x_0,y_0, z_0) be the center of sphere. Then points P(x,y,z) are locus of points that satisfy 
```math
(x_0-x)^2 + (y_0+y)^2 + (z_0+z)^2=r^2
```

This can be written as, 
```math
\Vert P-C \Vert=r^2
```
If the ray intersects, it would have common point(s) satisfying
```math
\Vert R-C \Vert=r^2
```
which is equivalent to
```math
dot((R-C),(R-C)) = r^2
```
```math
dot((O+tD-C),(O+tD-C)) = r^2 \\
dot(O,O)+t.dot(O,D)
```

## Technical Implementation Details

### Core Components

1. **vector.py**
   - Implements the `Vector` class for 3D vector operations
   - Functions:
     - `__init__(self, x=0.0, y=0.0, z=0.0)`: Initialize vector with x, y, z components
     - `dot_product(self, other)`: Calculate dot product with another vector
     - `normalize(self)`: Return normalized (unit) vector
     - Various operator overloads (`+`, `-`, `*`) for vector arithmetic
   - Essential for ray direction calculations and vector math throughout the raytracer

2. **point.py**
   - Implements the `Point` class for representing 3D points in space
   - Functions:
     - `__init__(self, x=0.0, y=0.0, z=0.0)`: Initialize point with x, y, z coordinates
     - Operator overloads for point-vector arithmetic
     - Used for defining object positions, light positions, and intersection points

3. **ray.py**
   - Implements the `Ray` class for ray tracing
   - Functions:
     - `__init__(self, origin, direction)`: Create ray with origin point and direction vector
     - Properties to access normalized direction
   - Represents light rays and view rays in the scene

4. **color.py**
   - Implements the `Color` class for RGB color representation
   - Functions:
     - `__init__(self, r=0.0, g=0.0, b=0.0)`: Initialize with RGB values (0-1 range)
     - `from_hex(hex_color)`: Create color from hex string (e.g., "#FF0000")
     - Operator overloads for color arithmetic (addition, multiplication)
     - `clamp(self)`: Ensure color values stay in valid range

### Scene Elements

5. **sphere.py**
   - Implements the `Sphere` class - the only 3D shape supported
   - Functions:
     - `__init__(self, center, radius, material)`: Create sphere with center point, radius, and material
     - `intersects(self, ray)`: Calculate if and where a ray intersects the sphere
     - `normal(self, surface_point)`: Calculate surface normal at given point
   - Uses quadratic equation solving for ray-sphere intersection

6. **material.py**
   - Contains two material classes for object surfaces
   - `Material` class functions:
     - `__init__(self, color, ambient, diffuse, specular, reflection)`: Create material with lighting properties
     - `color_at(self, position)`: Get color at surface position
   - `ChequeredMaterial` class functions:
     - `__init__(self, color1, color2, ambient, diffuse, specular, reflection)`: Create checkered pattern material
     - `color_at(self, position)`: Calculate checker pattern color at position
   - Controls how objects reflect and absorb light

7. **light.py**
   - Implements the `Light` class for point light sources
   - Functions:
     - `__init__(self, position, color)`: Create light at position with specific color
   - Used for illuminating the scene and calculating shadows

8. **scene.py**
   - Manages the entire 3D scene configuration
   - Functions:
     - `__init__(self, camera, objects, lights, width, height)`: Initialize scene with all components
   - Stores:
     - Camera position and viewing parameters
     - List of all objects in the scene
     - List of all light sources
     - Image dimensions

### Rendering Engine

9. **engine.py**
   - Contains the core `RenderEngine` class that implements the ray tracing algorithm
   - Main functions:
     - `render_multiprocess(self, scene, process_count, img_fileobj)`: Main rendering function with multi-processing
     - `render(self, scene, hmin, hmax, part_file, rows_done)`: Render a portion of the scene
     - `ray_trace(self, ray, scene, depth=0)`: Trace a single ray through the scene
     - `find_nearest(self, ray, scene)`: Find nearest object intersected by ray
     - `color_at(self, obj_hit, hit_pos, normal, scene)`: Calculate color at intersection point
   - Implements:
     - Phong illumination model (ambient, diffuse, specular)
     - Reflection calculations with maximum depth
     - Multi-process rendering for performance
     - Progress tracking

10. **image.py**
    - Handles image creation and file output
    - Functions:
      - `__init__(self, width, height)`: Create image buffer with dimensions
      - `set_pixel(self, x, y, col)`: Set color of pixel at (x,y)
      - `write_ppm(self, file)`: Write image in PPM format (text)
      - `write_ppm_raw(self, file)`: Write image in PPM format (binary)
    - Manages pixel data array and PPM file format output

11. **main.py**
    - Entry point of the application
    - Functions:
      - `main()`: Primary entry point
        - Parses command line arguments
        - Loads scene file dynamically
        - Sets up multi-processing
        - Initiates rendering process
    - Command-line interface for:
      - Scene file selection
      - Process count configuration
      - Output file handling

### Examples

The `examples/` directory contains sample scenes:
- `twoballs.py`: Example scene with two colored spheres and a checkered ground plane
- Output files are saved in PPM format (e.g., `2balls.ppm`)

## Usage Guide

### Running the Ray Tracer

The ray tracer can be executed using the command line interface:

```bash
python main.py <scene_file> [-p PROCESSES]
```

#### Command Line Arguments
- `scene_file`: Path to the scene file (without .py extension)
  - Scene file contains the complete scene description
  - Must be a valid Python module
- `-p, --processes`: Number of processes for parallel rendering
  - `0` (default): Automatically use all available CPU cores
  - `1`: Single process rendering (useful for debugging)
  - `n`: Use n processes for rendering

#### Example Commands
```bash
# Render using all CPU cores
python main.py examples/twoballs

# Render using 4 processes
python main.py examples/twoballs -p 4

# Render single-threaded
python main.py examples/twoballs -p 1
```

### Creating Custom Scenes

#### Scene File Structure
Each scene is defined in a Python file with the following required variables:

1. **Image Configuration**
   ```python
   WIDTH = 1920    # Image width in pixels
   HEIGHT = 1080   # Image height in pixels
   RENDERED_IMG = "output.ppm"  # Output filename
   ```

2. **Camera Setup**
   ```python
   CAMERA = Vector(0, -0.35, -1)  # Camera position in 3D space
   ```

3. **Object Definition**
   ```python
   OBJECTS = [
       # Ground plane (very large sphere)
       Sphere(
           Point(0, 10000.5, 1),
           10000.0,
           ChequeredMaterial(
               color1=Color.from_hex("#420500"),
               color2=Color.from_hex("#e6b87d"),
               ambient=0.2,
               reflection=0.2,
           ),
       ),
       # Reflective sphere
       Sphere(
           Point(0.75, -0.1, 1),
           0.6,
           Material(
               Color.from_hex("#0000FF"),
               ambient=0.1,
               diffuse=0.7,
               specular=1.0,
               reflection=0.6
           ),
       ),
   ]
   ```

4. **Lighting Setup**
   ```python
   LIGHTS = [
       Light(Point(1.5, -0.5, -10), Color.from_hex("#FFFFFF")),
       Light(Point(-0.5, -10.5, 0), Color.from_hex("#E6E6E6")),
   ]
   ```

#### Material Properties Explained
- `ambient`: Base visibility of object (0.0-1.0)
- `diffuse`: Matte lighting intensity (0.0-1.0)
- `specular`: Highlight intensity (0.0-1.0)
- `reflection`: Mirror reflection strength (0.0-1.0)

#### Example Implementations
See `examples/twoballs.py` for a complete scene setup showcasing:
- Multiple sphere objects
- Checkered ground plane
- Multiple light sources
- Different material properties


## Performance Optimizations

1. **Multiprocessing Implementation**
   - Scene divided into horizontal strips
   - Each strip rendered in separate process
   - Results merged into final image
   - Progress tracking across processes

2. **Ray Tracing Optimizations**
   - Early termination for shadow rays
   - Maximum reflection depth limit
   - Minimum displacement to prevent self-intersection
   - Normalized vectors for faster calculations
