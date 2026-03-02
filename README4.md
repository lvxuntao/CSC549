# HW3 Problem 5 - Autonomous Driving Agent Racing (PID Controller)

## Group Members

Member: Xuntao Lyu (xlyu5), Lihang Liu (lliu45)

## Overview

The goal of this problem was to create an aggressive autonomous agent that races to a destination as fast as possible, ignoring all traffic rules, while tracking simulation time and infractions.

A video demo is included to demonstrate our implementation:
https://github.com/lvxuntao/CSC549/blob/main/assets/hw3_p5_demo.mp4

### How the PID Controller was Implemented

1. **Agent Modification**: The `MyPID` class in `my_automatic_control.py` was created by inheriting from `BasicAgent`.
2. **PID Tuning & Aggression**: We overrode the default lateral and longitudinal PID dictionaries (`lateral_control_dict`, `longitudinal_control_dict`). By passing these through `opt_dict` into the initialization of `BasicAgent` (and consequently `LocalPlanner` and `VehiclePIDController`), we achieved faster acceleration (Longitudinal K_P/K_I/K_D) and sharper turning (Lateral K_P/K_I/K_D). Maximum constraints (`max_throttle`, `max_brake`, `max_steering`) were similarly elevated.
3. **Decoupling from Speed Limits**: We explicitly called `agent.follow_speed_limits(False)` and set the `target_speed` to 60 km/h, ensuring the `LocalPlanner` did not cap the PID target speed based on traffic signs.

### Other Details

* **Time tracking**: Recorded `world.hud.simulation_time` exactly when the agent loop starts, and subtracted it from the simulation time when `agent.done()` evaluated to True.
* **Traffic rule ignorance**: Adjusted the agent initialization to `opt_dict.setdefault('ignore_traffic_lights', True)` and `ignore_stop_signs`, preventing the vehicle from stopping at red lights.
* **Collision Tracking**: Tapped into the `CollisionSensor` to increment a penalty counter on each impact.
* **Lane Invasion Tracking**: Tapped into the `LaneInvasionSensor`, filtering out `carla.libcarla.LaneMarkingType.Broken` completely. We tracked the total number of penalties, keeping a separate tally for `NONE` type invasions so that we could deduct up to 2 `NONE` penalties from the final score at the end of the race before stopping the simulation.

## Execution Output Example

Below is an example of the output from running the script:

start time 0
invasion {carla.libcarla.LaneMarkingType.SolidSolid}
invasion {carla.libcarla.LaneMarkingType.SolidSolid}
invasion {carla.libcarla.LaneMarkingType.SolidSolid}
invasion {carla.libcarla.LaneMarkingType.SolidSolid}
invasion {carla.libcarla.LaneMarkingType.Broken}
invasion {carla.libcarla.LaneMarkingType.Broken}
invasion {carla.libcarla.LaneMarkingType.Solid}
invasion {carla.libcarla.LaneMarkingType.Solid}
invasion {carla.libcarla.LaneMarkingType.SolidSolid}
The target has been reached, stopping the simulation, total time is 578.9772780127823 invasions 7 collisions 0