# HW1 Problem 3 - Carla Stop Sign Agent

## Group Members

Member: Xuntao Lyu (xlyu5), Lihang Liu (lliu45)

## Overview

This solution implements a custom autonomous driving agent (`MyStopAgent`) for the Carla simulator that can detect and stop at stop signs for a brief moment (30 virtual clock cycles), while also respecting traffic lights and speed limits.

A video demo is included to demenstrate our implementation.

## Implementation Details

### Modified Files

1. **my_automatic_control.py**  
   Main control script (added the `MyStop` agent option and fixed start/destination in Town10HD)

2. **basic_agent.py**  
   Added a `MyStopAgent` subclass of `BasicAgent` that stops at stop signs for a fixed number of ticks


### Key Features

#### 1. MyStopAgent Class (basic_agent.py: 493-619)

The `MyStopAgent` extends `BasicAgent` and adds stop sign detection and stopping behavior.

**Initialization:**
- Explicitly sets `ignore_traffic_lights = False` and `ignore_stop_signs = False`
- Enables speed limit following
- Retrieves all stop signs in the world using `self._world.get_actors().filter("*stop*")`
- Initializes stop countdown counter for 30-cycle pause

**Key Parameters:**
- `_stop_cycles = 30`: Number of ticks to pause at stop sign (~3 seconds at 10 fps)
- `_base_stop_threshold = 7.0`: Detection distance in meters
- `_last_stop_sign`: Tracks last encountered stop sign to prevent repeated triggering

#### 2. Stop Sign Detection (_affected_by_stop_sign method)

**Algorithm:**
```
For each stop sign in the world:
  1. Project stop sign location to nearest driving lane waypoint
  2. Filter stop signs on the same lane as ego vehicle
  3. Check if stop sign is within detection distance and in front cone
  4. Return (True, stop_actor) if detected, else (False, None)
```

**Key Implementation Details:**
- Uses `get_waypoint(sign_loc, project_to_road=True)` to find the road waypoint near the stop sign
- Filters by lane ID to ensure only same-lane stop signs are detected
- Uses `is_within_distance()` with a 120-degree forward cone [0, 120] to detect signs ahead
- Implements hysteresis: once a stop sign is detected and passed, it won't re-trigger until vehicle moves sufficiently far away (2× max_distance)

#### 3. Stopping Behavior (run_step method)

**Control Flow:**
```
run_step() is called each virtual clock cycle:

1. If _stop_countdown > 0:
   - Decrement counter
   - Apply emergency brake
   - Return control (stay stopped)

2. Else:
   - Check for vehicle obstacles
   - Check for red traffic lights
   - Check for stop signs
   - If stop sign detected:
     * Set _stop_countdown = 30
     * Mark stop sign as _last_stop_sign
     * Set hazard_detected = True

3. Get control from local planner
4. If hazard detected, apply emergency stop
5. Return control
```

#### 4. Spawn and Destination Configuration (my_automatic_control.py)

**Custom Spawn Point (lines 196-200):**
- Agent type: `MyStop`
- Spawn location: `(-45.235935, -36.500095, 0.600000)`
- Map: Town10HD (default)

**Custom Destination (lines 882-886):**
- Destination: `(67.659737, 69.835068, 0.000000)`

**Vehicle Filter:**
- Vehicle type: `vehicle.jeep.wrangler_rubicon`

## Installation and Execution

### Prerequisites

Ensure you are on an ARC cluster node with Carla support:
```bash
salloc -p csc549
```

### Setup (one-time)

```bash
cd hw1/p3
mkdir -p CSC549/PythonAPI/examples
mkdir -p CSC549/PythonAPI/carla/agents/navigation

# Create symbolic links for required libraries
ln -s /usr/lib64/libomp.so libomp.so.5
ln -s /opt/nvidia/nsight-systems/2023.2.3/host-linux-x64/libjpeg.so.8 .
```

### Environment Setup (each session)

```bash
# Set environment variables
export UE4_ROOT=~fmuelle/carla-9.14/UnrealEngine_4.26/
export PYTHONPATH=~fmuelle/carla-9.14/PythonAPI/carla/dist/carla-0.9.14-py3.7-linux-x86_64.egg
export LD_LIBRARY_PATH=`pwd`:$LD_LIBRARY_PATH

# Upgrade pip and install pygame
pip3 install --user --upgrade pip
pip3 install --user pygame==2.5.1
```

### Start Carla Server (Terminal 1)

```bash
cd hw1/p3
sh ~fmuelle/carla-9.14/CarlaUE4.sh -fps=10 -benchmark-quality-level=Low &
```

Wait ~1 minute for the server to fully start.

### Run MyStop Agent (Terminal 2)

```bash
cd hw1/p3
pip3 install -r requirements.txt
python3 CSC549/PythonAPI/examples/my_automatic_control.py -a MyStop --filter vehicle.jeep.wrangler_rubicon
```

**Note**: The Carla Python API (0.9.14) is provided via the egg file in `$PYTHONPATH`.
