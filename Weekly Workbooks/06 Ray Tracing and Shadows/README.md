## Ray Tracing and Shadows
### <a href='https://uob.sharepoint.com/:f:/r/teams/UnitTeams-COMS30020-2022-23-TB-1-A/Shared%20Documents/General/Recordings/View%20Only?csf=1&web=1&e=aTmZb2' target='_blank'> Weekly Briefing ![](../../resources/icons/briefing.png) </a>
### Task 1: Introduction
 <a href='01%20Introduction/slides/segment-1.pdf' target='_blank'> ![](../../resources/icons/slides.png) </a> <a href='01%20Introduction/audio/segment-1.mp4' target='_blank'> ![](../../resources/icons/audio.png) </a> <a href='01%20Introduction/animation/segment-1.mp4' target='_blank'> ![](../../resources/icons/animation.png) </a>

As we have seen, rasterising is a fast and efficient approach to rendering a 3 dimensional scene. However it has limitations - there are a number of phenomena that are difficult (or even impossible) to achieve using this approach. In particular, rendering "light" and "shadow" is not well suited to rasterisation. For this reason, we now turn our attention to an alternative form of rendering: _ray tracing_. Take a look at the slides, audio narration and animation linked to above in order to gain a high-level understanding of the operation of _ray tracing_. Once you have grasped the basic principles, move on to the tasks in the rest of this workbook.

It is advisable to continue working with your current project and augment it with additional ray tracing functions. This will not only allow you to switch between different rendering modes (wireframe, rasterised, ray-traced) but will also permit "hybrid" rendering to take place.  


**Hints & Tips:**  
The code that you will write for this workbook will be VERY resource intensive. If you find that your renderer is running too slowly to allow you to navigate the camera around the scene effectively, then you might like to try using the optimised `make speedy` build rule.

Ray tracing is a challenging activity that will exercise all your abilities as a programmer. Due to the nature of a lot of the tasks in this workbook, traditional approaches to debugging will typically prove ineffective. To aid you in your work, we have provided a <a href="../../Debugging" target="_blank">debugging guide</a> - the techniques outlined in this document will allow you to more effectively detect and isolate errors in your code.  


# 
### Task 2: Closest Intersection
 <a href='02%20Closest%20Intersection/slides/segment-1.pdf' target='_blank'> ![](../../resources/icons/slides.png) </a> <a href='02%20Closest%20Intersection/audio/segment-1.mp4' target='_blank'> ![](../../resources/icons/audio.png) </a> <a href='02%20Closest%20Intersection/audio/segment-2.mp4' target='_blank'> ![](../../resources/icons/audio.png) </a> <a href='02%20Closest%20Intersection/animation/segment-1.mp4' target='_blank'> ![](../../resources/icons/animation.png) </a> <a href='02%20Closest%20Intersection/animation/segment-2.mp4' target='_blank'> ![](../../resources/icons/animation.png) </a>

The first objective we need to achieve when attempting to perform ray tracing is to be able to detect when (and more importantly where) a projected ray intersects with a model triangle. Watch the narrated slides and animations above to gain a theoretical understanding of how to perform this operation.

With the knowledge gained, your task is to write a function called `getClosestIntersection` that given:
- the position of the camera (as a `vec3`)
- the direction of a ray being cast from the camera into the scene (also as a `vec3`)

will search through the all the triangles in the current scene and return details of the _closest_ intersected triangle (if indeed there is an intersection !).

We appreciate that this is a complex task, so to help you achieve this, the code below is the C++ equivalent of the ray/triangle intersection equation that was shown in the slides above:  

``` cpp
glm::vec3 e0 = triangle.vertices[1] - triangle.vertices[0];
glm::vec3 e1 = triangle.vertices[2] - triangle.vertices[0];
glm::vec3 SPVector = cameraPosition - triangle.vertices[0];
glm::mat3 DEMatrix(-rayDirection, e0, e1);
glm::vec3 possibleSolution = glm::inverse(DEMatrix) * SPVector;
```

It is important to recognise that the `possibleSolution` calculated by the above code is NOT the `(x,y,z)` coordinates of a point in 3 dimensional space, but rather a three-element data structure that consists of:

- `t` the _absolute_ distance along the ray from the camera to the intersection point
- `u` the _proportional_ distance along the triangle's first edge that the intersection point occurs
- `v` the _proportional_ distance along the triangle's second edge that the intersection point occurs

