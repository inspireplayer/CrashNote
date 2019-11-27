[TOC]


# 一、OpenGL 简介

>  OpenGL 作为图形硬件标准，是最通用的图形管线版本
>  使用 OpenGL 自带的数据类型可以确保各平台中每一种类型的大小都是统一的
>
>  **OpenGL 只是一个标准/规范，具体的实现是由驱动开发商针对特定显卡来实现**



##1. OpenGL 状态机

由于 OpenGL 内部是一个类似于全局变量的状态机

- 切换状态 `glEnable()`、 `glDisable()` 
- 查询状态 `glIsEnabled()` 
- 存储状态 `glPushAttrib()`：保存 OpenGL 当前属性状态信息到属性栈中
- 恢复之前存储的状态 `glPopAttrib()`：从属性栈中获取栈首的一系列属性值



##2. OpenGL Context

OpenGL 命令执行的结果影响 OpenGL 状态（由 OpenGL context 保存，包括OpenGL 数据缓存）或 影响帧缓存

1. 使用 OpenGL 之前必须先创建 OpenGL Context，并 make current 将创建的 上下文作为当前线程的上下文

2. **OpenGL 标准并不定义如何创建 OpenGL Context，这个任务由其他标准定义**
   如GLX（linux）、WGL（windows）、EGL（一般在移动设备上用）

3. 上下文的描述类型有 **core profile (不包含任何弃用功能)** 或 **compatibility profile (包含任何弃用功能)** 两种
   如果创建的是 core profile OpenGL context，调用如 glBegin() 等兼容 API 将产生GL_INVALID_OPERATION 错误（用 glGetError() 查询）

4. 共享上下文

   一个窗口的 Context 可以有多个，在某个线程创建后，所有 OpenGL 的操作都会转到这个线程来操作
   两个线程同时 make current 到同一个绘制上下文，会导致程序崩溃

   一般每个窗口都有一个上下文，可以保证上下文间的不互相影响
   通过**创建上下文时传入要共享的上下文**，多个窗口的上下文之间图形资源可以共享
   可以共享的：纹理、shader、Vertex Buffer 等，外部传入对象
   不可共享的：Frame Buffer Object、Vertex Array Object 等 OpenGL 内置容器**对象**

   

## 3. OpenGL 的环境配置流程

**1. 动态获取 OpenGL 函数地址**

OpenGL 只是一个标准/规范，具体的实现是由驱动开发商针对特定显卡来实现，而且 OpenGL 驱动版本众多，它大多数函数的位置都**无法在编译时确定下来，需要在运行时查询**
因此，在编写与 OpenGL 相关的程序时需要开发者自己来获取 OpenGL 函数地址

