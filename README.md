# Unsupervised learning : Human activity recognition

This project aims to recognise patterns in timeseries data from a pair of movement sensors on the body. 

We start with timeseries data from pairs of gyroscope and accelerometer sensors. By detecting signal shifts in the data we can obtain recogniseable patterns in segmented timeseries data. Then we use the Tsfresh automated feature extraction library on all signals in each of these segments to cluster these patterns.

The model is then trained on unlabeled data, making this an unsupervised learning project. The final goal is to obtain clusters which can be identified as a specific human activity movement.

## Installation

This step by step guide will get you to have the development environment up and running.

1. Create and activate your virtual environment
2. Install necessary libraries from `requirements.txt`
 
## Usage

The project is split into different folders and Jupyter notebook files.<br>

### /analysis

1. For visual analysis `read_signal.ipynb` is used to plot the timeseries.<br>

### /processing

2. To fine tune thresholds for changepoint detection we use the provided labeled data:<br>
`segmentation_labeled.ipynb`

3. To process one file (preprocessing, changepoint detection, segmentation, and feature extraction): <br>
`feature_extraction.ipynb`

4. To process all files in a folder : <br>
`feature_extraction_RUN.ipynb` <br>

### /clustering

5. To run the clustering and visual validation :<br>
`clustering_features.ipynb`<br>
 
## Method

### 1. **Analysis**
The analysis of data was handed over to our data-analysis team and does not fall within the scope of this project.

We simply plot the x,y,z axes of the sensor data. For each timeseries we have data for 3 (axes) x2 (sensors) x 2 (devices) = 12 signals. 

### 2. **Preprocessing**
Each timeseries is downsampled, normalised and processed with HAAR filter.

### 3. **Processing**
This step involves splitting the available timeseries data (12 signals) into segments with the ruptures library, and then extracting features of these signals with tsfresh.

#### **A. Segmentation**

**Step 1 - Tuning on labeled data**

For splitting the timeseries into different segments we use a seperate dataset with labeled movements. We use the ruptures library to detect changepoints (signal shifts) on each of the signals. The challenge here is that these movements have only a small similarity to the unlabeled data, which was obtained from live usage.

By running the file `segmentation_labeled.ipynb` we can create segments (frames) from the *labeled* timeseries data by detecting so-called changepoints in the signals. For this I use the ruptures library.

To determine the sensitivity of this detection on the *unlabeled data*, we empirically apply different threshold settings for different (sets of) signals in the *labeled data*.

For example, we split the data according to signal shifts in the x axis of accelerometer 1 with threshold 1. We do the same for the y axis of accelerometer 2 with a different threshold, and so on...

In the next image, the background colours represent different labels, and the changepoints are marked as red vertical lines.

![image](/assets/segmentation_1_signal.png)

We do this for another batch of 3 signals in the same timeseries.

![image](/assets/segmentation_3_signal.png)

Finally we add each changepoint and apply it on the timeseries data with all the signals.

Through empirical method we obtained threshold values that are satisfying for the scope of the project. To reduce the number of resulting frames we remove changepoints that are too close to eachother and replace them with their mean datapoint. 

Our aim is that the segments are more or less visibly correlated to the provided labels, as we can see in the resulting image.

![image](/assets/segmentation_12_signals.png)

The original timeseries data has now been split into a number of frames that more or less correlate with the labels. We can now use this method and apply the respective thresholds on the unlabeled data.

**Step 2 - Applying on unlabeled data**

Now we need to apply this method on our unlabeled data. A demonstration of this can be seen in the notebook file `segmentation.ipynb` which serves no other purpose.

![image](/assets/unlabeled_segmentation.png)

The processing of the unlabeled data in the above image results in 9 frames

![image](/assets/unlabeled_frame_1.png)

In this image of the first frame we plotted 6 signals and two calculated signals. 

#### **B. Feature extraction**

By using tsfresh we will extract the features of all signals in each frame that was segmented from the unlabeled data.
 
The total number of signals is 14. 

Now we can start the feature extraction on each of the frames that were found with the above method. The result is saved in a **big vector** with 783 (extracted features) for 14 (signals), which results in 783*14 = 10962 dimensions per frame. 

#### Processing all files

By running the file `feature_extraction_RUN.ipynb` we extract the features from all the available timeseries data in one go, as described above.

### 4. **Clustering**
A clustering model is applied to a selection of the features from each of the frames

With the notebook file `clustering_features.ipynb` we can visualize our newly found clusters.

The script loads all the extracted features for all found frames and tries to determine 10 clusters from this data.

Because we have many dimensions we apply PCA. We can reduce the number of dimensions to the most efficient number with the method.

The project resulted in a n-dimensional model, with a pre-determined number of 10 clusters.

Then we can validate our model by printing frames which are identified to be in a cluster

### 5. **Validation**

We plot the newly found clusters to validate similarities in the patterns. In the notebook we print out the first 3 frames that were labeled as cluster 4.
 
## Limitations

Using all signals approach and seems exhaustive, but this is an MVP with the intention to fine-tune this approach if succesful. The method could be improved by selecting only the more relevant features. This would reduce the dimensionality and processing time and might improve the clustering results.

Therefore this project has the following limitation:

1. Processing time is an issue

- Feature extraction with TSFRESH is done without selecting the most relevant features. 
- Segmentation takes a long time because we use different thresholds for different signals.

2. A better understanding of the correlation between the signals could greatly improve the models performance.

3. Determining the number of frames, features before clustering is an experimental method. 

4. Dimension reduction could be obtained by not using all features and all signals. 

 
