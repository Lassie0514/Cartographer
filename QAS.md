# Stakeholder Analysis - Deliverable M2 - Team 2
This document presents Team 2's recent work determining the ASRs for the Cartographer project. We present a listing of the ASRs and a Utility Tree diagram to present this important project data.

## Group Members
1. [Liam Day](https://github.com/liamday "GitHub Page")
2. [Nathan Harmsworth](https://github.com/NHarmie "GitHub Page")
3. [Jingshan Zhang](https://github.com/Lassie0514 "GitHub Page")
4. [Zhe Chen](https://github.com/SiRumCz "GitHub Page")
5. [Beijia(Frances) Yu](https://github.com/francesyu90 "GitHub Page")
6. [Kurt Fairfield](https://github.com/kurtfairfield "GitHub Page")

 ## Background

 Google Cartographer is a project that was originally a Google in-house project that has now become open source. The original development information is not available, but a few key developers names can be picked out as having ownership of the project immediately after the project went public.

The mission statement of Google Cartographer is to advance Simultaneous Localization and Mapping [(SLAM)](https://en.wikipedia.org/wiki/Simultaneous_localization_and_mapping) as a technology, with a focus on Light Detection and Ranging (LiDAR) SLAM [[1]](https://opensource.googleblog.com/2016/10/introducing-cartographer.html "Google Blog on Cartographer"). The majority of work done for the project is done by the original Google group, though some newer developers have worked on the project since it went open source. 

The project has a large variety of users. Users include hobbyists and researchers as well as professional users like Google and Toyota [[1]](https://opensource.googleblog.com/2016/10/introducing-cartographer.html "Google Blog on Cartographer"). The project has been deployed onto a variety of robotics such as the Toyota HSR and a Levo RDS equipped Vacuum [[2]](https://research.google.com/pubs/pub45466.html "Research at Google Publication"). 

![2D SLAM Example GIF](https://j.gifs.com/wp3BJM.gif)

Cartographer has two main purposes; to provide localization of a device on a virtual map and to generate (or update) that very same map. The system requires integration with a robotics platform to provide sensor input, but once connected, the system can obtain 2D or 3D maps of a local area. The localization component involves computing the trajectory of a sense enabled device through a given metric space. The input to the system is sensor data and the output is a best estimate of the trajectory of the device that provided the sensor readings. This is a probabilistic process compare a sensor reading with the base map in an attempt to find a high probability match, thus indicating a specific location and pose for the device.

Additionally, cartographer relies on a module to take sensor readings and integrate a series of these into a 2D or 3D map that attempts to minimize the inconsistencies  created by the various readings taken at different time instances from different locations and poses. The system for localization is by necessity a near-realtime system. Processing power of the device requiring SLAM is often a limiting factor as the computations for localization can be significant.
 
 ## Architecturally Significant Requirements
 
 Architecturally Significant Requirements or ASRs are project requirements that if absent, would have a profound impact on the project outcomes. An ASR necessarily has a high business value.
 
 ASRs can be categorized into one of the following three areas:
 1. Functional Requirement - how the system must behave or react to stimulus
 2. Quality Attribute Requirement - a qualification of the functional requirement
 3. Constraint - a design decision with no degrees of freedom
 
 ### Listing of ASRs
 
 We considered the incoming business goal analysis as well as a comprehensive review of the documentation and online commentary sources to develop this list of ASRs for the Cartographer project. The system is meant to be implemented on mobile computing hardware, largely for researchers, so this playing an important part in our selection process.
 
 1. Interface using a Publish/Subscribe pattern with downstream consumers/creators of the data (ROS, etc.)
 2. Operate such that performance can be tuned to deliver trajectory projections, as messages in near real time
 3. Be able to readily access and process data from COTS sensor arrays and other system inputs
 4. Create an abstractable design that can be used in multiple environments and moved between said environments
 5. Allow the software to be version controlled during the course of large scale deployment
 6. Changes to the software can be tested on the fly to allow public involvement not to interfere with code integrity
 7. Mapping data should be saved in a reasonable manner
 8. Algorithm must be able to maintain the relative position of the sensor accurately
 9. Code must be easy to understand in order to be modifiable by other developers
10. It should openly support multiple sensors and platforms
11. Algorithm must be able to optimize both generated and visualized floor plans in real-time
12. Hardware expectations : 64bit CPU, ideally a core i7, to avoid libeigen alignment problems
13. Algorithms implemented must achieve real-time loop closure
14. 2D and 3D mapping algorithms should both be efficient

 ## Quality Attribute Scenarios
 
 QAS are used to quantify the qualitative nature of the attribute to be able to measure success. They require;
 * Source 
 * Stimulus
 * Environment 
 * Artifact
 * Response
 * Response Measure 
 
 #### Performance
| Scenario name| Process Position Data|
|:------------- |:------------- |
| Business Goals | Effective algorithm for calculating position, pose and trajectory of a mobile robot |
| Quality Attributes | Performance |
| Stimulus | Deploy the device in operating status |
| Stimulus Source | User needing localization and hardware request |
| Response | Process data from sensor and sending back mapping response |
| Response Measure | Mean accuracy <= 5mm |

#### Interoperability
| Scenario name| Cross-Platform Communication |
|:------------- |:------------- |
| Business Goals | Achieve inter-platform communication |
| Quality Attributes | Interoperability |
| Stimulus | Need for an alternate or novel platform to be able to utilize the system |
| Stimulus Source | User implementing on the other paltform, software based |
| Response | Able to read and write incoming and outgoing data in message form |
| Response Measure | Successfully send and receive data message or enable error correcting algorithms on failure |

#### Usability
| Scenario name| Save map to image |
|:------------- |:------------- |
| Business Goals | Whe system ceased operation, there must be an algorithm to store the resulting map to an accessible image. |
| Quality Attributes | Usability |
| Stimulus | Need from user to save the map to the computer |
| Stimulus Source | User stops the system and wants to view the map |
| Response | Able to save the resulting map had so far and output a recognizable format |
| Response Measure | Successfully save the map to the local machine or enable error handler |
 
 ### Listing of QASs
 
 As this system is a real time or near real time system, attributes like performance and usability were important. Interoperability and maintainability were also key issues which address the 'Advancing and democratizing SLAM as a technology' goal. It is not possible to draw a 1:1 relationship between goal and these QASs, as many address several business goals simultaneously.  Availability and Security play a role, but as the systems are usually used with other systems that address these areas as a group, these were not dominant in our analysis.
 
 #### Usability 
* When the system is in running mode, localization is delivered as a discrete message to the subscribing system in a format that results in a successful interpretation by the client device. Technical Complexity - medium : Business Priority - medium 
* When the system has ceased operation, the resulting map created by the algorithm must be stored as an image. The map can be veiwed on any jpeg viewed. Technical Complexity - medium : Business Priority - high

 #### Interoperability
 * When completing a build of the project, the build should be self-contained and deployable on multiple environments. Compatible with advertised ROS distributions at a minimum. Technical Complexity - high :  Business Priority - high
 
 #### Modifiability
 * Whenever the code base is being modified, by a development team, there will be versioning on the modifications. Each code change will have a separate version associated with it. Technical Complexity - low : Business Priority - low
  
 #### Performance
   * When the system is in running mode, the deployable position must be within 5mm of the algorithms calculated position. Technical Complexity - high : Business Priority - high
  * When an client of the system requests a localization or trajectory approximation, based on sensor data, the system will provide a valid response, with an estimate of error in the approximation, in less than 500 ms. Technical Complexity - high : Business Priority - medium
  
 #### Configurability
  * When designing an environment to deploy the project to, the sensor chosen must be modular, replaceable and accessible to designers to allow for flexibility in design. Compatible with all COTS as well as custom laser range sensors. Technical Complexity - low :  Business Priority - high 

#### Maintainability 

  * When a developer submits a change to the code, the project shall be tested to ensure the change meets a set of testing parameters. The change passes all automated testing parameters. Technical Complexity - medium : Business Priority - high
 
 ## Utility Tree
 
 ![resources](https://docs.google.com/drawings/d/e/2PACX-1vQUFx-Zpw_iB_SLsRlsA6kfiS4o-y2c2wRL8s5F-fqpn_3FF5oq4sWYcNh9fJjvVkemkM7C6sOuhzW7/pub?w=960&h=720)
 
 ## Summary
 
 This deliverable presented some key Architecturally Significant Requirements or ASRs for the Cartographer Open Source project. These were categorized and then presented as scenarios in a Utility Tree with both technical and business feasibility estimated for each.
