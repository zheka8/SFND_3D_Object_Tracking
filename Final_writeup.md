# Sensor Fusion Camera 3D Object Tracking Final Project Writeup
Eugene Klitenik

## FP.0 Final Report
This report covers rubric points FP.0 to FP.6, with a separate file (PerformanceEval.txt) provided containing detailed times and statistics.

## FP.1 Match 3D Objects
As recommended by the instructor, 3D object matching is implemented by using a multi-map and determining the highest occurence of matches between frames. This is implemented in camFusion_Student.cpp matchBoundingBoxes. After storing all potential matches in the multi-map, all match candidates in the multi-map which share the same box ID in the prev frame are found and counted. Bounding boxes with highest number of occurences are then determined and store in the map that is passed by reference to the function. This provides the mechanism for returning the result to the main function.

## FP.2 Compute Lidar-based TTC
Lidar based Time-To-Collision is computed using only the Lidar measurements from the matched bounding boxes between the current and previous frame. In order to be robust to outliers, the mean of the x-coordinate of all points is computed to use in the TTC calculations. This prevents an outlier being close to the ego vehicle in resulting in erroneous TTC. This could be tuned further to use a statistical approach based on the distribution of points. For example, the min point within 2 standard deviations could be calculated instead. The code for this calculation is in computeTTCLidar.

## FP.3  Associate Keypoint Correspondences with Bounding Boxes
Camera based TTC estimation is performed just like demonstrated in Lesson 3 of this course. Prior to performing camera based TTC, all keypoint matches that belong to each 3D object are found based on whether they are in side the region of interest. This is done in clusterKptMatchesWithROI. 

## FP.4 Compute Camera-based TTC
Camera based TTC is computed using a threshold based on the median distance of keypoint matches. This code is based on Lesson 3 of this course. Estimation is done in the computeTTCCamera function.

## FP.5 Performance Eval 1
All Lidar TTC estimations seemed to be within reasonable values, ranging from about 16 seconds at the start and decreasing to about 8 seconds. These values seem approximately correct based on the vehicle motion. Lack of cases with Lidar TTC being way off is likley because the mean x-coordinate of Lidar points is used for estimating TTC. This is very robust to outliers unless the number of outliers is large enough to skew the mean.

## FP.6 Performance Eval 2
Certain detector / descriptor combinations resulted in TTC values that seemed out of family with other camera TTC estimates and lidar TTC estimates. For example, Shi-Tomasi and BRIEF resulted in a 20s TTC whereas Lidar TTC was closer to 13 seconds. This is likely due to the number and quality of matches resulting form this detector / descriptor combination. In another case, Shi-Tomasi / FREAK resulted in a TTC of 11 seconds whereas the Lidar TTC for that frame was closer to 16 seconds. Similary, Harris / BRIKS resulted in a 9 second difference between Lidar and Camera TTC. In this case the number of keypoints detected was very low (~80) and is likely the cause of the large difference in TTC. Harris / BRIEF also resulted in a small number of keypoints and similarly had an outlier for a TTC. As did Harris / ORB,  Harris / FREAK, and Harris / SIFT.

Based on the results, the top 5 detector / descriptor combinations are below:
| Det |	 Desc	|  TTC Camera (avg)	| TTC Lidar (avg)|	 TTC Abs Delta (avg)|
|-----|---------|-------------------|----------------|----------------------|
|FAST	  |SIFT	|11.8415	|11.7444	|0.0971107|
|SHITOMASI|ORB|	11.8925	|11.7444	|0.148146	|
|SIFT	  |BRISK	|11.9215	|11.7444	|0.177155	|
|FAST	  |ORB	|11.9354	|11.7444	|0.191041	|
|FAST	  |BRIEF|	12.0361	|11.7444	|0.291723	|