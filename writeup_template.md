## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"
[undistortedImage]: ./output_images/unidstort1.png "Undistorted"
[road_undistorted]: ./output_images/road_undistorted.png "Undistorted Road Image"
[undistorted_chessboards]:  ./output_images/undistorted_chessboards.jpg "Undistorted Chessboards Example"

[BinaryImage]: ./output_images/BinaryImage.jpg "Binary Road Images"
[challengeBinary_NOK1]: ./output_images/challengeBinary_NOK1.jpg "NOK Challenge Road Image 1"
[challengeBinary_NOK2]: ./output_images/challengeBinary_NOK2.jpg "NOK Challenge Road Image 2"
[challenge_frames]: ./output_images/challenge_frames.jpg "Challenge Lighting road frames"

[thresholding]: ./output_images/thresholding.jpg "Thresholding Example"

[Birds_eye.jpg]: ./output_images/Birds_eye.jpg "Birds Eye Transform"
[lane_lines_poly.jpg]: ./output_images/lane_lines_poly.jpg "Left and Right Lane Line polynomial fitting"
[perspective_transform_src]: ./output_images/perspective_transform_src.jpg "Perspective Transform - src"
[warped_image]: ./output_images/warped_image.jpg "Warped perspective of road lanes"
[find_lane_pixels_histogram]: ./output_images/find_lane_pixels_histogram.jpg "find lane pixels - histogram "
[annotated_unwarped_road]: ./output_images/annotated_unwarped_road.jpg "Unwarped Road Image with annotations "

[project_video_out]: ./project_video_out.mp4 "Project Video Output "

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The followed the principles outlined to perform a calibration on the camera and correct lens distortion factor.

(1)   I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

(2) Use OpenCv function  `cv2.findChessboardCorners(gray, (nx,ny), None)` and pass in my grayscale converted chessboard images (`gray`), including the number of corners I expect in the x direction (`nx`) and the number of corners I expect in the y direction (`ny`)
Note that calibration images indexed {1,4,5,20} could not be undistorted using the 9x6 inner corner definition because the outermost corners either extended beyond the photo (or where at the image boundary)

(3) Use OpenCv function  `cv2.calibrateCamera(objpoints, imgpoints, gray.shape[::-1], None, None) `.  As described in the openCV documentation [here] (https://docs.opencv.org/2.4/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html).  The function uses a pin-hole camera model to return the camera matrix, distortion coefficients, output vector of rotation for each chessboard patern, and the output vector for tranlation required to consequently undistort the image.

Refer to ![undistorted_chessboards][undistorted_chessboards]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Test image 4 is a good example to show the undistortion beign applied to the original image. The white car originaly was in complete view in the image, however after the undistortion function it is translated such that it's rear is clipped in the image.
![road_undistorted][road_undistorted]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.

combined_threshold_image = thresh_colour2(undistortedRoad)
(1) Colour transformation to HLS image space. This was recommended in the course as being a robust -> filter for white and yellow lines. then returned a masked image (not binary image) for use in text threshold operation.  

binaryImage = thresholdHLS(undistortedRoad, s_thresh=(135, 255), sx_thresh=(10, 225))
(2) Apply sobel derivative in x direction to look for only vertical lane lines in the image
(3) Applied threshold to the saturation component of the image and combined the 2 to create a binary output.

I would describe my implementation as minimal, but it still robust to identify the lane pixels in the test images.

Testing this function out with the the challenge video images, it's clear that some further work here needs to be done especially when there is a gradient between the "road" and variable lighting conditions. I will have to explore thresholding on the L channel also and combining it with the sobel thresholding.

The algorithm was effective for the test images and I also tried this with frames in the challenge videos.
![thresholding][thresholding]

The algorithm could be improved for challenge images.
![challenge_frames] [challenge_frames]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I used the openCV functions `getPerspectiveTransform(src,dst)`, and `cv2.warpPerspective(undistortedRoad, M, (nx,ny))` to obtain the "bird's eye view" transformation of the road. 

I explored the idea of drawing grid lines first test images to narrow down the scope of the vertices used for the transformation.
I opted for the top vertices to be under the horizon and allow enough margin around the 1200 pixel mark. 

However this was not robust for the challenge video.  There was more curvature,  elevation in the road, and the vehicle was in an outer lane. So I will need to look my source point definition and decide whether this should be fixed. 


I defined the distortion transform to include neighbouring lanes.  
![perspective_transform_src][perspective_transform_src]

`src = np.float32([[595,451], [684,451], [1045,680] , [262,680])`
`dst = np.float32([ [405, 0], [850 - 5, 0], [850, 720], [404, 720]])`

![warped_image][warped_image].

This was done with the intent to reduce level of distortion on pixels in the distance / horizon region and made the line pixel search algo more effective.  

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.  


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I followed the logic outlined in the course, and that was to segment the image into sliding windows. Then used the numpy function `np.polyfit` to fit a line on the detected line pixels for the left lane line and the right lane line.

Refer to `find_lane_pixels(binary_warped):` and ` def fit_polynomial(binary_warped):` in `CARND-Advanced_Lane_Lines.ipyn` notebook.

![lane_lines_poly][lane_lines_poly]
![find_lane_pixels_histogram][find_lane_pixels_histogram]

I explored different implementations here (refer to `def find_window_centroids(image, window_width, window_height, window_margin):` and 
`def search_around_poly(binary_warped):` functions in line 118 and line 122.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Refer to `def measure_curvature_real(left_fit, right_fit, image_width, image_height) :`
Refer to 'https://www.intmath.com/applications-differentiation/8-radius-curvature.php'

The calculations were converted to meters.
`ym_per_pix =30./720  `
`xm_per_pix = 3.5/417` The U.S lane width regulation is upto 3.7m  I measured the pixel distance from left and right lines to be 417pixels.

![lane_lines_poly][lane_lines_poly]


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Refer to `def draw_lane(original_img, binary_img, l_fit, r_fit, Minv):` and `def draw_data(original_img, curv_rad, centre_offset):`


Note this time I used the inverse matrix `Minv` to transform the image back to the road image view.

![annotated_unwarped_road][annotated_unwarped_road.jpg]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a link to the output video [project_video_out](./project_video_out.mp4)


### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?


 - Need a more robust filter for the lane line detection.  
 - I attempted to stack up multiple binary filters.  First I was stripping slowly different features and returning a masked image to the next binary filter -> and so forth.

- I invisiged implementing an adaptive filter that would segment the "road" and the lines (something simlar to a "white balance" filter but for "road image".

- The adaptive filter would use ROI on "road" areas and return data such as road HLS range (this would be a helper function for road "image stats" where min, high average outliers etc would be returned. These stats would be used as an input for "adaptive threshold"
- The ROI may be line segments across the "road area" - with origin to the horizon, and also horizontal lines across various horizontal planes.
- would use these as flags for high variance frames where white/dark washing of image segments may be occuring due to lighting conditions.

I also had an idea of using the perpsective transform to to warp line ROIs to the birds eye view and pair them.  Pixels in the y-axis would be plausible lane pixels if they were the correct 3.5-3.7 meters distance apart. This could be added in for robustness of the fit-polynomial function.
