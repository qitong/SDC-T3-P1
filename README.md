# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program
Term 3 Project 1 by Qitong Hu

## Model Documentation
---
Overview

Given the waypoints that describe the map and sensor data that describe the environment along the road, the objective is making the car drive itself according to speed limit, no exceed the max acceleration and jerk, no colllision, keep lane and able to change lane for at least 4.32 miles.
My implementation is based on the lesson 5 Project Walkthrough and Q&A (Thanks David and Aaron) which mainly solves the driving and smoothing problem, then I tune the parameter for close and slow down and implement my own finite state machine for change lane strategy.

### The car drives according to the speed limit && Max Acceleration and Jerk are not Exceeded && Keep Lane && Smoothing
These problems are mainly solved by the Project Walkthrough.

I set the max speed at 49.5, which is hard coded as the max value for ref_vel. As mentioned in video, it takes into account the curvation of the road (in s,d coordinate) not only the line distance in XY coordinate. Then it splits s by N segs and ensure none of them pass the limit.

Similarly, the acceleration is set to 0.224 to avoid exceeding the max acceleration.

Then, using spline library to smooth the path to avoid exceeding the max jerk. The point generation strategy is as follow: First take the point left in the last iteration to ensure that the newly generated one follows mainly a same path to avoid shake, generate new anchor points for next 30,60,90 meters, add those anchor point and use the end point of the car to generate new path which is smooth and consistent with history.

Setting d value to (lane * 4 + 2) to make sure that car is driving in the middle of the lane.

The code from the video mainly solves the "driving" problem by itself.

### Change Lane
I implement my own finite state machine to solve the change lane problem.
It is a little difference from the course suggestion. Here is my strategy:

There are total 6 states in my FSM:
0 = Keep Lane 0, 
1 = Keep Lane 1 (Start State)
2 = Keep Lane 2
3 = Prepare Lane 0->1
4 = Prepare Lane 1->0
5 = Prepare Lane 1->2
6 = Prepare Lane 2->1
in which the left lane is Lane 0, middle lane is Lane 1 and right lane is Lane 2.

Action:
a = Keep Lane: No car in front of me in change_lane_perspect distance in my lane OR for lane 1 has car in front of me in my lane but still the openest lane
b = Prepare for Lane changing: Has car in front of me in change_lane_perspect distance in my lane (for lane 0 and lane 2)
c = In lane 1, has car in front of me in change_lane_perspect distance in my lane and lane 0 is the openest lane 
d = In lane 1, has car in front of me in change_lane_perspect distance in my lane and lane 2 is the openest lane
e = Unsafe to change
f = Safe to change lane
in which, openest means that the front car in that lane is farmost in all 3 lanes; safe to change means that the front and rear space in the target lane is big enough (defined by front_safe_distance and rear_safe_distance respectively).

I didn't use Prepare Left Lane Change nor Prepare Right Lane Change, but rather define those 3 lanes individually. Since the road is simple in this case, Lane 0 (left) and Lane 2 (right) has only 1 change lane choice respectively, it can only choose to keep lane & slow down or change to Lane 1, the choice is straightforward -- when it is safe, change; on the other hand, Lane 1 has 3 choices, keep lane & slow down, change to Lane 2 and change to Lane 0, I used to implemented it to choose left first (according to my own custom), but found it stucks in traffic a lot, then I made 3 changes to solve the problem: 1. make the change_lane_perspect to 50 instead of 30, prepare before stucking 2. make the safe judgement more aggresive (rear_safe_distance from 20 to 10 and slow_down_threshold from 30 to 15)  3.change to the lane that is openest.

The following is the video of path planning project.

![](https://github.com/qitong/SDC-T3-P1/raw/master/output/fsm.jpeg)  
[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/YfMcGAV4PJI/0.jpg)](https://www.youtube.com/watch?v=YfMcGAV4PJI&feature=youtu.be)
