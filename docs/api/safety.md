# Create® 3 Safety Overrides

By default, the Create® 3 robot has some safety features enabled.
Their purpose is to make sure that the robot does not get into dangerous sitations and it is able to detect and react properly to cliff hazards.

Safety features are configurable by the user through a ROS 2 parameter.
This will allow more adventurous users to have full control over the robot.

!!! important 
    The robot will temporarily re-enable all safety features whenever it is commanded to execute any of the built-in autonomous behaviors.

The `safety_override` parameter exposed by the `motion_control` ROS 2 node should be used to control and override safety features.
It's a `string` parameter and it accepts 3 possible values:

 - `none` Safety features are fully enabled. This is the default value. 
 - `backup_only` Overrides the robot backup limit safety feature, disabling it.
 - `full` Safety features are fully disabled.

For example, in order to completely disable safety mechanisms:

```bash
ros2 param set /motion_control safety_override full
```

Note that the parameter server will reject changes if there are typos in the new safety value set by the users.

The following sections briefly describe what the safety features do.

## Backup Limit

!!! attention 
    **If you disable the backup limit, the robot will not stop if there are cliffs while driving backward!**

The Create® 3 robot is equipped with cliff sensors, but they are located only in the front of the robot.
This means that the robot is not able to detect cliff hazards while driving backward.

In order to make the robot safe, the robot's default policy is to never move backward further than what would be safe should there be a cliff directly behind the robot.
Under standard circumstances, the robot is allowed to briefly move backward only if it has first traveled forward (i.e. if it has "explored" the space making sure that it does not present cliff hazards).

If the robot is kidnapped (i.e. first lifted by the user and then placed somewhere), the backup limit will immediately trigger.

The Create® 3 robot signals to the user when the backup limit is about to be triggered in multiple ways:
 
 - Through the `HazardDetectionsVector` ROS 2 message: an hazard of type `BACKUP_LIMIT` will be published in the vector.
 - By logging a warning message.
 - By changing the color of the lights to orange.

If the user ignores these alerts and keeps driving the robot backward, the robot will suddenly brake and come to a stop.
From this point, the robot will refuse any backward movement.
The user will have to drive the robot forward in order to re-enable backward movements.
As soon as the user drives the robot forward, the lights will go back to the default white color.
Safe backward movements will not be immediately re-allowed, but rather the robot will have to keep driving forward for a little while.
This moment will be identified by the `BACKUP_LIMIT` hazard disappearing from the `HazardDetectionsVector` message.

The backup limit and its related alerts are active only when `safety_override = none`.

## Maximum speed

!!! attention
    **If you increase the speed above the default one, the robot may not be able to stop in time when a cliff is detected!**

In order to stop when a cliff hazard is detected, it is also necessary to make sure that the robot is not driving too fast.
Safety features will thus limit the robot speed.

You can check the current maximum robot speed through the read-only parameter `max_speed` exposed by the `motion_control` ROS 2 node.

When `safety_override = none` or `safety_override = backup_only` the maximum speed will be limited to 0.306 m/s.

By fully disabling safety features, i.e. setting `safety_override = full` the robot will be allowed to drive at its true maximum speed of 0.460 m/s.

## Acceleration Limits

The robot exposes its commanding acceleration limits through the `wheel_accel_limit` parameter on the `motion_control` node.
The velocity commands sent to the wheels will be ramped with the acceleration profile associated with this value.
The value defaults to its maximum settable `900 mm/s^2`.  If using heavier payloads, it is advisable to decrease acceleration limits.

## E-Stop

The robot exposes a service with name `e_stop` that when sent `e_stop_on` true will turn off the robot's wheels.
The robot will not respond to velocity commands when `e_stop_on` is true.
See [EStop.srv](https://github.com/iRobotEducation/irobot_create_msgs/blob/main/srv/EStop.srv)

For example, in order to enable the E-Stop:

```bash
ros2 service call /e_stop "{e_stop_on: true}"
```

## Wheel Status

There is a `wheel_status` topic which publishes [WheelStatus.msg](https://github.com/iRobotEducation/irobot_create_msgs/blob/main/msg/WheelStatus.msg)
exposing whether the wheels are enabled (disabled if E-Stop is engaged) and the present PWM duty cycle and current through each wheel.
