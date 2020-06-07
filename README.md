## Advanced Lane Finding

![sdc-1-1024x427](https://user-images.githubusercontent.com/25509152/33837471-770be3a2-de9d-11e7-97cc-50c6224bfcc3.png)

---
### Table of Contents
---

- Overview
- steps of project
- steps detail
- pipeline
- Test on videos
- concolusion
- end

---
## Overview :point_left:
---

## Advanced Lane Detection Project which includes advanced image processing to detect lanes irrespective of the road texture, brightness, contrast, curves etc. Used Image warping and sliding window approach to find and plot the lane lines. Also determined the real curvature of the lane and vehicle position with respect to center.

---
![](gif.gif)
---
## steps of project :
---
### Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.

### Apply a distortion correction to raw images. Use color transforms, gradients, etc., to create a thresholded binary image

### Apply a perspective transform to rectify binary image ("birds-eye view").

### Detect lane pixels and fit to find the lane boundary.

### Determine the curvature of the lane and vehicle position with respect to center.

### Warp the detected lane boundaries back onto the original image.

### Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle

### applay this steps on video stream.


---
## Camera Calibration
---

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The first step in the pipeline is to undistort the camera. Some images of a 9x6 chessboard are given and are distorted. Our task is to find the Chessboard corners an plot them. For this, after loading the images we calibrate the camera. Open CV functions like findChessboardCorners(), drawChessboardCorners() and calibrateCamera() help us do this.

---
### image distortion

#### These images are Distortion Corrected :

- code 

```
objp = np.zeros((6*9,3), np.float32)
objp[:,:2] = np.mgrid[0:9,0:6].T.reshape(-1,2)
ret, corners = cv2.findChessboardCorners(gray, (9,6),None)

```
```
def cal_undistort(img, objpoints, imgpoints):
    # Use cv2.calibrateCamera() and cv2.undistort()
    ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, img.shape[1::-1], None, None)
    undist = cv2.undistort(img, mtx, dist, None, mtx)
    print( dist)
    return undist
```

![output_1](https://user-images.githubusercontent.com/34116562/49129716-d0936f80-f2f6-11e8-9165-fbbdbc7e96ec.png)
![output_2](https://user-images.githubusercontent.com/34116562/49129717-d1c49c80-f2f6-11e8-993a-03642ccce32a.png)
![output_3](https://user-images.githubusercontent.com/34116562/49129719-d2f5c980-f2f6-11e8-8875-a3a3445d4d00.png)
![output_4](https://user-images.githubusercontent.com/34116562/49129720-d4bf8d00-f2f6-11e8-9ed2-e2eb9256bbdf.png)
![output_5](https://user-images.githubusercontent.com/34116562/49129735-e4d76c80-f2f6-11e8-9a3e-af0ace294a08.png)
![output_6](https://user-images.githubusercontent.com/34116562/49129738-e6089980-f2f6-11e8-98a4-804e9e632a4a.png)

---

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

Detecting edges around trees or cars is okay because these lines can be mostly filtered out by applying a mask to the image and essentially cropping out the area outside of the lane lines. It's most important that we reliably detect different colors of lane lines under varying degrees of daylight and shadow. So, that our self driving car does not become blind in extreme daylight hours or under the shadow of a tree.
 
I performed gradient threshold and color threshold individually and then created a binary combination of these two images to map out where either the color or gradient thresholds were met called the combined_binary in the code.

![output_1](https://user-images.githubusercontent.com/34116562/49129596-68448e00-f2f6-11e8-8d0d-bfa31bbc1dbe.png)
![output_1](https://user-images.githubusercontent.com/34116562/49129605-6c70ab80-f2f6-11e8-9678-c058d70bd1a3.png)
![output_2](https://user-images.githubusercontent.com/34116562/49129599-6a0e5180-f2f6-11e8-9539-b4d630e90a84.png)
![output_2](https://user-images.githubusercontent.com/34116562/49129607-6e3a6f00-f2f6-11e8-926a-bc7c87323ce2.png)
---
```
thresh_min = 20
thresh_max = 100
s_thresh_min = 170
s_thresh_max = 255

```
---

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Perspective Transform is the Bird's eye view for Lane images. We want to look at the lanes from the top and have a clear picture about their curves. Implementing Perspective Transform was the most interesting one for me. I used values of src and dst as shown below: 

src = np.float32([[585, 460],[203, 720],[1127, 720],[695, 460]])

dst = np.float32([[320,0],[320,720],[960, 720],[960, 0]])

Also, made a function warper(img, src, dst) which takes in the BInary Warped Image and return the perspective transform using cv2.getPerspectiveTransform(src, dst) and cv2.warpPerspective(img, M, img_size, flags=cv2.INTER_NEAREST). The results are shown below:


![output_1](https://user-images.githubusercontent.com/34116562/49129781-1d774600-f2f7-11e8-80df-2d0d6b0a3950.png)
![output_2](https://user-images.githubusercontent.com/34116562/49129782-1f410980-f2f7-11e8-894a-94fdf6f519d8.png)
![output_3](https://user-images.githubusercontent.com/34116562/49129786-20723680-f2f7-11e8-9268-273b2526af11.png)
![output_4](https://user-images.githubusercontent.com/34116562/49129788-223bfa00-f2f7-11e8-9aed-0555271d53ed.png)
![output_5](https://user-images.githubusercontent.com/34116562/49129790-2405bd80-f2f7-11e8-90c8-61f1f9490cab.png)
![output_6](https://user-images.githubusercontent.com/34116562/49129795-2700ae00-f2f7-11e8-8987-eda7e5cc51a2.png)

---
```
def warper(img, src, dst):

    # Compute and apply perpective transform
    img_size = (img.shape[1], img.shape[0])
    M = cv2.getPerspectiveTransform(src, dst)
    warped = cv2.warpPerspective(img, M, img_size, flags=cv2.INTER_NEAREST)  # keep same size as input image

    return warped

```
---

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

After implementing all the steps, it's time to create the pipeline for one image. Created a function process_image() as the main pipeline function. Also, I put the Radius of Curvature and Center Offset on the final image using cv2.putText() function. The result is shown below: 

##### Lane Area Drawn without Information:

![output-without-info](https://user-images.githubusercontent.com/34116562/49130028-fec57f00-f2f7-11e8-8293-34e09bc07880.png)

##### Lane Area Drawn with Radius of Curvature and Central Offset:

![output-with-radius-and-offset](https://user-images.githubusercontent.com/34116562/49130030-008f4280-f2f8-11e8-8ed1-8a1aad84de5a.png)

---
## pipeline for video stream

```
white_output = 'test_videos_output/project_video_output.mp4'
clip1 = VideoFileClip("project_video.mp4")
white_clip = clip1.fl_image(process_image) #NOTE: this function expects color images!!
%time white_clip.write_videofile(white_output, audio=False)

```
End :raising_hand:
