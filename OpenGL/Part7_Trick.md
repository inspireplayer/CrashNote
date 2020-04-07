[TOC]

# 一、碰撞检测

一般出于性能的考虑，使用**重叠**在物体上的更简单的外形（通常有较简单明确的数学定义）来进行碰撞检测是常用碰撞检测的方法

缺点：这些外形通常无法完全包裹物体，当检测到碰撞时，实际的物体可能并没有真正的碰撞



## 1. Axis-Aligned Bounding Box

轴对齐碰撞箱（Axis-Aligned Bounding Box）

- 碰撞箱为 矩形/立方体，它的边与所包围物体的**世界坐标系对齐**（为了便于计算碰撞箱）

- 不会随所包装物体的旋转而旋转，只会**随所包装物体的旋转而缩放**
- 表示方法有很多，通常用两个坐标系各轴都是模型中的点 最大/最小 的点来表示
  描述 AABB 的几何结构为：碰撞箱的中心点 + 碰撞箱子各个坐标轴的半径（为了方便碰撞检测）





## 2. AABB 与 AABB 碰撞

通过：两个物体的 AABB 碰撞箱的 **水平** 和 **垂直** 边界是否重合 来判断





## 3. AABB 与 圆碰撞

通过：找到 AABB 上距离圆最近的一个点，如果圆到这一点的距离小于圆的半径，那么就产生了碰撞

![](./images/collisions_aabb_circle.png)

```c
GLboolean CheckCollision(BallObject &circle, GameObject &aabb_box)
{
    // 获取圆的中心 
    glm::vec2 circle_center(circle.Position + circle.Radius);
    // 计算AABB的信息（中心、半边长）
    glm::vec2 aabb_half_extents(aabb_box.Size.x / 2, aabb_box.Size.y / 2);
    glm::vec2 aabb_center(
        aabb_box.Position.x + aabb_half_extents.x, 
        aabb_box.Position.y + aabb_half_extents.y
    );
    // 获取两个中心的差矢量
    glm::vec2 difference = circle_center - aabb_center;
    glm::vec2 clamped = glm::clamp(difference, -aabb_half_extents, aabb_half_extents);
    // AABB_center 加上 clamped 这样就得到了碰撞箱上距离圆最近的点closest
    glm::vec2 p = aabb_center + clamped;
    // 获得圆心 C 和最近点 P 的矢量
    difference = p - circle_center;
  	// 判断 p 是否在圆内
    return glm::length(difference) < circle.Radius;
}
```





# 二、后处理特效


## 1. HDR

高动态范围（High Dynamic Range，HDR）

- 亮的东西可以变得非常亮，暗的东西可以变得非常暗，而且充满细节

- 显示器被限制为只能显示值为 0.0 到 1.0 间的颜色，但是在光照方程中却没有这个限制

- 通过使片段的颜色超过1.0，我们有了一个更大的颜色范围



HDR 原本只是被运用在摄影上，摄影师对同一个场景采取不同曝光拍多张照片，捕捉大范围的色彩值。这些图片被合成为 HDR 图片，从而综合不同的曝光等级使得大范围的细节可见






## 2. 泛光 Bloom

方法：

1. 根据一个阈值提取出图像中较亮的区域，把它们存储在一张渲染纹理上
2. 利用高斯模糊对这张渲染纹理进行模糊处理，从而模拟光线扩散的泛光效果
3. 将模糊后的图像和原图像进行混合





## 3. 反走样 Anti-Aliasing

方法一：超采样抗锯齿 Super Sample Anti-aliasing, SSAA

- 步骤：渲染一张比显示的纹理更高分辨率的帧缓冲，分辨率下采样到正常的分辨率
- 缺点：性能开销很大



方法二：多重采样抗锯齿 Multisample Anti-aliasing, MSAA

