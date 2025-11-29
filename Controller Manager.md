This file is the main node that's acting as the central manager and data enforcer, it acts as the central hub for controller the F10th car. Its primary job is to select the appropriate control algorithm (like Pure Pursuit, MPC, or a trailing controller) based on the car's state and then publish the final driving commands.

The file consists of a main `Controller_manager` node. It's function is to integrate various sensors and control algorithms to produce the final driving command.

It has a couple of key roles:
1. **State Machine Logic:** Selects the active controller (`MAP`, `PP`, `STMPC`, `KMPC`, or `FTGONLY`) based on the car's current `self.state` (received from the `/state_machine` topic).
||
2. **Data Management:** Collects real-time data from ROS topics (car position, speed, waypoints, obstacles, IMU, etc.).
||
3. **Control Cycle:** Runs the selected controller's main loop (`map_cycle`, `pp_cycle`, etc.) at a fixed rate ($\text{self.loop}_{\text{rate}} = 40 \text{ Hz}$).
||
4. **Command Publishing:** Publishes the resulting steering angle and speed in an `AckermannDriveStamped` message to the car's actuators.
||
5. **Visualization/Tuning:** Publishes various markers and debug data for visualization in tools like RViz, and subscribes to dynamic reconfigure updates for tuning

# Subscribers

| **Subscriber Topic**                | **ROS Message Type** | **Data Content**                                                                                                             | **Purpose**                                                                               |
| ----------------------------------- | -------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `/car_state/odom`                   | `Odometry`           | Ego car's linear speed (`linear.x`, `linear.y`) and angular speed/yaw rate (`angular.z`).                                    | Get **current speed** and yaw rate for control loops.                                     |
| `/car_state/pose`                   | `PoseStamped`        | Ego car's position ($x, y$) and orientation ($\theta$).                                                                      | Get **current position** and orientation in the map frame.                                |
| `/local_waypoints`                  | `WpntArray`          | A list of waypoints (including $x, y$, speed $v$, curvature $\kappa$, etc.) local to the car.                                | Provide the **reference path** for path-following controllers (`MAP`, `PP`, MPC).         |
| `/vesc/sensors/imu/raw`             | `Imu`                | Raw linear acceleration data (specifically used to derive longitudinal acceleration).                                        | Get **acceleration data** (`linear_acceleration.y`) for steer scaling and MPC state.      |
| `/car_state/odom_frenet`            | `Odometry`           | Ego car's position ($s, d$) and velocity ($v_s, v_d$) in the **Frenet frame**.                                               | Get track-relative state for MPC and trailing controllers.                                |
| `/perception/obstacles`             | `ObstacleArray`      | A list of detected obstacles/opponents, including Frenet coordinates ($s, d$) and speed ($v_s$).                             | Get **opponent state** for trailing and overtaking logic.                                 |
| `/state_machine`                    | `String`             | A string representing the car's current state (e.g., `"DRIVING"`, `"TRAILING"`, `"FTGONLY"`).                                | Select the **active control algorithm** in `controller_loop`.                             |
| `/scan`                             | `LaserScan`          | Raw data from the LiDAR sensor (an array of distance ranges).                                                                | Used directly by the **Follow The Gap (FTG)** controller for mapping/emergency maneuvers. |
| `/l1_param_tuner/`<br>`parameter_updates`<br> | `Config`             | Updated parameters (gains, lookahead distances, etc.) for L1/Pure Pursuit and Trailing controllers.                          | Allow **real-time tuning** (Dynamic Reconfigure).                                         |
| `/mpc_param_tuner`                  | `Config`             | (Subscribed within `init_controller` via `Client`) Updated parameters for the MPC controllers (`STMPCConfig`, `KMPCConfig`). | Allow **real-time MPC tuning**.                                                           |
.

# Publishers and topics
| **Publisher Topic**                                                        | **ROS Message Type**    | **Data Content**                                                                        | **Purpose**                                               |
| -------------------------------------------------------------------------- | ----------------------- | --------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| <br>`/vesc/high_level/`<br>`ackermann_cmd_mux`<br>`/input/nav_1` | `AckermannDriveStamped` | **Speed** and **Steering Angle** (the final control command).                           | Send commands to the VESC motor controller (actuators).   |
| `/lookahead_point`                                                         | `Marker`                | A marker showing the lookahead point (L1/Pure Pursuit target) or steering angle vector. | **Visualize** the target point in RViz.                   |
| `/trailing_opponent_marker`                                                | `Marker`                | A marker showing the opponent's predicted position in the map frame.                    | **Visualize** the opponent's position when trailing.      |
| `/my_waypoints`                                                            | `MarkerArray`           | A collection of markers representing the local waypoints.                               | **Visualize** the reference path in RViz.                 |
| `/trailing/gap_data`                                                       | `PidData`               | Values of the trailing PID controller (error, P, I, D, command).                        | Debug and **tune** the Trailing/ACC controller.           |
| `/controller/latency`                                                      | `Float32`               | The time taken to execute one control loop iteration.                                   | **Measure performance** if launched with `measure:=true`. |
| `/mpc_controller/states`                                                   | `Float64MultiArray`     | The predicted state trajectory from the MPC controller.                                 | **Visualize** the MPC planning horizon.                   |