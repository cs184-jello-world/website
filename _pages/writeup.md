---
layout: page
title: writeup
permalink: /writeup
mathjax: true
---

# Final Writeup

[Slides ](https://docs.google.com/presentation/d/1ozbsNFH_Sg-njZ_wx_bAXcgFaxxcuOdkhU3FMj_iLkg/edit?usp=sharing) | [Video](https://www.youtube.com/watch?v=qnUuMLBezdM)

Jello, world! In our final project, we implemented a realtime Jello simulation in Unity with support for keyboard and mouse interactivity that allows the user to manipulate the Jello. We created two mass-spring systems in the forms of a cube and a sphere and wrote mesh generators for both shapes. Initially, we started with the cube model by extending the 2D cloth model from Project 4 to our 3D lattice of masses and springs. Then, we devised our own algorithm for the sphere model using spherical coordinates and testing different spring configurations. Finally, we tuned spring parameters for realistic renderings. Since we used Unity for our project, we took advantage of the High Definition Rendering Pipeline, allowing us to focus on the construction of the object itself.

## Technical Approach

**Mass-Spring Cube**

Since our Jello supports a variable number of particles per axis, we used a similar implementation to Project 4 when deciding where to place each particle by computing a constant offset distance between each particle. We initialized an $$N \times N \times N$$ lattice of particles to prevent from underconstraining later when adding springs.

We took advantage of our existing knowledge of the spring system in project 4 and expanded the definitions of each spring to apply to three-dimensional space. Structural constraints join adjacent vertices along each axis, bending constraints skip a vertex in a certain axis, and shearing constraints join adjacent vertices along face and space diagonals. Below is a diagram illustrating the springs between each particle:

![cube-springs](../assets/img/writeup/cube-springs.png){:style="display:block; margin-left: auto; margin-right: auto; width:50%;"}

**Mass-Spring Sphere**

We devised our own algorithm from scratch to initialize vertex positions and add springs for our spherical mass-spring model.

Initializing the particles for a sphere was more complicated. To simplify the design, we chose to leverage the spherical coordinate system with angles $$(\theta, \phi)$$. We loop through the two coordinates in equal increments to set positions for each point by transforming back to the Cartesian grid: $$x = R\sin(\theta)\cos(\phi)$$, $$y = R\cos(\theta)$$, and $$z = R\sin(\theta)\sin(\phi)$$. After accounting for the edge case at the poles (e.g. $$\theta = 0$$ or $$\theta = \pi$$) which should only have one particle, we initialized one final particle at the origin. We’ll elaborate on the usage of this origin particle later.

We had to experiment a bit to see what springs were necessary to constrain our system. Similar to structural, shearing, and bending springs, we first connected all particles to (1) their direct neighbors by incrementing $$\theta$$ and $$\phi$$, (2) their diagonal neighbors, and (3) springs connecting every other vertex in both angular dimensions. At this point, we found that the sphere deformed too easily, as there were only enough particles to simulate its skeletal surface but nothing representing the inside. As a result, we added springs connecting the origin to all the other particles to prevent too much deformation. Still, we needed to constrain the sphere further, so we added springs connecting each vertex to its polar opposite vertex radially through the origin.

| Vertices                                                                                                              | Springs                                                                                                             |
| --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| ![sphere-vertices](../assets/img/writeup/sphere-vertices.png){:style="display:block; margin-right: auto; width:90%;"} | ![sphere-springs](../assets/img/writeup/sphere-springs.png){:style="display:block; margin-right: auto; width:90%;"} |

**Mesh Generation**

For both the cube and the sphere mesh, we used Unity's built-in triangle Meshes based on an indexed triangle representation. For every vertex on the outer surface of the body, we track the position in a vertex list. Then, we describe triangles as triplets of indices within the vertex list $$(i_{v1}, i_{v2}, i_{v3})$$. A tricky aspect of creating the mesh was being intentional about making sure that all the triangles were in a consistent winding order so that the normals were pointed outwards.

**Handling Collisions**

Collisions between rigid bodies and our jellies were handled by Unity’s physics engine, primarily relying on the outer vertices of our mesh representations. We needed to update the center of the models to correspond to the average of the vertices for Unity’s collision detection to work properly.

Collisions between jellies and themselves were much more difficult. We ran into problems with non-vertex points in the mesh falling through each other when colliding. As a workaround, we made the springs collide with one another so that the meshes were full convex shapes in three dimensions and could no longer fall through each other. Because our models were based heavily on springs, the collisions were rather bouncy, but we still found they looked quite realistic.

**Interactivity**

We implemented two types of interactivity: (1) movement with WASD and space and (2) click-and-drag to manipulate vertices.

The WASD keys allow the user to control the Jello in the $$x-z$$ plane. Upon every update, we detect keyboard inputs. For instance, if the W key is pressed, we add velocity in the $$+z$$ direction to the rigid body component of every unique particle in the object. We also supported spacebar inputs to make the Jello jump in the $$+y$$ direction.

To support clicking and dragging our jello, we tracked the position of the mouse and sent out a ray from the mouse screen coordinate to the world and tracked the collision point with the object. Since moving around the mouse could only change two spatial dimensions, we needed to limit the movement of the dragged object to a plane in 3-d space. The plane we chose was the plane normal to the camera viewing direction that included the initial intersection point. Any subsequent mouse movement would be raycasted to intersect that plane to determine the object’s new position.

**Challenges**

As this was all of our first times working with Unity, we ran into a number of challenges:

1. **Distributing particles in our spherical mass spring system:** We found there was a tradeoff between computational efficiency with fewer particles and sufficiently constraining the system with springs so that the object remained stable. At first, we only added particles to the outer face of the sphere and distributed them evenly. However, to adequately constrain the system, we also added a particle in the origin that was connected to every other particle in the system (much like a bicycle wheel and its spokes). Another issue we originally faced with setting particle positions was that we had to special case $$\theta = 0$$ and $$\theta = \pi$$ because rotating these positions in $$\phi$$ should result in the same vertex.
2. **Decreasing the vertex particle size deformed our system:**
   We concluded that the structure of our objects was maintained via the particle collisions, and kept the particle sizes large enough but didn’t render the particles themselves. We’re still not completely sure whether the deformations were caused by precision errors when calculating the normals or the mass of the particle decreasing as the size of the particle also decreases, which would cause spring correction forces to apply an outsized impact on the particles.
3. **Choosing the sphere model's spring configuration:** During many early iterations of our design, we found that many spring configurations were underconstrained, resulting in erratic motion and deformation. As a result, we added an origin point with springs connecting to all outer vertices as well as other sets of springs that connect polar opposite vertices. Additionally, we found that lowering the spring constant to the 100-5000 range led to the best results.
4. **Collision detection:** When attempting to implement collisions, we found that the objects would only detect a collision when particles collided. However, since the mesh spans more locations than simply the particle positions, this would lead to objects being “inside” of one another before a collision was detected. To fix this, we modified the position of the Game Object in Unity to match the center of the mesh so that collisions were detected properly. Additionally, we enabled collisions between springs so that objects could no longer fall through one another.
5. **Exploring other models:** One of the limitations we found with our current mass-spring method was that designing the system for a particular mesh shape was a manual and difficult process. We wanted to try a softbody model that could represent any mesh shape, so instead of creating the mesh based on the outer vertices of the mass spring system, we just used the mesh vertices themselves in the physics simulation. To enable elasticity and deformation, we let the vertices be displaced from their original frame and then corrected them back with an oscillating force. However, this didn't look realistic so we didn't pursue it and to truly pursue a mesh-shape independent model we might've needed tetrahedral meshes.

We learned a lot in this project! Here are a few:

1. We broke apart our models into its components to make our code cleaner and simpler.
2. Choosing the right parameters (e.g. damping, particle size, number of particles, spring constant) is really important for realistic simulations.
3. To debug, we wrote helper methods to visualize specific aspects of our Jello (e.g. just the wireframe). This also came in handy when rendering images for the final writeup.

## Final Results

In the videos below, we capture the primary features of our Jello! You can see the cube and spherical Jellos we implemented, as well as applying movement and jumping commands to the objects.

| Bowl (Cube)                                                                                               | Bowl (Sphere)                                                                                                 | Ramps                                                                                             |
| --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| ![cube-bowl](../assets/img/writeup/cube-bowl.gif){:style="display:block; margin-right: auto; width:90%;"} | ![sphere-bowl](../assets/img/writeup/sphere-bowl.gif){:style="display:block; margin-right: auto; width:90%;"} | ![ramps](../assets/img/writeup/ramps.gif){:style="display:block; margin-right: auto; width:90%;"} |

{:style="margin-bottom: 10px;"}

We also played around with the spring constant to manipulate the rigidity of the Jello. As you can see, lower spring constants make the Jello more jiggly, analogous to an extra bouncy bit of jello! Naturally, a higher spring constant meant that the system would settle quicker. Interestingly, the bounce height for the cube seemed to increase then decrease as the spring constant increased. This might be because lower spring constants could not apply enough force to lift the Jello while higher spring constants do not deform enough to spring the Jello upwards.

![spring-constants](../assets/img/writeup/spring-constants.gif){:style="display:block; margin-left: auto; margin-right: auto; width:90%;"}

We also handled collisions between two Jello objects by using mesh colliders and spring collision (left) and clicking/dragging our Jello object as an aspect of interactivity (right).

| Collisions                                                                                                | Clicking/dragging                                                                               |
| --------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| ![collision](../assets/img/writeup/collision.gif){:style="display:block; margin-right: auto; width:50%;"} | ![drag](../assets/img/writeup/drag.gif){:style="display:block; margin-right: auto; width:50%;"} |

{:style="margin-bottom: 10px;"}

## References

From a high level, other than referencing documentation on how certain built in tools for Unity worked, we used trial and error when building

- ClothSim served as the basis for the way we went about building the mass-spring system, as well as the springs we attached to the cube object.
  = To translate the springs from 2-D to 3-D for the cube object specifically, we referenced slides by [Carnegie Mellon University’s Fall 2002 offering of Computer Graphics 15-462](https://www.cs.cmu.edu/~barbic/jellocube_bw.pdf)
- [Unity documentation on how the mesh class works](https://docs.unity3d.com/Manual/UsingtheMeshClass.html) so we know the appropriate information to supply when rendering the mesh.

## Contributions

Noah created the mass-spring cube and sphere models, implemented interactivity with keypresses, and figured out collisions. Johnny created the mesh renderer for the cube Jello, implemented interactivity with clicking/dragging, visualized the spring system with a wireframe, and looked into the frame deformation model. Allen worked on texturing the Jello using Unity's build in HDRP and helped construct the sphere mass-spring system and mesh. Jedi helped on creating the Jello materials in Unity and worked on the sphere Jello mass-spring model, too. Everyone contributed equally to the milestone, renders, and final presentation.
