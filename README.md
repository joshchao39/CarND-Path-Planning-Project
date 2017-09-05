# CarND-Path-Planning
Self-Driving Car Engineer Nanodegree Program

---

## Background

This project aims to drive the car around the highway track passing slow cars without collision/speeding/discomfort(high jerk) in the Udacity [simulator](https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2).

The approach can be split into 3 steps:
- **Dynamics Modeling**: This step deals with the input/output requirements of the simulator. 
- **Trajectory Generation**: This step generates the desired trajectory as path points without causing citations. 
- **Behavior Planning**: This step decides high level behavior of the vehicle such as lane change and reference velocity.

## Dynamics Modeling

The following are the inputs provided by the simulator at each communication (taken from Udacity project repository):

- **x**: The car's x position in map coordinates
- **y**: The car's y position in map coordinates
- **s**: The car's s position in frenet coordinates
- **d**: The car's d position in frenet coordinates
- **yaw**: The car's yaw angle in the map
- **speed**: The car's speed in MPH

- **previous_path_x**: The previous list of x points previously given to the simulator
- **previous_path_y**: The previous list of y points previously given to the simulator

- **end_path_s**: The previous list's last point's frenet s value
- **end_path_d**: The previous list's last point's frenet d value

- **sensor_fusion**: A 2d vector of cars and then that car's [car's unique ID, car's x position in map coordinates, car's y position in map coordinates, car's x velocity in m/s, car's y velocity in m/s, car's s position in frenet coordinates, car's d position in frenet coordinates.

The following are the output sent back to the simulator to control the ego vehicle:

- **next_x**: A list of x coordinates for the ego vehicle to go in sequence spaced by 0.02 seconds
- **next_y**: A list of y coordinates for the ego vehicle to go in sequence spaced by 0.02 seconds

Note output (x,y) coordinates are executed perfectly without any physical limitation, and the simulator returns any leftover coordinates as `previous_path_x/y`. New path points should be appended to `previous_path_x/y` to ensure smooth transition.

The car should not exceed the speed limit of 50 mph, acceleration limit of 10 m/s^2, and jerk limit of 10 m/s^3.

## Trajectory Generation

This step converts desired lane and velocity into smoothed path points.

A set of raw path points are selected using the last two path points from previous path, along with three points at 30, 60, 90 meters ahead in Frenet converted back to global coordinate. A spline function is then used to smooth the path. Enough points are drawn from the spline function to fill 50 (x,y) coordinates to feed the simulator.

Most of the logic in this section are taken directly from the Udacity Q&A video.

## Behavior Planning
#### Vehicle Tracking
To decide whether to change lane or whether to speed up or slow down, we only need to know the position and speed of the cars in the front and in the adjacent lane(s). The logic goes through each car in the sensor fusion information to identify the closet cars in the front.

#### Cost Function
To keep the action space small and simple, the only three lane change actions at any given time are keep lane, left lane change, and right lane change.

Three cost items are used to compute the cost of each action:
1. The velocity of the car in the front in the center/left/right lane.
2. The distance of the car in the front in the center/left/right lane.
3. The possibility of left/right lane change. (ex. If the ego vehicle is on the right most lane, the cost of right lane change should be infinitely large.)

The weights are tuned manually, and (w1=1.0,w2=3.0,w3=99999.0) are the final weights. It is hard to compare the weights directly as they carry different units, but for sure we do not want to go off the road! 

#### Slow Down
A simple heuristic rule is used to slow down the car when the car in the front is less than 30 meters ahead to the speed of the car in the front. Otherwise speed up to the desired velocity, which is 49.5 mph.


## Example
<p align="center">
  <img src="example.gif" alt="Passing Slower Car"/>
</p>

Full video can be found [here](https://youtu.be/BIWhmaCRS7Q)

## Other Notes
1. This approach, while not optimal, works well for the designed constraint and requirement. The ego vehicle is capable of going around the track endlessly while passing slower cars.
2. There are a lot of room to improve for the trajectory generation step. Instead of using constants tuned for the worst case scenario, we can always operate right under the acceleration/jerk limit for maximal lane change speed. 
3. There are a lot of room to improve for the behavior planning step. We could expand the possible actions to consider double lane change or going to a slower lane for better access a faster lane two lanes away.
