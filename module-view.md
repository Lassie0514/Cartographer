# Module View 
This document presents Team 2's recent work examining the architecture of the open source Cartographer project. We provide a module view presentation as envisioned by the Carnegie Mellon Software Engineering Institute.

# Module Uses View

![image alt text](https://github.com/SENG480-18/project-team2/blob/master/M3%20Uses%20Diagram%206.png)

# Element Catalog

The following section presents the 9 major modules of the Cartographer system. We chose this architectural view as it presents the core components of the system. The designers did an excellent job of segmenting the code base into folders, which we follow as we present the various elements of the system shown above.

## Common

Common is primarily used for core functionalities such as object definitions, LUA integration and ensuring concurrency. There are also helper functions and cartographer port definitions.

## Ground Truth

Ground Truth contains machine learning that is core to the mapping algorithm. The code compares the current position to the position generated during the last loop and calculates the distance travelled.

## Internal

Internal has the helper functions and header files needed for Mapping, Mapping 2d, and Mapping 3d. Local trajectory builders for the three Mapping folders are also included.

## IO

I/O contains code to draw graphical representations of the programs actions during runtime. It also has filestream objects and other things for saving this information to memory.

## Mapping

Mapping is where the calculations for things such as pose, trajectory, and location probability are done. Mapping also builds the map using all the information from other files.

## Mapping 2d

Mapping 2D has similar functions to the mapping folder, although the calculations are only done in two dimensions. It creates a 2D submap and pose map as helper steps for the mapping folder.

## Mapping 3d

Mapping 3D provides functionality to create 3D pose maps and 3D submaps, as well as testing ranges in 3D space compared to the previous scans.

## Sensor

Sensor is used to take in the sensor information and turn it into variables that can be used in other functions. This includes things such as range, map position with respect to time, calculating trajectory and establishing any landmarks that can be used for position tracking.

## Transform

Transform uses Eigenvalues and matrix algebra to get information from the maps.

# Context Diagram

The following diagram shows the context for the operation of Cartographer. The main application has bi-directional flow from the hardware (mobile robot typically) which in turn takes input data from both the lidar sensor and the ground truthing odometry sensor.

System output is the area map, as well as providing trajectory (location, pose and rate of change of each) continuously in real time to the ROS client. These data artifacts can be packaged and reused as base maps for other mobile devices as if they were created by the using system.

![image alt text](https://docs.google.com/drawings/d/e/2PACX-1vQLYu96KAva7hOd7q-FZlhLAwSCJb2cpYbND56Sv-bOndEHwMJsyMGqxRk-duD9FhN8yWKLff4VGq3B/pub?w=960&h=720)
# Behaviour Diagrams
We selected two critical QASs to examine using behaviour diagrams. These QAS are presented below for clarity. 

## Quality Attribute - Performance

Performance is a critical QAS, as the mobile devices implementing Cartographer need to be able to localize faster than they can actually locomote themselves. This requires the accuracy of the positioning to be sufficiently high to allow confidence before sending motion command to the robots actuators.

<table>
  <tr>
    <td>Scenario name</td>
    <td>Process Position Data</td>
  </tr>
  <tr>
    <td>Business goals</td>
    <td>Effective algorithms for calculating position, pose and trajectory of a mobile robot</td>
  </tr>
  <tr>
    <td>Quality attribute</td>
    <td>Performance</td>
  </tr>
  <tr>
    <td>Stimulus</td>
    <td>Deploy the device in operating status</td>
  </tr>
  <tr>
    <td>Stimulus source</td>
    <td>User needing localization, hardware request</td>
  </tr>
  <tr>
    <td>Response</td>
    <td>Processing data from sensor and sending back mapping response</td>
  </tr>
  <tr>
    <td>Response measure</td>
    <td>Position mean accuracy <= 5mm</td>
  </tr>
</table>

### Performance - Use Case Diagram
![image alt text](https://github.com/SENG480-18/project-team2/blob/master/M3%20-%20Use%20Case%20Diagram.png)

### Performance - Sequence Diagram
![image alt text](https://github.com/SENG480-18/project-team2/blob/master/M3%20Performance%20Sequence%20Diagram.png)

## Quality Attribute - Interoperability
Interoperability is another important quality attribute that is addressed by Cartographer’s architecture. The core functionality should be able to be implemented by a variety of devices running a variety of client software. ROS is the dominate client platform, but there are many others in use, and will likely be more as hardware evolves. Without cross platform interoperability, the utility of the open source project is significantly reduced.

<table>
  <tr>
    <td>Scenario name</td>
    <td>Multiplatform Interoperability</td>
  </tr>
  <tr>
    <td>Business goals</td>
    <td>Achieve inter-platform communication</td>
  </tr>
  <tr>
    <td>Quality attribute</td>
    <td>Interoperability</td>
  </tr>
  <tr>
    <td>Stimulus</td>
    <td>Need to have an alternate or novel platform to be able to utilize the system</td>
  </tr>
  <tr>
    <td>Stimulus source</td>
    <td>User implementing other platform, software based</td>
  </tr>
  <tr>
    <td>Response</td>
    <td>Can read and write incoming and outgoing messages</td>
  </tr>
  <tr>
    <td>Response measure</td>
    <td>Successfully receive and send message or enable error correcting algorithms on failure</td>
  </tr>
</table>

### Interoperability - Sequence Diagram
![image alt text](https://github.com/SENG480-18/project-team2/blob/master/M3%20Interoperability%20Sequence%20Diagram.png)

# Interfaces
We provide two interfaces that are critical to the overall system; the ROS (mobile client) interface and the mapping interface.

![image alt text](https://github.com/SENG480-18/project-team2/blob/master/Interface%20Lead%20inDiagram.png)

## ROS Interface
![image alt text](https://github.com/SENG480-18/project-team2/blob/master/ROS%20Interface%20Diagram.png)
## Mapping Interface
![image alt text](https://github.com/SENG480-18/project-team2/blob/master/Mapping%20Interface.png)

# Rationale
Cartographer is a mathematically complex program that needs to be capable of being modified by a large number of developers. This is a function of the user base who are are typically researching and building custom robotics systems and architectures to support them. Cartographer’s design aims for high cohesion, in order to make the code easily modifiable by a diverse group of developers. By keeping the modules within well defined boundaries it is easy to find the code associated with a quality attribute.

Cartographer is built around nine core modules. One of Cartographer’s key modules is called Common.  It is dedicated to structures and functions with shared usage throughout the system. Cartographer does not aim for cohesion is it’s Common module. Performance is assisted by providing common programming objects, such as queues, in performance empathetic versions. Objects that work well with concurrency are prevalent throughout Common. By offering performance emphasised objects, Cartographer makes it easy for developers to program in a manner that achieves their performance goals. 

Additionally, the Common module is used to prevent copies of code that perform the same task from proliferating in the codebase. The downside to Common is it is difficult to gauge whether a functionality is part of it from just the functionalities description. Common also provides generic functions used throughout the system and LUA integration. 

Configurability is an important quality attribute, and Cartographer is architected around this concept. Configurability is enabled through a number of LUA scripts. LUA was chosen as it is easy a relatively simple language in which to perform high level changes. The LUA scripts add a degree of abstraction so a user can alter Cartographer without needing to alter the more complex C++ source code. The changes that can be provided through the configurablitty scripts are limited to common options the developers chose to provide. More complex changes to Cartographer are somewhat supported through the cohesion of the modules and robust documentation that supports the project. 

The following four modules; Internal, Mapping, Mapping 2D, and Mapping 3D are tightly coupled. Due to different approaches required for 2D and 3D mapping within Cartographer, as well as the ideology that not all systems will require both types, the mapping module has been segmented by dimension rank. This split increases the cohesion of the individual modules and supports modifiability. The downside to this architectural choice, splitting the Mapping module into three distinct modules is that it can be unclear where each component located. From a general perspective Mapping contains functions that are common to both 2D and 3D implementations and anything specific to either type of mapping is put in its perspective folder.

Internal contains helper functions and structures used throughout mapping while also containing connections to Sensor which captures the input data required as feedstock to the mapping process. Internal provides greater cohesion for functions that would otherwise be gathered in the Common module. As such, the choice to segregate Internal is effective at increasing the modifiability of the code. 

The Transform module, which provides unit conversions and transformations of vectors and matrices, is also strongly coupled to the mapping functions as these are key to the mapping process itself. While Transform may be tightly coupled to the mapping modules, it is also tightly coupled to the IO module. 

The IO, and Transform Libraries are very tightly coupled and act as a conduit for the output of all the mapping modules. As maps are created the data is interpreted through Transform. Once passed through Transform, data is changed into a graphical form as a voxel map and saved through the IO module. IO acts as the graphical output for the mapping functions. While IO is a somewhat generic name choice for a module, it is appropriate in this case as all of Cartographer’s output passes through this module. One exception to the inferred purpose of the IO module, is that Cartographer’s primary input is handled through the Sensor module, not IO. 

Sensor is a module that controls the data inputs into the program. LiDar and odometry information supplied via ROS (or other mobile client) comes into the rest of Cartographer via this module. Sensor abstracts the physical information provided to it into data which can be read by the rest of Cartographer. Sensor is key to Cartographers interoperability QA goals as it changes input from something sensor specific, into a format readable by Cartographer’s data processing modules. Cartographer’s interoperability is so strongly defined at this specific part of the system architecture, we can say that any implementation that can provide data in a way which Sensor can interpret and which can run Cartographer’s code has access to the the entire Cartographer system.

