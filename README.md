
# Advanced Lane Finding Project

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

[image1]: ./output_images/img1.png 
[image2]: ./output_images/img2.png
[image3_1]: ./output_images/img3_1.png 
[image3_2]: ./output_images/img3_2.png
[image3_3]: ./output_images/img3_3.png
[image3_4]: ./output_images/img3_4.png
[image3_5]: ./output_images/img3_5.png
[image3_6]: ./output_images/img3_6.png
[image3_7]: ./output_images/img3_7.png
[image3_8]: ./output_images/img3_8.png
[image4]: ./output_images/img4.png 
[image5]: ./output_images/img5.png 
[image6]: ./output_images/img6.png 
[image7]: ./output_images/img7.png 
[image8]: ./output_images/img8.png 
[image9]: ./output_images/img9.png 
[video1]: ./project_video.mp4 "Video"


---

### Writeup / README

All my code is here: "./examples/Notebook_Final.ipynb"

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first and second code cell of my notebook: Notebook_Final.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function, used `cv2.drawChessboardCorners` to indicate corners  and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

I had to change of channels from BGR to RGB because I used `cv2.imread()`. Thanks to coefficients, I made a `cv2.undistort()` with mtx and dist coefficients.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color RGB, color HLS and gradient threshold to generate a binary image. I used sobelx and gradient direction for gradient threshold and for color I used a threshold of R and G and a threshold of S and L. I combined all of them. The pipeline is in my notebook.

![alt text][image3_1]
![alt text][image3_2]
![alt text][image3_3]
![alt text][image3_4]
![alt text][image3_5]
![alt text][image3_6]
![alt text][image3_7]
![alt text][image3_8]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()` . The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[578,460],
    [190,720],
    [1127,720],
    [707,460]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```
I felt better with this points. Last submission, I have tried with different point because I was taking a curve as a reference. In this time I took a straight road and I had better results in the video.

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 578, 460      | 320, 0        | 
| 190, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 707, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. But in this example I wanted to see curved lines.

![alt text][image4]

In this example I used the same combination of color and gradient threshold as number 2.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In order to fit my lane lines with a 2nd order polynomial, I did a histogram to have a starting point for where to search for the lines. The two most prominent peaks in this histogram will be a good indicator of the x-position of the base of the lane lines.

![alt text][image6]

I used sliding window method to find the best window center positions in a thresholded road image.

![alt text][image9]


After, I decided to search in a margin around the previous line position like this:

![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is computed according to the formula and method described in the material. Since we perform the polynomial fit in pixels and whereas the curvature has to be calculated in real world meters, we have to use a pixel to meter transformation and recompute the fit again.

The mean of the lane pixels closest to the car gives us the center of the lane. The center of the image gives us the position of the car. The difference between the 2 is the offset from the center.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This is an example where you can see a highligthed zone of green color and the identification of radius and offset. 

![alt text][image8]


---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./examples/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

#### Election of combination of threshold
It takes me more time trying to find the best combination for a binary image. I spent much time choosing HSV, HLS, RGB channels or Gradient Threshold as a perfect combination. even when the white line almost seemed to have disappeared.

#### Calculating offset
The method that I used for calculating the offset was, first compute the mean of left and right curves position. Then, I assumed `img_size[1]/2` to be the position of the car. Finally, I calculated  the center of lanes with the mean of left and right curves and indicate the position of the car from the center on video.

#### Edges detection
The model could be improved If I use different methods like Canny Edges and Hough Transform for edge detection. I think reducing noises is one way to find better edges from binary images.

#### Curvature Radio
In the video you can see more than 2km as radio. Even further more. I think there is something wrong with the coefficients of polynomials. This is one point to improve. Although there are certain parts where the road is straight, so it would seem logical to obtain these results.
