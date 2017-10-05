
[//]: # (Image References)

[image1]: ./images/undistort.png "Undistorted"
[image2]: ./images/test1.png "Road Transformed"
[image3]: ./images/binary_combo_example.png "Binary Example"
[image4]: ./images/warped_lines.png "Warp Example"
[image5]: ./images/color_fit_lines1.png "Fit Visual"
[image6]: ./images/draw_data.png "Output"
[video1]: ./project_video_output.mp4 "Video"

---

### README

## Advanced Lane Finding Project

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in " project4.ipynb" 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

With camera calibration matrix and distortion coefficients achieved by camera calibration, I can use*** cv.undistort*** function to return the undistorted image, shown as above. 

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used multiple combinations of different color and gradient thresholds to generate a binary image (thresholding steps at In 51 ***function thresh_gradient() ***in `project4.ipynb`).  The final result I use is the combination of B-channel in LAB color space and L-channel channel in HLS color space to extract lines. Here's an example of my output for this step. 

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `corners_warper()`, which appears in In 52 in the file `project4.ipynb.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. The main steps in this function is shown as below:
    
     a) define 4 source points src = np.float32([[,],[,],[,],[,]])
     b) define 4 destination points dst = np.float32([[,],[,],[,],[,]])
     c) use cv2.getPerspectiveTransform() to get M, the transform matrix
     d) use cv2.warpPerspective() to warp your image to a top-down view
 
 I chose the hardcode the source and destination points in the following manner:


This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 230, 720      | 320, 720        | 
| 610, 460      | 320, 0      |
| 740, 460     | 960, 0      |
| 1170, 720      | 960, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used a sliding window_search fucntion to detect the the sum of nonzero pixels vertically in the lower half of each image frame, in this way, I can using the maximum value points in each left and right side of images to determine the base points. Then, I set sliding windows to identify the x and y positions of all nonzero pixels in the image and set margins of box to detect the nonzeros pixels within the slide windows. After I got all the lane pixels within slide windows. I fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

This part of code is shown in In[54] with a function call ' window_search()'
when I detect the base points and already have a lane fit, i will using another function I defined 'fit_using_previous()'

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines IN[93] in my code in `project4.ipynb.` as function of curvature_center_dist()
in this part, I calculate the radius of base point of image and the difference of lane center and image center to calculate the offset of vehicle position.
I also write a function to draw radius result and vehicle position regarding to the center line on each image frame.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `project4.ipynb.` in the function `draw_data()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

At beginning, I tried just using sobel gradients in x direction and s-channel to detect lane pixels. However, when vehicle go into the shadows areas, detection went really bad. Also, when the black car on the adjunct lane coming, the lane detection also get bad. Therefore, I try using L-channel in HLS to detect white lane and get a better result. After try different combination of color space and gradient. I finally use combination of B-channel in LAB color space and L-channel channel in HLS color space to extract lines.

Also, without smooth the lane, the lane is easy to shake around. I also add a function to store recent five best lanes to overcome this problem.

To further improve my algorithm, I am think using different images processing (different combination of color thresh and gradient) regarding to different road materials. And train a classifier to detect current road and light situation and using the most suitable image processing algoithm to current frame.

My algorithm may fail when white lane is not clear, the material od road reflect the white light or the sunshine is too bright which may cause white lane detection become unstable. If the white car is at the adjunct lane, slide window search may also detect that and cause the base point or fit curve have some issues. 
 