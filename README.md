# Update 20220714
- 新增了`header_only_ikdtree`分支，只有两个头文件，非常方便直接嵌入您自己的工程中。
- 在`header_only_ikdtree`分支中，修复了一些引起编译报警的代码(如int和unsigned int做比较，unused variable等)。

# 中文注解
「*读者在阅读此代码和注释前，应当至少阅读fast-lio2或者ikd-tree论文之一，并充分理解ikd-tree算法原理。*」  
在Fast-Lio2中，`class KD_TREE` 被调用的`接口函数`&`成员变量`有（按调用顺序）：  
- `acquire_removed_points()` //获取所有被标记为delete的点；在算法中虽然被调用了一次，但没有实际意义，可忽略。
- `Delete_Point_Boxes()` //删除指定Boxes中的点；配准用的localmap(也即ikdtree)的边界是需要跟着激光雷达移动的，该函数用于删除激光雷达移动后位于边界外的点。在算法循环的 **lasermap_fov_segment()** 函数中被调用一次。//注：lasermap_fov_segment()函数的作用正是更新localmap边界。
- `Root_Node` //成员变量，ikdtree根节点指针；算法随时调用。
- `set_downsample_param()` //设置体素降采样分辨率；仅在初始化ikdtree时调用一次。
- `Build()` //构建平衡的(sub)kdtree；仅在初始化时调用一次，用第一帧点云初始化ikdtree。
- `validnum()` //返回当前总kdtree中有效点的数量；每次算法循环都会调用一次，获取当前ikdtree地图的点云数量。
- `size()` //返回当前总kdtree中所有点的数量，包括无效点；算法随时调用。
- `PCL_Storage` //成员变量，ikdtree中维护的点云内存块指针；仅在需要可视化ikdtree地图时，在算法循环中被调用。
- `flatten()` //将指定Node(即kdtree结构中的节点)下的点云另存为线性化排列的点云；仅在需要可视化ikdtree地图时，在算法循环中被调用。  
- `Nearest_Search()` //支持kNN，ranged-kNN搜索；对该函数的调用被封装在h_share_model()函数中，而h_share_model()又被作为一个函数对象传递给 **class esekf** 的实例 **kf** 中，后者作为前端完成基于ESKF的位姿估算，体现为算法循环中的 **kf.update_iterated_dyn_share_modified()** 语句。
- `Add_Points()` //添加新的点到ikdtree结构中；用于在当前帧配准完成后，将当前帧中有价值的点插入进来，在专门负责增量式更新的 **map_incremental()** 中被调用。  

上述调用接口中的重要函数已详细注释，包括`Delete_Point_Boxes()`，`Build()`，`Nearest_Search()`，`Add_Points()`。

# ikd-Tree
**ikd-Tree** is an incremental k-d tree designed for robotic applications. The ikd-Tree incrementally updates a k-d tree with new coming points only, leading to much lower computation time than existing static k-d trees. Besides point-wise operations, the ikd-Tree supports several features such as box-wise operations and down-sampling that are practically useful in robotic applications.

## What does ikd-Tree support?

- Build a balanced k-d tree - `Build()`

- Dynamically insert points to or delete points from the k-d tree - `Add_Points() / Delete_Points()`

- Delete points inside given axis-aligned bounding boxes - `Delete_Point_Boxes()`

- K Nearest Neighbor Search with range limitation - `Nearest_Search()`

- Acquire points inside a given axis-aligned bounding box on the k-d tree - `Box_Search()`

- Acquire points inside a ball with given radius on the k-d tree - `Radius_Search()`

## User Manual

- Browse the [User Manual](https://github.com/hku-mars/ikd-Tree/blob/main/documents/UserManual.pdf) for using our ikd-Tree.

## Developers

- [Yixi CAI 蔡逸熙](https://github.com/Ecstasy-EC): Data structure design and implementation

- [Wei XU 徐威](https://github.com/XW-HKU): Incorporation into  [LiDAR-inertial odometry package FAST_LIO2 (TRO, 2022)](https://github.com/hku-mars/FAST_LIO)


## Related paper

- [ikd-Tree: An Incremental K-D Tree for robotic applications](https://arxiv.org/abs/2102.10808)

- [FAST-LIO2: Fast Direct LiDAR-Inertial Odometry](https://ieeexplore.ieee.org/abstract/document/9697912)

If you are using any code of this repo in your research, please cite at least one of the articles as following:
- **ikd-Tree**
```
@article{cai2021ikd,
  title={ikd-Tree: An Incremental KD Tree for Robotic Applications},
  author={Cai, Yixi and Xu, Wei and Zhang, Fu},
  journal={arXiv preprint arXiv:2102.10808},
  year={2021}
}
```
- **FAST-LIO2**
```
@article{xu2022fast,
  title={Fast-lio2: Fast direct lidar-inertial odometry},
  author={Xu, Wei and Cai, Yixi and He, Dongjiao and Lin, Jiarong and Zhang, Fu},
  journal={IEEE Transactions on Robotics},
  year={2022},
  publisher={IEEE}
}
```

## Build & Run demo
### 1. How to build this project
```bash
cd ~/catkin_ws/src
git clone git@github.com:hku-mars/ikd-Tree.git
cd ikd-Tree/build
cmake ..
make -j 9
```
### 2. Run our examples

**Note: To run Example 2 & 3, please download the PCD file ([HKU_demo_pointcloud](https://drive.google.com/file/d/1tMYiBIFn-fcjisaoIrmIKA09NICGG9KJ/view?usp=sharing))  into`${Your own directory}/ikd-Tree/materials`**

```bash
cd ${Your own directory}/ikd-Tree/build
# Example 1. Check the speed of ikd-Tree
./ikd_tree_demo
# Example 2. Searching-points-by-box examples
./ikd_Tree_Search_demo
# Example 3. An aysnc. exmaple for readers' better understanding of the principle of ikd-Tree
./ikd_tree_async_demo
```

**Example 2: ikd_tree_Search_demo** 

Box Search Result  |   Radius Search Result
:-------------------------:|:-------------------------:
![](materials/imgs/Box_Search.png) |  ![](materials/imgs/Radius_Search.png)

Points returned from the two search methods are shown in red.

**Example 3: ikd_tree_Async_demo**

Original Map:

<img src="materials/imgs/HKU_campus.png" style="zoom:50%;" />



Box Delete Results:

Points removed from ikd-Tree(red) |       Map after box delete        
:-------------------------:|:-------------------------:
![](materials/imgs/removed.png) |  ![](materials/imgs/remain.png)

This example is to demonstrate the asynchronous phenomenon in ikd-Tree. The points are deleted by attaching 'deleted' on the tree nodes (map shown in the ) instead of being removed from the ikd-Tree immediately. They are removed from the tree when rebuilding process is performed. Please refer to our paper for more details about delete and rebuilding.


## Acknowledgments
- Thanks [Marcus Davi](https://github.com/Marcus-Davi) for helps in templating the ikd-Tree for more general applications.

- Thanks [Hyungtae Lim 임형태](https://github.com/LimHyungTae) for providing application examples on point clouds. 

## License

The source code of ikd-Tree is released under [GPLv2](http://www.gnu.org/licenses/old-licenses/gpl-2.0.html) license. For commercial use, please contact Mr. Yixi CAI (<yixicai@connect.hku.hk>) or Dr. Fu ZHANG (<fuzhang@hku.hk>).
