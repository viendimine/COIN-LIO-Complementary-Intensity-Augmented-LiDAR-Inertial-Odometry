# COIN-LIO: Complementary Intensity Augmented LiDAR Inertial Odometry
Patrick Pfreundschuh, Helen Oleynikova, Cesar Cadena, Roland Siegwart, and Olov Andersson. "COIN-LIO: Complementary Intensity-Augmented LiDAR Inertial Odometry" accepted at ICRA 2024.

<p align="center">
  <img width='100%' src="coin_tracking.gif">
</p>

## Problem
Traditional LiDAR-based navigation systems that rely solely on geometric point cloud registration face several key problems in certain challenging scenarios:

### 1.Geometric degeneracy
In environments like tunnels, flat fields or areas with lack of geometric features, traditional point cloud registration techniques struggle because there are not enough distinct geometric shapes or structures to match against.
### 2.Lack of Robustness
When the geometry provides little information, traditional systems become brittle and can fail catastrophically in such geometrically degenerate environments.
### 3.Failure in Real-world Challenging Scenes
Many public datasets lack truly challenging real-world scenarios. Traditional methods may perform well on these datasets but fail in practical deployments in geometrically degenerate environments.

## LIDAR Inertial Odometry?
LiDAR Inertial Odometry (LIO) is an approach that fuses data from two key sensors: LiDAR and an Inertial Measurement Unit (IMU). LiDAR provides dense 3D point clouds representing the geometric structure of the environment, while the IMU directly measures the sensor's linear accelerations and rotational motions. By tightly integrating these complementary streams, LIO algorithms can estimate the sensor's trajectory over time with high accuracy and robustness. The IMU's motion data aids point cloud registration, reduces distortion, and provides a good initial guess, while the LiDAR's mapping constrains drift errors, enhancing odometry performance compared to using either modality alone.
![LIO_system](https://github.com/vishapraj/Cv_and_Drones_EEA/assets/126682925/a30702b9-ef9c-4787-99bf-4845ec8a927c)








Traditional LiDAR odometry approaches like LOAM work well in structured environments with plenty of geometric features like walls, corners, and planes. LOAM extracts edge and plane points from the point cloud and registers them against a map built incrementally. This approach performs robustly when there are enough distinct geometric features to match against.
However, consider a scenario where the robot enters a long tunnel or a vast open field. In such geometrically degenerate environments, there are few distinct edges or planes to match against. LOAM and similar feature-based methods struggle in these cases, leading to potential failures or inaccuracies in odometry estimation.
![Mappings](https://github.com/vishapraj/Cv_and_Drones_EEA/assets/126682925/bbf82793-67e3-41b5-8184-ed14a2c884a9)


## Solution-COIN-LIO

COIN-LIO is a new LiDAR-based navigation system that combines information from the 3D point cloud geometry as well as the intensity (brightness) values of the LiDAR returns.It is designed to work better in challenging scenarios like tunnels or flat fields, where there is a lack of geometric features that traditional LiDAR odometry systems rely on.
The system takes the intensity values from the LiDAR and projects them into a 2D image. It then applies novel image processing techniques to improve the brightness consistency within the image and across different scenes.
The processed intensity images are used as an additional modality (source of information) along with the point cloud geometry.
A new feature selection scheme is proposed that detects uninformative directions in the point cloud registration and explicitly selects patches from the intensity images that can provide complementary information.
The system then fuses the photometric (brightness) error minimization from the image patches with inertial measurements (from IMU) and the traditional point-to-plane registration in an Extended Kalman Filter.

[COIN-LIO](https://github.com/vishapraj/Cv_and_Drones_EEA/assets/126682925/cf70fc72-73f9-41fb-a7af-90ef5e2082d7)

## What COIN-LIO Adds: Photometric Component

### 1.Photometric Data from LiDAR:
* LiDAR not only provides 3D point clouds but also captures intensity (brightness) of reflected light from surfaces.
* COIN-LIO uses this intensity information as an additional data source.
### 2.Improving Brightness Consistency:
* A special filter processes the intensity images from the LiDAR point clouds to make brightness values more consistent and reduce sensor artefacts (errors or distortions).
* Example: Imagine taking a series of photos in varying light conditions. This filter would adjust the brightness so that each photo looks consistent, reducing glare or shadows.

### 3.Feature Selection:
* COIN-LIO selects specific features from these intensity images that provide valuable information where the 3D geometry might be less informative.
* Example: When a robot moves along a flat wall, the 3D shape doesn’t change much, but variations in the wall’s texture (captured by intensity) can help in understanding movement.

### 4.Feature Management:
* The system continuously checks if the selected features are still valid and detects if they get blocked or hidden (occlusions).
* Example: If the robot moves and an object comes between the sensor and the wall, the system detects that the wall’s texture features are now occluded and stops using them until they are visible again.

### 5.Integration into Kalman Filter:
* The photometric data (intensity information) is integrated into the existing Kalman Filter, which already combines LiDAR and IMU data.
* This combination helps in refining the movement estimation by using both geometric (3D shape) and photometric (brightness) information.

## Example to Illustrate
Imagine a robot navigating a corridor with plain white walls:
### 1.Without Photometric Data:
The robot uses LiDAR to detect the flat walls and IMU to understand its motion. However, in a long, featureless corridor, it might struggle to get accurate position updates due to the lack of distinctive 3D features.
### 2.With Photometric Data:
The robot also uses the intensity variations on the walls (e.g., small marks, textures, or light differences). Even if the walls are geometrically flat, the photometric data provides additional cues that help in accurately tracking the robot’s movement.

## 3-D point projection to 2-D
* The point 
𝐿
𝑗
𝑝
𝑗
=
[
𝑥
𝑗
,
𝑦
𝑗
,
𝑧
𝑗
]
L 
j
​
 p 
j
​
 =[x 
j
​
 ,y 
j
​
 ,z 
j
​
 ] represents a point in 3D space as seen by the LiDAR sensor.

### Spherical Projection:-
* This converts the 3D point into 2D image coordinates 
𝐶
𝑝
𝑗
Cp 
j
​
  using angles derived from the point's position: ![Screenshot from 2024-06-09 03-25-23](https://github.com/vishapraj/Cv_and_Drones_EEA/assets/126682925/3b1c4984-0bcc-4470-b9f2-6fddc504ecd1)
Here, 
𝑢
𝑗
u 
j
​
  and 
𝑣
𝑗
v 
j
​
  are the image coordinates, 
𝑤
w and 
ℎ
h are the image width and height, 
atan2
(
𝑦
𝑗
,
𝑥
𝑗
)
atan2(y 
j
​
 ,x 
j
​
 ) gives the horizontal angle, and 
arcsin
(
𝑧
𝑗
𝑅
)
arcsin( 
R
z 
j
​) gives the vertical angle, with:
𝑅
=
𝑥
𝑗
2
+
𝑦
𝑗
2
+
𝑧
𝑗
2

![Screenshot from 2024-06-09 03-26-51](https://github.com/vishapraj/Cv_and_Drones_EEA/assets/126682925/8db43648-8f75-4520-8413-fbf9688ba19e)

## RESULTS

### 1. Newer College Dataset:-
#### * Sensor Used:
A handheld 128-beam Ouster OS0 LiDAR.
#### * Sequences:
Various sequences are included to test different conditions, such as Cloister (large structures and slow motions), Quad-Hard (aggressive rotations), and Stairs (narrow stairways).

#### Conclusion
The proposed pipeline demonstrates superior performance and robustness across various challenging scenarios by effectively integrating intensity information with geometric data, while maintaining computational efficiency.
![Screenshot from 2024-06-09 03-42-35](https://github.com/vishapraj/Cv_and_Drones_EEA/assets/126682925/7d7d462d-b44a-4e94-81e2-ba1982a302ae)

### 2. ENWIDE Dataset:-
#### * Sensor Used:
Hand-held Ouster OS0 128-beam LiDAR with an integrated IMU (Inertial Measurement Unit).
#### * Environment:
Five distinct real-world environments were recorded, each with long sections of geometric degeneracy but with well-constrained areas at the beginning and end of the sequences.

#### Conclusion
The new dataset was meticulously designed to challenge LIO and LO algorithms with environments that lack distinct geometric features. By recording sequences in various challenging environments and including both smooth and dynamic motions, the dataset provides a comprehensive benchmark for evaluating robustness and accuracy of different approaches. This helps in understanding how well the proposed and existing methods perform in real-world scenarios where traditional geometric cues are minimal.
![Screenshot from 2024-06-09 03-49-52](https://github.com/vishapraj/Cv_and_Drones_EEA/assets/126682925/19dd95a4-734b-4323-8546-04fe88577ab5)



