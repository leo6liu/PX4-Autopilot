# TU ENGR PX4 Autopilot

This repository was forked on v1.12.3 for development of Temple University Department of Engineering's Roboboat Senior Design Project (FALL 2021).

## How to add a new "boat-frame"

The primary source for these instructions are provided on the PX4 offical docs: [Adding a New Airframe Configuration](https://docs.px4.io/master/en/dev_airframes/adding_a_new_frame.html).

The following steps demonstrate how to create the Aquatic Surface Vehicle (ASV) Dual-Thruster 30% power boat-frame and compile it using the PX4 stack.

1. Create the configuration file. This file sets the name and frame type, sets custom parameters, and specifies the mixer file.
    - Create the file: `22028_asv_dual_thruster_030` in the `PX4-Autopilot/ROMFS/px4fmu_common/init.d/airframes` directory (It is convention to prepend custom config files with a number in the range: [22000, 22999]).
    - The full config file for the ASV Dual-Thruster 30% can be seen here: [`22028_asv_dual_thruster_030`](ROMFS/px4fmu_common/init.d/airframes/22028_asv_dual_thruster_030)
    - The @name in the header sets the name of the boat frame which will appear in the QGC airframes menu.
    - The @type and @class in the header sets the type and class of airframe. In this case both are set to Boat so they will show up under the Boat catagory in the QGC airframes menu.
    - `. ${R}etc/init.d/rc.boat_defaults` This line calls some PX4 defaults for boat frames.
    - The `PWM_MAIN_DISARM`, `MAX`, and `MIN` define the main PWM ranges which will be used by the MAIN outputs. Units are in microseconds (μs).
    - `set MIXER asv_dual_thruster_030` This line specified the mixer file which should be used by this boat frame. The mixer file specifies how input signals (i.e. Roll, Pitch, Yaw, Throttle) should effect the output channels.
2. Add the configuration file to the airframes CMakeLists.txt.
    - Add the line `22028_asv_dual_thruster_030` under the `# [22000, 22999] Reserve for custom models` heading in [`ROMFS/px4fmu_common/init.d/airframes/CMakeLists.txt`](ROMFS/px4fmu_common/init.d/airframes/CMakeLists.txt)
3. Create the mixer file. This file specifies how input signals (i.e. Roll, Pitch, Yaw, Throttle) should effect the output channels.
    - Create the file: `asv_dual_thruster_030.main.mix` in the `PX4-Autopilot/ROMFS/px4fmu_common/mixers` directory (It is convention to prepend custom config files with a number in the range: [22000, 22999]).
    - The full mixer file for the ASV Dual-Thruster 30% can be seen here: [`asv_dual_thruster_030.main.mix`](ROMFS/px4fmu_common/mixers/asv_dual_thruster_030.main.mix)
    - Additional documentation on mixer files can be found here: [Mixing and Actuators](https://docs.px4.io/master/en/concept/mixing.html)
    - Only lines that begin with a capital letter followed by a colon are processed (e.g. `M:`, `O:`, `S:`). All other lines can be used for commenting and documentation.
    - The mixer file is made up of several mixers. There are four types of mixers: `R:` Multirover, `H:` Helicopter, `M:` Summing, `Z:` Null. Multiple mixers can be specified in a single mixer file. The outputs correspond to the order in which the mixers are specified in the file (e.g. "if you define a multi-rotor mixer for a quad geometry, followed by a null mixer, followed by two summing mixers then this would allocate the first 4 outputs to the quad, an "empty" output, and the next two outputs.").
    - Our mixer is for a dual thruster boat, so there should be two outputs. The first will be for the right thruster and the second for the left thruster.
    - The right thruster will use a summing mixer, meaning zero or more control inputs will control a single actuator output. Further documentation on summing mixers can be found here: [Summing Mixer](https://docs.px4.io/master/en/concept/mixing.html#summing-mixer).
    - Here is a line by line explanation of the summing mixer used for the right thruster:

The following two lines do not begin with a capital letter followed by a colon, so they are ignored.

`Output 1: right thruster`

`---------------------------------------`

This is the first mixer specifed in the file. As it is a summing mixer (`M:`), it will control a single output (main PWM output 1). The `2` indicates that it reads two control inputs. This requires two `S:` statements to appear in this mixer block.

`M: 2`

The `O:` defines the output scaler. Values are scaled by a factor of 10000, so 3000 would correspond to a scale value of 0.3. The first 3000 indicates we want a 0.3 scaling on negative output. The second 3000 indicates we want a 0.3 scaling on positive output. The 0 specifies an offset of 0. The -10000 indiates the lower limit is -1.0 and the 10000 indiates the upper limit is 1.0.

`O:         3000    3000      0  -10000  10000`

The `S:` defines the output behavior for the first control input. The 0 indiates the input is from [Control Group #0 (Flight Control)](https://docs.px4.io/master/en/concept/mixing.html#control-group-0-flight-control). The 2 indates channel 2 which corresponds 

`S: 0 2   -10000  -10000      0  -10000  10000`

`S: 0 3    10000   10000      0  -10000  10000`

## General Information

Reach out to the responsive PX4 community on Slack:
[![Slack](/.github/slack.svg)](https://join.slack.com/t/px4/shared_invite/zt-si4xo5qs-R4baYFmMjlrT4rQK5yUnaA)

This repository holds the [PX4](http://px4.io) flight control solution for drones, with the main applications located in the [src/modules](https://github.com/PX4/PX4-Autopilot/tree/master/src/modules) directory. It also contains the PX4 Drone Middleware Platform, which provides drivers and middleware to run drones.

PX4 is highly portable, OS-independent and supports Linux, NuttX and MacOS out of the box.

* Official Website: http://px4.io (License: BSD 3-clause, [LICENSE](https://github.com/PX4/PX4-Autopilot/blob/master/LICENSE))
* [Supported airframes](https://docs.px4.io/master/en/airframes/airframe_reference.html) ([portfolio](http://px4.io/#airframes)):
  * [Multicopters](https://docs.px4.io/master/en/frames_multicopter/)
  * [Fixed wing](https://docs.px4.io/master/en/frames_plane/)
  * [VTOL](https://docs.px4.io/master/en/frames_vtol/)
  * [Autogyro](https://docs.px4.io/master/en/frames_autogyro/)
  * [Rover](https://docs.px4.io/master/en/frames_rover/)
  * many more experimental types (Blimps, Boats, Submarines, High altitude balloons, etc)
* Releases: [Downloads](https://github.com/PX4/PX4-Autopilot/releases)


## Building a PX4 based drone, rover, boat or robot

The [PX4 User Guide](https://docs.px4.io/master/en/) explains how to assemble [supported vehicles](https://docs.px4.io/master/en/airframes/airframe_reference.html) and fly drones with PX4.
See the [forum and chat](https://docs.px4.io/master/en/#support) if you need help!


## Changing code and contributing

This [Developer Guide](https://docs.px4.io/master/en/development/development.html) is for software developers who want to modify the flight stack and middleware (e.g. to add new flight modes), hardware integrators who want to support new flight controller boards and peripherals, and anyone who wants to get PX4 working on a new (unsupported) airframe/vehicle.


## Supported Hardware

This repository contains code supporting Pixhawk standard boards (best supported, best tested, recommended choice) and proprietary boards.

### Pixhawk Standard Boards
  * FMUv6X and FMUv6U (STM32H7, 2021)
    * Various vendors will provide FMUv6X and FMUv6U based designs Q3/2021
  * FMUv5 and FMUv5X (STM32F7, 2019/20)
    * [Pixhawk 4 (FMUv5)](https://docs.px4.io/master/en/flight_controller/pixhawk4.html)
    * [Pixhawk 4 mini (FMUv5)](https://docs.px4.io/master/en/flight_controller/pixhawk4_mini.html)
    * [CUAV V5+ (FMUv5)](https://docs.px4.io/master/en/flight_controller/cuav_v5_plus.html)
    * [CUAV V5 nano (FMUv5)](https://docs.px4.io/master/en/flight_controller/cuav_v5_nano.html)
    * [Auterion Skynode (FMUv5X)](https://docs.px4.io/master/en/flight_controller/auterion_skynode.html)
  * FMUv4 (STM32F4, 2015)
    * [Pixracer](https://docs.px4.io/master/en/flight_controller/pixracer.html)
    * [Pixhawk 3 Pro](https://docs.px4.io/master/en/flight_controller/pixhawk3_pro.html)
  * FMUv3 (STM32F4, 2014)
    * [Pixhawk 2](https://docs.px4.io/master/en/flight_controller/pixhawk-2.html)
    * [Pixhawk Mini](https://docs.px4.io/master/en/flight_controller/pixhawk_mini.html)
    * [CUAV Pixhack v3](https://docs.px4.io/master/en/flight_controller/pixhack_v3.html)
  * FMUv2 (STM32F4, 2013)
    * [Pixhawk](https://docs.px4.io/master/en/flight_controller/pixhawk.html)
    * [Pixfalcon](https://docs.px4.io/master/en/flight_controller/pixfalcon.html)

### Manufacturer and Community supported
  * [Holybro Durandal](https://docs.px4.io/master/en/flight_controller/durandal.html)
  * [Hex Cube Orange](https://docs.px4.io/master/en/flight_controller/cubepilot_cube_orange.html)
  * [Hex Cube Yellow](https://docs.px4.io/master/en/flight_controller/cubepilot_cube_yellow.html)
  * [Airmind MindPX V2.8](http://www.mindpx.net/assets/accessories/UserGuide_MindPX.pdf)
  * [Airmind MindRacer V1.2](http://mindpx.net/assets/accessories/mindracer_user_guide_v1.2.pdf)
  * [Bitcraze Crazyflie 2.0](https://docs.px4.io/master/en/complete_vehicles/crazyflie2.html)
  * [Omnibus F4 SD](https://docs.px4.io/master/en/flight_controller/omnibus_f4_sd.html)
  * [Holybro Kakute F7](https://docs.px4.io/master/en/flight_controller/kakutef7.html)
  * [Raspberry PI with Navio 2](https://docs.px4.io/master/en/flight_controller/raspberry_pi_navio2.html)

Additional information about supported hardware can be found in [PX4 user Guide > Autopilot Hardware](https://docs.px4.io/master/en/flight_controller/).
