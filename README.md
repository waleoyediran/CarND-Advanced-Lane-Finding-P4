## Advanced Lane Finding

### The goal of this project is to write a software pipeline to identify the lane boundaries in a video. 

---


The steps taking to achieve this are listed below:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/undistortion-step.png "Undistorted"
[image2]: ./output_images/distortion-corrected-test.png "Road Transformed"
[image3]: ./output_images/binary-image.png "Binary Example"
[image4]: ./output_images/perspective-transform.png "Warp Example"
[image5]: ./output_images/lane-pixel-polynomial.png "Fit Visual"
[image6]: ./output_images/lane-plot.png "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third code cell of the IPython notebook located in "./P4.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (in the 8th code cell of the IPython notebook located in "./P4.ipynb").  Here's an example of my output for this step.
I used the S-channel (threshold 160-255) in the HLS color space and the x-gradient (threshold 20-100) to generate the binary image. 

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is a function called `getPerspectiveTransform()`, in the 9th code cell of the IPython notebook). 
I chose hardcoded source and their corresponding destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 190, 720      | 190, 720      | 
| 548, 480      | 190, 0        |
| 740, 480      | 1130, 0       |
| 1130, 720     | 1130, 720     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the Sliding Window method to search for lane lines in code cell. 
We basically divide an image into N windows and for each window, we calculate the histogram and use two prominent peaks as the left and right lane.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of curvature in code cell 13 in the function `radius_of_curvature()`
The function returns the radius of the left lane, right lane and the offset of the car to the lane center

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `process_image_final()` in lin code cell 14 of ny notebook.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The problem I encountered was the variations in Road conditions. Getting a binary image that works in all lighting and road textures is quite challenging.

I settled for using a combination of the S-channel in the HLS color space and the X gradient of the image. I settled on a threshold after trying various combination.

I also applied sanity checks (`line_sanity_check` in code cell 13) to check for the sanity of a line. I discarded lines that are not acceptable, and 
allows the algorithm to use lines from the previous frames. This approach will not be effective where there is a change in direction along side change a lighting and road texture.
  