In order to calculate the actual position of the intersection point in 3 dimensional space, you can _either_ use the distance `t` along the projected ray _or_ insert `u` and `v` into the formula below. Both approaches _should_ give the same solution (in fact it might be an interesting double-check to see if they agree on the same intersection point !)


  


![](02%20Closest%20Intersection/images/formula.jpg)

**Hints & Tips:**  
Your function is going to need to return various details about the intersection point found. To help you in achieving this, we have provided a class in the `libs/sdw` folder called `RayTriangleIntersection` that you might like to use to store and return all of the intersection details.

A single ray fired into a scene may well intersect with a number of different triangles (e.g. the front of the blue box, the back of the blue box, the back wall of the room etc.). Remember that we are interested in the intersection that is **closest** to the camera (i.e. has the smallest `t`).  


# 
### Task 3: Validating Intersections
 <a href='03%20Validating%20Intersections/animation/segment-1.mp4' target='_blank'> ![](../../resources/icons/animation.png) </a>

We need to be a little bit careful when searching for ray/triangle intersections because the `getClosestIntersection` function can often return false-positive results ! The reason for this is that formulae provided previously allow us to calculate the intersection between a line and a _plane_. We must however remember that the triangle is a discrete bounded segment of a plane. The code that you wrote in the previous task may well find solutions that are _on the same plane_ as the triangle, but not actually _within the bounds_ of the triangle itself (as illustrated by the X in the diagram below).

In order to solve this problem, we must validate the coordinates of any _potential_ intersections before we can accept it as a solution. Watch the video linked to at the top of this task for an explanation of how this is achieved. Once you have grasped the basic principles, implement a validation check in your `getClosestIntersection` function so that it checks a **possible** solution to make sure it is valid _before_ returning it as an **actual** solution. To help you in this task, integrate the following three tests into your code:

```
(u >= 0.0) && (u <= 1.0)
(v >= 0.0) && (v <= 1.0)
(u + v) <= 1.0
```

You should also check that the distance `t` from the camera to the intersection is positive - otherwise you may end up rendering triangles that are actually _behind_ the camera !
  


![](03%20Validating%20Intersections/images/outside-bounds.jpg)

**Hints & Tips:**  
Test your `getClosestIntersection` function by passing it a direction vector with a _known_ intersection point (e.g. the ray that is fired through the dead centre of the image plane _should_ intersect with the front of the blue box in the test model).  


# 
### Task 4: Ray Tracing the Scene
 <a href='04%20Ray%20Tracing%20the%20Scene/slides/segment-1.pdf' target='_blank'> ![](../../resources/icons/slides.png) </a> <a href='04%20Ray%20Tracing%20the%20Scene/audio/segment-1.mp4' target='_blank'> ![](../../resources/icons/audio.png) </a>

Now that you have a function that can identify the closest valid intersection of a single ray, we can use this to render an entire scene ! Write a new `draw` function that renders the Cornell Box model using Ray Tracing. Don't throw away your old "rasterising" `draw` function - you are going to need this later. Instead you should rename it to something appropriate (such as `drawRasterisedScene`).

In your new `draw` method, loop through each pixel on the image plane (top-to-bottom, left-to-right), casting a ray from the camera, _through_ the current pixel and into the scene (remembering to normalise any direction vectors to avoid scaling effects). Using your previously written `getClosestIntersection` function, determine if the ray intersects with a triangle in the model. When a valid intersection has been identified, paint the image plane pixel with the colour of the closest intersected triangle. 

Remember that in this task, you are converting _from_ a 2D canvas pixel position _to_ a direction in three dimensional space. As such you need to do the _opposite_ of your previously written `getCanvasIntersectionPoint` function (where you converted from a 3D vertex position into a 2D canvas position). For this reason, you need to do the same operations, but in _reverse_: subtracting when you previously added, dividing where you previously multiplied. The order which you applied the operations will also need to be reversed ! Clearly this is going to require a fair bit of thinking about !!!

When everything is working properly, you should end up with a render that looks similar to the one shown in the diagram below.  


![](04%20Ray%20Tracing%20the%20Scene/images/basic-ray-tracing.jpg)

