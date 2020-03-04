[TOC]

<h1><center>OpenGL 的版本演变</center></h1>



# OpenGL 1.X

## OpenGL 1.0

- **OpenGL 的每个版本由扩展组成**
- 每个版本会定义一些显卡必须支持的新扩展，当硬件的驱动全部支持相应的扩展的时候，相应的OpenGL版本就被支持了



## OpenGL 1.1

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



## OpenGL 1.2

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



##  OpenGL 1.2.1

- 没有什么重大的改变，但是专门介绍了 ARB 扩展的概念
  ARB 扩展是经过 OpenGL ARB 认证的扩展，**这样的扩展将被广泛地实现**



## OpenGL 1.3

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



## OpenGL 1.4

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

  

## OpenGL 1.5

- [GL_ARB_vertex_buffer_object](http://www.opengl.org/registry/specs/ARB/vertex_buffer_object.txt)
  出现了buffer object，用来取代过去的 vertex array 和立即模式，顶点数据可以从客户端内存上传到服务端内存
- [GL_ARB_occlusion_query](http://www.opengl.org/registry/specs/ARB/occlusion_query.txt)
  遮挡查询
- [GL_EXT_shadow_funcs](http://www.opengl.org/registry/specs/EXT/shadow_funcs.txt)



# OpenGL 2.X

## OpenGL 2.0

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



## OpenGL 2.1

- [GL_ARB_pixel_buffer_object](http://www.opengl.org/registry/specs/ARB/pixel_buffer_object.txt)
  增加了 pixel buffer object，用来更快地像素传输的工作
  支持将像素从 texture object 和 framebuffer object 到 pixel buffer object 的包装和解包装
  pixel buffer object 可以像普通的缓冲对象一样被 map 修改数据
- [GL_EXT_texture_sRGB](http://www.opengl.org/registry/specs/EXT/texture_sRGB.txt)
  支持 sRGB 格式的纹理对象



# OpenGL 3.X

## OpenGL 3.0

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



## OpenGL 3.1

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



## OpenGL 3.2

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



## OpenGL 3.3

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



# OpenGL 4.X

## OpenGL 4.0

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



## OpenGL 4.1

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



## OpenGL 4.2

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



## OpenGL 4.3

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



## OpenGL 4.4

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





# 引用

1. [从未停止！OpenGL的版本历史和发展](https://www.cnblogs.com/vertexshader/articles/2917540.html)
