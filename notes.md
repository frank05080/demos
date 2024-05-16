## Common Python Issue

when I run "python3", I got: python3: error while loading shared libraries: libpython3.10.so.1.0: cannot open shared object file: No such file or directory

Solution:
find / -name "libpython3.10.so.1.0"
sudo ln -s /usr/local/lib/libpython3.10.so.1.0 /usr/lib/libpython3.10.so.1.0
or
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

## ROS2 Underlay Overlay冲突

在ros2，如何理解：
[1.087s] WARNING:colcon.colcon_core.package_selection:Some selected packages are already built in one or more underlay workspaces:
        'composition' is in: /opt/ros/foxy
If a package in a merged underlay workspace is overridden and it installs headers, then all packages in the overlay must sort their include directories by workspace order. Failure to do so may result in build failures or undefined behavior at run time.
If the overridden package is used by another package in any underlay, then the overriding package in the overlay must be API and ABI compatible or undefined behavior at run time may occur.

If you understand the risks and want to override a package anyways, add the following to the command line:
        --allow-overriding composition


Ans:
在ROS 2中，这条警告消息是关于在构建和使用多个工作空间（workspaces）时可能出现的问题。让我们逐步解析这个消息：

### 1. 基本理解

- **colcon**：这是ROS 2的构建工具，用于构建、测试和安装ROS 2包。
- **underlay workspace**：这是一个已经存在并且可能已经构建过的工作空间。通常是一个基础的工作空间，比如 `/opt/ros/foxy`，其中包含了预编译的ROS 2包。
- **overlay workspace**：这是你当前正在构建或使用的工作空间，它可能依赖于一个或多个underlay工作空间。

### 2. 警告内容

```
[1.087s] WARNING:colcon.colcon_core.package_selection:Some selected packages are already built in one or more underlay workspaces:
        'composition' is in: /opt/ros/foxy
```

这部分告诉你，你选中的包（在本例中是`composition`包）已经在一个或多个underlay工作空间中构建过了，例如在 `/opt/ros/foxy` 中。

### 3. 潜在风险

```
If a package in a merged underlay workspace is overridden and it installs headers, then all packages in the overlay must sort their include directories by workspace order. Failure to do so may result in build failures or undefined behavior at run time.
```

如果你在overlay工作空间中覆盖了一个在underlay工作空间中已经存在的包，并且这个包安装了头文件，那么所有在overlay工作空间中的包必须按照工作空间的顺序来排序它们的include目录。如果不这样做，可能会导致构建失败或运行时出现未定义的行为。

```
If the overridden package is used by another package in any underlay, then the overriding package in the overlay must be API and ABI compatible or undefined behavior at run time may occur.
```

如果你覆盖的包在任何underlay工作空间中被其他包使用，那么在overlay工作空间中覆盖的这个包必须在API和ABI（应用程序二进制接口）上兼容，否则可能会导致运行时出现未定义的行为。

### 4. 如何处理

```
If you understand the risks and want to override a package anyways, add the following to the command line:
        --allow-overriding composition
```

如果你理解这些风险并且仍然想覆盖一个包，你可以在命令行中添加 `--allow-overriding composition` 参数。

### 总结

这条警告信息提醒你，在多个工作空间中构建和覆盖包时需要小心处理，以避免潜在的构建失败和运行时问题。如果你确定要覆盖包，确保你已经考虑了API和ABI兼容性，并且知道如何正确排序include目录。




如何理解如果你在overlay工作空间中覆盖了一个在underlay工作空间中已经存在的包，并且这个包安装了头文件，那么所有在overlay工作空间中的包必须按照工作空间的顺序来排序它们的include目录。如果不这样做，可能会导致构建失败或运行时出现未定义的行为。

Ans:

为了理解这一部分，我们需要先了解一些概念和背景知识。

### 概念

- **Underlay Workspace**：已经存在的工作空间，包含了预编译的包。
- **Overlay Workspace**：你当前正在构建或使用的工作空间，可以依赖于一个或多个underlay工作空间。
- **Include Directories**：在构建C++等语言的程序时，需要使用头文件（header files）。这些头文件的路径需要被指定，也就是包含目录。

### 背景

在ROS 2中，工作空间是层次化的。你可以有多个工作空间叠加在一起，overlay工作空间可以覆盖underlay工作空间中的包。

### 详细解析

假设你有以下两个工作空间：

1. **Underlay Workspace**：包含了包A和包B。
2. **Overlay Workspace**：包含了包A的一个新版本，以及包C。

如果包A在underlay工作空间中安装了头文件，这意味着这些头文件在 `/opt/ros/foxy/include`（假设这个路径是underlay工作空间）中可用。而在overlay工作空间中，你可能也安装了包A的头文件，这些头文件在你自己的工作空间路径中。

### 问题的根源

当你在overlay工作空间中覆盖了underlay工作空间中的包，并且这个包安装了头文件时，编译器需要知道使用哪些头文件。头文件的路径可能如下：

- Underlay Workspace的头文件路径：`/opt/ros/foxy/include`
- Overlay Workspace的头文件路径：`~/your_overlay_workspace/include`

如果编译器先搜索underlay工作空间的头文件路径，那么它会使用旧版本的头文件。反之，如果先搜索overlay工作空间的头文件路径，它会使用新版本的头文件。为了确保编译器使用正确的头文件，必须按正确的顺序排序include目录。

### 举例

假设包A安装了头文件 `a.h`，在编译包C时，需要包含 `a.h`。在CMakeLists.txt或类似的构建文件中，包含目录的顺序决定了编译器使用哪个版本的头文件：

```cmake
include_directories(
  ~/your_overlay_workspace/include
  /opt/ros/foxy/include
)
```

这样，编译器会首先搜索overlay工作空间的头文件路径，如果找不到才会搜索underlay工作空间的头文件路径。

### 风险

如果不按这个顺序排序：

```cmake
include_directories(
  /opt/ros/foxy/include
  ~/your_overlay_workspace/include
)
```

可能会导致编译器使用旧版本的头文件，从而引发以下问题：

1. **构建失败**：因为新版本和旧版本的API可能不兼容。
2. **运行时未定义行为**：即使成功构建，运行时可能会出现不可预见的问题，因为运行时可能使用了不兼容的二进制文件（ABI问题）。

### 总结

在overlay工作空间中覆盖underlay工作空间中的包时，确保所有包的include目录按照正确的顺序排序，即先包含overlay工作空间的目录，再包含underlay工作空间的目录。这有助于避免构建和运行时的问题，确保你使用的是最新的、正确的头文件和API。