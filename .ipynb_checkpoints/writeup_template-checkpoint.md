
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

[image1]: ./output_images/undistorted.png "Undistorted"
[image2]: ./output_images/undistorted2.png "Undistorted2"
[image3]: ./output_images/combined_binary.png "Binary Example"
[image4]: ./output_images/warped.png "Warp Example"
[image5]: ./output_images/window.png "Fit Visual"
[image6]: ./output_images/project_back.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup

#### Provide a Writeup / README that includes all the rubric points and how you addressed each one.

### 1. Camera Calibration

#### Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./porject.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained the unditorted image on the right side, the left side is the original image: 

![image1]

### 2.Pipeline (single images)

#### 2.1. Provide an example of a distortion-corrected image.

Use the parameters calculated from the last step, I apply cv2.undstort() funciton on a test image, and get the following result:

![alt text][image2]

#### 2.2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

On this step, I tested many methods to binarize an image: simple threshold on grayscale image, R,G,B channel threshold, H,L,S channel threshold, gradient threshold and so on.  
The 'gradient_channel_combined()'function combines the gradient in x direction and S channel thresholds, it gives a prety good result at lane lines detection.

Here's an example of my output for this step. The stacked image shows gradient result as green and S channel result as blue.

![alt text][image3]

#### 2.3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the first cell of 2.3 View transform. 
The `warper()` function takes an image (`img`) as input, as well as source (`src`) and destination (`dst`) points.  I chose the source and destination points as the following:


| Source        | Destination   | 
|:-------------:|:-------------:| 
| 563, 470      | 300, 100      | 
| 720, 470      | 1000, 100     |
| 1120, 720     | 1000, 720     |
| 190, 720      | 300, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image, after some adjustment, the lines appear parallel in the warped image.

![alt text][image4]

#### 2.4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then the next step is detecting the lane lines and fit them with second order polynomials. The 'find_lane_points()' function is designed to do this job. It takes the binirized and warped image as input. Uses the histogram of the bottom half of the image. Finds the two peaks on left and right side as the starting point for the two lines. Then uses sliding windiow to search for non-zero pixels. Based on the detected non-zero pixels, fits two second order polynomials to represent the two lines.

![alt text][image5]

#### 2.5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To calculate the radius of curvature on the bottom of the image, the equation is R = ((1+(2Ay+B)^2)^(3/2))/∣2A∣. The code is:
``` python
y_eval = np.max(ploty)
left_curverad = ((1 + (2*left_fit[0]*y_eval*ym_per_pix + left_fit[1])**2)**1.5) / np.absolute(2*left_fit[0])
right_curverad = ((1 + (2*right_fit[0]*y_eval*ym_per_pix + right_fit[1])**2)**1.5) / np.absolute(2*right_fit[0])
```

Assume the lane width is 3.7 meter, the vehicle is in the center of of the image. The x-value of the last points of each polynomial is the lane-lines position. We can compare the middle of the lane area with the position of the vehicle to calculate the desired result.

#### 2.6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function 'draw_lane()', it fills the lane area with green color on the warped image, transforms the view back, and then combines it with the original image. 

Here is an example of my result on a test image:

![alt text][image6]
Fot this image:
Left curve: 2478.20 m Right curve: 2304.90 m
Position : 0.05 m left of center


### 3. Process video

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The 'process_video()' function process the first frame of a video, and the 'next_frame()' function process the rest frames. The 'process_video()' function use the same step as described before on single image. The 'next_frame()' function search for non-zero pixels within a range based on the polynomials of the last frame. 'put_text()' function add radius of curvature and vehicle position information onto the video frames.

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I think the core of this lane detection process is to binarize the warped image. Fit the lines is just a easy step based on the binarized images. As can be seen, I tried many methods and combinitions of them, none is rubust throughout the whole video. The final choice still get a slight problem at 24th and 42th second. The lane-lines on warped images are almost in the vertical direction all the time, sobel calculator on x direction should have good results, the yellow lins are clearly separated with the background on S channel. The combinition of these two method works better than other methods I tried, but when applied it on the challenge video, it became a mess. There must be better solutions to create robust binary images, however testing different parameters and methods and their combinitions took too much time. If I happen to work in this area in the future, I'll make further test.

---
update: use hsv thresholds as seggested, the binarization result is much better.