- 步骤：将单一的采样点变为多个采样点（采样点的数量可以是任意的）
  不再使用像素中心的单一采样点，而是以特定图案排列的 4 个子采样点(Subsample)
  由顶点插值得到的像素颜色，会存储在**被图形遮盖住**的**每个**子采样点中，最终的像素颜色是子采样点的平均值
  如果不想以平均计算子采样颜色的方式，OpenGL 允许我们在 FS 阶段获取到每个子采样点的颜色并计算最终采样结果
- 缺点：颜色缓冲的大小会随着子采样点的增加而增加

![](./images/trick_anti_aliasing_sample.png)



## 4. 运动模糊

运动模糊：真实世界中的摄像机的一种效果（相机曝光时，拍摄场景发生变化，就会产生模糊效果）

方法一：

- 步骤：利用累积缓存（accumulation buffer）来混合多张连续的图像，将它们取平均值来作为最后的模糊图像
- 缺点：性能消耗很大，因为在同一帧里要渲染多次场景
- 优化：只保存上一帧的渲染效果，不断把上一帧图像和当前图像叠加（alpha 混合），从而产生运动轨迹的视觉效果



方法二：

- 步骤：利用速度缓存（velocity buffer）存储各个像素当前的运动速度，利用改值来决定模糊的方向和大小
- 优化：通过上一帧的相机位置和投影得到上一帧点的坐标与当前帧求深度位置差，即运动速度。最后，通过运动速度决定了 3 X 3 的均值滤波的各个方向的采样步长来做模糊





## 5. 全局雾化

雾的计算：$Color_{out} = f * Color_{fog} +(1-f) * Color_{origin}$

雾的系数 $f$ 计算：使用噪声纹理强度图，乘以雾的系数，让雾的系数变化更自然

1. 线性，$d_{max}$ 和 $d_{min}$ 分别表示受雾影响的最小距离和最大距离
   $f = {d_{max} - |z| \over d_{max} - d_{min}}$
2. 指数，$d$ 控制雾浓度的参数
   $f = e^{-d-|z|}$
3. 指数的平方，$d$ 控制雾浓度的参数
   $f = e^{-(d-|z|)^2}$





# 三、其他

## 1. 描边

方法一：

1. 让背面面片在视角空间下把模型顶点沿着法线的方向**向外扩张**一段距离，让背部轮廓可见
   为了防止背面面片有 Z 轴方向内凹的模型，先给让背面面片尽可能平整
   让所有背面面片的法线 Z 轴统一为一个定值，然后在归一化法线

   ```glsl
   viewNormal.z = -0.5;
   viewNormal = normalize(viewNormal);
   viewPos = viewPos + viewNormal * outlineWidth;
   ```

2. 使用轮廓线的颜色渲染背面的面片

3. 渲染正面面片



方法二：

1. 检测边是否为轮廓线
   通过判断两个相邻的三角片面是否一个朝正面，一个朝背面
   $(n_0 \cdot v) * (n_1 \cdot v) < 0$ 其中，$n_0$ 和 $n_1$ 为相邻两个三角面的法向量，$v$ 是从视角指向顶点的方向
2. 单独渲染轮廓线（可以进行风格化渲染，水磨笔触的描边）





## 2. 粒子系统





## 3. 3D 拾取







# 引用

- [3D Picking](http://ogldev.atspace.co.uk/www/tutorial29/tutorial29.html)
- [learnopengl-Bloom](https://learnopengl-cn.github.io/05 Advanced Lighting/07 Bloom/)
- [learnopengl-HDR](https://learnopengl-cn.github.io/05 Advanced Lighting/06 HDR/)
- [learnopengl-AntiAliasing](https://learnopengl-cn.github.io/04 Advanced OpenGL/11 Anti Aliasing/)
- [HDR Tone Mapping](https://zhuanlan.zhihu.com/p/26254959)
- [OGL-Particle System using Transform Feedback](http://ogldev.atspace.co.uk/www/tutorial28/tutorial28.html)
- [Open Dynamics Engine](http://www.ode.org)
- [Open Dynamics Engine Doc](http://ode.org/ode-latest-userguide.html)

