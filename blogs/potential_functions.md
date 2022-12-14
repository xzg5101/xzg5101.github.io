# Object Motion with Potential Vectors

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/katex.min.css" integrity="sha384-yFRtMMDnQtDRO8rLpMIKrtPCD5jdktao2TV19YiZYWMDkUR5GQZR/NOVTdquEx1j" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/katex.min.js" integrity="sha384-9Nhn55MVVN0/4OFx7EE5kpFBPsEMZxKTCnA+4fqDmg12eCTqGi6+BB2LjY8brQxJ" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/contrib/auto-render.min.js" integrity="sha384-kWPLUVMOks5AQFrykwIup5lo0m3iMkkHrD0uJ4H5cjeGihAutqP0yW0J6dpFiVkI" crossorigin="anonymous" onload="renderMathInElement(document.body);"></script>

### Guo Xiaoyang, 2022 Nov.29

## Table of content

- Preparation
- Math Walkthrough
- Preditor-prey Model
- Making Creatures More Vivid

## Before You Start

Before reading this blog, make sure you understand how to find the gradient of a function. Also make sure you are comfortable working with 3D space. If you wish to fully understand the forth section, it would be a good idea to review quaternions or Euler rotations before you reach there. Both are fine, but this article will focus on quaternions.

## Math and Theory Behind Potential Functions

### A Naive Point of View

The motion of an object is defined by its velocity vector, which has a direction and a speed. In the nature, a creature may rest, stroll, chase or decamp. This section will focus on chasing and decamping, which are essentially the same mechenism in terms of implementation, with only opposite dircections. Stroll will be discussed in last section.

To make a object chase a target, the most intuitive method would be finding the vector from this object to the target, and using this vector to be the velocity——which works, but have some issues. <br />
<img
    src="../media/potential_vector/img01_object_to_target.png"
    alt="Vector from the object to the target"
    height="150"
  />

First, the object will become slower as it get closer to the target. This may be desired in some specific situations, but is definetly not suitable for all situations. This issue can be addressed by normalize the vector and mutiply with a speed value. As you would see, no matter what method we use to compute the velocity vector, this operation will be preserved. Secondly, this method cannot really deal with situations where multiple target present in the space. The natural solution is to simply add up all the vectors from the object to each target. However this is not really a generic solution. For example, if there are two targets, then the object will move to the middle of the two target and stop there no matter where it begins. Once again, this is not suitable for most situations.<br />

<img
    src="../media/potential_vector/img02_two_targets.png"
    alt="Cannot handle two targets"
    height="150"
  />

So the task in front of us is to avoid equilibriums as much as possible. One method is to make the closer targets more "attractive" to the target. If this is true, then the object will go to not the middle point between the targets, but the closer target instead, which is desirable for many situations. How do we do this? Remember we metioned that without normalization, the velocity vector will become smaller and smaller as it approach the target, which means the inverse become larger and larger. Furthermore, by increase the power of the dinominator, the attraction of closer targets will be higher and higher compare to the targets that are further away from the object. Keep this in mind when you try to work out the math in next part of this section.

### Potential Functions and Gradient Descent

With the conceptual understanding, let's take a look at the math. As we mentioned in last part, by putting the distance from the object to the targets in the denominator, we can make the target that is closest to the object be more attractive. However, we want a more generic methodology such that we can deal with all sort of situations. We call all functions that define the objects' reaction on target potential functions. Potential functions can decide a lot of things. They could be attracted or repelled by the target, and if they are attracted, they may prefer some of the targets over others based on some properties, etc. Both functions we mentioned in last part are potential functions, and they perform differently. In this artical, the only property we care about is the distance from the object to the targets, thus we can define potential functions in a specific way: **A potential fucntion is a function that inputs the positions of the object and the target, and returns the desired behavior of the object.** Notice this is not a formal defination of potential function. The name is called "potential functions" because these functions do look like and perform like real world potential functions in terms of attraction and repulsion, such as gravity or magnetic fields. Our job is to figure out the desired motion pattern of the object when target presents, and design potential functions that meet the requirement. When multiple targets present, the overall potential should be obtained by simply add all the potential vectors from each target.

After designing potential functions, we need to figure out how to use them. The goal is to push the object to approach some equalibrium. Gradient descent is a very commonly used method in machine learning to find the local minimum of a function; our initial purpose is to minimize the distance between the object and the target, so naturally we addapt this concept. The gradient of a function gives the fastest direction to "move" along the fuction that will reach the minimum of the function.

