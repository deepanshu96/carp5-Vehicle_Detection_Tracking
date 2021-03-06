**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

---
### Writeup / README

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text](https://github.com/deepanshu96/carp5/blob/master/output_images/t1.png)

I used the udacity quiz hog pipeline function to extract the hog features from the image, moreover I used the LUV color space in order to extract the hog features(I used the L channel to be more precise) as it was giving the most robust result.
The code pipeline for hog extraction is given below:-
```
def get_hog_features(img, orient, pix_per_cell, cell_per_block, 
                        vis=False, feature_vec=True):
    # Call with two outputs if vis==True
    if vis == True:
        features, hog_image = hog(img, orientations=orient, 
                                  pixels_per_cell=(pix_per_cell, pix_per_cell),
                                  cells_per_block=(cell_per_block, cell_per_block), 
                                  transform_sqrt=True, 
                                  visualise=vis, feature_vector=feature_vec)
        return features, hog_image
    # Otherwise call with one output
    else:      
        features = hog(img, orientations=orient, 
                       pixels_per_cell=(pix_per_cell, pix_per_cell),
                       cells_per_block=(cell_per_block, cell_per_block), 
                       transform_sqrt=True, 
                       visualise=vis, feature_vector=feature_vec)
        return features
```

Here is an example using the `LUV` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text](https://github.com/deepanshu96/carp5/blob/master/output_images/t2.png)

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and the best hog image representation was being given by orientation 9 and pixels_per_cell = (8,8). Also these parameters were mentioned in udacity hog pipeline quiz and I preferred to use them as they were quite robust in the quiz also.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

Before training the classifier, I preprocessed the image using hog features, color histogram, and spatial binning for each image and combined the features obtained. Also I flipped each image along y-axis in order to generate more data for training the classifier. After that I applied an SVM classifier with linear kernel and obtained a good test set accuracy of '0.9958'. The code for the classifier is given below :-

```
xtrain = np.vstack((f1,f2)).astype(np.float64)
ytrain = np.hstack(( (np.ones(len(f1))) , (np.zeros(len(f2))) ))
print(xtrain.shape)
print(ytrain.shape)

X_train, X_test, y_train, y_test = train_test_split(xtrain, ytrain, test_size=0.2, random_state=22)
print("done1")

# Fit a per-column scaler
X_scaler = StandardScaler().fit(X_train)
# Apply the scaler to X
X_train = X_scaler.transform(X_train)
X_test = X_scaler.transform(X_test)

svc = LinearSVC() 
t=time.time() 
svc.fit(X_train, y_train) 
t2 = time.time()
print(round(t2-t, 2), 'Seconds to train SVC...')
print('Test Accuracy = ', round(svc.score(X_test, y_test), 4))
print("done")

```

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I implemented sliding window search in two ways as was given in the udacity quizes. First way was to search each image by selecting a frame and then extracting all the features for that frame and testing it with our trained model from previous steps. If an image of car was obtained, a rectangle was drawn. The second method was hog subsampling window search in which hog parameters were calculated for a single dimension for the entire image. This method was quite fast and robust and I also used it in the final pipeline. 

Search and classify 
![alt text](https://github.com/deepanshu96/carp5/blob/master/output_images/t4.png)

Hog Subsampling
![alt text](https://github.com/deepanshu96/carp5/blob/master/output_images/t5.png)

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using LUV 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text](https://github.com/deepanshu96/carp5/blob/master/output_images/t5.png)

The main step that I used to optimize the performance of classifier was to use hog subsampling with multiple windows and window sizes in order to detect cars that were far and near to our vehicle in the project video. Also I used other steps to remove false positives and multiple rectangular windows formed around vehicles which are mentioned below.

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](https://github.com/deepanshu96/carp5/blob/master/Major.mp4)

[Youtube link](https://www.youtube.com/watch?v=ctIXumr295w)

#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from  frame of video:- 

![alt text](https://github.com/deepanshu96/carp5/blob/master/output_images/t6.png)


### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text](https://github.com/deepanshu96/carp5/blob/master/output_images/t7.png)

### Updated

I stored previous 8 frames of the video(heatmap frames) and averaged over them to define the rectangular box around the vehicles. This helped me attain a better window detection and a lot smoother without any jitters. The code for the above procedure is given below:- 
```
def rollheat(heatmap):
    global history
    ans = np.zeros_like(heatmap)
    i = 1
    x = len(history)
    for hist in history:
        ans += hist
    ans+= heatmap
    
    ans = ans/x
    return ans
```

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I used two sliding frames to identify car images in the project video, one was aimed to identify far away vehicles and the other was used for identifying the near ones. My pipeline did not perform well to cover the whole car and was quite jittery. But the pipeline used, robustly identified the cars in video majorly and also there were false positive frames were minimised to a large extent. 

To make the pipeline more robust, we can store the vehicle locations of previous frames and perform a targeted search around those coordinates only in order to maximise the results. 

Also we can use Yolo object detection technique which is way more robust than conventional image processing and can identify various other objects and people too. 

