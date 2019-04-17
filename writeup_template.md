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
[BinaryImage]: ./output_images/BinaryImage.jpg "Binary Road Images"
[challengeBinary_NOK1]: ./output_images/challengeBinary_NOK1.jpg "NOK Challenge Road Image 1"
[challengeBinary_NOK2]: ./output_images/challengeBinary_NOK2.jpg "NOK Challenge Road Image 2"
[Birds_eye.jpg]: ./output_images/Birds_eye.jpg "Birds Eye Transform"
[lane_lines_poly.jpg]: ./output_images/lane_lines_poly.jpg "Left and Right Lane Line polynomial fitting" 

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

Refer to [undistortedImage]



![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Test image 4 is a good example to show the undistortion beign applied to the original image. The white car originaly was in complete view in the image, however after the undistortion function it is translated such that it's rear is clipped in the image.
![alt text][image2]
![alt text][road_undistorted]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.

(1) Colour transformation to HLS image space. This was recommended in the course as being a robust 
(2) Applying a Sobel gradient threshold (for x-direction)
(3) Applying threshold Saturation Channel

Refer to function definition `thresholdHLS(image, s_thresh=(130, 255), sx_thresh=(40, 100))` for details.

I would describe my implementation as minimal, but it still robus to identify the lane pixels in the test images.

Testing this function out with the the challenge video images, it's clear that some further work here needs to be done especially when there is a gradient between the "road" and variable lighting conditions. I will have to explore thresholding on the L channel also and combining it with teh sobel thresholding.

The algorithm was effective for the test images.
![BinaryImage]: ./output_images/BinaryImage.jpg "Binary Road Images"

The algorithm was not effective for the challenge images.
![challengeBinary_NOK1]: ./output_images/challengeBinary_NOK1.jpg "NOK Challenge Road Image 1"
![challengeBinary_NOK2]: ./output_images/challengeBinary_NOK2.jpg "NOK Challenge Road Image 2"

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I used the openCV functions `getPerspectiveTransform(src,dst)`, and `cv2.warpPerspective(undistortedRoad, M, (nx,ny))` to obtain the "bird's eye view" transformation of the road. 

I explored the idea of drawing grid lines first test images to narrow down the scope of the vertices used for the transformation.
I opted for the top vertices to be under the horizon and allow enough margin around the 1200 pixel mark. 

However this was not robust for the challenge video.  There was more curvature,  elevation in the road, and the vehicle was in an outer lane. So I will need to look my source point definition and decide whether this should be fixed. 

I chose to hardcode the source and destination points as below:

`src = np.float32([    
    [585, 455],[705, 455], [1130, 720], [190, 720]
])

pixel_offset = 190   
nx = img.shape[1]
ny = img.shape[0]

print(nx)
print(ny)

dst = np.float32([
    [pixel_offset, 0],
    [nx - pixel_offset, 0],
    [nx -  pixel_offset, ny], 
    [pixel_offset, ny] 
])`


![Birds_eye.jpg]: ./output_images/Birds_eye.jpg "Birds Eye Transform"

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I followed the logic outlined in the course, and that was to segment the image into sliding windows. Then used the numpy function `np.polyfit` to fit a line on the detected line pixels for the left lane line and the right lane line.

Refer to `find_lane_pixels(binary_warped):` and ` def fit_polynomial(binary_warped):` in `CARND-Advanced_Lane_Lines.ipyn` notebook.

[lane_lines_poly.jpg]: ./output_images/lane_lines_poly.jpg "Left and Right Lane Line polynomial fitting" 

I explored different implementations here (refer to `def find_window_centroids(image, window_width, window_height, window_margin):` and 
`def search_around_poly(binary_warped):` functions in line 118 and line 122.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
