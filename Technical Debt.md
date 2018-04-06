# Code Quality and Technical Debt

# TOC
- [Code Quality Review](#code-quality-review)
  * [Tools Used](#tools-used)
    + [Understand](#understand)
    + [CodeScene](#codescene)
      - [Hotspots](#hotspots)
      - [Temporal Coupling](#temporal-coupling)
      - [Complexity Trends](#complexity-trends)
    + [SonarQube](#sonarqube)
    + [Cpplint](#cpplint)
  * [Cumulative Summary of Results](#cumulative-summary-of-results)
- [Technical Debt](#technical-debt)
  * [Analysis of Potential Debt](#analysis-of-potential-debt)
    + [Lack of Inline Documentation - Comments](#lack-of-inline-documentation---comments)
    + [Refactoring Candidates](#refactoring-candidates)
  * [Design Tradeoffs Identified](#design-tradeoffs-identified)

# Code Quality Review

This code quality review serves to determine how well Cartographers code base conforms to best practices and the impacts of deviations from those on the Quality Attributes the user base cares about.

Generally, the Cartographer code base is very dense. They use protocol buffers to template many of the structures to improve interoperability. However, this comes at a cost of readability. Additionally, it is an implementation of a specific geometric algorithm coupled with some unique statistical inference. Additionally, the project group is actively transforming the key 2D and 3D libraries while simultaneously folding in gRPC structures to allow for multi agent SLAM.

We decided to analyze the code base using several of the tools as some were better than others and produced different inferences into the quality of the overall codebase.

## Tools Used

We used several tools to assess code quality. First, we used Understand as it was previously used to help with the Module and C&C milestones. This program has static code analysis built in and worked to show us some code smells and identify areas of potential technical debt.

Additionally we used SonarQube, which was very challenging to use. Several of our team members attempted to get it to run. Only one was even moderately successful, however this was on a local instance, so we are not able to share it for review like a cloud account.

CodeScene was used extensively as well as the CLI based cpplint. The results from all of these four tools is summarized after each is presented individually in the sections below. The summaries include a brief description of the tool, what is could look for, what it did not find some of the key findings.

### Understand

Understand is a static software analysis tool that was used in the earlier phases of the project, while not used as a comprehensive tool to evaluate code quality, it does have one interesting quality tool called Metrics Treemap a form of heat map, in these maps a gradient of color is applied across a segmentation map. The characteristic being examined is most expressed when the color is dark and the segment size can also be selected; by file, by class, etc.

![image alt text](image_0.png)

[https://files.slack.com/files-pri/T8AA0Q6FQ-F9TVB5EFM/image.png](https://files.slack.com/files-pri/T8AA0Q6FQ-F9TVB5EFM/image.png)

The maps shown in this section show the entire library build, with Cartographer shown highlighted in yellow. In many key metrics (line count, max cyclomatic, etc.) we found that the Cartographer code had better metrics than most included libraries, indicating that it was low in technical debt.

One area where it did show issue was in commenting. When we analyzed the ratio of comment to code and looked at Cartographer and ROS combined, we found quantitative support from what we had noted in the codebase qualitatively; a pronounced lack of comments. Commenting was widely variable and often insufficient to understand even the purpose of the class, let alone the inner workings. 

However, the metric used in Understand can be misleading as much of the comments showing below as well commented below (green) are simply license header comment blocks. See the [Technical Debt](#heading=h.2ndqokaumuy) section for more discussion.

![image alt text](image_1.png)

[https://files.slack.com/files-pri/T8AA0Q6FQ-F9TGDQYFK/image.png](https://files.slack.com/files-pri/T8AA0Q6FQ-F9TGDQYFK/image.png)

Understand also had another tool called CodeCheck that performs static analysis. We used this to identify code smells. We ran this tool on a subset of the code base; the cartographer source proper, not including any of the ROS code or math libraries.

The results showed lots of uncommented variables, many defined by unused identifiers and also lots of magic numbers (numeral literals with little reference as to the reason for their selection). This lack of commenting negatively impacts the maintainability QAS described in previous milestones.

![image alt text](image_2.png)

[https://files.slack.com/files-pri/T8AA0Q6FQ-F9UMYEY5B/image.png](https://files.slack.com/files-pri/T8AA0Q6FQ-F9UMYEY5B/image.png)

### CodeScene

CodeScene is a behavioral code analysis tool written in Clojure. Instead of doing traditional static analysis, it looks for patterns in version control data to understand the history and evolution of a code base: unravelling things like hotspots, temporal coupling as well as complexity trends.

#### Hotspots

A hotspot is complicated code that a developer has to work with often. In CodeScene, hotspots are calculated from two different data sources:

* Lines of code in each file as a proxy for complexity

* Change frequency of each file as a proxy for the effort one has spent on that code

The following image shows the hotspot analysis for Cartographer:

![image alt text](image_3.png)

Each large, blue circle represents a package in Cartographer. The packages in the figure follow the directory structure of Cartographer. Inside each package, source code files can be found. The biggest circles in darkest red are the top hotspots in the codebase. In other words, *pose_graph_2d* and *pose_graph_3d* are the two modules to which most of development activities tend to be located.  

#### Temporal Coupling

Temporal coupling means that two (or more) modules change together over time. CodeScene provides several different metrics for temporal coupling. The tool considers two modules coupled in time if they;

* are modified in the same commit, or

* are modified by the same programmer within a specific period of time, or

* refer to the same Ticket ID in their commit messages. 

The following figure shows the temporal coupling for Cartographer:

![image alt text](image_4.png)

The *pose_graph_2d.cc* and *pose_graph_3d.cc* have the highest sum of couplings - 656 and 654. If looking at the temporal coupling by commits, these two modules have very high degree of coupling: 82%, and the number of average revisions even reaches to 150. If looking at the temporal coupling across commits, the degree of coupling between these two modules is 83% and the number of average revisions is 117.

This tool helped us zero in on potential areas for technical debt as it tends to accumulate in proportion to code rework. When we examine *pose_graph_2d.cc* and *pose_graph_3d.cc* we notice one thing; lots of comments compared to the rest of the code base. These comments are instructional, bread crumbs for other developers that indicate that the code can be challenging to understand. All the commits, over time, motivated the developers to add these helpful in code artifacts, which based on the lack of comments elsewhere, indicates this as any area to investigate for issues.

#### Complexity Trends

Complexity trends are used to get more information around hotspots. A complexity trend is calculated by fetching each historic version of a hotspot and calculating the code complexity of those historic versions. 

![image alt text](image_5.png)

![image alt text](image_6.png)

The figures above show the complexity trends of *pose_graph_2d* and *pose_graph_3d* respectively, starting in mid 2016. It paints a worrisome picture since the complexity has started to grow rapidly since April 2017. 

![image alt text](image_7.png)

![image alt text](image_8.png)

As evident by the Complexity/Lines of Code ratio shown in figures above, the ratio is always over 2.0 for both modules even though there are some modifications somewhere between April to November in 2017. This ratio indicates that the code in the hotspots - *pose_graph_2d.cc* and *pose_graph_3d.cc* - may be overly challenging to understand, which negatively impacts modifiability, a key QA for this project.

### SonarQube

SonarQube is an enterprise solution to code quality analysis. SonarQube offers insight into Code Smells, Bugs, Debt, Testing Coverage, and Duplicated Sections of code. SonarQube markets itself as a continuously updating version of other tools, but it is a much more robust tool then that. SonarQube is made of two main pieces, a server and a scanner, making it a

lot more complex than other tools to get up and running.

![image alt text](image_9.png)

The server for SonarQube is a Java based system made to allow web distribution of results. The server is a robust platform which offers security, code quality report tracking, and multiple projects. It is reasonably easy to configure the server once its PATH variables are added. The server also supports a variety of plugins to offer greater versatility. Plugins can be added through drag and drop and then a restart of the server, making install relatively simple. Finally as the server is in Java it works on Windows, Linux, and Mac.

The SonarQube scanner is a client side way of publishing results to the server. Some configuration is required in order allow the scanner to talk to the server, but it is simple in a local environment. Our issue with this tool for our project is that it does not support the C++ that our project is made of primarily.

[CXX](https://github.com/SonarOpenCommunity/sonar-cxx) is a open source plugin that offers C/C++ support. One of the issues with CXX is that it does not actually do any code sniffing on its own. CXX purely acts as a tool to decode other tools output, like [CPPCheck](http://cppcheck.sourceforge.net/). We found our best solution was too output results from CPPCheck and then too use scanner as a way of publishing them. Configuring scanner in this way proved very difficult as it was not undocumented. As CPPCheck is not a very special code checker our attempt at SonarQube was left here. 

In the end SonarQube proved to be a powerful tool with huge upfront cost. In a work environment where employees are very spread out it would offer a unique solution through its web interface. If SonarQube is configured extensively it is the most powerful tool as it can consolidate information from other tools, but out of the box it proves a little lacking. 

It is worth noting that SonarQube was very effective at checking python code of which there is around 200 lines of in our project. While not key to our projects overall quality a small analysis of the python report is done below. 

![image alt text](image_10.png)

From above we can see that there were no bugs or vulnerabilities where found with the built in SonarQube quality profile, other profiles can be added as plugins as described above. As we did not run any batch testing our coverage is 0%. First taking a look at code smells we can see there are 8 smells present. 

![image alt text](image_11.png)

Those 8 smells are rated by Severity, Personnel Assignment, and time to complete. A number of other metrics to organize the smells are also available on the left. This provides a effective way for teams to look code smells. The Debt rating of ‘1h’ acts as a tracker of how much time would be needed to completely rectify the code to SonarQube standards. Finally Duplicated lines shows what sections are repeats and could be changed to functions. 

### Cpplint

[Cpplint ](https://github.com/google/styleguide/tree/gh-pages/cpplint)is an open source automated source code analyze tool written in Python script that can flag programming errors, bugs, stylistic errors, and suspicious constructs. This tool was first developed by Google in 2009. The ultimate goal of Cpplint is to make sure C++ code file follows [Google’s C++ coding style guide](https://google.github.io/styleguide/cppguide.html).

We ran this tool on an Anaconda environment with python version 2.7 installed. By using prewritten Windows batch file we recursively ran this python tool on each of the cpp and header files and stored the results for each module. Results are presented in a textual format, as seen in the screenshot below;

![image alt text](image_12.png)

After finished analyzing code file, it will return total error number found in this code file along with the full paths, including which lines are they and suggestions of how should they be fixed. It also gives issue type, for example it will add [build/include_what_you_use] when it found missing #include guard, and confidence scores ranging from one to five. Where five means that the tool is very certain of the problem and one refers to issues that could be a legitimate construct.

From the results we have, we see a numerous error reports found for each module that most of them are having style problem with *#ifndef* header guard and some *#include* guards. But also a major issue is there is some unapproved C++11 headers such as* <mutex> *and *<chrono> *which we think it could to be easily turned into technical debt. There is also some less important coding style issues such as, redundant semicolons after curly bracket or lines exceeding 80 characters in length.

Having source code before built examined and analyzed by such simple lint-like tool, although there could be many errors made due to the dependencies not being built and recognized, we still think it is showing some of the aspect of their coding quality and helped us to inspect the architecture more carefully in terms of the potential for technical debt.

## Cumulative Summary of Results

We reviewed several tools to pull in a wide range of analysis of code quality. We found that lacking of comments was reported by Understand which can leads to negatively impacts the maintainability. With the experience of CodeScence, we found that *pose_graph_2d.cc* and *pose_graph_3d* are the two modules which change most frequently  and the most challenging to understand, this may also lead to poor maintainability. We run a small analysis on SonarQube, based on the report we generated, the overall quality is good but still have minor problems like code smells and duplicated lines. Last but not least, coding style issues were found by Cpplint and this produced similar results to the output of the CodeCheck from Understand. 

From this we concluded the most significant code quality issues were lacking of commenting, and coding style issues because they can lead to negative repercussions on the maintainability QAS that are important to the system users.

# Technical Debt

The following section presents our findings on the technical debt accumulated in the Cartographer code base. The project is a Google codebase turned Open Source. As such it has excellent code quality, although as noted above is very dense and challenging to read. This is in part to it being a complicated geometric algorithm, but is not helped by almost a complete lack of comments, not including license headers.

Technical debt is an implied rework cost stemming from choosing a relatively easy to implement solution now rather than using a more time consuming to produce, but inherently more efficient approach. In our case lack of commenting negatively impacted the modifiability QAS’s associated with this project.

## Analysis of Potential Debt

Although it was challenging to find significant technical debt is a mature Google project, the following sections present our findings on the technical debt of Cartographer.

### Lack of Inline Documentation - Comments

One key area of potential technical debt, was inline documentation. Comments were ineffective at transmitting the intent of the developers in many cases. While a deep understanding of the geometric algorithms is a requirement, even comments to indicate the purpose of a file seem to be subservient to the obligatory Apache ULA banner.

Our analysis with various static code quality tools found that ratios were both inconsistent and very low. File size was quite low for many files and any acceptably high comment to code ratios were often attributable to the licensing banner. It would be better to use a tool that took into consideration if the comments were at the beginning of the file and also if they were nearly identical to other file headers, therefore conveying little specific meaning about the individual files. An example file (*ceres_pose.cc*) of a typical commenting gap, with the licence header issue illustrated as well.

![image alt text](image_13.png)

[https://files.slack.com/files-pri/T8AA0Q6FQ-F9TB8TGKA/image.png](https://files.slack.com/files-pri/T8AA0Q6FQ-F9TB8TGKA/image.png)

Google’s own style guide for C++ had this to say about comments;

"Though a pain to write, comments are absolutely vital to keeping our code readable. The following rules describe what you should comment and where. But remember: while comments are very important, the best code is self-documenting. Giving sensible names to types and variables is much better than using obscure names that you must then explain through comments.

When writing your comments, write for your audience: the next contributor who will need to understand your code. Be generous — the next one may be you!"

[https://google.github.io/styleguide/cppguide.html#Comments](https://google.github.io/styleguide/cppguide.html#Comments)

Clearly, if comments are vital, including none in a file is not meeting the expectations of the guidelines and is clearly technical debt worth addressing. This assessment was supported by the code smells identified in our code quality inspections. Many issues were noted for failing to document variables and numeric literal choices. This negatively impact the maintainability of the code base and was apparent throughout our entire assessment of this open source project released into the wild from Google Labs.

### Refactoring Candidates

![image alt text](image_14.png)

The figure above shows a list of prioritized refactoring targets for Cartographer. It is clearly that *pose_graph_2d.cc* and *pose_graph_3d.cc* are two refactoring targets with the top priority. As mentioned above, it is not hard to see why it is important to refactor these two modules:

* These two modules are the top hotspots in Cartographer meaning that lots of effort is put into implementing these two modules

* These two modules are highly coupled with high numbers of average revisions by commits or across commits

* The Complexity/Lines of Code ratio is always 2.0 even though there were fluctuations somewhere in time. 

Followed by *pose_graph_2d.cc* and *pose_graph_3d.cc*, there are other modules needed to be refactored, such as *constraint_builder_3d.cc*. Among these refactoring targets, the code complexity of some modules starts to rise as shown by the flag of complexity trend warning. In other words, when developing modules, such as *local_trajectory_builder_3d.cc*, it is important to refactor the code at the same time. 

## Design Tradeoffs Identified

Cartographer is a big piece of software, and as such several design tradeoffs were made. While Cartographer is open source, it started as a Google inhouse project and has a relatively strong foundation because of this. The first design trade off involves an increase of quality and complexity. 

The first tradeoff identified by our team involved protocol buffer. [Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data](https://developers.google.com/protocol-buffers/). By implementing protobuf the project's interoperability is increased greatly by allowing data structures to be easily passed between languages and platforms. The downside of this is that proto buffer needs to be implemented project wide. Changing proto files is straight forward but building all of them originally would take time. This tradeoff offers increased interoperability at the cost of code complexity.

Another major tradeoff identified involves [Google Remote Procedure Call (gRPC)](https://grpc.io/). This allows interoperability of clients running on various mobile platforms (as is often the case with mobile robots). This abstraction lets multiple agents work together on a mapping objective.  

![image alt text](image_15.png)

[https://grpc.io/](https://grpc.io/)

Originally gRPC was not implemented, but it was added to the project further in. Initially building the project without gRPC would be simpler, but it is more work to add a functionality late. This tradeoff was made to help the project get off the ground.

The final tradeoff that our team has identified is the splitting of the 2D and 3D mapping. Originally they were separated into two folders, which we believe is because they implemented only the 2D mapping first before moving to 3D. Laying out the code in such a way would make it very easy to check and work on originally. Recently the developers have been combing the two folders into a single one and merging the files to have 2D and 3D functionality. This is similar to the gRPC in the sense that it was done to reduce complexity while the code base was being established but has created technical debt that had to be fixed at a later date.

