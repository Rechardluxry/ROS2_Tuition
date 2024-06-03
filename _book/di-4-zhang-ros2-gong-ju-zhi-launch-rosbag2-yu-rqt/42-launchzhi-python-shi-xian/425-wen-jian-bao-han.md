### 4.2.4 文件包含

在 launch 文件中可以包含其他launch文件，需要使用的API为：launch.actions.IncludeLaunchDescription 和 launch.launch\_description\_sources.PythonLaunchDescriptionSource。

**需求：**新建 launch 文件，包含 4.2.3 中的 launch 文件并为之传入设置背景色相关的参数。

**示例：**

在 cpp01\_launch/launch/py 目录下新建 py04\_include.launch.py 文件，输入如下内容：

```py
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource

import os
from ament_index_python import get_package_share_directory

def generate_launch_description():

    include_launch = IncludeLaunchDescription(
        launch_description_source= PythonLaunchDescriptionSource(
            launch_file_path=os.path.join(
                get_package_share_directory("cpp01_launch"),
                "launch/py",
                "py03_args.launch.py"
            )
        ),
        launch_arguments={
            "background_r": "200",
            "background_g": "100",
            "background_b": "70",
        }.items()
    )

    return LaunchDescription([include_launch])
```

**代码解释：**

```py
include_launch = IncludeLaunchDescription(
        launch_description_source= PythonLaunchDescriptionSource(
            launch_file_path=os.path.join(
                get_package_share_directory("cpp01_launch"),
                "launch/py",
                "py03_args.launch.py"
            )
        ),
        launch_arguments={
            "background_r": "200",
            "background_g": "100",
            "background_b": "70",
        }.items()
    )
```

上述代码将包含一个launch文件并为launch文件传参。

在 IncludeLaunchDescription 对象中：

* launch\_description\_source：用于设置被包含的 launch 文件；
* launch\_arguments：元组列表，每个元组中都包含参数的键和值。

在 PythonLaunchDescriptionSource 对象中：

* launch\_file\_path：被包含的 launch 文件路径。



