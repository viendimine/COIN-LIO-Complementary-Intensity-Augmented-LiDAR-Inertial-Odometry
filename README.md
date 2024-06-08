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
