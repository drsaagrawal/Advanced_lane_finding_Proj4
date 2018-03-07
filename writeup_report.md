
## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 3rd cell and 7th cell of the IPython notebook.
I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, objp is just a replicated array of coordinates, and objpoints will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.
I then used the output objpoints and imgpoints to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function. I applied this distortion correction to the cheesboard image and test image using the cv2.undistort() function. Plots for chessboard and test images are in 9th and 14th cell respectively.![undistorted_test.png](attachment:undistorted_test.png)![drawcorners_chessboard.png](attachment:drawcorners_chessboard.png)![undistorted_chessboard.png](attachment:undistorted_chessboard.png)

### Pipeline (single images)


#### 1. Provide an example of a distortion-corrected image.

Code is in cell 12. To demonstrate this step, I will describe how I apply the distortion correction to one of the test images (test2.jpg) like this one:![test2_undistort.png](attachment:test2_undistort.png)
    

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Code is in cell 12. I used a combination of color and gradient thresholds to generate a binary imagez. Here's an example of my output for this step.![test2_threshold.png](attachment:test2_threshold.png)

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for this in cell 11. I used hard coded source and destination points. Used getPerspectiveTransform(source,dst) function and warpPerspective(img, M,img_size, flags=cv2.INTER_LINEAR) function to get the ouput. The source and destination points are:
This resulted in the following source and destination points:

src=np.float32([[239, 683],[571, 455],[751, 455],[1050, 683]])

dst=np.float32([[100, img_size[0]],[100, 0],[img_size[1]-100, 0],[img_size[1]-100, img_size[0]]])

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 239, 683      | 100, 720      | 
| 571, 455      | 100, 0        |
| 751, 455      | 1180, 0       |
| 1050, 683     | 1180, 720     |

I verified that my perspective transform was working as expected by drawing the src and dst points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. ![undistort_destination.png](attachment:undistort_destination.png)![undistort_source.png](attachment:undistort_source.png)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Code for this is in Cell 20. 
* After applying calibration, thresholding, and a perspective transform to a road image, I have a binary image where the lane lines stand out clearly. However, I still need to decide explicitly which pixels are part of the lines and which belong to the left line and which belong to the right line. I first take a histogram along all the columns in the lower half of the image like this:![histogram.png](attachment:histogram.png)
* With this histogram I am adding up the pixel values along each column in the image. In my thresholded binary image, pixels are either 0 or 1, so the two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. I can use that as a starting point for where to search for the lines. From that point, I can use a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame.

Steps for code:
* Choose the number of sliding windows
* Set height of windows
* Identify the x and y positions of all nonzero pixels in the image
* Current positions to be updated for each window
* Set the width of the windows +/- margin
* Set minimum number of pixels found to recenter window
* Create empty lists to receive left and right lane pixel indices
* Step through the windows one by one
* Identify window boundaries in x and y (and right and left)
* Identify the nonzero pixels in x and y within the window
* Append these indices to the lists
* If you found > minpix pixels, recenter next window on their mean position
* Concatenate the arrays of indices
* Extract left and right line pixel positions
* Fit a second order polynomial to each


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.


Code for this in cell 20.
Steps:
* Define conversions in x and y from pixels space to meters
* Fitting the polynomial
* Calculate radii by formula - ![formula.png](attachment:formula.png)


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Implemented this in cell 23.
![final.png](attachment:final.png)

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

[Output_Video](./outpy.mp4)

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In the beginning sometimes the coloured area was covering the left most part also then I changed the gradient, source and destination ponits to tackle it.

The edges were flickering very much so I have taken the average for smoothening.

This may fail in very sharp curves. To make pipeline more robust we can have dynamic thresholding (separate threshold parameters for different horizontal slices of the image).
