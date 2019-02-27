# Advanced Lane Finding_Project02

## Udacity Self driving engineer nano degree program - Project 01 - Finding Lane Lines

### In this project, lanes on the road have been identified by following as series of techniques such as, Distortion correction, Color and Gradient thresholding, Second order polynomial fit and checking other lane conditions. The goal of this project is to develop a software pipeline to identify the lane boundaries and plot it on a video stream.

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

[image1]: ./output_images/UndistortedCalImage.png "UndistortedCalImage"
[image2]: ./output_images/UndistortedImage.png "UndistortedImage"
[image3]: ./output_images/ColorThreshold.png "ColorThreshold"
[image4]: ./output_images/Combined.png "Combined"
[image5]: ./output_images/Totcombined.png "Total Combined"
[image6]: ./output_images/warped.png "warped"
[image7]: ./output_images/sliding.png "sliding"
[image8]: ./output_images/searcharound.png "searcharound"
[image9]: ./output_images/drwaonimage.png "drawonimage"

[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/AdvancedLaneFidning_P02.ipynb" in my Github repo.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The below image shows the original and the undistorted version of the projects lane image after applying 'cv2.undistort()' using the previously calculated camera matrix and distortion coefficients. 
![alt text][image2]

The effect of undistortion is subtle but can mainly observed at the edges. 

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. 

Below are the resulting images from Color Thresholds that I liked the most:

![alt text][image3]

Further a combination absolute sobel, Magnitude and Directional gradient thresholds are used to generate binary images and below is a example of the binary image from this combined gradient thresholding:

![alt text][image4]

Below is a side by side comparision of the Thresholds I have chosen so far:

![alt text][image5]

Finally upon careful obsevation, Just the R channel binary yielded much better visuals with minimal noise and has been chosen for thresholding with values between 200 and 255.

The code for this section is found in Step3 of my Jupyter notebook "./examples/AdvancedLaneFidning_P02.ipynb" in my Github repo(lines 7 through 12).

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a custome function called `birds_eye_view()` which appears in Step4 in the Jupyter notebook(line 14).  The `birds_eye_view()` function takes as inputs an image (`img`) and warps it based on source and destinatin points.  I chose the hardcode the source and destination points in the following manner:

```python
    offset = 400
    l_upper_corner  = [568,470]
    r_upper_corner = [717,470]
    l_lower_corner  = [300,650]
    r_lower_corner = [1040,650]
    
    src = np.float32([l_upper_corner, l_lower_corner, r_upper_corner, r_lower_corner])
    dst = np.float32([[offset,0], [offset,650], [img_size[0]-offset,0], [img_size[0]-offset,650]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 568, 470      | 400, 0        | 
| 717, 470      | 900, 0        |
| 300, 650      | 400, 650      |
| 1040, 650     | 900, 650      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a series of test images and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

'find_lane_pixels' and 'search_around_poly' functions have been used inorder to identify lane-line pixels and fit their positions with a second order polynomial.

'find_lane_pixels' function uses a sliding window method to identify the lane line pizels. Initially a histogram is genereated where we values along each column in the image are added up. In our thresholded binary image, pixels are either 0 or 1, so the two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. This is used as a starting point for where to search for the lines. From this point,  a sliding window, placed around the line centers is used, to find and follow the lines up to the top of the frame. A total of 9 windows with a width of 100 are used to find the lane pixels. In each iteration the windows are recentered based on the concentration of pixels points with a value =1. Below image shows the implementation of thie sliding window approach on a series of test frames.
![alt text][image7]

This approach is used for the initial frame and further frames use 'search_around_poly' function that would search in a margin around the previous line position, like in the below image. 
![alt text][image8]

The green shaded area shows where we searched for the lines this time. So, once you know where the lines are in one frame of video, you can do a highly targeted search for them in the next frame.

Code section: Under Step5 - Same Jupyter notebook

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In the last section,lane line pixels have been located, and I used their x and y pixel positions to fit a second order polynomial curve:

f(y)=Ay2+By+Cf     

I am fitting for f(y), rather than f(x), because the lane lines in the warped image are near vertical and may have the same x value for more than one y value.

The radius of curvature  at any point x of the function x=f(y)x = f(y)x=f(y) is given as follows(Reference Udacity Self Driving Car Engineer nano degree program material):

Rcurve=[1+(dxdy)2]3/2∣d2xdy2

In the case of the second order polynomial above, the first and second derivatives are:

f′(y)=dxdy=2Ay+B

f′′(y)=d2xdy2=2A

So, our equation for radius of curvature becomes:

Rcurve=(1+(2Ay+B)2)3/2∣2A∣

The y values of my image increase from top to bottom, so if, for example, if I wanted to measure the radius of curvature closest to the vehicle, the above formula could be evaluated at the y value corresponding to the bottom of your image, or in Python, at 'yvalue = image.shape[0]'.

I have further done conversion to have a representation in meters as below:
```python
    ym_per_pix = 30/720 # meters per pixel in y dimension
    xm_per_pix = 3.7/700 
```

Below are the results of a sample frame:
Left_curvature 1730.03700072
right_curvature 1917.67787259
Center Vehicle is -0.03m to the left

Code section: Under Step6 - same Jupyter notebook.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in Step7 in my code in the function `draw_on_image()`. Left line and right line curvature as well as the vehicles offet from the center of the lane have been drawn on the image along with a polygon filled in 'green' to show the indentified lane area in that frame.
Here is an example of my result on a test image:

![alt text][imag9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./result_video/project_video_output.mp4)
[![link to my youtube video of result](https://img.youtube.com/vi/1eM0_rFIre4/0.jpg)](https://www.youtube.com/watch?v=1eM0_rFIre4)


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Most time has been spent in visualizing video frames after applying the color and gradient thresholding techniques to see which method yields a good image in general with minimal noise to be able to properly detect the lanes. Also, for the sake of warping and applying sliding window techniques, a specific polygon(area on the lane) has been selected manually after observing the overall video stream. This is not the best way, as objects like other cars crossing paths within these boundaries will completely ruin the lane identification. Also, in extreme weather conditions like snow and rain, other thresholding appraoches might have to be used and the R channel binary might not be very generic to cover all these conditions.

Another very important factor to consider is lane changing which is not tested with the current appraoch where we have the video stream for a single lane. A more robust appraoch would be when these various other conditions such as weather, objects crossing paths and lane changing can be taken into account and somehome bringing in predictive models inorder to implement actual dynamic behviour of vehicles and humans on roads.


Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
