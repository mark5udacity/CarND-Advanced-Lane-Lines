## Writeup Template

### This here, modified, writeup template is used to address each rubric point. 
---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

1) Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
2) Apply a distortion correction to raw images.
3) Use color transforms, gradients, etc., to create a thresholded binary image.
4) Apply a perspective transform to rectify binary image ("birds-eye view").
5) Detect lane pixels and fit to find the lane boundary.
6) Determine the curvature of the lane and vehicle position with respect to center.
7) Warp the detected lane boundaries back onto the original image.
8) Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./writeup/good_undistort.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./writeup/good_color.png "Binary Example"
[image4]: ./writeup/transformed.png "Warp Example"
[image5]: ./writeup/lane_lines_warped.png "Fit Visual"
[image6]: ./writeup/example_output.png "Output"
[image7]: ./writeup/bad_example_thresh.png "Bad Thres Example"
[image8]: ./writeup/MagDirGradient_bad_example_test4_orig.png "Bad Gradient Example"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

#### > For brevity, the Jupyter notebook containing all my code has been labelled accordingly with the goal sections as described above.

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the a code cell of the IPython notebook located in "./AdvancedLaneFinding.ipynb" with the heading (1).

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result, 
^^ This is from original template, below contains my statement:

with a side-by-side comparison to original to see how well it does: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used solely color-based thresholds to generate a binary image (as can be found in Step 3 on color-based sorting.  I originally tried to come up with a magnitude and gradient thresholding that could be ANDed with the color threshold, but ran into too many problems related to shadows, as discussed later.  In addition, on the forums and slack, many people made comments that they soley used color-based thresholding to pass the challenge.

Here's an example of my output for this step.  

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `transform()`, which can be found in Step 4 of the Jupyter Notebook `AdvancedLaneFinding.ipynb` (AdvancedLaneFinding.ipynb).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.
  
  I originally tried to make the transformation agnostic to image size, but then suspected it was causing some problems for me so I picked hardcoded values as found in some discussion and properly linked and pointed to, with a little bit of narrowing on my part to avoid extra lanes. 


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the sliding window approach as outlined in the lectures and fitted my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did what can be found in Step 6 of the Jupyter Notebook `AdvancedLaneFinding.ipynb` (AdvancedLaneFinding.ipynb), which was a straightforward application of the lecture videos.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in Step 7 of the Jupyter Notebook `AdvancedLaneFinding.ipynb` (AdvancedLaneFinding.ipynb).  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output/challenge_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  


As mentioned briefly earlier, my original goal was to have an excellent thresholding that could then be ANDed with the color threshold to get good results.  What I discovered is that the gradient-based thresh techniques do very poorly in the shadows cast by trees and bridges.  

Here's a video on the extra hard challenge, before I had implemented average-lane smoothing and curve-rejection, the thresholding with colors did better than the colors-alone, at least subjectively (at least I tried!)
[link to my video result](./output/harder_challenge_video.mp4)


As mentioned in the forums and elsewhere, the video we needed to pass can be done with just color thresholding.  I discovered that wasn't entirely true, and the 'optional' parts of rejecting bad lane lines basically had to be implemented to handle going under the bridge, which even a human couldn't see.  

It makes sense some basic memory is retained about where lanes are.  The primary reason I abandoned using magnitude gradient thresholding (if I did use a gradient-based threshold, I had dropped all but direction), was because it fell apart in very bright conditions for bright roads with bright lane-markers.   These result in less strong gradients, even as I adjusted my sobel kernel size. 

![alt text][image8]


In the end, I abandoned gradient-based thresholding completely, because the challenge video in the beginning had a line-lane that was painted over, which caused a gradient different in the image, but did not match the color.  It is not clear to me how useful this will be, but this does expose a challenge for computers, and even humans, as these two great following examples succintly show:
 
 [Kramer paints over lines](https://twitter.com/Seinfeld2000/status/912003492041990144)
 
["Good luck" self-driving cars](https://twitter.com/mat_kelcey/status/904551257808953344)

Anyway, after hours spread-out over days of adjustments, the best my threshold/color techniques could do was the following, which had a bad lane line indeed.

![alt text][image7]


Some final thoughts on where to improve from here, I can implement a parallel check for left/right lane lines, adjust the warped points better perhaps and think more about how gradient-based thresholding could be brought in to supplement the color selection.
