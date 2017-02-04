TOAdd in end:
For the complex video
Delta from one line
Based on one line

Debugging video includes
Original frame
Undistorted frame

Warped - undistorted


Result frame


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

[image1]: ./output_images/undistorted_calibration1.jpg "Undistorted Calibration Image"
[image2]: ./test_images/test2.jpg "Original"
[image3]: ./output_images/02undistorted_test.jpg "Undistorted"
[image4]: ./output_images/03sobel_magnitude_test.jpg "Sobel magnitude mask"
[image5]: ./output_images/04hls_s_test.jpg "HLS S mask"
[image6]: ./output_images/05white_yuv_test.jpg "White YUV mask"
[image7]: ./output_images/06combined_mask_test.jpg "Combined mask"

[image10]: ./output_images/09warped_mask_test.jpg "Perspective warp mask"


[image14]: ./output_images/14mid_lines_test.jpg "Initial detection at max Y"
[image15]: ./output_images/15points_test.jpg "Detected points"
[image16]: ./output_images/16polylines_test.jpg "Polyline fit to the ponts"

[imagep1]: ./output_images/p0persp_transf_orig_straight.jpg "Straight Lines Original transformation rectangle"
[imagep2]: ./output_images/p1persp_transf_warped_straight.jpg "Straight Lines Warped transformation rectangle"
[imagep3]: ./output_images/p0persp_transf_orig_test.jpg "Original transformation rectangle"
[imagep4]: ./output_images/p1persp_transf_warped_test.jpg "Warped transformation rectangle"

[image20]: ./output_images/20plotted_test.jpg "Results plotted on warped"
[image21]: ./output_images/21plotted_unwarp_test.jpg "Results un-warped"
[image22]: ./output_images/22result_test.jpg "Results overlaid on undistorted image"
[image23]: ./output_images/23result_texttest.jpg "Final result"

###Further there are each of the points in rubric described:
###Camera Calibration
Function that performs camera calibration is defined in notebook cell #2.
It tries to read the saved values from pickle file calibration.pkl
If file loading fails with exception then it recalculates the calibration matrix and saves it to the pickle file. It was implemented in order to reduce processing times.

Calibration calculation initializes the grid for 10x7 grid point detection.
Object points are fixed points for all images representing the x,y grid on a plane, where z coordinate = 0.
Image points are read from each of the images in camera_cal folder and coordinates are found using cv2 function findChessboardCorners() using the grayscale image.
If the image points are found then they are added to the imgpoints array and also Object points are added to objpoints array.

This succesfullty processes 17 out of 20 images, the ones with different number of points are rejected in this implementation.

Objpoints and Imgpoints arrays are processed by cv2 function calibrateCamera() that returns distortion matrices including the mtx and dist, that are saved to the pickle file and used in pipeline.

Example of undistorted calibration image:
![alt text][image1]


###Pipeline (single images)
Note: Processing pipeline is performed by function process_frame(image, debug, name), that as input takes one RGB image.
It is possible to pass additional parameter debug, that has default value of 'None'.
There can be 2 additional values set:
'Save' - it will output the pipeline steps to 'output_images' folder with the prefix filename passed in name parameter
'Debug' - it will output the result as 1920x1080 image that will consist of multiple lower resolution frames, that each will show one of the pipeline steps.

####1. Provide an example of a distortion-corrected image.
Each image (or frame) is passed to function undistort(), that takes input parameters of image and distortion matrices:mtx and dist.
mtx and dist are calculated once inthe befinninf from the camera calibration step.
undistort() function uses cv2.undistort().

test_images/test1.jpg is processed by undistort and saved in output_images/undistorted_test1.jpg:
![alt text][image2]
![alt text][image3]


####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

After undistorted image is obtained then the various binary masks are calculated.
In the submission pipelines following 3 masks are used:
Sobel gradient magnitude (np.sqrt(sobelx* *2 + sobely* *2))
S channel from HLS (yellow and white)
White from YUV colorspace - in addition to above

Code for the image thresholding functionas are in notebook cell 3.

During testing there were multiple additional binary masks calculated, e.g.:
Sobel in one direction X or Y
Grayscale - initial, used to test pipeline failover to full scan, when there are many bad detections
Channels form RGB
Etc., however, they did not provide significant improvements. Also, the more different calculation are done, the individual frame processign performance becomes, which could be significant factor if the processing should be performed real time.

It was also tested in perspective warping should be performed beofre or after thresholding, however, especially in case of Sobel, it turned out that it is better to threshods original image and only then perform perspective warping on binary image.
If perspective warping was performed before Sobel, then there were many artifacts introduced that did not allow good quality Sobel result.


Addtional point is that various parameters can be tuned at this step in order to get the result better and what works in one video, might not transfer very well to other videos, especially the hardest challenge where there is quite different environment.

Here are the images:
![alt text][image4]
![alt text][image5]
![alt text][image6]
![alt text][image7]


####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Code for the perspective transformation functionas are in notebook cell 4.
There are two functions defined, one for calculating transformation and inverse transformation matrix and the second function for actually applying perspective warp by passing image and transformation matrix.

