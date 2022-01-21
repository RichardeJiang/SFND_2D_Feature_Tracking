# Report

## Overview
In this project, we implemented various corner detectors + feature descriptors, and have concatenated their workflow such that keypoints are extracted from the images, matched, and the computation time from diff combinations are logged and compared.

## Running and Result
You can try diff combinations of detectors and descriptors by changing the string in MidTermProject_Camera_Student file, line 49 and 51. Note that for the below sections, I'm mainly computing the value for each image, and then take the average of all 9/10 data points when focusing on one particular instance/pair.

In order to record the metrics, I made some changes to the function signature, where some vectors will also be passed in, such that we could log the computation time and size. These vectors will lastly be printed, and for my runs, the output is recorded inside the file src/RawOutput.md.

## Implementation MP 1: Data buffer
This part is to add in support for a ring-buffer such that we could cinstantly push in new images / data points into the buffer vector, therefore we could compare the current buffer size against the maximum size (2 in this case), and once it exceeds, simply remove the first entry (achieved by `vector.erase()` method).

## Implementation MP 2 and 4: Add in modern detectors and descriptors
The basic Shi-Tomasi and HARRIS corner detectors have been demonstrated during lectures, so the primary focus here is to implement the other types. Interestingly, note that common practice from OpenCV tutorials is to simply specify the feature extractor via things like `detector->detectAndCompute(image1, cv::Mat(), keyPoints1, descriptors1);`. here in order to compare all combinations of detectors and descriptors, we are splitting the detection step and description step, therefore, we'd first check the passed-in detector/descriptor type in string format, and then specify the high level detector/descriptor like `detector = cv::ORB::create();`, and subsequently perform `->detect` and `->compute` separately.

