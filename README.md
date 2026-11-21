# Mini-NOVA | Obstacle perception and steering for a research catamaran

**Autonomous Systems Laboratory · University of Duisburg-Essen · 2026**

Mini-NOVA is a small research catamaran used to study how an autonomous vessel can reach a target while responding to obstacles. A crossing vessel illustrates the engineering problem: the boat might need to adjust its route, slow down, or wait for the crossing to clear. This project overview focuses on two pieces of the prototype for which Python source is available: a stereo-depth perception node and an embedded steering bridge.

## Reading obstacles from a depth image

The ROS 2 node in `zed_obstacle_node.py` subscribes to registered depth images on `/zed/zed_node/depth/depth_registered`. It converts supported depth formats into metres, crops the image vertically to 30–75% of its height, and divides that strip into left, centre, and right regions.

For each region, the node considers finite depth readings between 0.25 and 2.50 m. It requires at least 300 valid pixels and estimates obstacle distance using the 10th percentile. This makes the estimate less sensitive to a single unusually close pixel. The node publishes its regional assessment on `/mini_nova/zed_obstacle_info`. The distances above are configured filter limits.

## Turning an obstacle report into a steering command

The Python node in `teensy_obstacle_bridge.py` subscribes to `/mini_nova/obstacle_info`. It reads the selected in-path obstacle, or filters the obstacle list for eligible candidates and selects the nearest by forward distance. An obstacle on the front right calls for a left turn, and one on the front left calls for a right turn. The configured steering offset ranges from 5° to 30° around a 90° centre, according to the reported danger level.

The bridge sends paired servo-angle commands to a Teensy over a 115200-baud serial connection. It tracks the last commanded pair to suppress duplicate writes. The ZED node and the steering bridge use different obstacle topics and message fields. Connecting these particular nodes requires a producer or adapter that supplies the bridge's expected obstacle report.

## Platform hardware

| Component | Role in the project |
| --- | --- |
| NVIDIA Jetson | Onboard computer for perception and ROS 2 processing |
| Stereolabs ZED | Stereo depth input for the camera perception node |
| Ouster OS0-32 | 3D LiDAR listed in the sensor inventory |
| Teensy 4.1 and steering servos | Microcontroller and actuators in the steering design |

## Project scope

The lab brief includes waypoint navigation and responses to static and moving obstacles. The software described here covers depth processing and steering-command generation. End-to-end navigation, physical steering response, and on-water collision avoidance require separate validation. No measured detection accuracy or sea-trial results are reported here.
