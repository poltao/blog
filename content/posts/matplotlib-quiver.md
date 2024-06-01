+++
title = "matplotlib.pyplot.quiver使用指南"
description = "matplotlib.pyplot.quiver使用指南，包括基本的函数参数介绍与示例程序。这是我在CSDN写过所有的博客里面至今为止浏览量和收藏最多的文章，所以在迁移博客的时候也一并迁移过来了。"
date = 2020-12-18
[taxonomies]
tags= ["Python3", "matplotlib"]
+++

# 1. 引子

```python
X = np.arange(-10, 10, 1)
Y = np.arange(-10, 10, 1)
# meshgrid 生成网格，此处生成两个 shape = (20,20) 的 ndarray, 详见参考资料2,3
U, V = np.meshgrid(X, Y)
C = np.sin(U)
fig, ax = plt.subplots()
# 绘制箭头
q = ax.quiver(X, Y, U, V, C)
# 该函数绘制一个箭头标签在 (X, Y) 处， 长度为U, 详见参考资料4
ax.quiverkey(q, X=0.3, Y=1.1, U=10,
             label='Quiver key, length = 10', labelpos='E')
plt.show()
```

效果如下图所示：

<div align="center">
    <img src="https://i.loli.net/2020/12/17/6kSNPoBr4wDbITA.png" alt="pyplot" width="600" height="400">
</div>

# 2. matplotlib.pyplot.quiver

根据 [matplotlib.pyplot.quiver.html](https://matplotlib.org/3.3.3/api/_as_gen/matplotlib.pyplot.quiver.html)显示，该函数原型 
```py3
matplotlib.pyplot.quiver(*args, data=None, **kw)
``` 
签名如下：

<font color="orange" size=4>
quiver([X, Y], U, V, [C], **kw)
</font>

其中 X, Y 定义了箭头的位置, U, V 定义了箭头的方向, C 作为可选参数用来设置颜色。  
下面是几个常见的参数：

- **units：** 此参数指示了除箭头长度外，其他尺寸的变化趋势，以该单位的倍数测量。可取值为 `{'width', 'height', 'dots', 'inches', 'x', 'y' 'xy'}`, 默认是 `'width'`。需要配合 `scale` 参数使用。
- **scale:** `float, optional`。此参数是每个箭头长度单位的数据单位数，通常该值越小，意味着箭头越长，默认为 `None` ,此时系统会采用自动缩放算法。箭头长度单位由 `scale_units` 参数指定。
- **scale_units:** 此参数是可选参数，其中包含以下值：`{'width', 'height', 'dots', 'inches', 'x', 'y', 'xy'}`, 一般当 `scale=None` 时该选项指示长度单位，默认为 `None`。

- **angles:** 此参数指定了确定箭头角度的方法，可以取 `{'uv', 'xy'}` 或者 `array-like`, 默认是 `'uv' `。 设计原因是 因为图的宽和高可能不同，所以 `x` 方向的单位长度和 `y` 方向的单位长度可能不同，这时我们需要做出选择，一是不管长度对不对，角度一定要对，此时 `angles='uv'`，二是不管角度了，只要长度对就可以了，此时 `angles='xy'`。当该值为一个 `array` 的时候，该数组应该是以度数为单位的数组，表示了每一个箭头的方向，如下所示：

```python
fig, ax = plt.subplots()
# 以水平轴按照 angles 参数逆时针旋转得到箭头方向， units='xy' 指出了箭头长度计算方法
ax.quiver((0, 0), (0, 0), (1, 0), (1, 3), angles=[60, 300], units='xy', scale=1, color='r')
plt.axis('equal')
plt.xticks(range(-5, 6))
plt.yticks(range(-5, 6))
plt.grid()
plt.show()
```

<div align="center">
    <img src="https://i.loli.net/2020/12/17/MpESeTBI6jnZUwH.png" alt="pyplot" width="600" height="400">
</div>

- **pivot:** `{'tail', 'mid', 'middle', 'tip'}`, 默认为 `'tail'`。该参数指定了箭头的基点(旋转点)。
- width:此参数是轴宽，以箭头为单位。
- headwidth:该参数是杆头宽度乘以杆身宽度的倍数。
- headlength:此参数是长度宽度乘以轴宽度。
- headwidth:该参数是杆头宽度乘以杆身宽度的倍数。
- headaxislength:此参数是轴交点处的头部长度。
- minshaft:此参数是箭头缩放到的长度，以头部长度为单位。
- minlength:此参数是最小长度，是轴宽度的倍数。

# 3. quiver 示例

- 以箭头中间为基点，并在该点画出一个红色的点：

```python
X, Y = np.meshgrid(np.arange(0, 2 * np.pi, .2), np.arange(0, 2 * np.pi, .2))
U, V = np.cos(X), np.sin(Y)
fig, ax = plt.subplots()

ax.set_title("pivot='mid'; every third arrow; units='inches'")
Q = ax.quiver(X[::3, ::3], Y[::3, ::3], U[::3, ::3], V[::3, ::3], units='inches', pivot='mid', color='g')
qk = ax.quiverkey(Q, 0.9, 0.9, 1, r'$1 \frac{m}{s}$', labelpos='E',
                   coordinates='figure')
ax.scatter(X[::3, ::3], Y[::3, ::3], color='r', s=5)
plt.show()
```

<div align="center">
    <img src="https://i.loli.net/2020/12/18/MFhOvrsIqY4jlgx.png" alt="pyplot" width="600" height="400">
</div>

- 头部， 并为每个箭头设置颜色和大小

```python
X, Y = np.meshgrid(np.arange(0, 2 * np.pi, .2), np.arange(0, 2 * np.pi, .2))
U, V = np.cos(X), np.sin(Y)
fig, ax = plt.subplots()
# M 为颜色矩阵
M = np.hypot(U, V)
ax.set_title("pivot='tip'; scales with x view")
Q = ax.quiver(X, Y, U, V, M, units='xy', scale=1 / 0.15, pivot='tip')
qk = ax.quiverkey(Q, 0.9, 0.9, 1, r'$1 \frac{m}{s}$', labelpos='E',
                  coordinates='figure')
ax.scatter(X, Y, color='r', s=1)
plt.show()
```

<div align="center">
    <img src="https://i.loli.net/2020/12/18/GLcWSgTJ9ydfCFu.png" alt="pyplot" width="600" height="400">
</div>

# 4. 参考资料

[1. Quiver Simple Demo](https://matplotlib.org/3.3.3/gallery/images_contours_and_fields/quiver_simple_demo.html)

[2. numpy.meshgrid](https://numpy.org/doc/stable/reference/generated/numpy.meshgrid.html#numpy.meshgrid)

[3. Numpy 中的 meshgrid()函数](https://blog.csdn.net/qq_38701868/article/details/99694048?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control)

[4. matplotlib.axes.Axes.quiverkey](https://matplotlib.org/3.3.3/api/_as_gen/matplotlib.axes.Axes.quiverkey.html#matplotlib.axes.Axes.quiverkey)

[5. matplotlib quiver:清奇的脑回路](https://www.cnblogs.com/gujianmu/p/12853643.html)

[6. Advanced quiver and quiverkey functions](https://matplotlib.org/3.3.3/gallery/images_contours_and_fields/quiver_demo.html#sphx-glr-gallery-images-contours-and-fields-quiver-demo-py)
