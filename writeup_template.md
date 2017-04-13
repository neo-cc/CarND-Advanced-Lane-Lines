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
[image0]: ./camera_cal/calibration1.jpg "Original Camera" 
[image01]: ./output_images/undistorted/camera/calibration1.jpg "Calibration"

[image1]: ./test_images/test5.jpg "Test Image"
[image11]: ./output_images/undistorted/test5.jpg "Road Transformed"
[image12]: ./output_images/threshold/test5.jpg "Binary Example"
[image13]: ./output_images/warped/test5.jpg "Warp Example"
[image14]: ./output_images/stage2/test5.jpg "Fit Visual"

[video1]: ./project_video.mp4 "Test Video"


## [Rubric Points](https://review.udacity.com/#!/rubrics/571/view) 
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cell [1] of the IPython notebook located in "P4.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function (code cell [2]).  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Original Camera][image0]
![Calibration][image01]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Test Image][image1]

After distortion-correction, the image look like this:
![Road Transformed][image11]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (code cell [4], the code is from Udacity Course directly).  Here's an example of my output for this step. 

![Binary Example][image12]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`(code cell [6]) .  The `warp()` function takes as inputs an image (`img`).  I also applied a image mask to mask out outer region of the warped image. The code is from project1 (code cell [8]).

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 150, 720      | 300, 720      | 
| 490, 450      | 300, 0        |
| 690, 450      | 980, 0        |
| 1130, 720     | 980, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image13]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I defined finding_lines() function to detect the lines. The code is from [Finding the Lines](https://classroom.udacity.com/nanodegrees/nd013/parts/fbf77062-5703-404e-b60c-95b78b2f3f9e/modules/2b62a1c3-e151-4a0e-b6b6-e424fa46ceab/lessons/40ec78ee-fb7c-4b53-94a8-028c5c60b858/concepts/c41a4b6b-9e57-44e6-9df9-7e4e74a1a49a) (code cell [10]). This function slices the picture into 9 windows, then find the peak of the left and right halves of the histogram for each window, then step through the windows one by one. Later, it extracts left and right line pixel positions, fit a second order polynomial to each line and then return the polynomial.

If the lines are detected in previous frame, I will use update_finding_lines() to detect lines in new frame (code cell 11). This function takes input as image, and fitted second order polynomial in previous image. 
Once the lane lines are found in one frame of video, I don't need to search blindly in the next frame. I simply search within a window around the previous detection.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code cell [13] set_radius_of_curvature(). 
The methed to calculate curvature is from [Measuring Curvature](https://classroom.udacity.com/nanodegrees/nd013/parts/fbf77062-5703-404e-b60c-95b78b2f3f9e/modules/2b62a1c3-e151-4a0e-b6b6-e424fa46ceab/lessons/40ec78ee-fb7c-4b53-94a8-028c5c60b858/concepts/2f928913-21f6-4611-9055-01744acc344f).


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in drawing() (code cell 14). This function takes original image and fitted curved lines as input, then draw the lane onto the warped blank image, and then warp the blank back to original image space using inverse perspective matrix (Minv). At last combine the result with the original image. 

![alt text][image14]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video/project_video_output.mp4).

For "processing challenge_video.mp4", here is the [result](./output_video/project_video_output_chanllenge.mp4).

For "processing project_video_output_harder_challenge.mp4", here is the [result](./output_video/project_video_output_harder_challenge.mp4).

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

I implemented sanity_check in Line() class (code cell [13]), which checked if the new image fitted polynomial is close to previous image's polynomial. If there are different a lot (>500%), the Line.detected flag is set to false. 

I also checked set_line_base_pos to see if the car is far off the center. If the car is 60cm away from center,  the Line.detected flag is also set to false.

If Line.detected flag is false, I will reset the Line class and do the line search using finding_lines() with the whole image. 

I also do smoothing to average over the last n frames of video to obtain a cleaner result. Each time I get a new high-confidence measurement, I append it to the list of recent measurements and then take an average over n past measurements to obtain the lane position.

There is not too much issue to process the "project_video.mp4" video. The road is consistant and there was no big variance for the fitted line. The video result looks ok.

The current algorithm is NOT good at processing challenge or harder challenge video. It's even hard to detect lines at the beginning. The finding_lines and sanity check functions need to be improved if given more time. 






