## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


In this project, your goal is to write a software pipeline to identify the lane boundaries in a video, but the main output or product we want you to create is a detailed writeup of the project.  Check out the [writeup template](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) for this project and use it as a starting point for creating your own writeup.  

The Project
---

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

The images for camera calibration are stored in the folder called `camera_cal`.  The images in `test_images` are for testing your pipeline on single frames.  If you want to extract more test images from the videos, you can simply use an image writing method like `cv2.imwrite()`, i.e., you can read the video in frame by frame as usual, and for frames you want to save for later you can write to an image file.  

To help the reviewer examine your work, please save examples of the output from each stage of your pipeline in the folder called `output_images`, and include a description in your writeup for the project of what each image shows.    The video called `project_video.mp4` is the video your pipeline should work well on.  

The `challenge_video.mp4` video is an extra (and optional) challenge for you if you want to test your pipeline under somewhat trickier conditions.  The `harder_challenge.mp4` video is another optional challenge and is brutal!

If you're feeling ambitious (again, totally optional though), don't stop there!  We encourage you to go out and take video of your own, calibrate your camera and show us how you would implement this project from scratch!

## How to write a README
A well written README file can enhance your project and portfolio.  Develop your abilities to create professional README files by completing [this free course](https://www.udacity.com/course/writing-readmes--ud777).

[//]: # (Image References)

[abs_sobel]: ./output_images/abs_sobel.png "ABS sobel"
[calibration]: ./output_images/calibration.png "Calibration"
[car_undistort]: ./output_images/car_undistort.png "Calibration"
[color_spaces]: ./output_images/color_spaces.png "Calibration"
[combined_hls_sobel]: ./output_images/combined_hls_sobel.png "Combined pipeline"
[dir_sobel]: ./output_images/dir_sobel.png "Direction sobel"
[draw_lane_with_details]: ./output_images/draw_lane_with_details.png "Draw lane with details"
[histogram]: ./output_images/histogram.png "Polyfit histogram"
[mag_sobel]: ./output_images/mag_sobel.png "Magnitude Sobel"
[perspective_transform]: ./output_images/perspective_transform.png "Perspective transform"
[perspective_area]: ./output_images/perspective_area.png "Perspective area"
[pipeline_transform]: ./output_images/pipeline_transform.png "Pipeline transform"
[polinomial]: ./output_images/polinomial.png "Polinomial fit"
[previous_polyfil]: ./output_images/previous_polyfil.png "Previous Polyfill"
[undistort]: ./output_images/undistort.png "Undistort image"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the notebook located in "./project.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

![Find chessboard corners][calibration]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Undistort chessboard image][undistort]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Applied `undistort` function in one of the sample images, with the following result, the difference can be easily appreciated on the car dashboard.

![Undistort car image][car_undistort]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In the `Color spaces` cell of "./project.ipynb". 
I tested the diffenct color spaces for the same test images for RGB and HLS.

![Color spsaces][color_spaces]

Then checked different grading sobel options.

- Absolute Sobel
![Absolute sobel][abs_sobel]

- Magnitude Sobel
![Magnitude sobel][mag_sobel]

- Direction Sobel
![Direction sobel][dir_sobel]

For my processing pipeline I applied the function `sobel_hls_threshold` which is a combination of `abs_sobel_thresh` with a threshold of `(90, 255)` and a combination of the `L` and `S` channels of the image converted to `HLS` color space.

The processing pipleine performs the following steps:

- Undistort the images, based on the camera calibration described previosly
- Perform a combination of gradient and color space transforms using `sobel_hls_threshold`
- Transform the image to bird's eye, which is described in step 3

The results are:

![Processing pipeline transform][pipeline_transform]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform can be found in the `Perspective transform` cells of "./project.ipyng".

In a first step I tried to identify the area of the road I will like to apply my transformation, choosing values for the bottom top as `720` up to `455`.

![Perspective area][perspective_area]

Based on the previous area and applying and offset of `200` pixels I found the `src` and `dst` points that will be applied to get a bird's eye transformation of the road ahead of the vehicle.

Source
```
[ 585.  455.]
 [ 705.  455.]
 [1130.  720.]
 [ 190.  720.]
```

Destination
```
[ 200.    0.]
 [1080.    0.]
 [1080.  720.]
 [ 200.  720.]
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 455      | 200, 0        | 
| 705, 455      | 1080, 0       |
| 1130, 720     | 1080, 720     |
| 190, 720      | 200, 0720     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image to verify that the lines appear parallel in the warped image.

![Perspective transform][perspective_transform]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To perform this task I applied the functions `sliding_window_polyfit` and `polyfit_using_prev_fit`, which can be found in the notebook "./project.ipynb".

#### sliding_window_polyfit

This function finds the base position of the left and right lines, which effectively folllows the line roads and returns the lanes and its points applying numpy `polyfit` function.

![Polinomianl fit][polinomial]

As it can be seen in the next histagram, the function identifies the lanes based on the two peaks near the left and rifht side of the image.

![Histogram][histogram]

#### polyfit_using_prev_fit

This function performs the same task as `sliding_window_polyfit` but using data from the previous fit to help identify the lane.

Next image shows the green area for the previous fit and current image values.

![Previous Polyfit][previous_polyfil]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

`calc_curv_rad_and_center_dist` calculates the radius of curvature and the position of the vehicel with respect to the center of the road.

```
Radius of curvature for example: 348.55093066238 m, 266.2984254798789 m
Distance from lane center for example: 0.02690423734879785 m
```

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this in the functions `draw_lane` and `draw_image_data` in the notebook "./project.ipynb".

- `draw_lane` adds a transparent green overlay for the area in the middle of the left and right lanes.
- `draw_image_data` adds a text overlay, in the top left position of the image, with details for curvature of the lane and the position of the vehicle with respect of the center.

![Lane identified][draw_lane_with_details]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link](./project_video_output.mp4) to the video result for the project.

I tried both the challenge, [link](./challenge_video_output.mp4), and the harder challenge, [link](./harder_challenge_video_output.mp4).

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The main problems I found were due to lightning conditions, shadows, which worked fine for the main project video, specially after using a combination of the `L` and `S` channel of the HLS color space. But for more twisted roads, or shadows were the road is not easy to identify I have quite a few problems tuning the pipeline.

Also the values choosen for perspective transform didn't help in a twisted road condition, as I did choose to only transform the from of the car if the curve is high then the perspective transform won't find the proper lanes. This can be checked in ther results of the harder challeng video, [link](./harder_challenge_video_output.mp4).

I will like to revisit the project in the future, can't at the moment due to time contraints, and tune up better the color space/gradient and the perspective transform to get better results in the challenge videos.

