# TU ENGR PX4 Autopilot

This repository was forked on v1.12.3 for development of Temple University Department of Engineering's Roboboat Senior Design Project (FALL 2021).

## How to setup the development environment

*Note: this is only a suggested build environment. Any text editor and container runtime has the potential to work.*

- install [VS Code](https://code.visualstudio.com/)
    - installed the [Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension
- install [Docker Desktop](https://www.docker.com/products/docker-desktop)
- install [Git](https://git-scm.com/downloads) *(git on WSL, macOS, and Linux will also work. GitHub Desktop WILL NOT WORK because it does not perform a recursive clone.)*
- install [QGroundControl](http://qgroundcontrol.com/downloads/)

1. Download the source code using the command: `git clone https://github.com/leo6liu/PX4-Autopilot.git --recursive`
2. Start Docker Desktop (or your container runtime of choice)
3. Open the PX4-Autopilot directory in VS Code
    - Do not install any suggested extensions at this time
    - There will be a notification in the bottom-right prompting you to `Reopen in Container`
      ![image](https://user-images.githubusercontent.com/46534486/142738240-c240856f-8efe-4302-8661-3a0ee84c0d44.png)
    - Click `Reopen in Container`
4. Once VS Code reopens in the container environment, it will install all the recommended extensions into the container and begin to configure the project using the CMake Tools extension.
5. Click the CMake Target target on the VS Code bottom taskbar. This will open a menu where you can select your build target (e.g. to build for Pixhawk 4, select px4_fmu-v5_default).
   ![image](https://user-images.githubusercontent.com/46534486/142738772-7b4fdf5d-c0a5-4657-afa1-e6780fc5b917.png)
6. Wait for the `Configuring Project: Configuring Project` notification from CMake Tools to finish (this could take several minutes on slower systems).
7. Click on the Terminal tab in VS Code and execute the following command: `make clean`
8. Execute the following command for the intended build target: `make px4_fmu-v5_default`
   *Note: each build must `make clean` before `make <target>`*
9. Wait for the build process to complete (this could take several minutes on slower systems).
10. In the VS Code file "Explorer" menu, navigate to the build/<target> directory (e.g. build/px4_fmu-v5_default).
11. Find the <target>.px4 file (e.g. px4_fmu-v5_default.px4), right click it, then Download it onto your computer.
12. Launch QGroundControl with a Pixhawk connected.
13. Navigate to the Firmware tab under Vehicle Configuration.
14. Follow the firmware flashing steps, selecting a custom firmware file. The file that should be loaded is the downloaded <target>.px4 file from above.
15. If a new "boat-frame" was added, relaunch QGroundControl, and it should appear in the Airframes tab.

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

The `S:` defines the output behavior for a control signal input. The 0 indiates the signal is from [Control Group #0 (Flight Control)](https://docs.px4.io/master/en/concept/mixing.html#control-group-0-flight-control). The 2 indates channel 2 which corresponds to Yaw. The next two values correspond to behavior to negative and positive yaw signals. For this dual-thruster design we specify a negative value for negative yaw (turn left) inputs so that the right motor thrusts forwards (neg * neg = pos). We specify a negative value for positive yaw (turn right) inputs so that the right motor thrusts backwards (pos * neg = neg). The next 0 indicates we want an offset of 0. The -10000 and 10000 indicate that we want the full possible range of outputs (-1 to 1) (no cuttoffs).

`S: 0 2   -10000  -10000      0  -10000  10000`
    
The next `S:` defines output behavior for the next control signal input. This time the 3 corresponds to Throttle. The next two values correspond to behavior to negative and positive throttle signals. For this dual-thruster we specify a positive value for negative throttle (translate backward) inputs so that the right motor thrusts backwards (neg * pos = neg). We specify a positive value for positive throttle (translate forward) inputs so that the right motor thrusts forwards (pos * pos = pos). Again, we want no offset value and a full range of outputs.

`S: 0 3    10000   10000      0  -10000  10000`

4. Add the mixer file to the mixers CMakeLists.txt.
    - Add the line `asv_dual_thruster_030.main.mix` to the end of the list of current mixers [`ROMFS/px4fmu_common/mixers/CMakeLists.txt`](ROMFS/px4fmu_common/mixers/CMakeLists.txt)
5. Compile the PX4 stack to generate a .px4 file as specified in the "How to setup the development environment" section above.
    - `make clean` then `make <target>` (e.g. `make px4_fmu-v5_default` for Pixhawk 4)
    
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
