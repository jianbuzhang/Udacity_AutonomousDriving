# Self-Driving Car Engineering Nanodegree Program
## Advanced Lane Finding Project

The goals/steps of this project are the following:
- Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
- Apply a distortion correction to raw images.
- 

[//]: # (Image References)

[img01]: ./process_images/01-calibration.png "Chessboard Calibration"
[img02]: ./process_images/02-undistort_chessboard.png "Undistorted Chessboard"
[img03]: ./process_images/03-undistort.png "Undistorted Dashcam Image"
[img04]: ./process_images/04-unwarp.png "Perspective Transform"
[img05]: ./process_images/05-colorspace_exploration.png "Colorspace Exploration"
[img06]: ./process_images/06-sobel_absolute.png "Sobel absolute"
[img07]: ./process_images/11-hls_l_channel.png "HLS L-Channel"
[img08]: ./process_images/12-lab_b_channel.png "LAB B-Channel"
[img09]: ./process_images/13-pipeline_all_test_images.png "Processing Pipeline for All Test Images"
[img10]: ./process_images/14-slide_window_polyfit.png "Sliding Window Polyfit"
[img11]: ./process_images/15-slide_window_histogram.png "Sliding Window Histogram"
[img12]: ./process_images/16-polyfit_by_previous_fit.png "Polyfit Using Previous Fit"
[img13]: ./process_images/17-draw_lane.png "Lane Drawn onto Original Image"
[img14]: ./process_images/18-draw_data.png "Data Drawn onto Original Image"

[video1]: ./project_video_output.mp4 "Video"

---
## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
*Here I will consider the rubric points individually and describe how I addressed each point in my implementation.*

---
### Writeup/README
#### 1.Provide a Writeup/README that include all the rubric points and how you addressed each one. You can submit your writeup as markdown or pdf.[Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.

You're reading it!

### Camera Calibration
#### 1.Briefly state how to compute the camera matrix and distortion coefficients.Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first two code cessl of the jupyter notebook `Advanced Lane Detection Proj.ipynb`.

The OpenCV function `findChessboardCorners` and `CalibrateCamera` are the backbone of the image calibration.A number of images of a chessboard, taken from different angles with the same camera,comprise the input.Arrays of object points, corrseponding to the location(essentially indices) of internal corners of a chessoard, and image points, the pixel locations of the internal chessboard, and image points, the pixel locations of the internal chessboard corners determined by `findChessboardCorners`, are fed to `calibrateCamera` which returns camera calibration and distortion coefficients.These can then be used by the OpenCV `undistort` function to undo the effects of distortion on any image produced by the same camera.
Generally,These coefficients will not change for a given camera(and lens). The below image depicts the corners drawn onto twenty chessboard images using the OpenCV function `drawChessboardCorners`:

![alt text][img01]

*Note:Some of the chessboard images do not appear for the reason that `findChessboardCorners` was unable to detect the desired number of internal corners.*

The image below depicts the results of applying `undistort`, using the calibration and distortion coefficients, to one of the chessboard images:

![alt text][img02]

### Pipeline (for single images)

#### 1. Provide an example of a distortion-correcting image.

The following image depicts the results of applying `undistort` to one of the project dashcam images:

![alt text][img03]

The effect of `undistort` is subtle, but can be perceived from the difference in shape of the hood of the car at the bottom corners of the image.

#### 2.Describe on how to use color transform,gradients or other ways to create a threshold binary image.Provide an example of a binary image result.

I explored several combinations of sobel gradient thresholds and color channel thresholds in multiple color spaces.These are labeled clearly in the Jupyter notebook.Below is an example of the sobel absolute threshold:

![alt text][img06]

The following image shows the various channels of three different color for the same image:

![alt text][img05]

Finally, I choose to use L channel of the HLS color space to separate white lines and the B channel of the LAB colorspace to isolate yellow lines.

I finely tune the threshold for each channel to be minimanly tolerant to changes in lighting. As part of this, I normalize the maximum value of the LAB B channel and HLS L channel to 255.Below are the exampls of thresholds in the HLS L channel and LAB B channel:

![alt text][img07]
![alt text][img08]

The results of applying the binary thresholding pipeline to various sample images are the following:

![alt text][img09]

#### 3. Describe how to perform a perspective transform and provide an example of a transformed image.

The code for my perspective transform is named as "Perspective Transform" in the Jupyter notebook.The function is `unwarp()`,the charceters are an image(`img`), source(`src`) and destination(`dst`) points. The source and destination points are the following:
```
src = np.float32([(575,464),
                  (707,464), 
                  (258,682), 
                  (1049,682)])
dst = np.float32([(450,0),
                  (w-450,0),
                  (450,h),
                  (w-450,h)])
```
Finally I verified the source and destination points on a test image to verify that the lines appear parallel in the warped image.

![alt text][img04]

#### 4. Describe how to identify lane-line pixels and fit their positions with a polynomial.

The function `slide_window_polyfit` and `polyfit_using_prev_fit` can identify lane lines and fit a second order polynomial to the right and left lane lines.
The first of these computes a histogram of the bottom half of the image and finds out the bottom-most x position(or "base") of the left and right lines.
Originally these locations were identified from the local maxima of the left and right halves of the histogram, but in my final process I changed these to quarters of the histogram just left and right of the midpoint which helped to reject lines from adjacent lanes. The function then identifies ten windows from which to identify lane pixels, each one centered on the midpoint of the pixels from the window below. This effectively "follows" the lane lines up to the top of the binary image, and speeds processing by only searching for activated pixels over a small portion of the image. Pixels belonging to each lane line are identified and the Numpy `polyfit()` method fits a second order polynomial to every set of pixels. The image below demonstrates how it works:

![alt text][img10]

The function `slide_window_polyfit` generated the histogram, in which the two peaks nearest the center are clearly visible.

![alt text][img11]

The `polyfit_by_prev_fit` function performs basically the same task,but alleviates big difficulty of the search process by leveraging a previous fit (from a previous video frame) and only searching for lane pixels within a certain range of the fit.

![alt text][img12]

#### 5.Describe how to calculate the radius of the curvature of the lane and the position of the vehicle with respect of the center.

The calculation of the radius of curvature is based upon [the website](http://www.intmath.com/applications-differentiation/8-radius-curvature.php) and coded in the cell `Radius of Curvature and Distance from Lane Center Calculation` using this line :
```
curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])
```

In this example, `fit[0]` is the first coefficient of the second order polynomial fit, while `fit[1]` is the second coefficient. `y_0` is the y position within the image upon which the curvature calculation is based.

The position of the vehicle in terms of the center of the lane is calculated with the following lines of code:
```
lane_center_position = (r_fit_x_int + l_fit_x_int) /2
center_dist = (car_position - lane_center_position) * x_meters_per_pix
```

`l_fit_x_int` and `r_fit_x_int` are the x-intercepts of the left and right fits, respectively, which requires evaluating the fit at the maximum y value(719 - the bottom of the image ) because the smallest y value is at the top.
The car position is the difference between the intercept points and the image midpoint(for the condition that the camera is mounted at the center of the vehicle).

#### 6.Plot back my result down on the road in an example image so to identify the lane area clearly.

I accomplish this function in the cell named `Draw the detected lane area back onto the basic image` and `Draw curvature radius and distance from center data onto basic image` in the file. A polygon is created according to the left and right fits, warped back to the perspective matrix `Minv` and overlaid onto the original image. The image below is an example of the results of the `draw_lane_area` function:

![alt text][img13]

The following is the result of the `add_data` function, in which the curvature radius and vehicle position data are added in the basic image:

![alt text][img14]

### Pipeline(video)

#### 1.Make sure thewobbly lines are ok but no catastrophic failure which may cause the car driveout off the road.The following is my final video link.

This is [my video result](./project_video_output.mp4)

---
### Argument

#### 1. Briefly discuss the problems or issues I met in my project,for example, where and when my pipeline would fail or how to improve it.

- The problems I met were apparent due to light conditions ,shadows, discoloration and so on. This is't challenging to dial in threshold parameters to get the pipeline to work well on the original video even in the gray section that comprised the most difficult sections. The lane line do not necessarily occupy the sanme pixel value range on the video so the normalization and scaling technologies had great help despite the fact that it may created problems like large noisy areas when the white lines did not contrast with the rest of the image. So this would clearly create an issue in a situation where a bright white car is driving among dull white lane lines.By creating a pipeline, lane lines can reliably be identified .Besides, smoothing the video output by averaging the last `n` found good fits is also useful. 
- My method also invalidates fits if the laft and right base points are not certain distance apart assuming that the lane width will remain relatively constant.

- I have thought of a few possible approaches for making my algorithm more suitable, which include dynamic thresholding(onsidering separate threshold parameters for different horizontal slices of the image), designating a confidence level for fits and rejecting new fits to deviate beyond a certain amount or rejecting the right fit if the confidence in the left fit is high and rightfit deviates too much.

---
That is the end of of my `Readme` for this project, Thanks!

