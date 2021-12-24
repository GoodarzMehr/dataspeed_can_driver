Dataspeed Drive-by-Wire Kit CAN Driver for CARMA
================================================

This is a fork of the [dbw_mkz_ros](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/src/master/) package that is used for connecting to and configuring the [Dataspeed Drive-by-Wire Kit](https://www.dataspeedinc.com/adas-by-wire-system/) for Lincoln MKZ / Ford Fusion vehicles. This fork has been modified to allow for building a Docker image that can serve as a CAN driver for the [CARMA Platform](https://github.com/usdot-fhwa-stol/carma-platform).

Ubuntu 20.04 Installation
-------------------------
Assuming the CARMA Platform is installed at `~/carma_ws/src`,
```
cd ~/carma_ws/src
git clone https://github.com/VT-ASIM-LAB/dataspeed_can_driver.git
cd dataspeed_can_driver/docker
sudo ./build-image.sh -d
```
After the Docker image is successfully built, connect the Drive-by-Wire Kit USB cable to your device and run `lsusb` in the terminal to determine which bus and device number it has been assigned to. Assuming here that it is Device 007 on Bus 001, add the following lines to the appropriate `docker-compose.yml` file in the `carma-config` directory, and make sure that the current user (and not `root`) is the owner of `/dev/bus/usb/001/007`.
```
dataspeed-can-driver:
  image: usdotfhwastoldev/carma-dataspeed-can-driver:develop
  container_name: dataspeed-can-driver
  network_mode: host
  privileged: true
  devices:
    - /dev/bus/usb/001/007:/dev/bus/usb/001/007
  volumes_from:
    - container:carma-config:ro
  environment:
    - ROS_IP=127.0.0.1
  volumes:
    - /opt/carma/logs:/opt/carma/logs
    - /opt/carma/.ros:/home/carma/.ros
    - /opt/carma/vehicle/calibration:/opt/carma/vehicle/calibration
  command: bash -c '. ./devel/setup.bash && export ROS_NAMESPACE=$${CARMA_INTR_NS} && wait-for-it.sh localhost:11311 -- roslaunch /opt/carma/vehicle/config/drivers.launch drivers:=dataspeed_can'
```
Finally, add the following lines to the `drivers.launch` file in the same directory as `docker-compose.yml`.
```
<include if="$(arg dataspeed_can)" file="$(find dbw_mkz_can)/launch/dbw.launch">
  <arg name="frame_id" value="base_link"/>
</include>
```

ROS API (stable)
----------------

### dbw_mkz_can

#### Nodes
* `can_node`
* `vehicle/dbw_node`

#### Published Topics
* `can_node/can_err [can_msgs/Frame]`: publishes error messages read from the vehicle [CAN bus](https://en.wikipedia.org/wiki/CAN_bus).
* `can_node/can_rx [can_msgs/Frame]`: publishes CAN messages read from the vehicle [CAN bus](https://en.wikipedia.org/wiki/CAN_bus) (772 Hz).
* `can_node/version [std_msgs/String]`: publishes the Dataspeed CAN USB Driver version.
* `vehicle/antilock_brakes_active [std_msgs/Bool]`: publishes whether the vehicle's anti-lock brake system (ABS) is active (50 Hz).
* `vehicle/brake_feedback [automotive_platform_msgs/BrakeFeedback]`: publishes the current brake pedal position (50 Hz).
* `vehicle/brake_info_report [dbw_mkz_msgs/BrakeInfoReport]`: publishes braking-related data including information on wheel torques, vehicle acceleration, brake pedal quality factor, hill start assist system, anti-lock braking system (ABS), stability control system, traction control system (TCS), and parking brake (50 Hz).
* `vehicle/brake_report [dbw_mkz_msgs/BrakeReport]`: publishes braking data including information on brake pedal position, braking torque, braking deceleration, and braking status (50 Hz).
* `vehicle/dbw_enable [std_msgs/Bool]`: publishes whether the Drive-by-Wire system has been enabled.
* `vehicle/driver_assist_report [dbw_mkz_msgs/DriverAssistReport]`: publishes information pertaining to advanced driver assistant systems (ADAS), including whether any such systems ([forward collision warning (FCW)](https://www.kbb.com/car-advice/how-does-forward-collision-warning-work/), [automatic emergency braking (AEB)](https://www.jdpower.com/cars/shopping-guides/what-is-automatic-emergency-braking), and [adaptive cruise control (ACC)](https://en.wikipedia.org/wiki/Adaptive_cruise_control)) are enabled or active as well as vehicle deceleration.
* `vehicle/fuel_level_report [dbw_mkz_msgs/FuelLevelReport]`: publishes fuel level and battery voltage information of the vehicle (10 Hz).
* `vehicle/gear_report [dbw_mkz_msgs/GearReport]`: publishes gear data including gear status and current gear enumeration (20 Hz).
* `vehicle/gps/fix [sensor_msgs/NavSatFix]`: publishes the Navigation Satellite fix for any [Global Navigation Satellite System (GNSS)](https://en.wikipedia.org/wiki/Satellite_navigation#Global_navigation_satellite_systems), specified using the [WGS84](https://en.wikipedia.org/wiki/World_Geodetic_System#1984_version) reference ellipsoid (1 Hz).
* `vehicle/gps/time [sensor_msgs/TimeReference]`: publishes time reported by the [Global Navigation Satellite System (GNSS)](https://en.wikipedia.org/wiki/Satellite_navigation#Global_navigation_satellite_systems) in use (1 Hz).
* `vehicle/gps/vel [geometry_msgs/TwistStamped]`: publishes linear and angular velocity (twist) measured by the [Global Navigation Satellite System (GNSS)](https://en.wikipedia.org/wiki/Satellite_navigation#Global_navigation_satellite_systems) in use (1 Hz).
* `vehicle/imu/data_raw [sensor_msgs/Imu]`: publishes data from the vehicle's IMU (100 Hz).
* `vehicle/misc_1_report [dbw_mkz_msgs/Misc1Report]`: publishes miscellaneous data obtained from the [CAN bus](https://en.wikipedia.org/wiki/CAN_bus), including information on turn signals, high beam, windshield wipers, ambient light sensor, outside air temperature, steering wheel buttons, doors, passenger seat, and seat belt (20 Hz).
* `vehicle/parking_brake [std_msgs/Bool]`: publishes whether the vehicle's parking brake is active (50 Hz).
* `vehicle/sonar_cloud [sensor_msgs/PointCloud2]`: publishes a point cloud created from the vehicle's ultrasound sensors observation.
* `vehicle/steering_feedback [automotive_platform_msgs/SteeringFeedback]`: publishes the current steering wheel angle (100 Hz).
* `vehicle/steering_report [dbw_mkz_msgs/SteeringReport]`: publishes steering data including information on the steering wheel angle, steering torque, and vehicle speed (100 Hz).
* `vehicle/stability_ctrl_active [std_msgs/Bool]`: publishes whether the vehicle's stability control system is active (50 Hz).
* `vehicle/stability_ctrl_enabled [std_msgs/Bool]`: publishes whether the vehicle's stability control system is enabled (50 Hz).
* `vehicle/surround_report [dbw_mkz_msgs/SurroundReport]`: publishes data obtained from the vehicle's ultrasound sensors as well as cross traffic alert (CTA) and blind spot information system (BLIS) information.
* `vehicle/throttle_feedback [automotive_platform_msgs/ThrottleFeedback]`: publishes the current throttle pedal position (100 Hz).
* `vehicle/throttle_info_report [dbw_mkz_msgs/ThrottleInfoReport]`: publishes throttle-related data including information on the throttle pedal position and rate of change, engine rpm, gear number, ignition status, and battery current (100 Hz).
* `vehicle/throttle_report [dbw_mkz_msgs/ThrottleReport]`: publishes throttle pedal data (50 Hz).
* `vehicle/tire_pressure_report [dbw_mkz_msgs/TirePressureReport]`: publishes tire pressure data (2 Hz).
* `vehicle/traction_ctrl_active [std_msgs/Bool]`: publishes whether the vehicle's traction control system (TCS) is active (50 Hz).
* `vehicle/traction_ctrl_enabled [std_msgs/Bool]`: publishes whether the vehicle's traction control system (TCS) is enabled (50 Hz).
* `vehicle/transmission_state [j2735_msgs/
* `vehicle/discovery`: publishes the CARMA [DriverStatus](https://github.com/usdot-fhwa-stol/carma-msgs/blob/develop/cav_msgs/msg/DriverStatus.msg) message.

#### Subscribed Topics
* `vehicle/can_tx [can_msgs/Frame]`: `can_node` subscribes to this topic to receive commands that should be published to the vehicle [CAN bus](https://en.wikipedia.org/wiki/CAN_bus).

#### Services
* ``

#### Parameters
* `can_node/bitrate`: bit rate of the [CAN bus](https://en.wikipedia.org/wiki/CAN_bus).
* `can_node/mask_0`: [CAN bus](https://en.wikipedia.org/wiki/CAN_bus) filter mask.
* `can_node/match_0`: [CAN bus](https://en.wikipedia.org/wiki/CAN_bus) filter match.

Examples
--------

See the `dbw.launch` file in the `dbw_mkz_can/launch` directory that is used to launch the Drive-by-Wire Kit.

Original Dataspeed ADAS Development Vehicle Kit Documentation
=============================================================
![rviz screenshot](https://bytebucket.org/DataspeedInc/dbw_mkz_ros/raw/d74e90d89c4e3da56d4c9e008d5feda8cff2447d/img/mkz_rviz.png)

Documentation and firmware updates
----------------------------------
The latest release can be found on the [downloads](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/downloads) page

ROS
---
If using ROS, setup your workspace and get started with the joystick demo [here](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/src/d74e90d89c4e3da56d4c9e008d5feda8cff2447d/ROS_SETUP.md).  
Get started early with recorded data [here](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/src/d74e90d89c4e3da56d4c9e008d5feda8cff2447d/ROS_BAGS.md).  