## Implementation MP 3: Only retain keypoints on the preceeding vehicle
This part is mainly about manipulating the keypoints from the previous MP 2, against a pre-defined region specified by `cv::Rect`. Note that the OpenCV KeyPoint class has a field named `pt` (see explanation [here](https://docs.opencv.org/3.4/d2/d29/classcv_1_1KeyPoint.html)), which is the coordinate of the point, hence we could iterate through all the retrieved keypoints, and only keep those falling into the rectangle region.

## Implementation MP 5: FLANNBASED matching
Matcher can be specified following the tutorial [here](https://docs.opencv.org/3.4/d5/d6f/tutorial_feature_flann_matcher.html). Several notes here: first one is when using the SIFT descriptor, we should be using the L2-norm as the distance computation scheme; secondly, as specified [here](https://stackoverflow.com/q/43830849/3455398), FLANNBASED requires the descriptor to be in type CV_32F so we need to do the conversion first.

## Implementation MP 6: Distance ratio
Sample explanation can be found on the same [tutorial](https://docs.opencv.org/3.4/d5/d6f/tutorial_feature_flann_matcher.html): after we got the 2 nearest distances between match pair, to avoid false matches, we can perform a quick filter: we'll only treat the closest pair as a __valid__ pair when its distance is smaller than the second smallest value by certain amount. With the 0.8 used as the pre-defined threshold, it means 
```
if (closest_dist / second_closest_dist < 0.8) // treat the closest_dist as a valid match
```

## Performance MP 7: Number of detected keypoints comparison
The average number of detected keypoints for all 10 images can be found in the below table.

|   Detector type    | Shi-Tomasi |   HARRIS   |   FAST   |  BRISK  |    ORB    |    AKAZE    |   SIFT   |
| :---------------:  | :--------: | :---------:| :-------:| :-----: | :--------:|  :--------: |:--------:|
|Avg cnt of detection|    117.9   |    24.8    |   149.1  |  276.2  |   116.1   |      167    |  138.6   |

As for the distribution of the keypoints and the nighborhood sizes: 
* Shi-Tomasi, HARRIS, FAST share some common attributes: the detected keypoints neighborhood sizes are roughly on the same level (detected circles with similar sizes). However, it can be seen clearly that HARRIS detected far fewer points than the other 2, actually far fewer than any other detectors, which is also shown by the above table;
* BRISK, ORB, AKAZE, SIFT have varying sizes, with BRISK and ORB yielding slightly larger points than the other 2. However, BRISK and ORB also show much overlapping for the detection bubbles.

## Performance MP 8: Number of matched keypoints for all pairs
The average number of matched keypoints on all 9 image-pair can be found in the below table.

|  Detector/Descriptor |   BRISK  |   BRIEF  |   ORB   |  FREAK  |  AKAZE  |   SIFT  |
|:--------------------:|:--------:|:--------:|:-------:|:-------:|:-------:|:-------:|
|     **SHITOMASI**    |  76.6667 |  90.6667 | 85.3333 | 63.7778 |   N/A   |   103   |
|      **HARRIS**      |  13.4444 |  15.6667 | 16.1111 | 13.6667 |   N/A   | 18.1111 |
|       **FAST**       |  86.2222 |  98.1111 | 95.7778 | 74.1111 |   N/A   | 116.222 |
|       **BRISK**      |  144.222 |  149.333 | 103.667 | 121.444 |   N/A   | 182.889 |
|       **ORB**        |  72.1111 |    50    | 58.8889 | 38.4444 |   N/A   | 84.7778 |
|       **AKAZE**      |    N/A   |    N/A   |   N/A   |   N/A   | 130.222 |   N/A   |
|       **SIFT**       |  59.5556 |  66.3333 |   N/A   | 56.2222 |   N/A   | 88.8889 |

It can be seen that the matched number roughly stay at the same scale of detected points from MP 7: when the detector could mark more keypoints, then we could establish more matches. However, as AKAZE detector could only be used with AKAZE descriptor, its associated row and column are left as N/A. Additionally, the pair (__SIFT detector, ORB descriptor__) produces core-dumped error even after multiple tries, so I'm leaving it out as well.

Sample output, using the Shi-Tomasi detector and BRISK descriptor:
![Sample Results](shitomasi-brisk.png)

## Performance MP 9: Computation time for all pairs
The average computation time for each detector and descriptor (averaged over 10 images) can be found in the below table.

|  Detector/Descriptor |         BRISK        |         BRIEF        |         ORB         |         FREAK         |         AKAZE        |          SIFT         |
|:--------------------:|:--------------------:|:--------------------:|:-------------------:|:---------------------:|:--------------------:|:---------------------:|
|     **SHITOMASI**    |   28.374 / 4.73258   |   20.0672 / 1.44203  |  20.4109 / 1.20803  |   13.1788 / 44.1685   |          N/A         |    14.387 / 19.022    |
|      **HARRIS**      |   22.277 / 1.80621   |    20.1692 / 1.028   |  22.6975 / 0.947442 |   15.9006 / 43.8079   |          N/A         |   20.0442 / 22.2612   |
|       **FAST**       |   1.08345 / 2.66857  |   0.908898 / 1.69396 |   0.88179 / 1.17789 |   0.850901 / 46.7111  |          N/A         |   0.875459 / 33.4035  |
|       **BRISK**      |  431.542 / 3.67109   |   444.374 / 1.33813  |  438.758 / 5.15977  |   424.533 / 48.3636   |          N/A         |   435.894 / 66.3313   |
|       **ORB**        |  9.57067 / 1.89368   |   8.89255 / 0.73962  |   7.7929 / 4.70806  |   8.24856 / 46.0275   |          N/A         |   10.0542 / 79.5501   |
|       **AKAZE**      |          N/A         |          N/A         |         N/A         |           N/A         |   130.222 / 109.853  |          N/A          |
|       **SIFT**       |   161.46 / 1.95341   |    138.6 / 0.919097  |         N/A         |   174.587 / 49.6796   |          N/A         |    155.124 / 109.643  |

Among all the results, it's easy to see that in terms of computation speed, we should go with detector __FAST__, where on average a rough 0.8 to 1 ms is spent, while the rest will take at least 10 times longer. However, in my opinion in order to tell the best pair, the number of detected/matched points should also be considered: e.g., the combination of FAST + SIFT produces 116 matched points, the largest one for all FAST results, but the average descriptor time for SIFT is 33 ms, much more than BRISK, BRIEF, ORB. Therefore my recommendation would be:
1. FAST + BRIEF;
2. FAST + ORB;
3. FAST + BRISK.

These pairs give the fastest processing time, while maintaining a reasonable number of matched points at the same time (all around 80 to 90 matched points).
