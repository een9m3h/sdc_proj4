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

[image1]: ./writeup_images/radial_distortion.png "Radial distorted"
[image2]: ./writeup_images/corner_detection.png "Corner detection"
[image3]: ./writeup_images/correction.png "Image before and after correction"
[image4]: ./writeup_images/binary_image.png "Binary Image"
[image5]: ./writeup_images/perspective_transform.png "Perspective Transform"
[image6]: ./writeup_images/histogram.png "Histogram"
[image7]: ./writeup_images/sliding_window.png "Sliding Window Search"
[image8]: ./writeup_images/warm_window.png "Warm Start SearchWindow "
[image9]: ./writeup_images/unwarp.png "Unwarp"
[video1]: ./project_video_out.mp4 "Video"


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cells 2,3,4 of the IPython notebook located in "./v2.ipynb".  An example of the type of camera distortion to be corrected can be observed in the radial distortion in image below:

![Radial Distortion][image1]

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  Chessboard detection is shown below:

![Corner detection][image2]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 



### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![Distortion correction][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of HLS color transforms and gradient thresholds to generate a binary image (threshold steps and HLS transforms at code blocks 6-9  `v2.ipynb`).  The saturation channel and absolute gradient threshold provides the best results. Here's an example of my output for this step.  

![Binary Image][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, which appears in the file `v2.ipynb` .  The `perspective_transform()` function takes as inputs an image (`img`), as well as source (`src`).  I chose to calculate the destination points from the target image shape:

```python
dst = np.float32([[offset, offset], [img_size[0]-offset, offset], 
                                     [img_size[0]-offset, img_size[1]-offset], 
                                     [offset, img_size[1]-offset]])
```
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Perspective Transform][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The procedure for detecting lines differs depending on two states, i.e. cold_start and warm_start. An overview of the two states is given below:

Cold_Start: Either there is no previously detected line or the line has not been correctly detected for several video frames. To implement this the following steps are applied:
 

 1. First take a **histogram** along all the columns in the _lower half_ of the image. With this histogram add up the pixel values along each column in the image. In the thresholded binary image, pixels are either 0 or 1, so the two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. Use that as a starting point for where to search for the lines. An histogram example is illustrated below:
![Histogram][image6]
 2. From that point a sliding window can be placed around the line centers to find and follow the lines up to the top of the frame. Each sliding widow is of fixed height (depending upon the number of windows to fit in the image) and the center is positioned on the average of the previous window. This can be seen by:
  ![Sliding Window][image7]
  3. Once the 'valid' points are retained a 2nd deg polynomial can be fitted.

Cold_Start: Once the lane lines have been detected this can be used as a prior for the next frame. Here a margin around the previous lines can be used to classify valid points to fit the next frame. This is shown in the illustration below with the previous lines designated in yellow, the search margin around the line in green and blue and red valid points for the left/right lines respectively.
![Warm Start Window][image8]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the `curvature()`function in `v2.ipynb`.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this in my code in `v2.ipynb` in the function `transform_back()`.  Here is an example of my result on a test image:

![Unwarp][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline will likely fail when the road is constantly changing curvature. In my pipeline I implemented a  infinite-impulse response filtering of the lane line. If the line doesn't change curvature often the alpha term provides less weighting to the latest line calculated and more to the history. In rapid changing curvature like a country road the averaging needs to be reduced. Classifying the speed/road type can help change this alpha parameter. 
