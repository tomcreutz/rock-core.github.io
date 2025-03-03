---
section: ROS/Rock bridge
title: The ROS-Rock Bridge
sort_info: 0
---

This section of the documentation will detail how to integrate ROS within a Rock
system.

Current state
-------------

In its current state, the Rock and ROS can interoperate at the level of the
dataflow, i.e. one can transfer data between Rock ports and ROS topics.

This section will guide you through:

 - what ROS messages are generated by a Rock system, and how to modify the
   default behaviour in that respect
 - how to connect Rock components with ROS nodes

Future Work
-----------

 - extend the runtime behaviour to be able to bypass roslaunch completely within
   [Ruby runtime scripts](../runtime/)
 - allow usage of ROS nodes within [Syskit](../system/)

