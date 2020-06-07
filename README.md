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

## Advanced Lane Detection Project which includes advanced image processing to detect lanes irrespective of the road texture, ## brightness, contrast, curves etc. Used Image warping and sliding window approach to find and plot the lane lines. Also determined the real curvature of the lane and vehicle position with respect to center.

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
```
def cal_undistort(img, objpoints, imgpoints):
    # Use cv2.calibrateCamera() and cv2.undistort()
    ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, img.shape[1::-1], None, None)
    undist = cv2.undistort(img, mtx, dist, None, mtx)
    print( dist)
    return undist
```
End :raising_hand:
