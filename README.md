# Info
The pptk library which is provided via the *Python Package Index* supports Python versions <=3.7. In order to work on higher versions pptk has to be built from source. Additionally, some files have to adapted to work on *Ubuntu 22.4.1 LTS* with *oneTBB* release 2021.7.0. For the updated installation guide see the **Install** section. The following files had to be adapted:

**pptk/kdtree/src/kdtree-impl.h**
```python
# old
class BuildTask : public tbb::task {...}
# new
class BuildTask : public tbb::task_group {...}

# old
tbb::task* execute() {...}
# new 
tbb::task_group* execute() {...}

# Added after BuildTask<T, dim>* left_task = NULL;
tbb::task_group tg

# old
left_task = new (tbb::task::allocate_child()) BuildTask<T, dim>(current_node->left, begin_index_, split_index, indices_, node_box_,points_, num_points_, build_params_);
# new
left_task = BuildTask<T, dim>(current_node->left, begin_index_, split_index, indices_, node_box_,points_, num_points_, build_params_);
tg.run(left_task);

# old
right_task = new (tbb::task::allocate_child()) BuildTask<T, dim>(current_node->right, split_index, end_index_, indices_, node_box_,points_, num_points_, build_params_);
# new
right_task = BuildTask<T, dim>(current_node->right, split_index, end_index_, indices_, node_box_, points_, num_points_, build_params_);
tg.run(right_task);

# old
if (left_task && right_task) {
  set_ref_count(3);
  spawn(*right_task);
  spawn_and_wait_for_all(*left_task);
} else if (right_task) {  // left empty
  set_ref_count(2);
  spawn_and_wait_for_all(*right_task);
} else {  // right empty
  set_ref_count(2);
  spawn_and_wait_for_all(*left_task);
}
# new
tg.wait();

# old
BuildTask<T, dim>& root_task = *new (tbb::task::allocate_root()) BuildTask<T, dim>(root, 0, (int)num_valid_points, indices, bounding_box, points, num_points, build_params);
# new 
BuildTask<T, dim>& root_task = *new BuildTask<T, dim>(root, 0, (int)num_valid_points, indices, bounding_box, points, num_points, build_params);
```

**pptk/processing/estimate_normals/estimate_normals.cpp**
```python
# old
tbb::task_scheduler_init(1);
# new
tbb::task_arena(1);
```

# pptk - Point Processing Toolkit

Copyright (C) 2011-2018 HERE Europe B.V.

The Point Processing Toolkit (pptk) is a Python package for visualizing and processing 2-d/3-d point clouds.

At present, pptk consists of the following features.

* A 3-d point cloud viewer that
  - accepts any 3-column numpy array as input,
  - renders tens of millions of points interactively using an octree-based level of detail mechanism,
  - supports point selection for inspecting and annotating point data.
* A fully parallelized point k-d tree that supports k-nearest neighbor queries and r-near range queries
  (both build and queries have been parallelized).
* A normal estimation routine based on principal component analysis of point cloud neighborhoods.

[Homepage](https://heremaps.github.io/pptk/index.html)

![pptk screenshots](/docs/source/tutorials/viewer/images/tutorial_banner.png)

The screenshots above show various point datasets visualized using pptk.
The `bildstein1` Lidar point cloud from Semantic3D (left),
Beijing GPS trajectories from Geolife (middle left),
`DistrictofColumbia.geojson` 2-d polygons from US building footprints (middle right),
and a Mobius strip (right).
For details, see the [tutorials](https://heremaps.github.io/pptk/tutorial.html).

## License

Unless otherwise noted in `LICENSE` files for specific files or directories,
the [LICENSE](LICENSE) in the root applies to all content in this repository.

## Install

Install pptk from the .whl file that results from [building pptk from source](#build).

```
>> pip install <.whl file>
```

## Quickstart

In Python, generate 100 random 3-d points.

```
>> import numpy as np
>> x = np.random.rand(100, 3)
```

Visualize.

```
>> import pptk
>> v = pptk.viewer(x)
```

Set point size to 0.01.

```
>> v.set(point_size=0.01)
```

For more advanced examples, see [tutorials](https://heremaps.github.io/pptk/tutorial.html).

## Build

We provide CMake scripts for automating most of the build process, but ask the
user to manually prepare [dependencies](#requirements) and record their paths
in the following CMake cache variables.

* `Numpy_INCLUDE_DIR`
* `Eigen_INCLUDE_DIR`
* `TBB_INCLUDE_DIR`
* `TBB_tbb_LIBRARY`
* `TBB_tbb_RUNTIME`
* `TBB_tbbmalloc_LIBRARY`
* `TBB_tbbmalloc_RUNTIME`
* `Qt5_DIR`

To set these variables, either use one of CMake's GUIs (ccmake or cmake-gui),
or provide an initial CMakeCache.txt in the target build folder
(for examples of initial cache files, see the CMakeCache.<platform>.txt files)

##### Requirements

Listed are versions of libraries used to develop pptk, though earlier versions
of these libraries may also work.

* Ubuntu 22.4.1 LTS
* [QT](https://wiki.qt.io/Building_Qt_5_from_Git#Getting_the_source_code) 5.4
* [oneTBB](https://www.threadingbuildingblocks.org/) 2021.7.0
* [Eigen](http://eigen.tuxfamily.org) 3.4.0
* [Python](https://www.python.org/) 3.10
* [Numpy](http://www.numpy.org/) 1.23.5

Additionally install **Patchelf**:
```
sudo apt update
sudo apt install patchelf
```

1. Create an empty build folder

```
>> mkdir <build_folder>
```

2. Create an initial CMakeCache.txt under <build_folder> and use it to provide
values for the CMake cache variables listed above. (e.g. see CMakeCache.win.txt)

3. Type the following...

```
>> cd <build_folder>
>> cmake -G "NMake Makefiles" <source_folder>
>> nmake
>> python setup.py bdist_wheel
>> pip install dist\<.whl file>
```

