# orio_free_fleet

Demo of ORIO, free fleet and a physical robot.

## Compiling Instructions

```bash
cd ~/orio/orio_free_fleet
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
```

Change the gazebo plugins.

```bash
# Change door plugin
cd ~/orio_free_fleet/build/orio_magni_worlds/maps/office_door
sed -i 's/libdoor.so/liborio_door.so/' office_door.world

# Change lift plugin
cd ~/orio_free_fleet/build/orio_magni_worlds/maps/office_lift
sed -i 's/liblift.so/liborio_lift.so/' office_lift.world

cd ~/orio_free_fleet
colcon build
source install/setup.bash
```

Align the coordinate frames

## Authentication

The PLC endpoint is secured by a username and password, which is passed into the launch file of each world here:

```xml
<param from="$(find-pkg-share orio_magni_worlds)/worlds/$(var world_name)/credentials/http_auth.yaml"/>
```

Create the yaml file with the following format

```yaml
<put manager node name here>:
  ros__parameters:
    username: "xxx"
    password: "xxx"
```

## Running demos - Door

Make sure there is a connection to the Orio PLC first.

```bash
# Run free_fleet server
ros2 launch orio_magni_ff_server free_fleet_server.launch.xml

# run simulation
ros2 launch orio_magni_worlds office_door.launch.xml 

ros2 launch orio_magni_worlds test_office_door_simple_loop.launch.xml start_point:=1_d end_point:=1_a

```

There are 4 points from `1_a` to `1_d`.

## Running demos - Lift

Make sure there is a connection to the Orio PLC first. And make sure it is at either L1 or L2.

```bash
ros2 launch orio_magni_ff_server free_fleet_server.launch.xml

# run simulation
ros2 launch orio_magni_worlds office_lift.launch.xml 

ros2 launch orio_magni_worlds test_office_lift_simple_loop.launch.xml start_point:=1_a end_point:=2_c
```
