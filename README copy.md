# Introduction

Google Cartographer is a project that was originally a Google in-house project that has now become open source. The mission statement of Google Cartographer is to advance Simultaneous Localization and Mapping \(SLAM\) as a technology, with a focus on Light Detection and Ranging \(LiDAR\) SLAM. The majority of work done for the project is done by the original Google group, though some newer developers have worked on the project since it went open source.

Cartographer has two main purposes: 1\) to provide localization of a device on a virtual map and, 2\) to generate \(or update\) that very same map. The system requires integration with a robotics platform to provide sensor input, but once connected, the system can obtain 2D or 3D maps of a local area. The localization component involves computing the trajectory of a sense enabled device through a given metric space. The input to the system is sensor data, and the output is the best estimate of the trajectory of the device that provided the sensor readings. This is a probabilistic process compare a sensor reading with the base map in an attempt to find a high probability match, thus indicating a specific location and pose for the device.

Additionally, cartographer relies on a module to take sensor readings and integrate a series of these into a 2D or 3D map that attempts to minimize the inconsistencies created by the various readings taken at different time instances from different locations and poses. The system for localization is by necessity a near-real-time system. The processing power of the device requiring SLAM is often a limiting factor in the computations for localization can be significant.

  