Let's look at an example. Assume that we are in 3D space, and we decide to use the second potential function mentioned in last part. The object is currently locating at $$P_o = (x_o, y_o, z_o)$$, and a target presents at $$P_t = (x_t, y_t, z_t)$$. The vector from the object to the target would be $$P_t-P_o$$, and the distance between them would be $$\lVert P_t-P_o\rVert^2$$. The potential function can be designed as:

$$f(P_o, P_t) = \frac{1}{\lVert P_t-P_o\rVert^2+\epsilon}$$

Where $$\epsilon$$ is a small constant added to avoid divide-by-zero error when the object and the target are at the same position. The gradient of the potential function would therefore be:

$$
\nabla f =
\begin{bmatrix}
\frac{2(x_t-x_o)}{(\lVert P_t-P_o\rVert^2+\epsilon)^2} \\
\frac{2(y_t-y_o)}{(\lVert P_t-P_o\rVert^2+\epsilon)^2} \\
\frac{2(z_t-z_o)}{(\lVert P_t-P_o\rVert^2+\epsilon)^2}
\end{bmatrix}
$$

This gradient gives us the vector that defines the motion of the object. In another word, this gradient vector is the "potential vector" give by the target. Simply change the sign of this vector will make the object attracted or repulsed by the target. When multiple targets present simultaneously, simply summing up all the potentials from all targets gives the overall potential. Also, do not forget to normalize it before you apply it to your creature.

## Preditor-prey Model

We've finished the math induction and ready to look at some implementation wise concepts. One common pattern of motions is a preditor-prey model. The basic ideal is from the nature. In an environment, there are two type of creatures, preditors and preys. A preditor will try to catch the preys, and the preys try not to be caught by the preditors. By simply implementing the methods mentioned above to preditors, and use the negative vector on preys, the output works fine in a infinite space. In every frame, the preditor will compute all the potential vectors from all preys, and sum them up to get the overall motion vector; similarly, the prey will compute all the potential vectors from all preditors, and repeat the same process. When a preditor touched a prey, the prey is caught, and therefore gets eaten and disappears from the environment. A creature can be preditor and prey at the same time, and many creatures together can form a complex food chain.

## Making Creatures More Vivid

Now we have finished the very naive move——the preditor will chase the prey and the prey will try to run away. But if we are to create an aquarium or vavarium, this is far away from success. There are some basic operations you can do to greatly increase the liveliness (or in the contrast, if you do not implement them, the creatures would be very much lifeless).

### "Eyes on The Road"

The most important feature in this section would be make you creature heading the correct direction while it is moving. A crab may look on the side, a Nautilidae will face backward, and most creatures will head toward and moving toward the same direction. This can be done with a quaternion.

### Collision Detection and Walls

It wouldn't make any sense if two creature's body overlaps, so collision detection is necessary. One naive way to implement is to simply comput the the distances from the object to all other creatures, and reflect (usually mirrow about the plane defined by the vector from the object to the creature it is colliding with) the direction of motion when the distance to any other creature is smaller then the sum of the bound of the two objects. This is also known as a spherical hard bound. Spherical means the collision take place at a constant distance, which form a bounding sphere in 3D space; hard bound means the creatures' bounding sphere will not overlap at all. This is not perfect as the creatures will be making instantaneous U turns when their bounding sphere collides. A more sophisticated solution is to once again use a potential function. For every creature, define a relatively large soft bound and a smaller hard bound. When two creatures' soft bound overlaps, a potential function that repulse each other will be applies and therefore the turn will be soft. When the hard bounds overlaps each other (maybe there is a stronger potential from a preditor is pushing them together), treat the hard bound as before. Similar approach can apply to walls if you are working with a finite space (which is true for most cases).

There are more complex way of dealing with collisions, such as bounding boxs, or hierarchical bounding boxs; but I am not going to go into details with them in this article.

### Creature's Periodic Animation

Placeholder.

### Chase/Escape Strategies

Placeholder.

#### Target Preference

Placeholder.

#### Change in Speed

Placeholder.

#### Corner Avoidence

Placeholder.

### Group Behavior

Placeholder.

## Demo

A short demo of author's implementation can be viewed at：

[Link to My Demo](https://www.bilibili.com/video/BV1xg411v71t/)

This is not the final outcome, but very close to it.

(Not Finished)
