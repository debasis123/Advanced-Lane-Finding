## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


In this project, our goal is to write a software pipeline to identify the lane boundaries in a video.


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

The images for camera calibration are stored in the folder called `camera_cal`. I tested my pipeline with the images from the folder `test_images`. The output of these images for different stages of pipeline are stored in the folder `output_images`.


[//]: # (Image References)

[image1]: ./output_images/undistorted.png "Undistorted"
[image2]: ./test_images/test2.jpg "Original test image"
[image3]: ./output_images/binary_combo_example.png "Binary Example"
[image4]: ./output_images/warped_straight_lines.png "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/example_output.png "Output"
[video1]: ./project_video_tracked.mp4 "Video"


---

### Camera Calibration

<!-- #### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image. -->

Section `Camera calibration` in the jupyter notebook describes the code for this.

I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. I assumed that the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

Then I used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]


### Image Thresholding Pipeline (single images)

<!-- #### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result. -->

Section `Image thresholding` in the jupyter notebook describes the image thresholding process. Here I describe how I apply the distortion correction and image thresholding to identify the road to one of the test images like this one:
![alt text][image2]

I used absolute values, magnitude, and direction of the gradient for both x and y directions after applying sobel operator,and applied various thresholds on it to identify the road. To improve upon it, I also used the S channel of the HLS color space and V channel of the HSV (with thresholds) to identify the road in the test image.

Here's an example of my output for this step.

![alt text][image3]


### Perspective Transform (single images)

<!-- #### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image. -->

Section `Perspective transform` section in the jupyter notebook describes the code for this.

The code for my perspective transform includes a function called `get_perspective_transform()`. The function takes as inputs an image (`img`). The transform uses two matrices as source (`src`) and destination (`dst`) points. I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[0,h-30], [w,h-30], [730,460], [540,460]])  # from bottom left, counter-clockwise
dst = np.float32([[0,h], [w,h], [w,0], [0,0]])  # in the same order as src
```
where `w` is the width of the image (=1280) and `h` is the height of the image (=720).

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 0, 690        | 0, 720        |
| 1280, 690     | 1280, 720     |
| 1730, 460     | 1280, 0       |
| 540, 460      | 0, 0          |

<!-- I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. -->

The following picture shows the above test image before and after the perspective transform. The left and right lanes indeed seem parallel.

![alt text][image4]


### Fitting polynomial to lanes

<!-- #### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial? -->

Section `Lane detection` in the jupyter notebook describes rest of the pipeline. First I defined a class `Tracker` which keeps the current and past information about two lanes, so as to decide on next frame of the video intelligently based on previous frames. This helps to keep the fitting polynomial to the lines steadier. I used the convolution method to approach the window sliding while fitting the polynnomial to the lanes. The function `sliding_window_using_convolution()` describes it. The basic idea and code snippet is from the lesson.

I used a 2nd order polynomial like this to fit to the lanes. The code is implemented by the function `fit_lane_boundaries()`.

![alt text][image5]


### Radius of Curvature and position of the vehicle with respect to center

<!-- #### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center. -->

I implemented this step in functions `radius_of_curvature()` and `calculate_offset_of_car_on_road()`. The relevant sections are mentioned in the jupyter notebook and concepts are from the lession.


### Identifying lane area with color

<!-- #### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly. -->

`visualize_road()` function in the relevant section describes the method. Here is an example of my result on a test image:

![alt text][image6]

---


### Pipeline (video)

Here's a [link to my video result](./project_video_tracked.mp4)

---

### Discussion

<!-- #### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust? -->

There are many parameters to tune at different parts of the project, for example, during image thresholding, perspective transform, fitting polynomial, window width and height during lane detection, etc. More experimentation with them will definitely improve this pipeline.

Mixing this pipeline (or compare) with the pipeline by deep learning method will be an interesting next project direction.

The video clearly shows that the pipeline needs better work where there is shadow on the road. Further, the right lane fit is not very steady as the lane itself is not continuous. Averaging over larger number of past frames for the right lane may fix this problem.