**Hints & Tips:**  
At this stage, it probably doesn't seem like you have made any progress - if anything, this is a _slower_ way of getting the same results as with the rasteriser. In the following tasks however (and next week as well), we will be doing things that are either hard or impossible to do with a rasteriser - so stick with it, this is all going to be worth it !  


# 
### Task 5: Casting Shadows
 <a href='05%20Casting%20Shadows/slides/segment-1.pdf' target='_blank'> ![](../../resources/icons/slides.png) </a> <a href='05%20Casting%20Shadows/audio/segment-1.mp4' target='_blank'> ![](../../resources/icons/audio.png) </a>

Shadows are a key element of 3D rendering that we have until this point not addressed. We see shadows all the time in the real world (just take a look around you now !). If we want our renders to look realistic, we are going to need to simulate them.

Determining whether or not a particular point on a surface should be drawn in shadow is conceptually relatively straight-forward. All we have to do is to ask ourselves the question: can a particular point "see" the light ? However, there are some additional complexities that we have to deal with when implementing a consistent shadow effect. Review the slides and audio narration above and then implement the concepts in your Ray Tracer.

It is worth noting that your already-implemented `getClosestIntersection` function does a lot of the work required in order to check for the visibility of the light from a point on a surface. You may however need to invest a bit of time refactoring it to make it versatile enough to be used for this purpose.

Once you have implemented a shadow feature, your render of the Cornell box should look something a little bit like the image below (depending on where you position your light source !). Note that your shadow may well look a little bit "speckly" and you might have shadows appearing where you wouldn't expect to see them. Don't worry - this is all to be expected - we will fix this in the following task !   


![](05%20Casting%20Shadows/images/without-ambient.jpg)

**Hints & Tips:**  
You are going to need a new `vec3` variable to hold the position of a single-point light source. A location in the middle of the room, above the origin, somewhere near the ceiling (but still inside the room) would seem a sensible place for it.

You will probably run into a few problems creating a consistent and complete shadow. It is worth reading the following section now (on "Shadow Acne") in case this helps you in completing the basic shadow task.  


# 
### Task 6: Shadow Acne
 <a href='06%20Shadow%20Acne/slides/segment-1.pdf' target='_blank'> ![](../../resources/icons/slides.png) </a> <a href='06%20Shadow%20Acne/audio/segment-1.mp4' target='_blank'> ![](../../resources/icons/audio.png) </a> <a href='06%20Shadow%20Acne/audio/segment-2.mp4' target='_blank'> ![](../../resources/icons/audio.png) </a>

When implementing your shadow you may notice that is has a "speckled" appearance and/or spills out of the area where the shadow should appear (as illustrated in the image below). This effect is known as "shadow acne" and it a commonly encountered phenomenon in ray traced shadows. View the slides and audio narration linked to above for a discussion of the cause of this effect and an explanation of one solution to this problem. Once you have gained this understanding, add some additional code to your project to remove this phenomenon from your render.  


![](06%20Shadow%20Acne/images/center.jpg)

**Hints & Tips:**  
If you need to compare two triangles for equality, make sure you use their **index** values as the basis for comparison (i.e. their positions in the triangle array). If you try to compare two instances of the `ModelTriangle` class using the `==` operator, you may inadvertently be testing the memory address of those variables (rather than checking to see if those two variables actually hold the same triangle !)

The `RayTriangleIntersection` class has an attribute called `triangleIndex` that can be used to store the index number of a triangle being intersected. Note that you'll have to fill this attribute yourself with a suitable value when your `getClosestIntersection` function detects a valid intersection.   


# 
### Task 7: Interactive Renderer Switching


As a final task in this workbook, add some additional key event handlers to your program that allows the user to interactively switch between the three main modes of rendering:

- Wireframe: Scene drawn using simple stroked/outline triangles
- Rasterised: Scene drawn using filled rasterised triangles
- Ray Traced: Scene drawn using filled ray traced triangles

Being able to switch between these different modes will prove very useful later on in the unit when attempting manual interactive testing: it will allow you to navigate the camera around the scene using the FAST wireframe or rasterised renderers. Then, once the camera is in the desired position, you can switch to the (much slower) Ray Traced renderer to view the fully lit scene from the current position.  


# 
### End of workbook

Make sure you push all your work to your GitHub repository !<br>
Use of GitHub is a key development skill.<br>
It will allow us to keep track of your progress (and we will use it in the final marking process).