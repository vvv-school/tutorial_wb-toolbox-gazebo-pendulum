# How to control a simple pendulum in Gazebo through WB-Toolbox
This tutorial describes the steps needed to simulate in Gazebo a simple pendulum controlled through a Simulink model built using the [WB-Toolbox](https://github.com/robotology/wb-toolbox).

Before moving to the core of the tutorial, please check that you installed the needed dependecies, which are:
- Gazebo
- Matlab/Simulink
- [robotology-superbuild](https://github.com/robotology/robotology-superbuild) with the options `ROBOTOLOGY_ENABLE_CORE`, `ROBOTOLOGY_USES_GAZEBO` and `ROBOTOLOGY_ENABLE_DYNAMICS` enabled

A description on how to set-up your environment is provided in [Setting-up your environment](#setting-up-your-environment). If your environment is already set-up correctly, move directly to [Running the simulation](#running-the-simulation).

## Setting-up your environment

### 1. Add the simple pendulum model in Gazebo
If you installed [robotology-superbuild](https://github.com/robotology/robotology-superbuild), you should have some sample models in the directory `GazeboYARPPlugins/tutorial/model/`. 
To install these models in Gazebo, append the directory `GazeboYARPPlugins/tutorial/model/` to the `GAZEBO_MODEL_PATH` environment variable by adding the following lines to the `.bashrc`:

```bash
export PENDULUM_PATH=${ROBOTOLOGY_SUPERBUILD_ROOT}/robotology/GazeboYARPPlugins/tutorial/model
export GAZEBO_MODEL_PATH=${GAZEBO_MODEL_PATH}:${PENDULUM_PATH}
```
Now if you open Gazebo, you can see that the path to the sample models is added in the `Insert` tab and you can finally use the simple pendulum model in Gazebo.

### 2. Make the simple pendulum model visible to YARP
To be able to interface the simple pendulum model with YARP, you should update the `$PATH` environment variable by adding the following line to the `.bashrc`:
```bash
export YARP_DATA_DIRS=$YARP_DATA_DIRS:$PENDULUM_PATH/single_pendulum
```
Now you need to let YARP know which is the robot you are going to use by setting the `YARP_ROBOT_NAME`:
```bash
export YARP_ROBOT_NAME=singlePendulumGazebo
```
:bulb: In case you do not know the `YARP_ROBOT_NAME` of your robot, check the field `robot` in the configuration file `yarpmotorgui.ini`.

By typing `yarp resource --find model.urdf` in the command line, you can check if YARP finds the correct Gazebo model.

### 3. Install WB-Toolbox in Simulink
To be able to use [WB-Toolbox](https://github.com/robotology/wb-toolbox) in Simulink, you need to update the default search path in MATLAB. An easy way to do it is to add the following paths to the `pathdef.m` file located at `Documents/MATLAB`:

```bash
'$ROBOTOLOGY_SUPERBUILD_INSTALL_PREFIX/share:', ...
'$ROBOTOLOGY_SUPERBUILD_INSTALL_PREFIX/mex:', ...
'$ROBOTOLOGY_SUPERBUILD_INSTALL_PREFIX/share/WBToolbox:', ...
'$ROBOTOLOGY_SUPERBUILD_INSTALL_PREFIX/share/WBToolbox/cmake:', ...
'$ROBOTOLOGY_SUPERBUILD_INSTALL_PREFIX/share/WBToolbox/images:', ...
```
This process can be done automatically by running **once** a [startup matlab script](https://github.com/robotology/wb-toolbox/blob/master/matlab/startup_wbitoolbox.m.in) that is installed inside the `$WBTOOLBOX_INSTALL_PREFIX` if **WB-Toolbox** is compiled outside the **robotology-superbuild**, or inside the `$ROBOTOLOGY_SUPERBUILD_INSTALL_PREFIX` if **WB-Toolbox** is installed through the **robotology-superbuild**.

:warning: Note that you need to run MATLAB from `Documents/MATLAB` to be able to access the WB-Toolbox (it is raccomended to make an alias for simplicity — see [WB-Toolbox troubleshooting](https://robotology.github.io/wb-toolbox/mkdocs/troubleshooting/)).

## Running the simulation
**STEP 0)** Set `YARP_ROBOT_NAME=singlePendulumGazebo` in the `.bashrc`.

**STEP 1)** In a first terminal, run a YARP server by typing the command:
```bash
yarp server
```
**STEP 2)** In a second terminal, start Gazebo with the clock option enabled to sychronize it with YARP:
```bash
gazebo -slibgazebo_yarp_clock.so
```
Now you can add the simple pendulum model in Gazebo (download the model from [this link](https://github.com/robotology/gazebo-yarp-plugins/tree/master/tutorial/model)). 

![pendulum_gazebo](https://user-images.githubusercontent.com/12125604/50785860-24a85100-12b2-11e9-84c5-d58d477ac251.png)

**STEP 3)** In a third terminal, run the command:
```bash
yarp name list
```
to retrieve the control board name corresponding to the model in Gazebo. For the simple pendulum, it should be `singlePendulumGazebo`.

**STEP 4)**  Now you can open Simulink and start to construct your controller using WB-Toolbox. The building blocks needed to build the controller are:
- `Set References` to control the pendulum either in position, velocity or torque.
- `Get measurement` to read the position, velocity or torque of the model used in Gazebo.
- `Simulator Synchronizer` to set the control period (e.g. 0.01).
- `Configuration` to actually connect the Simulink model to the Gazebo model. Change the parameters `Robot Name`, `Controlled Joints` and `Control Board Names` according to the model used in Gazebo. For the simple pendulum, these parameters should be set to `singlePendulumGazebo`, `joint` and `body` (see `yarpmotorgui.ini` for `Robot Name` and `Control Board Names`, while see `conf/gazebo_controlboard.ini` for `Controlled Joints`).

![pendulum_config_wbt](https://user-images.githubusercontent.com/12125604/50785690-c1b6ba00-12b1-11e9-8b61-a8b2f77afcba.png)

After connecting the needed building blocks, the resulting controller should look like this (see `pendulum_tutorial.mdl`. 

![pendulum_simulink](https://user-images.githubusercontent.com/12125604/50785684-bbc0d900-12b1-11e9-9f96-b54dc0d3ea95.png)

To make the controller work, note that you have to:
- save the Simulink model as `.mdl` file.
- run the Simulink model using the `discrete (no continuous states)` solver in `Fixed-step`. It is recommended to set the simulation step size equal to `0.01`. 

![pendulum_config_model](https://user-images.githubusercontent.com/12125604/50785868-2e31b900-12b2-11e9-93b5-3d53d6fc54be.png)

**STEP 5)**  Finally, you can run the Simulink model. **[ADD GIF]**
