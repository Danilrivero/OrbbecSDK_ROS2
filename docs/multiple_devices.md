# Multiple devices

* To get the `usb_port` of the camera, plug in the camera and run the following command in the terminal:

```bash
ros2 run orbbec_camera list_devices_node
```

* Set the `device_num` parameter to the number of cameras you have.
* Go to the `OrbbecSDK_ROS2/launch/multi_xxx.launch.py` file and change the `usb_port`.
* Don't forget to put the `include` tag inside the `group` tag.
  Otherwise, the parameter values of different cameras may become contaminated.

```python
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, IncludeLaunchDescription, GroupAction, ExecuteProcess
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch_ros.actions import Node
from ament_index_python.packages import get_package_share_directory
import os

def generate_launch_description():
    # Node configuration
    cleanup_node = Node(
        package='orbbec_camera',
        executable='ob_cleanup_shm_node',
        name='camera',
        output='screen'
    )

    # Include launch files
    package_dir = get_package_share_directory('orbbec_camera')
    launch_file_dir = os.path.join(package_dir, 'launch')
    launch1_include = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(
            os.path.join(launch_file_dir, 'gemini2.launch.py')
        ),
        launch_arguments={
            'camera_name': 'camera_01',
            'usb_port': '6-2.4.4.2',  # replace your usb port here
            'device_num': '2'
        }.items()
    )

    launch2_include = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(
            os.path.join(launch_file_dir, 'gemini2.launch.py')
        ),
        launch_arguments={
            'camera_name': 'camera_02',
            'usb_port': '6-2.4.1',  # replace your usb port here
            'device_num': '2'
        }.items()
    )

    # If you need more cameras, just add more launch_include here, and change the usb_port and device_num

    # Launch description
    ld = LaunchDescription([
        cleanup_node,
        GroupAction([launch1_include]),
        GroupAction([launch2_include]),
    ])

    return ld

```

* Note that the astra camera uses semaphores for process synchronization.
  If the camera start fails, the semaphore file may be left in `/dev/shm`,
  causing the next start to become stuck. To avoid this, run the following command before launching:

```bash
ros2 run orbbec_camera ob_cleanup_shm_node
```

This will clean up `/dev/shm/`.

* To launch the cameras, run the following command:

```bash
ros2 launch orbbec_camera multi_camera.launch.py
```