Transformation matrix is calculated by hardcoded values, finding out correct values turned out more complex than initially estimated.
It is not as simple as to take straight road segment and identify the points. It should be adjusted how long along vertical (Y) axis the points are chosen and how much to offset lines horizontaly from borders in order to get good enough results and still remain with consistent transformation on curved roads.

In the final solution values from Udacity classroom is used, as they did wery well in order to avoid big transformation errors on curved road segments.

The difficulties are there because, even if we take points on an image with straight road, then there are additional variables that impact accuracy:
a) Car might not be centered within lane
b) Camera might not be centered in car
c) Camera angle (axis) might not be accuratelly aligned with car axis
d) Car is not driving parallel to the straight road lanes
In real life, it would be much better to calibrate camera by physically marking (measuring) the points, e.g. in parking spot, so that car is car and camera is centered and aligned with the measured axis. That way the small errors could be eliminated.

Here are the points that are used for the transformation drawn on the undistorted original and warped sample for the image of Straight lines.
![alt text][imagep1]
![alt text][imagep2]

And for the Test image:
![alt text][imagep3]
![alt text][imagep4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

As an input the binary mask that is obtained by using the perspective transformation warping function above:
![alt text][image10]

There are two distinct functions for detection, one is for full frame and other is for partial frame based on previously fit polynomial.
Which detection is run is determined by state class.

####4.1 Full detection:
Code is within cells 5&6.

Full detection is run on initial frame or if it was not succesfully detected in 10 sequential frames.
Full detection is always added to the detected line array.

Full detection is done by initially splitting frame to two at the middle point of the image. Then for each half, the maximum histogram points are detected, by checking histogram and reverse histogram indexes (described below) for frame between vertical point 480 and maximum. This results in peak point over full frame.
![alt text][image15]
This peak is used to narrow down search also for full frame detection and it checks 80 pixels around that peak in order to determine the line points. After each vertical loop, it adjusts the peak position to detected one previously, that allows to quickly recognize main lines and also follow then if they are curved.

####4.2 Partial detection:
Code is within cells 7&8.

Partial detection is done when there are previously fit polynomial line.
Partial detection is similar to the full detection, but it checks area only 40 pixels to both sides form the previous polunomial line.
Within that frame it checks histogram and reverse histogram (described below) and outputs detected points.

Succesfull partial detection is when both lines has at least some points detected (in order to fit polynomial) and also if detected polynomial passes test to determine if it was not too far off from the previous detected line average.

Polynomial function that is used is: ax^2+bx+c
Delta check is done based on b value of the polynomial function, because it determines main curvature of the line. If the difference between averaged polynomial fit over previous 7 frames and latest detected line is over the threshold then the line is rejected and marked as not detected.


####4.3 Both detection functions:
######Function for maximum point identification
Initailly function "signal.find_peaks_cwt" was used, however its performance was very slow, therefore some other functions was tested. In the end, final decision was to use histogram function and numpy argmax, that returns index of first maximum appearance.

#####Histogram and reverse histogram
Using histogram and np.argmax, run into cases when the lines shifted to the left (lower) indexes. This was because argmax returns first index and there might be case when there are multiple values at the maximum value.
For example, in the array:
0,0,1,2,5,5,5,5,2,1
Argmax would output index 4 (as it is first element with maximum value of 5)
In order to get more accurate result, the histogram was reversed and argmax was taken from the reverse histogram, in this case it would be a value of 2, which leads to index 9-2=7.
The maximum is then averaged rounded down 4+7=11/2=index position of 5.

That creates special case, when histogram contains just 0 values that middle value is returned, but this was covered by performing additional check and return NaN value if this is the case.

######Vertical frame size
Height of 20 pixels is checked each time, this gave reasonable detections (lower false positives from occasional points), but kept the number of points high for the polynomial fit functions.

####4.4 Polynomial fit:
From the detected points:
![alt text][image15]

Pipeline passes detected points to the numpy function np.polyfit(), that calculates 2nd order polynomial.
![alt text][image16]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for this is in Notebook cell 1, class Lane and function add().

It follows the class instructions of calculating pixel to metre conversion.
I take the averaged values over the last multiple succesfull frames and values are calculated from this averaged polyfit.
Curvature that is output to video is average between left and right lanes, in order to have less output parameters.
In addition to that, if curvature is too large (>4000m), then text "Curvature: straight" is displayed instead, as this is more meaningful value at such large curvatures.

Off center position is calculated by difference (in meters) between middle pixel between left and right polyfit position at maximum Y coordinate and image center position.
Output is provided in Centimeters without decimal places.


####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for drawign results are in function process_frame() under comment "Drawing results on road image".

Here are the images, after drawing results on warped image:
![alt text][image20]
After reverse perspective transform:
![alt text][image21]
And overlaid on original unwarped image:
![alt text][image22]

And there is the final image with curvature and offcenter position:
![alt text][image23]

Note: Single image always runs in full detection mode as there is no previous information for polyline fit. 

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a link to the final video:

And two debuging videos:

First, for color transformations and masking:

Second, for undistortions and perspective warping:


---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