相关库可以提供 OpenGL 函数获取地址后的头文件：[GLAD](https://github.com/Dav1dde/glad)



**2. 创建上下文**

OpenGL 创建上下文的操作在不同的操作(窗口)系统上是不同的，所以需要开发者自己处理：**窗口的创建、定义上下文、处理用户输入**

相关库可以摆脱平台的限制，提供一个较为统一的接口和窗口、上下文用来渲染：[GLUT](http://freeglut.sourceforge.net/)、SDL、SFML、[GLFW](http://www.glfw.org/download.html)




## 4. OpenGL 的执行模型（Client - Server 模型）

> 主函数在 CPU 上执行，图形渲染在 GPU 上执行
> 虽然 GPU 可以编程，但这样的程序也需要在 CPU 上执行来操作 GPU

基本执行模型：CPU 上 push command 命令，GPU 上执行命令的渲染操作

- **应用程序 和 GPU 的执行通常是异步的**
  OpenGL API 调用返回 != OpenGL 在 GPU 上执行完了相应命令，但保证按调用顺序执行
  同步方式：**glFlush()** 强制发出所有 OpenGL 命令并在此函数返回后的有限时间内执行完这些 OpenGL 命令
  异步方式：**glFinish()** 等待直到**此函数之前**的 OpenGL 命令执行完毕才返回

- **应用程序 和 OpenGL 可以在也可以不在同一台计算机上执行**
  一个网络渲染的例子是通过 Windows 远程桌面在远程计算机上启动 OpenGL 程序，应用程序在远程计算机执行，而 OpenGL 命令在本地计算机执行（**将几何数据**而不是将渲染结果图像通过网络传输）

  > 当 Client 和 Server 位于**同一台计算机**上时，也称 GPU 为 Device，CPU 为 Host
  > Device、Host 这两个术语通常在用 GPU 进行通用计算时使用

- **内存管理**
  CPU 上由程序准备的缓存数据（buffer、texture 等）存储在显存（video memory）中，这些数据从程序到缓存中拷贝，也可以再次拷贝到程序的缓存中
  
- **数据绑定发生在 OpenGL 命令调用时**
  应用程序传送给 GPU 的数据在 OpenGL API 调用时解释，在调用返回时完成
  例，指针指向的数据给 OpenGL 传送数据，如 glBufferData()  在此 API 调用返回后修改指针指向的数据将不再对 OpenGL 状态产生影响




## 5. Shader 接口一致性

- Vertex Shader 的 输入 和 应用程序的顶点属性数据接口 一致
- Vertex Shader 的 输出 和 Fragment Shader 对应的 输入 一致
- Fragment Shader 的 输出 和 帧缓存的颜色缓存接口 一致



固定管线功能阶段需要的一些特定输入输出由着色器的内置输出输入变量定义，如下图

![](images/vertexToFragmentAPI.png)



# 三、OpenGL 的版本演变

## 1. OpenGL 1.X

**OpenGL 1.0**

- **OpenGL 的每个版本由扩展组成**
- 每个版本会定义一些显卡必须支持的新扩展，当硬件的驱动全部支持相应的扩展的时候，相应的OpenGL版本就被支持了



**OpenGL 1.1**

- [GL_EXT_vertex_array](http://www.opengl.org/registry/specs/EXT/vertex_array.txt)
  顶点数组取代了`glVertex*` 这类立即模式绘图函数，多个数据可以被一个函数调用绘制了，降低了调用函数带来的 CPU 循环开销
- [GL_EXT_polygon_offset](http://www.opengl.org/registry/specs/EXT/polygon_offset.txt)
  解决了[z-fighting ](http://en.wikipedia.org/wiki/Z_fighting)和 stitching 的问题
- [GL_EXT_blend_logic_op](http://www.opengl.org/registry/specs/EXT/blend_logic_op.txt)
  在 pre-fragment operation 开始支持逻辑操作
- [GL_EXT_texture](http://www.opengl.org/registry/specs/EXT/texture.txt)
  支持纹理代理(texture proxy)和 纹理环境映射(texture environment)
- [GL_EXT_copy_texture](http://www.opengl.org/registry/specs/EXT/copy_texture.txt)、[GL_EXT_subtexture](http://www.opengl.org/registry/specs/EXT/subtexture.txt)
从 frameuffer 复制像素至 texture 或 subtexture
- [GL_EXT_texture_object](http://www.opengl.org/registry/specs/EXT/texture_object.txt)
  texture object 的出现改变了过去只能使用 display list 来静态地使用纹理的方法，现在纹理和参数能被改变了



**OpenGL 1.2**

- [GL_EXT_texture3D](http://www.opengl.org/registry/specs/EXT/texture3D.txt)
  可以用于体渲染(volume rendering) 和体纹理(solid texture)
- [GL_EXT_bgra](http://www.opengl.org/registry/specs/EXT/bgra.txt)
  BGRA 和 BGA 的出现主要是**为了兼容某些平台和硬件**
- [GL_EXT_packed_pixels](http://www.opengl.org/registry/specs/EXT/packed_pixels.txt)
  使得像素可以在不同的对象之间进行像素传输(pixel transfer)](http://www.opengl.org/wiki/Pixel_Transfer)这是像素缓冲对象(pixel buffer object)的前身
- [GL_EXT_rescale_normal](http://www.opengl.org/registry/specs/EXT/rescale_normal.txt)
- [GL_EXT_separate_specular_color](http://www.opengl.org/registry/specs/EXT/separate_specular_color.txt)
- [GL_SGIS_texture_edge_clamp](http://www.opengl.org/registry/specs/SGIS/texture_edge_clamp.txt)
  将 texture coordinate 规范在 [0,1] 这个区间
- [GL_SGIS_texture_lod](http://www.opengl.org/registry/specs/SGIS/texture_lod.txt)
  带来了重要的 [MipMap ](http://en.wikipedia.org/wiki/Mipmap) 技术，可以通过对纹理参数的控制来完成对 MipMap 的控制
- [GL_EXT_draw_range_elements](http://www.opengl.org/registry/specs/EXT/draw_range_elements.txt)



**OpenGL 1.2.1**

- 没有什么重大的改变，但是专门介绍了 ARB 扩展的概念
  ARB 扩展是经过 OpenGL ARB 认证的扩展，**这样的扩展将被广泛地实现**



**OpenGL 1.3**

- [GL_ARB_texture_compression](http://www.opengl.org/registry/specs/ARB/texture_compression.txt)
  纹理压缩可以有效地减少存储和带宽的压力
- [GL_ARB_texture_cube_map](http://www.opengl.org/registry/specs/ARB/texture_cube_map.txt)
  主要用于在天空盒、动态反射(dynamic reflection)等技术上
- [GL_ARB_multisample ](http://www.opengl.org/registry/specs/ARB/texture_cube_map.txt)删除了 [GL_ARB_multitexture](http://www.opengl.org/registry/specs/ARB/multitexture.txt)
  支持纹理和 framebuffer 的 [MSAA](http://en.wikipedia.org/wiki/Multisample_anti-aliasing) 抗锯齿技术，代替了过去在光栅化状态中趋近无用的抗锯齿设置
- [GL_ARB_texture_env_add](http://www.opengl.org/registry/specs/ARB/texture_env_add.txt)
  [GL_ARB_texture_env_combine](http://www.opengl.org/registry/specs/ARB/texture_env_combine.txt)
  [GL_ARB_texture_env_dot3](http://www.opengl.org/registry/specs/ARB/texture_env_dot3.txt)
- [GL_ARB_texture_border_clamp](http://www.opengl.org/registry/specs/ARB/texture_border_clamp.txt)
- [GL_ARB_transpose_matrix](http://www.opengl.org/registry/specs/ARB/transpose_matrix.txt)



**OpenGL 1.4**

- [GL_SGIS_generate_mipmap](http://www.opengl.org/registry/specs/SGIS/generate_mipmap.txt)
  支持纹理自动生成 Mipmap

- [GL_NV_blend_square](http://www.opengl.org/registry/specs/NV/blend_square.txt)

- [GL_ARB_depth_texture](http://www.opengl.org/registry/specs/ARB/depth_texture.txt)
  [GL_ARB_shadow](http://www.opengl.org/registry/specs/ARB/shadow.txt)
  
- [GL_EXT_fog_coord](http://www.opengl.org/registry/specs/EXT/fog_coord.txt)

- [GL_EXT_multi_draw_arrays](http://www.opengl.org/registry/specs/EXT/multi_draw_arrays.txt)

- [GL_ARB_point_parameters](http://www.opengl.org/registry/specs/ARB/point_parameters.txt)
  顶点光栅化的参数设置

- [GL_EXT_secondary_color](http://www.opengl.org/registry/specs/EXT/secondary_color.txt)

- [GL_EXT_blend_func_separate](http://www.opengl.org/registry/specs/EXT/blend_func_separate.txt)

- [GL_EXT_stencil_wrap](http://www.opengl.org/registry/specs/EXT/stencil_wrap.txt)

- [GL_ARB_texture_env_crossbar](http://www.opengl.org/registry/specs/ARB/texture_env_crossbar.txt)

- [GL_ARB_texture_mirrored_repeat](http://www.opengl.org/registry/specs/ARB/texture_mirrored_repeat.txt)

- [GL_ARB_window_pos](http://www.opengl.org/registry/specs/ARB/window_pos.txt)

  

**OpenGL 1.5**

- [GL_ARB_vertex_buffer_object](http://www.opengl.org/registry/specs/ARB/vertex_buffer_object.txt)
  出现了buffer object，用来取代过去的 vertex array 和立即模式，顶点数据可以从客户端内存上传到服务端内存
- [GL_ARB_occlusion_query](http://www.opengl.org/registry/specs/ARB/occlusion_query.txt)
  遮挡查询
- [GL_EXT_shadow_funcs](http://www.opengl.org/registry/specs/EXT/shadow_funcs.txt)



## 2. OpenGL 2.X

**OpenGL 2.0**

- [GL_ARB_shading_language_100](http://www.opengl.org/registry/specs/ARB/shading_language_100.txt)
  [GL_ARB_shader_objects](http://www.opengl.org/registry/specs/ARB/shader_objects.txt)
  [GL_ARB_vertex_shader](http://www.opengl.org/registry/specs/ARB/vertex_shader.txt)
  [GL_ARB_fragment_shader](http://www.opengl.org/registry/specs/ARB/fragment_shader.txt)
  用户可以自定义 GPU 里 VS 和 FS 阶段的计算方法，固定管线和可编程管线并存
  ARB 选择了 3Dlabs 的 Dave 设计的着色语言成为 OpenGL 原生的着色语言
- [GL_ARB_draw_buffers](http://www.opengl.org/registry/specs/ARB/draw_buffers.txt)
  片元着色器输出可以输出到帧缓冲的多个渲染目标(render target)上
- [GL_ARB_texture_non_power_of_two](http://www.opengl.org/registry/specs/ARB/texture_non_power_of_two.txt)
  **纹理也不再有必须是 2^n 次方的尺寸限制**
- [GL_ARB_point_sprite](http://www.opengl.org/registry/specs/ARB/point_sprite.txt)
- [GL_EXT_blend_equation_separate](http://www.opengl.org/registry/specs/EXT/blend_equation_separate.txt)
- [GL_EXT_stencil_two_side](http://www.opengl.org/registry/specs/EXT/stencil_two_side.txt)



**OpenGL 2.1**

- [GL_ARB_pixel_buffer_object](http://www.opengl.org/registry/specs/ARB/pixel_buffer_object.txt)
  增加了 pixel buffer object，用来更快地像素传输的工作
  支持将像素从 texture object 和 framebuffer object 到 pixel buffer object 的包装和解包装
  pixel buffer object 可以像普通的缓冲对象一样被 map 修改数据
- [GL_EXT_texture_sRGB](http://www.opengl.org/registry/specs/EXT/texture_sRGB.txt)
  支持 sRGB 格式的纹理对象



## 3. OpenGL 3.X

**OpenGL 3.0**

从 3.0 开始，OpenGL 分 core profile 和 compatibility profile，并且 compatibility profile 内容将逐渐淘汰
改变了过去向下兼容的特性

> 将淘汰的功能有 Bitmaps、Shading language 1.10 and 1.20、Begin/End primitive specification、Edge flags、Fixed function vertex processing、Client-side vertex arrays、Rectangles、Current raster position、Two-sided color selection、Non-sprite points、Wide linee and line strip、Quadrilateral and polygon primitives、Sepatate polygon draw mode、Polygon stripple、Pixel transger modes and operations、Pixel drawing、Application-generated object names、Color index mode、Legacy OpenGL 1.0 pixel formats、Legacy pixel formats、Depth texture mode、Texture wrap mode CLAMP、Texture border、Automatic mipmap generation、Fixed function fragment processing、Alpha test、Accumulation buffers、Context、framebuffer size queries、Evaluators、Selection and feedback mode、Display lists、Hints、Attribute stacks、Unified extension string

- [GL_EXT_gpu_shader4](http://www.opengl.org/registry/specs/EXT/gpu_shader4.txt)
- [GL_EXT_framebuffer_object](http://www.opengl.org/registry/specs/EXT/framebuffer_object.txt)
  帧缓冲对象之间可以互相拷贝像素到持有的不同的 render target，是性能上的提升
- [GL_EXT_framebuffer_blit](http://www.opengl.org/registry/specs/EXT/framebuffer_blit.txt)
- [GL_ARB_texture_float](http://www.opengl.org/registry/specs/ARB/texture_float.txt)
  [GL_ARB_color_buffer_float](http://www.opengl.org/registry/specs/ARB/color_buffer_float.txt)
  [GL_NV_depth_buffer_float](http://www.opengl.org/registry/specs/NV/depth_buffer_float.txt)
  [GL_EXT_packed_float](http://www.opengl.org/registry/specs/EXT/packed_float.txt)
  [GL_EXT_texture_shared_exponent](http://www.opengl.org/registry/specs/EXT/texture_shared_exponent.txt)
  增加了浮点型和整型的 texture 和 depth 的 image format
- [GL_EXT_texture_compression_rgtc](http://www.opengl.org/registry/specs/EXT/texture_compression_rgtc.txt)
  支持 RGTC 纹理压缩格式
- [GL_EXT_transform_feedback](http://www.opengl.org/registry/specs/EXT/transform_feedback.txt)
  数据可以经过 vertex shader 和 geometry shader 之后，又输出回 buffer 而不经过 rasterization 以及之后的阶段，在物理和粒子的计算上面非常的有用
- [GL_APPLE_vertex_array_object](http://www.opengl.org/registry/specs/APPLE/vertex_array_object.txt)
  增加的 vertex array object 方便管理 buffer object 以及 vertex attrib pointer 和其开启/关闭状态，不必每次在渲染前都要设置一遍了
- [GL_NV_conditional_render](http://www.opengl.org/registry/specs/NV/conditional_render.txt)
- [GL_EXT_texture_integer](http://www.opengl.org/registry/specs/EXT/texture_integer.txt)



**OpenGL 3.1**

- [GL_ARB_draw_instanced](http://www.opengl.org/registry/specs/ARB/draw_instanced.txt)
  减轻了同类物体绘制所占有的带宽压力
- [GL_ARB_copy_buffer](http://www.opengl.org/registry/specs/ARB/copy_buffer.txt)
  Copy buffer的出现，是让数据在 client 端进行拷贝，也是一种性能的优化
- [GL_ARB_texture_buffer_object](http://www.opengl.org/registry/specs/ARB/texture_buffer_object.txt)
  让 buffer object 像 texture 那样被访问
- [GL_ARB_texture_rectangle](http://www.opengl.org/registry/specs/ARB/texture_rectangle.txt)
- [GL_ARB_uniform_buffer_object](http://www.opengl.org/registry/specs/ARB/uniform_buffer_object.txt)
  OpenGL 每个函数的调用所消耗的 CPU 循环都非常的大，频繁地调用 `glUniform*` 会带来很大的性能问题，开放了 uniform buffer object，通过 map/unmap 更新数据，函数调用开销明显地减少
- [GL_NV_primitive_restart](http://www.opengl.org/registry/specs/NV/primitive_restart.txt)



**OpenGL 3.2**

- [GL_ARB_geometry_shader4](http://www.opengl.org/registry/specs/ARB/geometry_shader4.txt)
  支持几何着色器（geometry shader）用来生成新的图元类型（点、线和三角形）
- [GL_ARB_sync](http://www.opengl.org/registry/specs/ARB/sync.txt)
 Fence sync objects  
- [GL_ARB_vertex_array_bgra](http://www.opengl.org/registry/specs/ARB/vertex_array_bgra.txt)
- [GL_ARB_draw_elements_base_vertex](http://www.opengl.org/registry/specs/ARB/draw_elements_base_vertex.txt)
- [GL_ARB_seamless_cube_map](http://www.opengl.org/registry/specs/ARB/seamless_cube_map.txt)
- [GL_ARB_texture_multisample](http://www.opengl.org/registry/specs/ARB/texture_multisample.txt)
  Texture 正式支持 multisample，可以作为 render target 来进行 framebuffer object上 的抗锯齿
  而不是经过的 WGL_ARB_multisample 和 GLX_ARB_multisample 进行窗口的抗锯齿
- [GL_ARB_fragment_coord_conventions](http://www.opengl.org/registry/specs/ARB/fragment_coord_conventions.txt)
- [GL_ARB_provoking_vertex](http://www.opengl.org/registry/specs/ARB/provoking_vertex.txt)
- [GL_ARB_depth_clamp](http://www.opengl.org/registry/specs/ARB/depth_clamp.txt)
  Fragment depth clamping



**OpenGL 3.3**

- [GL_ARB_blend_func_extended](http://www.opengl.org/registry/specs/ARB/blend_func_extended.txt)
- [GL_ARB_explicit_attrib_location](http://www.opengl.org/registry/specs/ARB/explicit_attrib_location.txt)
  改变了程序需要查询输入变量（attribute）的 location 的方式，可以像 HLSL 指定 semantic 一样在 shader 里指定 layout，减少了相应 API 的调用
- [GL_ARB_occlusion_query2](http://www.opengl.org/registry/specs/ARB/occlusion_query2.txt)
- [GL_ARB_sampler_objects](http://www.opengl.org/registry/specs/ARB/sampler_objects.txt)
  将 texture object 和 sampler state 解耦，增加了 sampler object
  sampler object 也可以绑定到 ACTIVE_TEXTURE 上了
- [GL_ARB_texture_swizzle](http://www.opengl.org/registry/specs/ARB/texture_swizzle.txt)
- [GL_ARB_timer_query](http://www.opengl.org/registry/specs/ARB/timer_query.txt)
- [GL_ARB_instanced_arrays](http://www.opengl.org/registry/specs/ARB/instanced_arrays.txt)
- [GL_ARB_texture_rgb10_a2ui](http://www.opengl.org/registry/specs/ARB/texture_rgb10_a2ui.txt)
  [GL_ARB_vertex_type_2_10_10_10_rev](http://www.opengl.org/registry/specs/ARB/vertex_type_2_10_10_10_rev.txt)
  new texture format for unsigned_10_10_10_2 and new vertex attributes for 2_10_10_10



## 4. OpenGL 4.X

**OpenGL 4.0**

- [GL_ARB_tessellation_shader](http://www.opengl.org/registry/specs/ARB/tessellation_shader.txt)
- [GL_ARB_texture_query_lod](http://www.opengl.org/registry/specs/ARB/texture_query_lod.txt)
  [GL_ARB_gpu_shader5](http://www.opengl.org/registry/specs/ARB/gpu_shader5.txt)
  [GL_ARB_gpu_shader_fp64](http://www.opengl.org/registry/specs/ARB/gpu_shader_fp64.txt)
  [GL_ARB_texture_gather](http://www.opengl.org/registry/specs/ARB/texture_gather.txt)
  [GL_ARB_shader_subroutine](http://www.opengl.org/registry/specs/ARB/shader_subroutine.txt)
  支持 Shader Language 4.0 
  subroutine 提供了在运行时刻不需要切换着色器或者是重新编译或者使用 if 判断选择不同功能的方法，降低了切换着色器程序所带来的巨大开销（切换着色器的 CPU 循环消耗真的非常的惊人）
- [GL_ARB_sample_shading](http://www.opengl.org/registry/specs/ARB/sample_shading.txt)
- [GL_ARB_draw_buffers_blend](http://www.opengl.org/registry/specs/ARB/draw_buffers_blend.txt)
  让 fragment shader 输出的每条 buffer 都可以完成各自的 pre-fragment operaion，而不是像过去那样每条都完成相同的 pre-fragment operation
- [GL_ARB_draw_indirect](http://www.opengl.org/registry/specs/ARB/draw_indirect.txt)
- [GL_ARB_transform_feedback2](http://www.opengl.org/registry/specs/ARB/transform_feedback2.txt)
  [GL_ARB_transform_feedback3](http://www.opengl.org/registry/specs/ARB/transform_feedback3.txt)
  提供了 transform feedback object，以及 transform feedback 相关的控制（比如pause之类）



**OpenGL 4.1**

- [GL_ARB_ES2_compatibility](http://www.opengl.org/registry/specs/ARB/ES2_compatibility.txt)
  把 OpenGL ES 的一些功能划入 core profile 的范围
- [GL_ARB_get_program_binary](http://www.opengl.org/registry/specs/ARB/get_program_binary.txt)
  可以将 shader 事先编译好序列化进入二进制文件
- [GL_ARB_separate_shader_objects](http://www.opengl.org/registry/specs/ARB/separate_shader_objects.txt)
  Ability to bind programs individually to programmable stages
- [GL_ARB_viewport_array](http://www.opengl.org/registry/specs/ARB/viewport_array.txt)
  Multiple viewports for the same rendering surface, or one per surface
- [GL_ARB_shader_precision](http://www.opengl.org/registry/specs/ARB/shader_precision.txt)
  Documents precision requirements for several FP operaions
- [GL_ARB_vertex_attrib_64bit](http://www.opengl.org/registry/specs/ARB/vertex_attrib_64bit.txt)
  提供了 64 位的浮点型输入变量，提升了数据精度



**OpenGL 4.2**

- [GL_ARB_base_instance](http://www.opengl.org/registry/specs/ARB/base_instance.txt)
- [GL_ARB_compressed_texture_pixel_storage](http://www.opengl.org/registry/specs/ARB/compressed_texture_pixel_storage.txt)
  Allows for sub-rectangle selections when transferring compressed texture data
- [GL_ARB_conservative_depth](http://www.opengl.org/registry/specs/ARB/conservative_depth.txt)
- [GL_ARB_internalformat_query](http://www.opengl.org/registry/specs/ARB/internalformat_query.txt)
- [GL_ARB_map_buffer_alignment](http://www.opengl.org/registry/specs/ARB/map_buffer_alignment.txt)
- [GL_ARB_shading_language_420pack](http://www.opengl.org/registry/specs/ARB/shading_language_420pack.txt)
- [GL__ARB_texture_storage](http://www.opengl.org/registry/specs/ARB/texture_storage.txt)
  Allows texture objects to have immutable storage, and allocating all mipmap levels and images in one call. The storage becomes immutable, but the contents of the storage are not
- [GL_ARB_transform_feedback_instanced](http://www.opengl.org/registry/specs/ARB/transform_feedback_instanced.txt)
- [GL_ARB_shader_atomic_counters](http://www.opengl.org/registry/specs/ARB/shader_atomic_counters.txt)
- [GL_ARB_shader_image_load_store](http://www.opengl.org/registry/specs/ARB/shader_image_load_store.txt)
- [GL_ARB_texture_compression_bptc](http://www.opengl.org/registry/specs/ARB/texture_compression_bptc.txt)
  Allows the use of certain advanced compression formats（至此 OpenGL 开始支持所有的 Block Compression 格式）



**OpenGL 4.3**

- [GL_ARB_arrays_of_arrays](http://www.opengl.org/registry/specs/ARB/arrays_of_arrays.txt)

- [GL_ARB_clear_buffer_object](http://www.opengl.org/registry/specs/ARB/clear_buffer_object.txt)

- [GL_ARB_compute_shader](http://www.opengl.org/registry/specs/ARB/compute_shader.txt)
  可以用于并行计算

- [GL_ARB_copy_image](http://www.opengl.org/registry/specs/ARB/copy_image.txt)

- [GL_KHR_debug](http://www.opengl.org/registry/specs/KHR/debug.txt)/[GL_ARB_debug_output](http://www.opengl.org/registry/specs/ARB/debug_output.txt)
  可以获取更多调试信息

- [GL_ARB_ES3_compatibility](http://www.opengl.org/registry/specs/ARB/ES3_compatibility.txt)

- [GL_ARB_explicit_uniform_location](http://www.opengl.org/registry/specs/ARB/explicit_uniform_location.txt)
  像 HLSL 指定 semantic 一样指定 layout 的方法

- [GL_ARB_framebuffer_no_attachments](http://www.opengl.org/registry/specs/ARB/framebuffer_no_attachments.txt)

- [GL_ARB_internalformat_query2](http://www.opengl.org/registry/specs/ARB/internalformat_query2.txt)

- [GL_ARB_invalidate_subdata](http://www.opengl.org/registry/specs/ARB/invalidate_subdata.txt)

- [GL_ARB_multi_draw_indirect](http://www.opengl.org/registry/specs/ARB/multi_draw_indirect.txt)

- [GL_ARB_program_interface_query](http://www.opengl.org/registry/specs/ARB/program_interface_query.txt)

- [GL_ARB_shader_storage_buffer_object](http://www.opengl.org/registry/specs/ARB/shader_storage_buffer_object.txt)

- [GL_ARB_shader_image_size](http://www.opengl.org/registry/specs/ARB/shader_image_size.txt)

- [GL_ARB_stencil_texturing](http://www.opengl.org/registry/specs/ARB/stencil_texturing.txt)

- [GL_ARB_fragment_layer_viewport](http://www.opengl.org/registry/specs/ARB/fragment_layer_viewport.txt)

- [GL_ARB_texture_query_levels](http://www.opengl.org/registry/specs/ARB/texture_query_levels.txt)

- [GL_ARB_texture_storage_multisample](http://www.opengl.org/registry/specs/ARB/texture_storage_multisample.txt)

- [GL_ARB_texture_view](http://www.opengl.org/registry/specs/ARB/texture_view.txt)
  用来共享已经创建纹理的内容

- [GL_ARB_vertex_attrib_binding](http://www.opengl.org/registry/specs/ARB/vertex_attrib_binding.txt)

- [GL_ARB_robust_buffer_access_behavior](http://www.opengl.org/registry/specs/ARB/robust_buffer_access_behavior.txt)

  [GL_ARB_robustness_isolation](http://www.opengl.org/registry/specs/ARB/robustness_isolation.txt)

  [WGL_ARB_robustness_isolation](http://www.opengl.org/registry/specs/ARB/wgl_robustness_isolation.txt)

  [GLX_ARB_robustness_isolation](http://www.opengl.org/registry/specs/ARB/glx_robustness_isolation.txt)



**OpenGL 4.4**

- [GL_ARB_buffer_storage](http://www.opengl.org/registry/specs/ARB/buffer_storage.txt)

- [GL_ARB_clear_texture](http://www.opengl.org/registry/specs/ARB/clear_texture.txt)

- [GL_ARB_enhanced_layouts](http://www.opengl.org/registry/specs/ARB/enhanced_layouts.txt)
  uniform block 内部指定 layout，相当于 Direct3D 的 registry

- [GL_ARB_multi_bind](http://www.opengl.org/registry/specs/ARB/multi_bind.txt)
  允许通过一次调用来绑定多个资源，将绑定资源的开销分摊到一个调用上，并且和 Direct3D11 的接口相互兼容

- [GL_ARB_query_buffer_object](http://www.opengl.org/registry/specs/ARB/query_buffer_object.txt)

- [GL_ARB_texture_mirror_clamp_to_edge](http://www.opengl.org/registry/specs/ARB/texture_mirror_clamp_to_edge.txt)

- [GL_ARB_texture_stencil8](http://www.opengl.org/registry/specs/ARB/texture_stencil8.txt)

- [GL_ARB_vertex_type_10f_11f_11f_rev](http://www.opengl.org/registry/specs/ARB/vertex_type_10f_11f_11f_rev.txt)

- [GL_ARB_compute_variable_group_size](http://www.opengl.org/registry/specs/ARB/compute_variable_group_size.txt)

- [GL_ARB_indirect_parameters](http://www.opengl.org/registry/specs/ARB/indirect_parameters.txt)

- [GL_ARB_seamless_cube_texture_per_texture](http://www.opengl.org/registry/specs/ARB/seamless_cubemap_per_texture.txt)

- [GL_ARB_shader_draw_parameters](http://www.opengl.org/registry/specs/ARB/shader_draw_parameters.txt)

- [GL_ARB_shader_group_vote](http://www.opengl.org/registry/specs/ARB/shader_group_vote.txt)

  

## 5. Shader Language

通过**首行使用** `#version` 来说明当前 OpenGL Shader Language 版本



### 5.1 GLSL 版本号对应

OpenGL 和 OpenGL 的 Shading Language 版本对应

| **Version OpenGL** | 2.0 | 2.1 | 3.0 | 3.1 | 3.2 | 3.3 | 4.0 | 4.1 | 4.2 | 4.3 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **Version GLSL** | 110 | 120 | 130 | 140 | 150 | 330 | 400 | 410 | 420 | 430 |

OpenGL ES 和 OpenGL ES 的 Shading Language 版本对应

| **Version OpenGL ES** | 2.0 | 3.0 |
| --------------------- | --- | --- |
| **Version GLSL ES**   | 100 | 300 |



### 5.2 GLSL 版本功能区别

1. GLSL 130+ 版本
   用 `in` 和  `out` 替换了 `attribute` 和 `varying`
2. GLSL 330+ 版本
   用 `texture` 替换了 `texture2D` 
   增加了 layout 内存布局功能
3. [其他版本重要功能变化](https://github.com/mattdesl/lwjgl-basics/wiki/glsl-versions)




# 四、渲染管线

> 所谓 OpenGL 管线（OpenGL pipeline），就是指 OpenGL 的渲染过程，即从输入数据到最终产生渲染结果数据所经过的通路及所经受的处理

## 1. 顶点变换过程

模型变换 $\to$ 视野变换 $\to$ 顶点处理（可能含有光照） $\to$ 
透视区域裁剪（得到裁剪后的坐标空间）$\to$ 齐次变换（得到标准化的坐标空间） $\to$ 视角变换（得到屏幕坐标）  $\to$ 光栅化 $\to$ 片源处理，纹理，光照处理 $\to$ 光栅化（可选） $\to$ 缓冲帧

![](images/coordinate.png)



## 2. 渲染管线流程

可编程：可以在需要时由 shader 实现
不可编程：具体方法由 OpenGL API 的驱动实现

简单流程

![](images/pipeline1.gif)



OpenGL 4.4 渲染管线

![](images/pipeline.png)



# 五、渲染同步

## 1. 同步异步的渲染方式 glFlush/glFinish

> 提交给 OpenGL 的指令并不是马上送到驱动程序里执行的，而是放到一个缓冲区里面，等这个缓冲区满了再一次过发到驱动程序里执行，glFlush 可以只接提交缓冲区的命令到驱动执行，而不需要在意缓冲区是否满了

同步方式：[void glFlush()](https://www.khronos.org/opengl/wiki/GLAPI/glFlush) 强制发出所有 OpenGL 命令并在此函数返回后的**有限时间**内执行这些 OpenGL 命令（这些命令可能没有执行完）
异步方式：[void glFinish()](https://www.khronos.org/opengl/wiki/GLAPI/glFinish) 等待直到**此函数之前**的 OpenGL 命令执行完毕才返回



## 2. 垂直同步 vsync

定义：确保显卡的运算频率（GPU 一秒绘制的帧数）和 显示器刷新频率（硬件决定）一致，防止在快速运动场景下，由于**显卡运算速率大于显示器运算速率**导致快速运动的动作割裂情况（画面撕裂）

流程：`显卡绘制一帧时间 > 显示器刷新一帧时间 ? 显示器刷新(显卡等待) : 显示器显示上一帧，等待显卡绘制完成(屏幕卡顿);`

缺点：开启垂直同步，画面会有延迟，但并没有卡顿

规避缺点的方法：用三重缓冲代替垂直同步（三重缓冲：在双缓冲的基础上加了一个缓冲，显卡不需要等待显示器把画面显示完整了再开始画下一幅，而是选择直接在另一个后缓冲画图）




# 引用

1. [OpenGL 加载库](https://www.khronos.org/opengl/wiki/OpenGL_Loading_Library)

2. [更多 OpenGL 的 lib 库文件](http://www.opengl-tutorial.org/miscellaneous/useful-tools-links/)

3. [从未停止！OpenGL的版本历史和发展](https://www.cnblogs.com/vertexshader/articles/2917540.html)

4. [水平同步 垂直同步](https://blog.csdn.net/hankern/article/details/90344384)

5. [Android 的 16ms 和垂直同步以及三重缓存](https://www.jianshu.com/p/3750db831aca)

6. [GLSL Versions](https://github.com/mattdesl/lwjgl-basics/wiki/glsl-versions)
