# GICI-LIB Dataset

This dataset is collected for the development of GICI-LIB. The platform is shown in the following figure.

<p align="left">
  <img alt="sensorsuit" src="./figures/experiment_platform.png" width="500"> 
</p>

We developed a GICI board to collect IMU and camera data and applied hardware synchronization with other sensors in the whole platform. The onboard IMU and camera are Bosch BMI088 and Onsemi MT9V034 respectively. The GNSS receiver is a Tersus David30 multi-frequency receiver. We also collected the reference station data from the Qianxun SI stream for RTD and RTK formulations, and the State-Space-Representation (SSR) data from the International GNSS Service (IGS) stream for PPP formulations. The fiber optic IMU is used to provide the ground truth by post-processing its data together with GNSS raw data.

We collected two kinds of datasets: short-term (several minutes) experiments (1.1 ~ 4.3) in different scenes, and long-term (tens of minutes) experiments (5.1 ~ 5.2) covering multiple scenes. For the short-term experiments, we categorize the scenes into 4 types: Open-sky, tree-lined, typical urban, and dense urban. And for each scene, we present 2 ~ 3 trajectories. For the long-term experiments, we provide two trajectories collected in the Shanghai city center that cover those scenes.

Here is the list of the datasets

| ID | Scene | Size | Date | Scene View | 
|:--------------|:--------------|:--------------|:--------------|:--------------|
| 1.1 | Open-sky | 0.7 GB | 2023.03.20 | [Images](figures/typical_scene/README_1.1.md) | 
| 1.2 | Open-sky | 0.5 GB | 2023.03.27 | [Images](figures/typical_scene/README_1.2.md) |
| 2.1 | Tree-lined | 1.4 GB | 2023.03.20 | [Images](figures/typical_scene/README_2.1.md) |
| 2.2 | Tree-lined | 0.6 GB | 2023.03.27 | [Images](figures/typical_scene/README_2.2.md) |
| 3.1 | Typical urban | 1.7 GB | 2023.03.27 | [Images](figures/typical_scene/README_3.1.md) |
| 3.2 | Typical urban | 1.4 GB | 2023.03.27 | [Images](figures/typical_scene/README_3.2.md) |
| 3.3 | Typical urban | 1.9 GB | 2023.03.27 | [Images](figures/typical_scene/README_3.3.md) |
| 4.1 | Dense urban | 1.4 GB | 2023.05.21 | [Images](figures/typical_scene/README_4.1.md) |
| 4.2 | Dense urban | 0.8 GB | 2023.03.27 | [Images](figures/typical_scene/README_4.2.md) |
| 4.3 | Dense urban | 1.6 GB | 2023.03.27 | [Images](figures/typical_scene/README_4.3.md) |
| 5.1 | Long-term | 8.2 GB | 2023.05.21 | [Images](figures/typical_scene/README_5.1.md) |
| 5.2 | Long-term | 5.8 GB | 2023.05.21 | [Images](figures/typical_scene/README_5.2.md) |

You can download them on [OneDrive](https://1drv.ms/f/s!Aq2sJkkB0M10jXecBfNuHSuEnrzM?e=Jz91kp).

## 1. Run with Non-ROS Interface

We provide various example YAML configuration files, see \<gici-root-directory\>/option. Remember to replace all the \<path\> and "start_time".

Then, you can run the dataset by 

```
./gici_main <gici-config-file>
```

To connect the real-time output stream to [RTKLIB](https://rtklib.com/), you should do the following steps:

a) Specify a TCP server output in NMEA format. The example configuration is shown in pseudo_real_time_estimation_RTK_RRR.yaml. 

b) Open RTKPLOT in a Windows computer. To access the IP address of your Linux computer, your Windows computer must be on the same network segment.

c) Click file->Connection Settings. Enable a TCP client. Click Opt to configure the TCP client options. Fill in the Server Address (IP of your Linux computer) and Port (Configured in GICI YAML file).

d) Click file->Connect to form connection. Then you can see the real-time plots.

## 2. Convert Raw Data to rosbags

We provide a tool, converting the bin files to rosbags, see \<gici-root-directory\>/tools/ros/gici_tools/src/gici_files_to_rosbag.cpp. Its configuration file is at \<gici-root-directory\>/tools/ros/gici_tools/option/convert_rosbags.yaml. Remember to replace all the \<path\> and "start_time".

You can compile the convertor by 

```
cd \<gici-root-directory\>/tools/ros
catkin_make -DCMAKE_BUILD_TYPE=Release
```

Then you can run the convertor by

```
./devel/lib/gici_tools/gici_files_to_rosbag <config-file>
```

## 3. Run with ROS Interface

We also provide various example YAML configuration files for ROS interface, see \<gici-root-directory\>/ros_wrapper/src/gici/option. Remember to replace all the \<path\> and "start_time".

Before you run the ROS executable, remember to run a roscore. Then, you can run the executable by 

```
rosrun gici_ros gici_ros_main <gici-config-file>
```
or

```
cd \<gici-root-directory\>/ros_wrapper
./devel/lib/gici_ros/gici_ros_main <gici-config-file>
```

After that, you can play the rosbags converted from our bin files by

```
rosbag play <data1.bag> <data2.bag> <data3.bag> ...
```

For visualization, you can run our RVIZ configuration by

```
rviz -d \<gici-root-directory\>/ros_wrapper/src/gici/rviz/gici_gic.rviz
```

## 4. Evaluation

We provide ground_truth.txt for each dataset. The ground truth data is in the frame of fiber optic IMU. You should apply a coordinate convertion before comparing the results. 

For the estimators containing IMU, GICI outputs solution in the IMU frame. We provide tools converting the ground truth to the IMU frame.

First, you should compile the tools by

```
cd \<gici-root-directory\>tools/evaluation/alignment
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j8

cd \<gici-root-directory\>tools/evaluation/format_converters
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j8
```

Then you can convert the ground truth by

```
\<gici-root-directory\>tools/evaluation/format_converters/build/ie_to_nmea ground_truth.txt
\<gici-root-directory\>tools/evaluation/alignment/build/nmea_pose_to_pose ground_truth.txt.nmea
```

The default settings in nmea_pose_to_pose.cpp is converting poses from the fiber optic IMU frame to IMU frame for our dataset. If you have other requirements, you should modify the parameters in nmea_pose_to_pose.cpp.

Now you get a ground truth file ground_truth.txt.nmea.transformed in NMEA format. For easy visualization, you can convert this file to the TUM format by

```
\<gici-root-directory\>tools/evaluation/format_converters/build/nmea_to_tum ground_truth.txt.nmea.transformed
```

You can also convert the GICI NMEA output to the TUM format, and then compare them by any software.

For the GNSS-only estimators, GICI outputs solution in the GNSS antenna frame. You should further convert the ground truth to GNSS antenna by

```
\<gici-root-directory\>tools/evaluation/alignment/build/nmea_pose_to_position ground_truth.txt.nmea.transformed
```

Now you get a ground truth file ground_truth.txt.nmea.transformed.translated. Then you can continue the operations above.
