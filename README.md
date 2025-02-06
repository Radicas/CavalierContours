# Summary(概要)

NOTE: This C++ library is not being actively developed. Development is continuing in Rust (with a C FFI), the repository is [here](https://github.com/jbuckmccready/cavalier_contours). This repository will remain available but I do not plan to add new features or fix bugs due to lack of time and motivation! I recommend using the Rust library C API from C++, but if you'd like to help maintain the C++ then pull requests are welcome.

> **注意：** 此 C++ 库**目前不再进行积极开发**。开发已转向 **Rust**（带有 C FFI），仓库地址为：[这里](https://github.com/jbuckmccready/cavalier_contours)。  
> 该 C++ 仓库将保持可用，但由于时间和精力有限，我不打算添加新功能或修复 Bug！  
> **推荐**在 C++ 中使用 **Rust** 库的 **C API**，但如果你愿意帮助维护 C++ 版本，欢迎提交 Pull Request。

C++14 header only library (with a C API available) for processing 2D polylines containing both straight line and constant radius arc segments. Supports contour/parallel offsetting, boolean operations (OR, AND, NOT, XOR) between closed polylines, and other common functions (winding number, area, path length, distance to point, etc.). For interactive UI and development go to the development project [CavalierContoursDev](https://github.com/jbuckmccready/CavalierContoursDev). For quick code examples look in the [examples](examples/). Live web demo available [here](https://jbuckmccready.github.io/CavalierContoursWebDemo/) (note the page is quite large and may take a minute to download, it's created by building the CavalierContoursDev project to web assembly using Emscripten). For the C API header look [here](c_api_include/cavaliercontours.h).

> 这是一个 **C++14** 头文件库（提供 C API），用于处理包含直线和恒定半径弧段的 2D 折线。支持闭合折线之间的轮廓/平行偏移、布尔运算（OR、AND、NOT、XOR）以及其他常见功能（如绕数、面积、路径长度、点到折线的距离等）。
>
> - **交互式 UI 和开发**：请访问开发项目 [CavalierContoursDev](https://github.com/jbuckmccready/CavalierContoursDev)。
> - **快速代码示例**：可以在 [examples](examples/) 中找到。
> - **在线演示**：访问 [这里](https://jbuckmccready.github.io/CavalierContoursWebDemo/) 进行体验（注意：页面较大，可能需要一些时间下载，这是通过将 CavalierContoursDev 项目构建为 WebAssembly 生成的）。
> - **C API 头文件**：可以在 [这里](c_api_include/cavaliercontours.h) 找到。

<img src="https://github.com/jbuckmccready/CavalierContoursDoc/blob/master/gifs/PolylineOffsets.gif" width="400"/>

<img src="https://github.com/jbuckmccready/CavalierContoursDoc/blob/master/gifs/PolylineCombines.gif" width="400"/>

<img src="https://raw.githubusercontent.com/jbuckmccready/CavalierContoursDoc/master/images/pretty_examples/example1.png" width="400"/>

<img src="https://raw.githubusercontent.com/jbuckmccready/CavalierContoursDoc/master/images/pretty_examples/islands_example1.png" width="400"/>

<img src="https://raw.githubusercontent.com/jbuckmccready/CavalierContoursDoc/master/images/pretty_examples/example6.png" width="800"/>

# Table of Contents(内容列表)

- [Summary(概要)](#summary概要)
- [Table of Contents(内容列表)](#table-of-contents内容列表)
- [Quick Code Example(快速编码示例)](#quick-code-example快速编码示例)
- [Polyline Structure(Polyline 结构)](#polyline-structurepolyline-结构)
- [Other Programming Languages(其他编程语言)](#other-programming-languages其他编程语言)
- [Offset Algorithm and Stepwise Example(偏移算法和每步样例)](#offset-algorithm-and-stepwise-example偏移算法和每步样例)
  - [Original input polyline, _pline_ in blue, vertexes in red(原始输入 polyline，线是蓝色，顶点是红色)](#original-input-polyline-pline-in-blue-vertexes-in-red原始输入-polyline线是蓝色顶点是红色)
  - [Raw offset segments generated in purple (Step 1)(生成的紫色原始偏移段 (步骤 1))](#raw-offset-segments-generated-in-purple-step-1生成的紫色原始偏移段-步骤-1)
  - [Raw offset polyline created from raw offset segments, _pline1_ (in green) (Step 2)(由原始偏移段生成的原始偏移 polyline，pline1(绿色)(步骤 2))](#raw-offset-polyline-created-from-raw-offset-segments-pline1-in-green-step-2由原始偏移段生成的原始偏移-polylinepline1绿色步骤-2)
  - [Raw offset polyline self intersects (dark cyan) (Step 4)(原始偏移 polyline 自交(深蓝绿色)(步骤 4))](#raw-offset-polyline-self-intersects-dark-cyan-step-4原始偏移-polyline-自交深蓝绿色步骤-4)
  - [Valid open polyline slices created from self intersects (in green, red, and blue) (Step 5 \& 6)(从自交点创建的有效开放折线切片(绿色，红色，蓝色)(步骤 5 和 6))](#valid-open-polyline-slices-created-from-self-intersects-in-green-red-and-blue-step-5--6从自交点创建的有效开放折线切片绿色红色蓝色步骤-5-和-6)
  - [Open polyline slices stitched together (in red and blue) (Step 7)(开放的折线切片缝合在一起(红色和蓝色)(步骤 7))](#open-polyline-slices-stitched-together-in-red-and-blue-step-7开放的折线切片缝合在一起红色和蓝色步骤-7)
- [Interactively Exploring the Algorithm(交互式探索此算法)](#interactively-exploring-the-algorithm交互式探索此算法)
- [Performance(性能)](#performance性能)
  - [Benchmarks(Benchmarks)](#benchmarksbenchmarks)
    - [CavalierContours (Arcs Approximated) vs. Clipper](#cavaliercontours-arcs-approximated-vs-clipper)
    - [CavalierContours (Arcs Included) vs. Clipper](#cavaliercontours-arcs-included-vs-clipper)
- [Implementation Notes and Variations(实现说明和变更)](#implementation-notes-and-variations实现说明和变更)
  - [Float Comparing and Thresholding](#float-comparing-and-thresholding)
  - [Joining Raw Offset Segments](#joining-raw-offset-segments)
  - [Stitching Open Polylines](#stitching-open-polylines)
- [Development(开发)](#development开发)
- [API Stability(接口稳定性)](#api-stability接口稳定性)
- [Project Motivation and Goal(工程的动力和目标)](#project-motivation-and-goal工程的动力和目标)
- [Algorithm Complexity and 2D Spatial Indexing(算法复杂度和 2D 索引)](#algorithm-complexity-and-2d-spatial-indexing算法复杂度和-2d-索引)
  - [Packed Hilbert R-Tree(打包的希尔伯特 R-Tree)](#packed-hilbert-r-tree打包的希尔伯特-r-tree)
- [References(引用)](#references引用)

# Quick Code Example(快速编码示例)

```c++
#include "cavc/polylineoffset.hpp"

// input polyline
// 输入 polyline
cavc::Polyline<double> input;
// add vertexes as (x, y, bulge)
// 添加顶点 (x, y, bulge)
input.addVertex(0, 25, 1);
input.addVertex(0, 0, 0);
input.addVertex(2, 0, 1);
input.addVertex(10, 0, -0.5);
input.addVertex(8, 9, 0.374794619217547);
input.addVertex(21, 0, 0);
input.addVertex(23, 0, 1);
input.addVertex(32, 0, -0.5);
input.addVertex(28, 0, 0.5);
input.addVertex(39, 21, 0);
input.addVertex(28, 12, 0);
input.isClosed() = true;

// compute the resulting offset polylines, offset = 3
std::vector<cavc::Polyline<double>> results = cavc::parallelOffset(input, 3.0);
```

NOTE: If the offset results are wrong in some way you may need to adjust the scale of the numbers, e.g. scale the inputs up by 1000 (by multiplying all the X and Y components of the vertexes by 1000), perform the offset (with the offset value also scaled up by 1000), and then scale the output result back down by 1000. This is due the fixed bit representation of floating point numbers and the absolute float comparing and thresholding used by the algorithm.

> 注意： 如果偏移结果出现错误，可能需要调整数值的缩放比例。例如，可以通过将所有顶点的 X 和 Y 分量乘以 1000 来将输入放大 1000，执行偏移（偏移量也需要放大 1000），然后再将输出结果缩小回 1000。这是由于浮点数的固定位表示以及算法中使用的绝对浮点比较和阈值设置所导致的。

# Polyline Structure(Polyline 结构)

Polylines are defined by a sequence of vertexes and a bool indicating whether the polyline is closed or open. Each vertex has a 2D position (x and y) as well as a bulge value. Bulge is used to define arcs, where `bulge = tan(theta/4)`. `theta` is the arc sweep angle from the starting vertex position to the next vertex position. If the polyline is closed then the last vertex connects to the first vertex, otherwise it does not (and the last vertex bulge value is unused). See [[2]](#references) for more details regarding bulge calculations.

> 折线由一系列顶点和一个布尔值定义，该布尔值表示折线是闭合的还是开放的。每个顶点具有 2D 坐标（x 和 y）以及一个弯曲值（bulge）。弯曲值用于定义弧线，其中 `bulge = tan(theta / 4)`，`theta` 是从起始顶点位置到下一个顶点位置的弧度角度。如果折线是闭合的，那么最后一个顶点会连接到第一个顶点，否则不连接（并且最后一个顶点的弯曲值会被忽略）。有关弯曲值计算的更多详细信息，请参见 [[2]](#references)。

# Other Programming Languages(其他编程语言)

CavalierContours is written in C++ and makes available a C API. Here are some wrappers in other languages:

> CavalierContours 是用 C++编写的，并且带有 C API。以下是一些其他语言的包装：

[Python](https://github.com/proto3/cavaliercontours-python) (wraps the C API)

# Offset Algorithm and Stepwise Example(偏移算法和每步样例)

1. Generate raw offset segments from the input polyline, _pline_.
2. Create the raw offset polyline, _pline1_, by trimming/joining raw offset segments acquired in step 1.
3. If the input polyline, _pline_, has self intersections or is an open polyline then repeat steps 1 and 2 with the offset negated (e.g. if the offset was 0.5 then create raw offset polyline with offset of -0.5), this is known as _pline2_.
4. Find all self-intersects of _pline1_. If step 3 was performed then also find all intersects between _pline1_ and _pline2_. If _pline_ is an open polyline then also find intersects between _pline1_ and circles at the start and end vertex points of _pline_ with radius equal to the offset.
5. Create a set of open polylines by slicing _pline1_ at all of the intersect points found in step 4.
6. Discard all open polyline slices whose minimum distance to _pline_ is less than the offset.
7. Stitch together the remaining open polyline slices found in step 6, closing the final stitched results if _pline_ is closed.
   > 1. 从输入折线 pline 生成原始偏移段。
   > 2. 通过修剪/连接步骤 1 获取的原始偏移段，创建原始偏移折线 pline1。
   > 3. 如果输入折线 pline 存在自交或是开放折线，则重复步骤 1 和 2，但偏移取相反值（例如，如果偏移量为 0.5，则创建偏移量为 -0.5 的原始偏移折线），该折线称为 pline2。
   > 4. 查找 pline1 的所有自交点。如果执行了步骤 3，则还需查找 pline1 和 pline2 之间的所有交点。如果 pline 是开放折线，还需查找 pline1 与以 pline 起点和终点为圆心、偏移量为半径的圆之间的交点。
   > 5. 在步骤 4 发现的所有交点处对 pline1 进行切割，生成一组开放折线。
   > 6. 丢弃所有与 pline 的最小距离小于偏移量的开放折线片段。
   > 7. 将步骤 6 剩余的开放折线片段拼接在一起，并在 pline 为闭合折线时闭合最终拼接的结果。

The algorithm is mostly based on Liu et al. [[1]](#references) with some differences since the algorithm they describe for GCPP (general closest point pair) clipping fails for certain inputs with large offsets (or at least I am unable to make their algorithm work).

> 这个算法主要基于 Liu 等人的方法 [1]，但有一些不同之处，因为他们描述的 GCPP（一般最近点对）裁剪算法在某些输入和较大偏移量下失败（或者至少我无法使他们的算法正常工作）。

The key clarifications/differences are:

> 关键的区别在于：

- When raw offset segments are extended to form a raw offset polyline they are always joined by an arc to form a rounded constant distance from the input polyline.
- Dual offset clipping is only applied if input polyline is open or has self intersects, it is not required for a closed polyline with no self intersects.
- If the polyline is open then a circle is formed at each end point with radius equal to the offset, the intersects between those circles and the raw offset polyline are included when forming slices.
- GCPP (general closest point pair) clipping is never performed and instead slices are formed from intersects, then they are discarded if too close to the original polyline, and finally stitched back together.
- No special handling is done for adjacent segments that overlap (it is not required given the slice and stitch method).
- Collapsing arc segments (arcs whose radius is less than the offset value) are converted into a line and specially marked for joining purposes.
  > - 当原始偏移段被延伸形成原始偏移折线时，它们总是通过弧线连接，以保持与输入折线的恒定距离，形成圆角。
  > - 双偏移裁剪仅在输入折线是开放的或有自交时应用，对于没有自交的闭合折线，则不需要进行此操作。
  > - 如果折线是开放的，则在每个端点处生成一个半径等于偏移量的圆，在形成切片时，这些圆与原始偏移折线的交点会被包含在内。
  > - 从不执行 GCPP（一般最近点对）裁剪，而是通过交点形成切片，若切片距离原始折线过近，则被丢弃，最后将剩余切片拼接回一起。
  > - 对于相邻的重叠段，不做特殊处理（由于使用了切片和拼接方法，因此不需要额外处理）。
  > - 对于弧形段（半径小于偏移量的弧形段），会将其转换为直线，并特别标记用于拼接。

Here is example code and visualizations of the algorithm operating on a closed polyline with no self intersects as input.

> 以下是该算法在输入为无自交的闭合折线时的示例代码和可视化结果。

## Original input polyline, _pline_ in blue, vertexes in red(原始输入 polyline，线是蓝色，顶点是红色)

![Input Polyline](https://raw.githubusercontent.com/jbuckmccready/CavalierContoursDoc/master/images/algorithm_steps/input_polyline.png)

## Raw offset segments generated in purple (Step 1)(生成的紫色原始偏移段 (步骤 1))

![Raw Offset Segments](https://raw.githubusercontent.com/jbuckmccready/CavalierContoursDoc/master/images/algorithm_steps/raw_offset_segments.png)

## Raw offset polyline created from raw offset segments, _pline1_ (in green) (Step 2)(由原始偏移段生成的原始偏移 polyline，pline1(绿色)(步骤 2))

![Raw Offset Polyline](https://raw.githubusercontent.com/jbuckmccready/CavalierContoursDoc/master/images/algorithm_steps/raw_offset_polyline.png)

## Raw offset polyline self intersects (dark cyan) (Step 4)(原始偏移 polyline 自交(深蓝绿色)(步骤 4))

![Raw Offset Polyline Intersects](https://raw.githubusercontent.com/jbuckmccready/CavalierContoursDoc/master/images/algorithm_steps/raw_offset_polyline_intersects.png)

## Valid open polyline slices created from self intersects (in green, red, and blue) (Step 5 & 6)(从自交点创建的有效开放折线切片(绿色，红色，蓝色)(步骤 5 和 6))

![Valid Slices](https://raw.githubusercontent.com/jbuckmccready/CavalierContoursDoc/master/images/algorithm_steps/valid_slices.png)

## Open polyline slices stitched together (in red and blue) (Step 7)(开放的折线切片缝合在一起(红色和蓝色)(步骤 7))

![Final Output Polylines](https://raw.githubusercontent.com/jbuckmccready/CavalierContoursDoc/master/images/algorithm_steps/output.png)

# Interactively Exploring the Algorithm(交互式探索此算法)

An interactive UI app (implemented using Qt and QML) is available ([CavalierContoursDev](https://github.com/jbuckmccready/CavalierContoursDev)) to visualize and explore in real time the offset algorithm. This app is also compiled to web assembly, for the live web version go [here](https://jbuckmccready.github.io/CavalierContoursWebDemo/). The app was used to generate all the images in this markdown.

> 一个**交互式 UI 应用**（使用 **Qt 和 QML** 实现）可用于实时可视化和探索偏移算法，代码仓库在 [CavalierContoursDev](https://github.com/jbuckmccready/CavalierContoursDev)。
> 该应用还编译为 **WebAssembly**，在线演示版本可访问 [这里](https://jbuckmccready.github.io/CavalierContoursWebDemo/)。本文档中的所有图片均由该应用生成。

# Performance(性能)

The implementation is not entirely geared around performance but some profiling has been done to determine where to reserve memory up front and avoid trig functions where possible. I suspect there is quite a bit of performance to be gained in using a custom memory allocator, and there may be ways to modify the algorithm to prune invalid segments faster.

> 该实现并非完全以性能为中心，但已经进行了某些性能分析，以确定在哪些地方可以预先分配内存并尽可能避免使用三角函数。我怀疑通过使用自定义内存分配器可以获得相当大的性能提升，并且可能有办法修改算法，以便更快速地剪枝无效段。

Additionally the structures and control flow are factored for readability/maintainability and may be limiting how much is compiled to simd instructions.

> 此外，结构和控制流的设计主要考虑了可读性和可维护性，这可能会限制编译为 SIMD 指令的效率。

## Benchmarks(Benchmarks)

The following is a series of benchmarks that were run on windows 10 64 bit with an i7 6700k @ 4.00Ghz, code was built using MSVC 2019 compiler version 19.24.28314 targeting 64 bit (similar results were found using MingGW 7.3 64 bit). Each benchmark profile has different characteristics, and each profile is repeatedly offset both inward and outward with increasing delta.

> 以下是一些基准测试结果，这些测试在 Windows 10 64 位系统上运行，使用的硬件为 i7 6700k @ 4.00GHz，代码是使用 MSVC 2019 编译器版本 19.24.28314 构建的，目标为 64 位（使用 MingGW 7.3 64 位时也得到了类似的结果）。每个基准测试配置有不同的特征，每个配置都分别执行了向内和向外的多次偏移，且偏移量逐渐增加。

All offsets were performed with rounded joins (maintaining exact offset distance from original input), and 1e8 was used to scale the double inputs into 64 bit integers for input into the Clipper library (but scaling and copying is not included in the benchmarks to avoid penalizing Clipper). Offsets are always computed from the original input (not from previous offset results) to avoid the explosion of vertexes from rounded joins approximated by line segments created by Clipper.

> 所有偏移操作都使用了**圆角连接**（保持与原始输入的精确偏移距离），并且使用了 **1e8** 将双精度浮点数输入缩放为 64 位整数，以便输入到 Clipper 库中（但为了避免对 Clipper 造成额外开销，缩放和复制操作没有包含在基准测试中）。偏移始终是从原始输入开始计算（而不是从之前的偏移结果计算），以避免通过 Clipper 创建的线段近似的圆角连接引起顶点数爆炸。

- BM_square is a simple square with 4 edges.
- BM_circle is a simple circle defined by two vertexes with bulge = 1 (half circle arcs).
- BM_roundedRectangle is a rectangle with corner radii.
- BM_Profile1 is a small closed polygon with 4 arcs and 2 line segments.
- BM_Profile2 is a slightly larger closed polygon with 7 arcs and 4 line segments.
- BM_PathologicalProfile1/N is a circle whose perimeter is composed of half circle arcs alternating clockwise and counter clockwise. N is the number of half circles that make up the perimeter. This is a pathological input for 2d bounding box spatial indexing: as the offset delta increases all of the raw offset segment's bounding boxes start to overlap, and all of the segments start to intersect.
  > - **BM_square**：一个简单的正方形，包含 4 条边。
  > - **BM_circle**：一个简单的圆形，由两个顶点定义，弯曲值（bulge）为 1（半圆弧）。
  > - **BM_roundedRectangle**：一个矩形，具有圆角半径。
  > - **BM_Profile1**：一个小的闭合多边形，包含 4 条弧和 2 条线段。
  > - **BM_Profile2**：一个稍大的闭合多边形，包含 7 条弧和 4 条线段。
  > - **BM_PathologicalProfile1/N**：一个由半圆弧组成的圆形，半圆弧交替旋转顺时针和逆时针排列，N 是构成圆形周长的半圆数。这是一个对 2D 边界框空间索引非常有挑战性的输入：随着偏移量（delta）的增加，所有原始偏移段的边界框开始重叠，且所有段开始相交。

1e-2 (0.01) and 1e-3 (0.001) at the end of "arcs approx." refers to the error when approximating an arc as a series of line segments, it is the maximum allowed distance between the approximating line segments and the original arc. Note the profile vertex distances are in the range of 0-50 so the 1e-2 (0.01) error is intolerable for many applications, but significantly cuts down on the number of generated segments.

> `1e-2 (0.01)` 和 `1e-3 (0.001)` 在“弧线近似”末尾表示的是将弧线近似为一系列线段时的误差，它是近似线段与原始弧线之间允许的最大距离。请注意，轮廓顶点的距离范围是 0 到 50，因此 `1e-2 (0.01)` 的误差对于许多应用来说是无法接受的，但它显著减少了生成的线段数量。

| benchmark                   | vertex count | arcs approx. 1e-2 vertex count | arcs approx. 1e-3 vertex count |
| --------------------------- | ------------ | ------------------------------ | ------------------------------ |
| BM_square                   | 4            | 4                              | 4                              |
| BM_circle                   | 2            | 142                            | 446                            |
| BM_roundedRectangle         | 8            | 56                             | 164                            |
| BM_Profile1                 | 6            | 80                             | 241                            |
| BM_Profile2                 | 11           | 162                            | 494                            |
| BM_PathologicalProfile1/10  | 10           | 400                            | 1240                           |
| BM_PathologicalProfile1/25  | 25           | 625                            | 1975                           |
| BM_PathologicalProfile1/50  | 50           | 900                            | 2800                           |
| BM_PathologicalProfile1/100 | 100          | 1300                           | 4000                           |

All times are wall time. Exact details of the benchmark profiles and offsets run can be found under the dev project [here](https://github.com/jbuckmccready/CavalierContoursDev). For more on the clipper library see [[11]](#references).

> 所有的时间都是实际的墙面时间（wall time）。基准测试配置和执行的偏移操作的详细信息可以在开发项目中找到，链接为：[CavalierContoursDev](https://github.com/jbuckmccready/CavalierContoursDev)。有关 Clipper 库的更多信息，请参阅 [[11]](#references)。

### CavalierContours (Arcs Approximated) vs. Clipper

| benchmark                   | cavc arcs approx. 1e-2 (ms) | clipper arcs approx. 1e-2 (ms) | cavc vs. clipper |
| --------------------------- | --------------------------- | ------------------------------ | ---------------- |
| BM_square                   | **0.21**                    | 0.72                           | 3.5x             |
| BM_circle                   | **9.11**                    | 9.47                           | 1.0x             |
| BM_roundedRectangle         | **3.92**                    | 8.46                           | 2.2x             |
| BM_Profile1                 | 8.55                        | **6.33**                       | 0.7x             |
| BM_Profile2                 | 17.57                       | **16.59**                      | 0.9x             |
| BM_PathologicalProfile1/10  | **34.97**                   | 136.46                         | 3.9x             |
| BM_PathologicalProfile1/25  | **70.93**                   | 241.95                         | 3.4x             |
| BM_PathologicalProfile1/50  | **138.17**                  | 514.95                         | 3.7x             |
| BM_PathologicalProfile1/100 | **336.37**                  | 1347.09                        | 4.0x             |

| benchmark                   | cavc arcs approx. 1e-3 (ms) | clipper arcs approx. 1e-3 (ms) | cavc vs. clipper |
| --------------------------- | --------------------------- | ------------------------------ | ---------------- |
| BM_square                   | **0.21**                    | 1.71                           | 8.1x             |
| BM_circle                   | **38.04**                   | 61.37                          | 1.6x             |
| BM_roundedRectangle         | **13.14**                   | 61.61                          | 4.7x             |
| BM_Profile1                 | 27.07                       | **24.44**                      | 0.9x             |
| BM_Profile2                 | **52.64**                   | 73.64                          | 1.4x             |
| BM_PathologicalProfile1/10  | **109.96**                  | 1383.46                        | 12.6x            |
| BM_PathologicalProfile1/25  | **227.51**                  | 2824.67                        | 12.4x            |
| BM_PathologicalProfile1/50  | **385.46**                  | 6170.26                        | 16.0x            |
| BM_PathologicalProfile1/100 | **821.15**                  | 15797.60                       | 19.2x            |

The above benchmarks were taken by converting all arcs to line segments before running the profile through the offsetting algorithm. Even with no arcs in the input CavalierContours is competitive with a noticeable speedup in many cases. This is in part due to CavalierContours not having to construct rounded joins from line segments (an exact arc can be constructed instead).

> 上述基准测试是在将所有弧线转换为线段后，才通过偏移算法运行配置文件的。即使输入中没有弧线，CavalierContours 在许多情况下仍表现出显著的速度提升，且具有竞争力。这部分得益于 CavalierContours 无需从线段构造圆角连接（可以直接构造精确的弧线）。

### CavalierContours (Arcs Included) vs. Clipper

| benchmark                   | cavc w/ arcs (ms) | clipper arcs approx. 1e-2 (ms) | cavc vs. clipper |
| --------------------------- | ----------------- | ------------------------------ | ---------------- |
| BM_square                   | **0.21**          | 0.72                           | 3.4x             |
| BM_circle                   | **0.12**          | 9.47                           | 76.4x            |
| BM_roundedRectangle         | **0.49**          | 8.46                           | 17.3x            |
| BM_Profile1                 | **0.76**          | 6.33                           | 8.4x             |
| BM_Profile2                 | **1.55**          | 16.59                          | 10.7x            |
| BM_PathologicalProfile1/10  | **1.61**          | 136.46                         | 84.9x            |
| BM_PathologicalProfile1/25  | **7.17**          | 241.95                         | 33.8x            |
| BM_PathologicalProfile1/50  | **23.98**         | 514.95                         | 21.5x            |
| BM_PathologicalProfile1/100 | **85.10**         | 1347.09                        | 15.8x            |

| benchmark                   | cavc w/ arcs (ms) | clipper arcs approx. 1e-3 (ms) | cavc vs. clipper |
| --------------------------- | ----------------- | ------------------------------ | ---------------- |
| BM_square                   | **0.21**          | 1.71                           | 8.1x             |
| BM_circle                   | **0.12**          | 61.37                          | 494.7x           |
| BM_roundedRectangle         | **0.49**          | 61.61                          | 126.1x           |
| BM_Profile1                 | **0.76**          | 24.44                          | 32.3x            |
| BM_Profile2                 | **1.55**          | 73.64                          | 47.6x            |
| BM_PathologicalProfile1/10  | **1.61**          | 1383.46                        | 861.0x           |
| BM_PathologicalProfile1/25  | **7.17**          | 2824.67                        | 394.2x           |
| BM_PathologicalProfile1/50  | **23.98**         | 6170.26                        | 257.3x           |
| BM_PathologicalProfile1/100 | **85.10**         | 15797.60                       | 185.6x           |

The above benchmarks compare CavalierContours taking in the original input (arcs included) vs. Clipper (arcs must be approximated). The performance benefits of processing the arcs directly is quickly realized.

> 上述基准测试比较了 CavalierContours 直接处理原始输入（包括弧线）与 Clipper（弧线必须近似处理）之间的差异。直接处理弧线的性能优势显而易见。

# Implementation Notes and Variations(实现说明和变更)

## Float Comparing and Thresholding

When comparing two float values for equality, `a` and `b`, this implementation uses a simple fuzzy comparing method of `abs(a - b) < epsilon`, where `epsilon` is a very small number, e.g. 1e-8 or 1e-5 depending on the context. These numbers were picked through anecdotal use case trial/error where input values are typically between 0.1 and 1000.0. If finer comparisons are required, or if the input values get quite large or small then numerical stability issues may arise and the means of fuzzy comparing will need to be adjusted. See [mathutils.hpp](include/cavc/mathutils.hpp) for c++ implementation.

> 在比较两个浮动值 `a` 和 `b` 的相等性时，这个实现使用了一个简单的模糊比较方法 `abs(a - b) < epsilon`，其中 `epsilon` 是一个非常小的数，例如 1e-8 或 1e-5，具体取决于上下文。这些数值是通过实际使用场景的试验/错误选择的，通常输入值在 0.1 到 1000.0 之间。如果需要更精细的比较，或者如果输入值非常大或非常小，则可能会出现数值稳定性问题，需要调整模糊比较的方式。C++ 实现的具体代码请参见 [mathutils.hpp](include/cavc/mathutils.hpp)。

## Joining Raw Offset Segments

When joining raw offset segments together the current implementation always uses an arc to connect adjacent segments if they do not intersect, the arc maintains a constant distance from the original input polyline. Alternatively these joins could be done by extending the original line/arc segments until they intersect or by some other means entirely. Note that any other type of join will result in the offset polyline not being at a constant offset distance from the original input.

> 在将原始偏移段连接在一起时，当前实现始终使用弧线连接相邻的段，前提是它们没有相交，弧线保持与原始输入多边形的恒定距离。另一种方法是通过延长原始线段/弧线直到它们相交，或者采用其他完全不同的方式进行连接。请注意，任何其他类型的连接方式都会导致偏移多边形与原始输入之间的偏移距离不再保持恒定。

## Stitching Open Polylines

When stitching the open polylines together the current implementation only looks to stitch end points to start points. This assumes that the slices will always stitch together this way, this is a valid assumption if all slices are from the same raw offset polyline, but in the case that you want to stitch slices from different offsets together the implementation must look at both start and end points since directionality may change.

> 在将开放多边形连接在一起时，当前的实现只考虑将末端点与起始点连接。这个假设认为所有的切片都会以这种方式拼接，这在所有切片来自同一原始偏移多边形时是有效的。但如果需要将来自不同偏移的切片拼接在一起，必须同时考虑起始点和末端点，因为方向可能会发生变化。

When stitching open polylines together the current implementation attempts to stitch the slices into the longest polylines possible (there are multiple possibilities when slices become coincident or tangent at end points with one another). Alternatively one could implement it in a way to stich slices into the shortest polylines possible. This may be useful if in the case of coincident stretches the result should be marked or discarded.

> 在将开放多边形连接在一起时，当前的实现尝试将切片拼接成最长的多边形（当切片在末端点处重合或相切时，可能有多种拼接方式）。另一种方法是可以将切片拼接成最短的多边形。如果遇到重合的区间，可以标记或丢弃这些结果，这种方式可能会有用。

# Development(开发)

Pull requests, feature requests/ideas, issues, and bug reports are welcome. Please attempt to follow the code style and apply clang-format (using the .clang-format file) before making a pull request.

> 欢迎提交拉取请求（Pull requests）、功能请求/建议、问题报告以及错误报告。在提交拉取请求之前，请尝试遵循代码风格，并使用 .clang-format 文件进行格式化。

# API Stability(接口稳定性)

There is not an official release yet - all functions and structures are subject to change. This repository for now serves as an implementation reference that is easy to understand and possibly transcribe to other programming languages. The code can be used as is, but there is no guarantee that future development will maintain the same functions and structures. Ideas/pull requests for what a stable API interface should look like are welcome.

> 目前还没有正式发布——所有的函数和结构体都可能会发生变化。这个仓库目前作为一个实现参考，旨在提供一个易于理解的代码结构，可能会被转录到其他编程语言中。代码可以直接使用，但不能保证未来开发会保持相同的函数和结构。如果有关于稳定 API 接口应该是什么样子的想法或拉取请求，欢迎提出。

Tentatively the C API is stable, see header file [here](c_api_include/cavaliercontours.h), but has not yet been solidified in a 1.0 release.

> 暂定 C API 已经稳定，可以参考头文件 [这里](c_api_include/cavaliercontours.h)，但还没有在 1.0 版本中正式固化。

# Project Motivation and Goal(工程的动力和目标)

I set out to generate tool compensated milling tool paths for profile cuts, in the process I found many papers on offsetting curves for CAD/CAM uses [[1][9][10][16]](#references), but there is often no reference implementation. Most algorithms described are dense and difficult to reproduce. Issues such as numeric stability, how to handle coincident segments, process collapsing arcs, etc. are often not mentioned, and the algorithmic description detail is inconsistent. It may be very clear how to perform some of the steps, but other steps are quickly glossed over, and it becomes unclear how to go about implementing. All of these issues would not be much of a problem if an open source reference implementation was supplied, but for all the papers I have read not a single implementation was given.

> 我着手生成用于轮廓切割的刀具补偿铣削路径，在这个过程中，我发现了许多关于 CAD/CAM 中曲线偏移的论文 [[1][9][10][16]](#references)，但这些论文中往往没有给出参考实现。大多数描述的算法都很复杂，难以复现。诸如数值稳定性、如何处理重合的线段、如何处理弧形等问题常常没有提及，算法描述的细节也不一致。有些步骤的执行方法很清楚，但其他步骤却被匆忙略过，导致实现时变得不明确。如果能提供一个开源的参考实现，这些问题可能就不那么严重了，但我读过的所有论文中，都没有给出任何实现。

In addition to papers being difficult to utilize as a pragmatic tool, most papers focus on offsetting straight segment polylines or polygons (sometimes referred to as point sequence curves) [[9][10][16]](#references). And there are a few notable open source libraries that work only on straight segment polylines as well [[11][12][13]](#references). Unfortunately if only straight segments are supported then all curves, even simple constant radius arcs, must be approximated using straight segments. This leads to an inefficient memory footprint, and additional algorithmic steps if arcs must be reconstructed from points after offsetting. Constant radius arcs are very common in CAD/CAM applications (tool compensation, tool offsetting for cleanout, part sizing, etc.), and arcs may be used to approximate other types of curves more memory efficiently than straight line segments, e.g. for Bezier curves [[14]](#references).

> 除了论文作为实用工具难以利用之外，大多数论文集中在偏移直线段多边形或多边形（有时称为点序列曲线）[[9][10][16]](#references)。还有一些著名的开源库也仅支持直线段多边形[[11][12][13]](#references)。不幸的是，如果仅支持直线段，那么所有的曲线，甚至简单的恒定半径弧形，也必须通过直线段进行近似。这会导致内存占用效率低，并且如果在偏移后需要从点重建弧形，还需要额外的算法步骤。恒定半径弧形在 CAD/CAM 应用中非常常见（刀具补偿、清理刀具偏移、零件尺寸等），并且弧形在内存效率方面比直线段更适合用于近似其他类型的曲线，例如贝塞尔曲线[[14]](#references)。

There are a few papers on offsetting curves with arc segments [[1][15]](#references), some involve a Voronoi diagram approach [[15]](#references), and others are more similar to the approach this library takes by processing self-intersects and applying a clipping algorithm [[1]](#references). These papers do not provide an opensource reference implementation, and many of them are difficult to understand, and even more difficult to reproduce.

> 有一些关于使用弧段偏移曲线的论文[[1][15]](#references)，其中一些采用了 Voronoi 图的方法[[15]](#references)，而其他则更类似于本库所采用的方法，通过处理自交并应用裁剪算法[[1]](#references)。这些论文没有提供开源的参考实现，而且许多论文难以理解，甚至更难以复现。

The goal of this project is to provide a simple, direct, and pragmatic algorithm and reference implementation for polyline offsetting (supporting both lines and arcs, open and closed) for use in CAD/CAM applications. It includes the minimum computational geometry building blocks required (vectors, spatial indexing, etc.) to accomplish this goal. The code is written with the intent that it can be easily read, modified, and transcribed to other programming languages. The [interactive UI app](https://github.com/jbuckmccready/CavalierContoursDev) can be used to help understand the algorithm visually and make parts of the code easier to digest.

> 该项目的目标是为 CAD/CAM 应用提供一个简单、直接且务实的多边形偏移算法和参考实现（支持直线和弧形，开口和闭合）。它包含实现这一目标所需的最基本的计算几何构建模块（如向量、空间索引等）。代码编写时考虑到易于阅读、修改，并能够转写为其他编程语言。[互动 UI 应用](https://github.com/jbuckmccready/CavalierContoursDev)可以帮助直观地理解算法，并使代码中的部分内容更易于理解。

# Algorithm Complexity and 2D Spatial Indexing(算法复杂度和 2D 索引)

The algorithm requires finding self-intersects, testing the distance between points and the original polyline, testing for intersects between new segments and the original polyline, and stitching open polylines together end to end. The naïve approach to such steps typically results in O(n<sup>2</sup>) algorithmic complexity where each segment must be tested against every other segment and each point must be tested against every segment.

> 该算法需要查找自交、测试点与原始多边形线段之间的距离、测试新线段与原始多边形线段之间的交点，并将开口多边形线段首尾连接起来。对这些步骤采用朴素的方法通常会导致 O(n<sup>2</sup>) 的算法复杂度，因为每个线段都必须与其他所有线段进行测试，每个点也必须与每个线段进行测试。

The approach used by this library is to use a packed Hilbert R-Tree [[3]](#references). See [staticspatialindex.hpp](include/cavc/staticspatialindex.hpp) for c++ implementation. See [[4]](#references) and [[5]](#references) for more implementation references. This results in an algorithm complexity of O(n log n) for typical inputs, with worse case O(n<sup>2</sup>) for pathological inputs.

> 该库使用的方法是使用一个打包的 Hilbert R-Tree [[3]](#references)。C++ 实现请参见 [staticspatialindex.hpp](include/cavc/staticspatialindex.hpp)。更多实现参考见 [[4]](#references) 和 [[5]](#references)。这种方法将典型输入的算法复杂度降至 O(n log n)，而在极端情况下，复杂度为 O(n<sup>2</sup>)。

## Packed Hilbert R-Tree(打包的希尔伯特 R-Tree)

Here is an image of a closed polyline approximating a circle using 100 line segments (blue lines and red vertexes) with spatial index bounding boxes made visible (magenta, orange, and light green boxes). The root of the R-Tree is the light green box, its children are the orange boxes, and its grand children are the magenta boxes.

> 这是一个闭合多边形近似圆形的图像，使用了 100 个线段（蓝色线段和红色顶点），并且显示了空间索引的边界框（品红色、橙色和浅绿色框）。R-Tree 的根是浅绿色框，它的子节点是橙色框，孙节点是品红色框。

![Simple Spatial Index](https://raw.githubusercontent.com/jbuckmccready/CavalierContoursDoc/master/images/spatial_index/simple_example.png)

The packed Hilbert R-Tree is very fast to build and query, but requires rebuilding the tree anytime the data changes – however for computational geometry algorithms such as those performed in this library we start with no indexed data, and our data does not change for the duration of the algorithm, making the tradeoff ideal for the use case.

> 打包的 Hilbert R-Tree 在构建和查询时非常快速，但每当数据发生变化时需要重新构建树。然而，对于像本库中执行的计算几何算法来说，我们从没有索引的数据开始，并且数据在算法执行过程中不会改变，这使得这种方法对于这个用例非常适合。

There are alternative approaches, e.g. by combining an Interval Tree [[6]](#references) and binary heap [[7]](#references) a sweep line algorithm [[8]](#references) approach can be taken to avoid O(n<sup>2</sup>) complexity.

> 也有替代方法，例如通过结合区间树 [[6]](#references) 和二叉堆 [[7]](#references)，可以采用扫描线算法 [[8]](#references) 来避免 O(n<sup>2</sup>) 的复杂度。

Note that there are pathological input cases that will still result in O(n<sup>2</sup>) behavior even with a spatial index or sweep line algorithm. E.g. a polyline for which all segments have axis-aligned bounding boxes that overlap. But for common inputs seen in CAD/CAM applications this is not the case, and a run time complexity of O(n log n) will result.

> 需要注意的是，即使使用空间索引或扫描线算法，仍然存在某些极端输入情况会导致 O(n<sup>2</sup>) 的行为。例如，一个多边形的所有线段都有重叠的轴对齐包围盒。但对于 CAD/CAM 应用中常见的输入来说，通常不会发生这种情况，因此运行时复杂度会保持在 O(n log n)。

# References(引用)

[1] Liu, X.-Z., Yong, J.-H., Zheng, G.-Q., & Sun, J.-G. (2007). An offset algorithm for polyline curves. Computers in Industry, 58(3), 240–254. doi:10.1016/j.compind.2006.06.002

[2] Bulge conversions: http://www.lee-mac.com/bulgeconversion.html

[3] https://en.wikipedia.org/wiki/Hilbert_R-tree#Packed_Hilbert_R-trees

[4] JavaScript spatial index implementation: https://github.com/mourner/flatbush/

[5] Fast 2D to 1D Hilbert curve mapping used for spatial index: https://github.com/rawrunprotected/hilbert_curves

[6] https://en.wikipedia.org/wiki/Interval_tree

[7] https://en.wikipedia.org/wiki/Binary_heap

[8] https://en.wikipedia.org/wiki/Sweep_line_algorithm

[9] Lin, Z., Fu, J., He, Y., & Gan, W. (2013). A robust 2D point-sequence curve offset algorithm with multiple islands for contour-parallel tool path. Computer-Aided Design, 45(3), 657–670. doi:10.1016/j.cad.2012.09.002

[10] Kim, D.-S. (1998). Polygon offsetting using a Voronoi diagram and two stacks. Computer-Aided Design, 30(14), 1069–1076. doi:10.1016/s0010-4485(98)00063-3

[11] Clipper library: http://www.angusj.com/delphi/clipper.php (github fork here: https://github.com/jbuckmccready/clipper-lib)

[12] CGAL library for offsetting polylines: https://doc.cgal.org/latest/Straight_skeleton_2/index.html

[13] Boost geometry: https://www.boost.org/doc/libs/1_53_0/libs/geometry/doc/html/index.html

[14] Bezier biarc approximating: https://github.com/domoszlai/bezier2biarc

[15] Held, M., & Huber, S. (2009). Topology-oriented incremental computation of Voronoi diagrams of circular arcs and straight-line segments. Computer-Aided Design, 41(5), 327–338. doi:10.1016/j.cad.2008.08.004

[16] Kim, H.-C., Lee, S.-G., & Yang, M.-Y. (2005). A new offset algorithm for closed 2D lines with Islands. The International Journal of Advanced Manufacturing Technology, 29(11-12), 1169–1177. doi:10.1007/s00170-005-0013-1
