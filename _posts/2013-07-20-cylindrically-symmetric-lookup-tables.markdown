---
layout: post
date: 2013-07-02 17:18
title: Efficient Indexing into Cylindrical or Rotation Symmetric Functions
published: false
---

So, you have found yourself modeling a cylinder or something with rotation symmetry.  Now the challenge is, "How do I index into it in an efficient manner?"  Thankfully, this is an advantageous situation.  Cylindrically symmetric functions can simply be modeled as planes.  This reduces dimensionality and memory requirements.  This is best demonstrated in the examples below.  Let's start with an even simpler case.  Turning a 2D rotation symmetric function into a 1D line function.

## Turning a Circle into a Line ##

### Step One ###
Take a cylindrically symmetric function.

![Cylindrically symmetric function](https://dl.dropboxusercontent.com/u/16585299/blog_images/cylinder_target_step1.png)

### Step Two ###
Identify the axis of symmetry.

![Cylindrically symmetric function with axes](https://dl.dropboxusercontent.com/u/16585299/blog_images/cylinder_target_step2.png)

### Step Three ###
Take a unit slice of the function along the axis of symmetry.

![Axial slice of cylindrically symmetric function](https://dl.dropboxusercontent.com/u/16585299/blog_images/cylinder_target_step3.png)

### Step Four ###
Now rotate your slice 360 degrees (2 pi) about the central axis.  If you recover your original circle, you are good to go!

![](https://dl.dropboxusercontent.com/u/16585299/blog_images/cylinder_target_step4.png)
![Cylindrically symmetric function](https://dl.dropboxusercontent.com/u/16585299/blog_images/cylinder_target_step1.png)

## Turn a Cylinder into a Plane ##
Taking one step up the dimensional ladder we can see how a cylindric function can be represented not by rotating a line, but by rotating a plane.

### Step One ###

### Step Two ###

### Step Three ###

### Step Four ###

### Step Five (maybe) ###
It is worth noting that if your plane from *Step Three* does not vary with depth, then you can reduce your problem even further to the 2D circle case.  Congratulations, you have dramatically simplified your problem.  Proceed directly to *Step One* of the section *Turning a Circle into a Line*!  

# The Real World is 3D #
The world we live in is 3D.  We have a 3D function and have reduced it to 2D by taking advantage of symmetry.

### Setting up the problem ###
Assume we have a camera (or whatever defines the center of our cylinder) whose position is represented in cartesian coordinates [x,y,z] using a variable `cameraPosition` and said camera is pointing in the direction represented by a variable `cameraDirection`.  Assume the location we are interested in sampling is the center of a voxel at position `voxelPosition`.
The following code illustrates how to efficiently index into our line using C++ and [Eigen (a fantastic library)](http://eigen.tuxfamily.org/).  Please note that as directly implemented this function may not be in its most computationally efficient form.  It is assumed that we have given the compiler enough leeway to optimize memory usage and instruction ordering when optimization is turned on.

    Eigen::Vector3f voxelPosition;
    Eigen::Vector3f cameraPosition;
    Eigen::Vector3f cameraDirection;
    voxelPosition -= cameraPosition;
	const float distance = voxelPosition.norm();
	const float dotValue = voxelPosition.dot(cameraDirection);
	// ^^ if the distance evaluates to 0 it will be caught in the next iff because dot'ing a vector whose norm is 0 with another vector will produce 0
	// ^^ since negative results are not possible, simply check for distance == 0 case
	if ( (dotValue > distance) || (dotValue <= 0.0) ) //here instead of distValue > 1.0 the other comparison is used to delay the division operation
	{
		return 0.0f;
		// ^^ zero is returned here because either
		// a: the angle is greater than or equal to pi/2 which we consider zero (if your camera has a field-of-view less than 180 degrees then you can change this value to reflect that)
		// or b: the diff_position == 0 in which case if the distance is almost zero, then the cell is essentially
		// on top of the camera, where we assume the value is zero.
	} else
	{
		const float xCoordinate = sqrt(abs(distance*distance-dotValue*dotValue));
		const float yCoordinate = dotValue; //dotValue is the y coordinate and guaranteed to be positive by the encapsulating if
		if ( (yCoordinate > m_computedYBoundary) || (xCoordinate > m_computedXBoundary) )
		{
			return 0.0f;
			// ^^ the target voxel is outside the lookup table, we consider this to be zero
		} else
		{
			return m_responseLineFunction->getValueXY(xCoordinate, yCoordinate);
		}
	}