
## Code Project 路径
    代码路径：171机器(10.118.28.171) /home/yangyu/workspace/4DMOS/

    环境路径：conda activate /home/yangyu/envs/4dmos

## 数据准备

```
./
└── data/sequences
  ├── 24/          数据文件夹名
  |   ├── pcd/	   原始点云数据
  |   |	├── 1.pcd
  |   |	├── 2.pcd
  |   |	└── ...
  │   ├── velodyne/	   坐标转换后（激光雷达为原点坐标）的点云数据
  |   |	├── 000000.bin
  |   |	├── 000001.bin
  |   |	└── ...
  │   └── mapping_pose.txt 原始位姿
  |   └── poses.txt  转换后的位姿（齐次矩阵形式）
  |   └── calib.txt  对角矩阵
  └── ...
```

1. 在当前工程文件夹data/sequences路径下创建一个新的文件夹，例如

     ```
    mkdir data/sequences/24          # 数据文件夹
    mkdir data/sequences/24/pcd      # 存放原始点云数据
    mkdir data/sequences/24/velodyne # 存放坐标转换（每帧转换为激光雷达为原点的坐标系下）后的点云
     ```

2. 将点云数据软连接到刚创建的```pcd```文件夹下：

    ```ln -s my_pcd_data data/sequences/24/pcd```

3. 将位姿矩阵转换为齐次矩阵形式

    (1) ```cp my_pose.txt data/sequences/24/mapping_pose.txt```

    (2)运行位姿转换代码，得到齐次矩阵形式的位姿poses.txt：
    ```
    若位姿为"x y z roll pitch yaw"形式：
        python utils/transform_pose.py
        注意修改代码第17,45行的路径，第26,33行读取列数
    
    若位姿为"qw qx qy qz x y z"形式：
        python utils/transform_pose2.py
        注意修改代码第27,55行的路径，第36,43行读取列数

3. 将点云的坐标转换为激光雷达原点坐标系

    运行坐标转换代码，在data/sequences/24/velodyne下得到转换后的点云数据：
    ```
    若位姿为"x y z roll pitch yaw"形式：
        python utils/transform_points.py
        注意修改代码第55,56,58行的路径，第63行读取列数
    
    若位姿为"qw qx qy qz x y z"形式：
        python utils/transform_points2.py
        注意修改代码第65,66,68行的路径，第73行读取列数
    ```
 4. 创建一个calib.txt，使用对角阵即可，内容为：

    Tr: 1.0 0.0 0.0 0.0 0.0 1.0 0.0 0.0 0.0 0.0 1.0 0.0

## 配置文件

1. 修改config/config.yaml第40行，为数据所在文件夹的名字，如上方的24
2. 修改config/semantic-kitti-mos.yaml第189行，为数据所在文件夹的名字，如上方的24
3. 修改scripts/predict_confidences.py第29行，为数据集所在文件夹得名字，如上方的[24]，支持多个序列同时测试，如[23,24,25...]
4. 修改scripts/confidences_to_labels.py第30行，为数据集所在文件夹得名字，如上方的[24]，支持多个序列同时测试，如[23,24,25...]

## 运行代码

```
conda activate /home/yangyu/envs/4dmos
export DATA=/home/yangyu/workspace/4DMOS/data/sequences
python scripts/predict_confidences.py -w ckpt/10_scans.ckpt
python scripts/confidences_to_labels.py -p predictions/10_scans/poses/confidences/24这里按照实际路径给出
```

## 清理运动点

```
清除每帧中运动物体的点，在predictions/clean_scans/sequences/24文件夹下保存清理后的点云
python utils/scan_cleaner.py
```

## 可视化

```
python utils/visulization.py #注意修改158,184,187行 为实际路径
python utils/make_video.py
```
