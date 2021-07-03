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
[image1]: ./output_images/Undistort.png "Undistorted"
[image3]: ./output_images/test3.jpg "HLS"
[image4]: ./output_images/perspective.jpg "Perspective"
[image5]: ./examples/example.PNG "Fit Visual"
[video1]: ./output_images/clip.mp4 "Video"



### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

Cameras typically use curved lenses to capture an image, therefore they do not detect the "real" image and therefore require algorithmic calibration and corrections to utilize in lane detection. For this project, I corrected these distortion factors by calibrating the camera using a chessboard. 

First, I prepared the object points (objpoints) and image points (imgpoints) arrays which hold the real 3D points and 2D image points respectively.

Using a given library of chessboard images, I stepped through each image and performed the following steps:
    1. Used the cv2.findChessboardCorners() function to find the chessboard corners. 
    2. If corners were detected then used the cv2.drawChessboardCorners() function to draw the corners onto the image.
    3. Used the cv2.calibrateCamera() function to calcuation the camera matrix and distortion coefficients.
    
The code for this step is contained in Cell Block 2 of the IPython notebook located in "PROJECT.ipynb".
    
These coefficients will be used later in the project to undistort the image in my pipeline.


![alt text][image1]

### Pipeline 

#### 1. Calibrate the image

As described in the previous section, calibration using chessboards is performed to assist in correction of the camera images. In the images below, the first step will detect corners of a given image and return the same image with corners detected.

The result of this will yield our mtx and dist values.

#### 2. Undistort the image

In this step, I used the cv2.undistort() function and our mtx and dist values from the previous step to undistort the camera images for the project. In the images below, it can be shown that our images our undistorted.

![alt text][image2]

#### 3. Generating a thresholded binary image

In order to detect lane lines, it is necessary to color transform the image into a binary (black/white) image so that our algorithm later in the pipeline can easily detect the lanes. For this project, I used an HLS transform. By filtering the images to find the Saturation layer, I was able to output accurate binary images to detect lane lines on. I experimented with other transforms such as the sobel, magnitude, and direction combination, however it did not provide for the same amount of flexibility.

Below is an example of a threshold binary image that is generated through my HLS filtering:
![alt text][image3]

#### 4. Perspective transform


Another setup task before lanes are actually detected is finding the perspective transform. Since objects further away from the current perspective provide distorted/skewed images, it is necessary to find a "bird's eyed view" of our scope in order to accurately define our outputs. In my perspective() function, I utilized the cv2.getPerspectiveTransform and cv2.warpPerspective functions to obtain this bird's eyed view. Since we know the camera is mounted in the center of the vehicle, we can use hard-coded values to get the generic range of where the lanes are going to be. After applying the warpPerpective() function, I was able to convert the image to the desired view. 

Below are my before and after images:


![alt text][image4]

#### 5. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
Once all the calibrations, undistorting, and transformations are complete, lanes are calculated within the find_lane_pixels(), fit_polynomial(), fit_poly(), and previous() functions.

Using the binary threshold warped image, a histogram can be found to find peaks in the image for areas of activated zones. These activated zones will be our lane lines. From there, I used a sliding window algorithm to generate the left and right fit variables used to calculate the actual fits. This algorithm and histogram is used within the find_lane_pixels() function which gets called by the fit_polynomial() function. The fit_polynomial() function is used to fit the polynomials generated by the find_lane_pixels() function.

Below is an example of the find_lane_pixels() and fit_polynomial() functions in action:

![alt text][image5]

In addition, I implemented the fit_poly() and previous() functions to optimize the pipeline. Since the previously mentioned method of finding lane lanes is a computationaly slow process, it is faster to only use it when necessary. Therefore, the previous() function utilizes a history of lane lines with a defined margin = 100 to fit lines that are close to the fitted line averages.

Below is an example of the previous() function which finds the polynomial fit within the defined margin(in green).


![alt text][image5]



### Pipeline (video)

#### 1. Project Output

Here's a [link to my video result](./output_images/clip.mp4)


### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

One major issue within my project was handling bumps in the road which caused my detected lanes to wobble significantly. I believe that this is caused by the physical change in the camera's location which would invalidate the perspective transform that was calculated in the pipeline. As mentioned previously, we used values for our src array which hold values where the lanes should be detected. When the vehicle comes over bumps, these values must change since the camera's orientation/position is changing as well. To make this project more robust, I could potentially create an algorithm which dynamically detects the areas where lanes should appear as opposed to hardcoding them.
