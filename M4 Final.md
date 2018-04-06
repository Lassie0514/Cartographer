# Component and Connector View - Deliverable M4 - Team 2

# Table of Contents
- [C&C View](#c-and-c-view)
  * [Voxel Filter Pipeline](#voxel-filter-pipeline)
  * [Submap Generator Pipeline](#submap-generator-pipeline)
  * [Pose Extrapolator Pipeline](#pose-extrapolator-pipeline)
- [Element Catalog](#element-catalog)
  * [Laser Range Data](#laser-range-data)
  * [Voxel Filter](#voxel-filter)
  * [Ceres Scan Matching](#ceres-scan-matching)
  * [Submaps](#submaps)
  * [Robotic Odometry](#robotic-odometry)
  * [Pose Extrapolator](#pose-extrapolator)
- [Context Diagram](#context-diagram)
- [Interface Documentation](#interface-documentation)
  * [Voxel Grid Interface](#voxel-grid-interface)
  * [Adaptive Voxel Filter Interface](#adaptive-voxel-filter-interface)
- [Variability Guide](#variability-guide)
- [Rationale](#rationale)
- [Related Views](#related-views)

# C and C View

## Voxel Filter Pipeline

![image alt text](https://github.com/SENG480-18/project-team2/blob/master/pipe1.png)

Voxel Filter Pipeline, which is between the input of LiDAR data and the submap generator component, acts as the primary data input pipeline in cartographer. The input of the voxel filter consists of one three dimensional float vector marking the origin and two point cloud structures marking hits or misses from the scanner. Point clouds are a grouping of three dimensional vectors defined by floats. The filter itself takes in three integers as [modifiers](https://github.com/googlecartographer/cartographer/blob/master/cartographer/sensor/proto/adaptive_voxel_filter_options.proto) that change the output. These modifiers are the max length of the voxels edge, the minimum number of points that must pass through this filter, and the max range the vector can be from the origin to pass through the filter. If the minimum number of points is not met, the max length is increased until the minimum can be met. After passing through the filter, the result is a hits point cloud and a misses points cloud with all the vectors that met the filters criteria.

## Submap Generator Pipeline

![image alt text](https://github.com/SENG480-18/project-team2/blob/master/Pipe%203.png)

The Submap Generator Pipeline deals with the submap generation and the Ceres scan matching algorithm. The pipeline is placed after the voxel filtered point clouds are created and acts as the map builder for Cartographer. The input from the voxel filter pipeline is point clouds outputted from the Voxel Filter pipeline. This pipeline also takes in the pose from the pose extrapolator pipeline which is stored as a Rigid3d data type. Rigid3d consists of two main parts, a three dimensional vector with the data stored in doubles, as well as a quaternion that also has its data points stored as doubles. A quaternion is a representation that makes spatial rotation calculations fast and efficient. From a data type perspective, they can be thought of as a 2 x 2 matrix of complex numbers, or typically represented for computation, as a 4 x 4 real matrix.

Inside the filter, the point clouds are matched onto the submaps and a new pose is sent to the pose extrapolator pipeline as a poseGraph structure, this is the pipelines output. The [poseGraph](https://github.com/googlecartographer/cartographer/blob/master/cartographer/mapping/proto/pose_graph.proto) structure consists of a submap ID and node ID, as well as a rigid3d structure with translation and rotation weights added on.

## Pose Extrapolator Pipeline

![image alt text](https://github.com/SENG480-18/project-team2/blob/master/pipe2.png)

The pose extrapolator pipeline takes information from Ceres the Odometry to infer position and orientation at a time. The robotic odometry sends the pipeline the current system time as a 64 bit integer as well as a Rigid3d structure of its pose calculated by the robots delta position. The Ceres scan matching algorithm also sends the pose that was used during its last calculation as another source of input for this pipeline. Using both of these pose inputs, the extrapolator checks to make sure that the information is valid, and infers pose if one of the two seem invalid based on previous pose extrapolations. The output is sent the Ceres scan matching pipeline as a Rigid3d structure. 

# Element Catalog

## Laser Range Data

This filter element represents a hardware source of data, in our case a LiDaR sensor module. This device reports time stamped point clouds of data in either 1, 2 or 3D. In some cases 1 or 2D devices are oscillated or swept across an area to create higher dimensional views. In any case, the data set is referenced from a device origin, and is a series of time stamped, ray tracing, distance to return measurements (laser hit something solid) along any number of vectors originating from the origin and radiating into spatial areas of interest. 

![https://goo.gl/images/WVrptG](https://github.com/SENG480-18/project-team2/blob/master/UserData_PointCloud3_Large.jpg)

[http://velodynelidar.com/hdl-64e.html](http://velodynelidar.com/hdl-64e.html)

This data format is a function of the mechanism of collection, but can be used to generate point clouds for series of instants in time as shown in the sample image above, created from a rotating line scanner with 64 discrete points or rays. Rotation frequency of the device is relevant and therefore corrections for the moving origin are required to compensate for robot motion. Also notice the shadow cast by the person near the center of the point cloud.

## Voxel Filter

The voxel filter element takes the incoming range data and converts it to a sparse point cloud.  Returns beyond a certain range or distance are trimmed to reduce the overall point cloud size to aid in achieving real time performance. The sparse point cloud is then binned in coarse resolution voxels.  If there are sufficiently high number of returns in a given voxel, it is considered solid, not enough points, empty space. These control criteria can be specified via the [LUA dictionary](https://github.com/kurtfairfield/cartographer/blob/master/cartographer/common/lua_parameter_dictionary.cc). Voxel size is also adjustable to allow performance to be tuned dynamically. 

![image alt text](https://github.com/SENG480-18/project-team2/blob/master/maxresdefault.jpg)

[Excellent video link showing progressive build - https://www.youtube.com/watch?v=dTTrCQuamPk](https://www.youtube.com/watch?v=dTTrCQuamPk)

Cartographer uses a coarse voxel filter for fast localization and loop closure as well as a higher resolution voxel representation for display and refinement of the global map. A sample image of a sparse (much empty space) voxel map is shown above for reference.

## Ceres Scan Matching

[Ceres Scan Matching](https://github.com/ceres-solver/ceres-solver) filter element is a third party library that implements spatial matching algorithms. It can take a voxel submap and find the highest probability location and orientation or pose estimate for that given submap. Due to the successive scan nature of the process, the next submap is highly overlapping typically, so that the maps, once matched and located with one another create a self seeding composite map or current map.

To assist in successfully locating a given set of scan data, a high fidelity estimate of the pose origin of that particular scan data set is very useful in reducing convergence time in the Ceres matching logic. This estimate is based on where the odometry component expects the scan at a given time to have originated. The better the estimate, the faster the match time.

## Submaps

Submaps are individual voxel maps that show the probability that a voxel is occupied or more technically; a regular probability grid where each discrete grid location represents the probability that the corresponding physical grid location is occupied or empty, in reality. 

When several submaps are aggregated together or properly aligned with respect to each other, they provide multiple estimates on voxel occupancy. These multiple probabilities are again aggregated to create the current map, or the collective best estimate of the available scans of the surrounding surface area.

Currently, the Cartographer group is moving to allow the combination of multiple sources of submaps to be used to create a common current map. This would allow multiple agents to complete SLAM collectively. This is visible by the [gRPC code being morphed into the codebase](https://github.com/kurtfairfield/cartographer/tree/master/cartographer_grpc).

## Robotic Odometry

The robot odometry is a combination of spatially aware sensors. Often a set of rotary encoders to map wheel based motion. An IMU is also integrated to provide relative change in inertial forces as well as provide a consistent gravity vector to help estimate pose. These aggregate data are forwarded to the pose extrapolator and onto the scan matching components to aid in better estimating the origin location of the scan data in the composite submap structure and hence better predicting the submap location and orientation.

## Pose Extrapolator

This filter element takes data from robotic odometry component and uses this ground truth data to extrapolate the robot pose - location and orientation of the device as well as the range sensor. It also is updated from the scan matching functions and provides updated pose estimates for future scan matching by the Ceres component.

# Context Diagram

The following diagram shows the context for the dataflow of Cartographer. The system has two main inputs, Laser Range Data and Robotic Odometry Data. Each of the inputs move through a pipeline which accepts some feedback from other pipelines as seen below. 

![image alt text](https://github.com/SENG480-18/project-team2/blob/master/M4%20-%20Pipeline.png)

The dataflow is relatively convoluted, but can be abstracted to follow a pattern as above. Output from this process is the area map, as well as providing trajectory (location, pose and rate of change of each) continuously in real time to the ROS client.

# Interface Documentation

## Voxel Grid Interface

1. Identity

    1. SubmapGenerator

2. Resources

    2. Syntax:

        1. [Submap](https://github.com/googlecartographer/cartographer/blob/def4048e95b07edc8e30009a0bf245a43baa0798/cartographer/mapping/submaps.h#L59) SubmapGenerator([PointCloud](https://github.com/googlecartographer/cartographer/blob/def4048e95b07edc8e30009a0bf245a43baa0798/cartographer/sensor/point_cloud.h#L32) pointCloud) ~ or;

        2. [Submap](https://github.com/googlecartographer/cartographer/blob/def4048e95b07edc8e30009a0bf245a43baa0798/cartographer/mapping/submaps.h#L59) SubmapGenerator([TimedPointCloud](https://github.com/googlecartographer/cartographer/blob/def4048e95b07edc8e30009a0bf245a43baa0798/cartographer/sensor/point_cloud.h#L39) timedPointCloud)

    3. Semantics

        3. Pre-Condition: Takes in point clouds which are formed by a set of ray tracing range data coming from LiDaR sensor. Timed point cloud is a point cloud with seconds since the last point was acquired stores as the last point.

        4. Post-Condition: Each point cloud is placed on a voxel grid that says where it is in relation to the other point clouds. When enough point clouds have been placed submaps can be generated.

3. Datatype

    4. Point Cloud is a collection of three dimensional vectors. Each vector has three floats that store the X, Y, and Z coordinates. The timedPointCloud contains a fourth float that has time since last point was acquired.

4. Error Handling

    5. Compares point clouds to others on the voxel grid and can change other point clouds grid location if needed once more data has shown it to be placed the the wrong grid.

5. Rationale

    6. The interface was designed to be updated based on data coming in from the voxel filter so that outlying data can be smoothed out and a proper submap can be created.

## Adaptive Voxel Filter Interface

1. Identity

    1. VoxelFilter

2. Resources

    2. Syntax

        1. [PointCloud](https://github.com/googlecartographer/cartographer/blob/def4048e95b07edc8e30009a0bf245a43baa0798/cartographer/sensor/point_cloud.h#L32) AdaptivelyVoxelFiltered(const [proto::AdaptiveVoxelFilterOption](https://github.com/googlecartographer/cartographer/blob/def4048e95b07edc8e30009a0bf245a43baa0798/cartographer/sensor/proto/adaptive_voxel_filter_options.proto#L19)& options, const [PointCloud](https://github.com/googlecartographer/cartographer/blob/def4048e95b07edc8e30009a0bf245a43baa0798/cartographer/sensor/point_cloud.h#L32)& point_cloud)

    3. Semantics

        2. Pre-condition:Takes in filter options which are stored as a proto message and takes in point cloud which is formed by a set of range data coming from sensors.

        3. Post-condition: After running through the function the original variables won’t change, but the function returns a new filtered point cloud which is based on user’s options such as maximum voxel edge and minimum number of scan points in each cell.

3. Datatype

    4. Point Cloud is a collection of three dimensional vectors. Each vector has three floats that store the X, Y, and Z coordinates. Options is a proto structure that contains max_length, min_num_points, and max_range which are stored as doubles.

4. Error Handling

    5. It uses some conditional statements to handle the errors on point cloud, and if any error occurs it will return the original point cloud.

5. Rationale

    6. Before a set of sensor data(point cloud) can be inserted into the voxel grid, cartographer runs a real-time voxel filter on this data to make the voxels stay at a consistent size. This improves the voxel grid creation which improves the accuracy of the submap when enough data has been gathered.

# Variability Guide

Robots are not always going to be looking at the inside of a lab full of squared off edges and solid walls that provide clear edges to build a map from. In the field, robots need to be able to generate maps with obstacles such a rising slopes or lack of defined edges on the area being mapped. Due to the difference in various terrains that robots operate in Cartographer needs to be able to sort the incoming data in a way that still performs when in any of the possible terrains. Another issue is that the robots orientation or position changes in no clear pattern to Cartographer as they may be human controlled. Placing the data into a submap grid greatly improves the variability of the overall system in both of these situations as data can be compared to only certain grids.

Another form of variability is language variability. ROS is the most common implementation of the Cartographer controller and is written in C++ like Cartographer is. However, other implementations that interface with the Cartographer SLAM service do so via another platform and some of which are written in languages other than C++. To accommodate this variability, in support of the [interoperability QAS presented in M3](https://github.com/SENG480-18/project-team2/blob/master/M3%20Final.md#quality-attribute---interoperability) the developers implemented protocol buffers to create a simple way to maintain a simple, XML like format of global data types. Then at compile time a precompiler is run to generate the language specific classes and header files, which are then compiled with the existing codebase, which references these data structures.

# Rationale

Cartographer employs a variety of data types to support good information flow. Our C&C view of Cartographer covers 3 pipelines that are core to Cartographer. Cartographer also aims to address performance and interoperability goals with its data flow design. 

Cartographer addresses performance concerns by allowing different systems to run concurrently. LiDAR data can be gathered, but the actual processing can be delayed by the system until an appropriate amount of data is present. In Cartographers Submaps component small maps are collected until all of the submaps can be assembled with reasonable confidence, by the Ceres Problem Solver, into a single map. By deferring the final assembly of the map until all the submaps are correctly assembled or positioned, the processing time spent by Ceres is reduced. In order to increase simplicity of mapping, the maps are filtered, in the Voxel Filter, to lower resolution. This optimization helps Cartographer run in real time.

Interoperability goals in Cartographer are key to Cartographers design. Proto is used in Cartography to mark up data types in a platform independent way. Other languages can access proto structures as needed which helps lower coupling in Cartographer. Data from sensors is passed through ROS allowing a standardization of sensor input. Cartographer is designed to handle basic sensor data from the OS in order to work with a variety of sensors appropriately.

In our Context Diagram three major pipelines are outlined. Pipeline 1 handles data coming in from the Lidar sensors, going through the Voxel Filter, and exiting towards Submaps, Pose, and Ceres. Pipeline 2 covers data coming from Robotic Odometry, Ceres, and the Voxel Filter which is then stored in Pose for use in Submaps and Ceres. Finally, Pipeline 3 Covers the transition of information from Ceres to Submaps and the various inputs to both. 

Voxel Filter Pipeline, which is between the input of LiDAR data and the submap generator component, acts as the primary data input pipeline in cartographer. The Voxel Filter takes in a collection of points in 3D space that represent the laser’s hits as well as the origin, from which the lasers are shot. The filter then performs a process similar to clustering in order to group points into voxels. The voxel maps created are fed into storage and more processing in the Submap Generator Pipeline. 

In the Submap Generator Data is fed into Submaps from the Voxel Filter, these submaps are stored for future use. Pose information from the Pose Extrapolator Pipeline is also stored within the Ceres Scan Matching. Ceres takes in Submaps from the voxel filter and pose information. Ceres aims to find how all the submaps align and outputs its submap organization into submaps for storage. 

The pose extrapolator pipeline aims to place the robot in the 3D maps it has created at a distinct time. Data concerning the robots movement is taken from Robotic Odometry as well as map information from Ceres. This Pose information is then sent to Ceres and Submaps for their use. 

As seen in [M3’s module view](https://github.com/SENG480-18/project-team2/blob/master/M3%20Final.md#module-uses-view) the data dependencies in Cartographer are numerable and complex. Tracing dataflow in Cartographer is quite difficult and does not support modifiability. As stated above the dataflow design instead aims to be fast and abstracted. Cartographer itself is not the area where modifiability is expressed, rather this is implemented in the ROS (or other platform) interface.

# Related Views

Please see the following additional views relevant to the project:

[M3 - Module View](https://github.com/SENG480-18/project-team2/blob/master/M3%20Final.md) (https://github.com/SENG480-18/project-team2/blob/master/M3%20Final.md)

Return to:
[Table of Contents](#table-of-contents)

