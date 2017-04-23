## Writeup

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/undistorted_calibration_output.png "undistorted_calibration_output"
[image2]: ./output_images/undistorted_straight_lines2.png "undistorted straight_lines2"
[image3]: ./output_images/undistorted_test6.png "undistorted test6"
[image4]: ./output_images/filter_color_grad.png "color and gradient filter"
[image5]: ./output_images/filter_mag_dir.png "magnitute of gradient and gradient direction filter "
[image6]: ./output_images/filter_combined.png "combined binary filter"
[image7]: ./output_images/perspective_transform.png "binary filter transformed in perspective view"
[image8]: ./output_images/lines_detected.png "detected lines"
[image9]: ./output_images/test6_lane.png "lane line"

[video1]: ./project4_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
The code was based on the code provided by Udacity lessons. All the code is available in a Ipython notebook named [advanced_lanes](https://github.com/lcfgr/CarND-Advanced-Lane-Lines/blob/master/advanced_lanes.ipynb). It is very comprehensive with titles of all the parts contained in this writeup along with the code used for the creation of all the images presented here in this writeup.
--- 
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.

You're reading it!

### Camera Calibration

#### 1. The code used for the Camera calibration is available in the 2nd cell.
  
I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to two of the test images:   
   
One were the car is going straight:  

![alt text][image2]

One while the car starts to turn right:   
   
![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

A large set of experiments was made to determine the best possible combination of filters used to generate a binary image.  
All the filter functions are available in the 3rd cell of the Ipython notebook. The function that combines the finally chosen combination of filters is named **detect_lines**. The final filter has the following combination of 2 subfilters:   
   
The first filter is an OR combination between the color filter, which selects white and yellow pixels in the image and an AND filter of gradients of x and y. The second filter is an AND combination between the Magnitude of the Gradient Filter and Direction of the Gradient. These two filters are combined into the final filter with OR.   
   
Below we can see the results of the first filter (color filter and x,y gradient):   
   
![alt text][image4]
   
Below we can see the results of the second filter (magnitute of gradient and gradient direction):   
![alt text][image5]
   
Here is the combination of all the filters used to create the binary image:   
![alt text][image6]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is visible in the 4th notebook cell, in a function called **perspective_transform**.The transformation matrices were manually selected.  
I decided to transform the binary image and create a wraped binary image to feed into the line detection algorithm. The output of the image can be seen below:   

![alt text][image7]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code used to detect the lane-line pixels, based on Udacity lecture code, can be found in the 5th and 6th notebook cells. The two functions used are **fit_lines** and **fast_fit_lines**. The fast version of the function does not scan for the lines from scratch but only to a pre-defined window around the initially detected lines. I also included the radius of curvature of the lane and the position of the vehicle in these functions, since it allowed easier implementation.   

The polynomial lines fitted for the above test image can be seen below:   

![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

As mention above, this is done also into the 5th and 6th notebook cells, within the **fit_lines** and **fast_fit_lines** functions. The radius of curvature is calculated as proposed in the Udacity lectures. To calculate the offset of the center of the lane the following approach is used, with the assumption that the camera is fitted in the center of the vehicle:   
The 10% of the detected left and right pixels that are near the bottom of the image are used. They are averaged, to create a better assessment of the left and right lane position. Therefore we can calculate the lane width. Then the offset in pixels is calculated, by substracting the center of the image from the center of the lane). In the end the real offset in meters is calculated, thanks to the ratio provided by Udacity (3.7 / 700 # meters per pixel in x dimension).   

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 7 of the notebook, with the function **draw_lane**.   
Here is an example of my result on the test image:

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's the [video](./project4_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

The approach seems robust in very good conditions such as that presented in the project. However there are several problems where the algorithm would not work well. These problems include:   

* Different/variable camera distortion   
* Difficult lighting conditions (shadows, reflections, etc)   
* Dirty camera/windshield   
* Other moving vehicles and/or objects entering the lane   
* Worn out /missing color lines

One approach to improve the performance would be, as proposed by Udacity, to average all the calculations done within multiple frames and recalculate from scratch when a sanity check fails. This approach can be also problematic, when there are many opposite turns in a row, so the number of frames used for averaging can be also variable and adaptive.  Also deep learning algorithm can be used to adapt the thresholds of the various functions, along with propabilistic based calculations. Furthermore, a auto-calibration algorithm can be implemented to calculate the camera distortion and the pixel to meter ratio, when comparing the images with known shapes and sizes (i.e. the sizes of traffic signs, the length of the discontinous lines, markers on the roads, etc). Last but not list, other sensors such as LIDAR can be used to measure distances and acceleration sensors to calibrate data related with the movement (speed/curvature,etc).
