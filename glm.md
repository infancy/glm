# GLM 阅读笔记

一系列声明文件, 具体实现在 detail, ext 里

hpp + inc, 相当于**声明+实现**

每个 hpp 的开头会写这个文件提供的功能

- common.hpp: 很多常用的标量, 向量函数: abs, sign, floor, ceil, mod, min, max, clamp...实现在 "detail/func_common.inl", "detail/func_common_simd.inl", "simd/common.h" 里
- exponential：pow, exp, log, exp2, log2, sqrt, inversesqrt
- ext：所有 ext/ 的文件
- fwd：一大堆 typedef
- geometric：length、distance、dot、cross、normalize、faceforward、reflect、refract
- glm：包含了 /glm 下的所有头文件
- integer：bit 相关
- mat*x*：包含 ext/matrix_float*x*.hpp, ext/matrix_float*x*_precision.hpp
- matrix：transpose, determinant, inverse
- packing：看不懂
- trigonometric：radians、degrees、sin、cos……
- vec*：like mat*x*
- vector_relational：>=, <=, any, all

## api 

*/glm/doc/api/index.html
*/glm/doc/api/modules.html
*/glm/doc/api/files.html



# GLM/DETAIL

##  overview

\_vectorize.hpp：传入 function<...> 和 vec, 简化了实现
detail/core
type_：类型的实现, func_：函数的实现
vec、mat 的具体实现
glm 里各种便利类型的实现
mat2, mat3, mat4 比别的矩阵多一些方法, 但 fay 里为了统一没有实现这些方法

qualifier：精度, low、medium、high；aligned；storage；
	
	typedef qualifier precision;
	
	通过 vec<...>::typename detail::storage<1, T, detail::is_aligned<Q>::value>::type data; 来控制对齐, 这与精度无关
	template<typename T>
	struct storage<3, T, true>
	{
		typedef struct alignas(4 * sizeof(T)) type {
			T data[4];
		} type;
	};

	// 上个特化版本, 如果用了 SIMD, 编译器就会用这个特化版本
	#if GLM_ARCH & GLM_ARCH_SSE2_BIT
	template<>
	struct storage<4, float, true>
	{
		typedef glm_f32vec4 type;
	};
	
关于 glm 的精度：https://stackoverflow.com/questions/25592975/glm-precision-qualifier 
, OpenGL 和 glm 的精度都没什么用, 只是为了兼容 OpenGL ES, 和精度相关的代码都可以跳过.

setup.hpp：一些通过宏定义设置配置的技巧, 值得借鉴

## type_*

type_float: union float_t<float>, union float_t<double>, union float_t<vec3> 提取指数和尾数, cool~

## func_*

func_common.inl
func_common_simd.inl

func_vector_relational: any, all, not



# GLM/EXT 稳定拓展

matrix/scalar/vector_type_n 都是从 detail 里实例化模板生成的一堆类型, 没什么看头

## matrix
matrix_clip_space：根据左右手、z轴深度分成好几种：ortho(left, right, bottom, top/*, near, far*/), frustum, perspective, perspectiveFov, infinitePerspective, tweakedInfinitePerspective（实现一部分就行了）

matrix_common：mix, 类似 lerp？？？

matrix_double*x*_precision：带精度的一堆类型定义
matrix_double*x*：一堆类型定义

matrix_projection:
projectZO(zero-one), projectNO(Negative-one): 从对象坐标系映射到窗口坐标系, 实现有一点意思. 
unProjectZO 进行反转
pickMatrix 的操作就让人看不懂了

matrix_relational: equal, not_equal
matrix_transform:


## quaternion

quaternion: todo

## scalar

scalar_common: min/max, fmin/fmax (for 3 to 4 scalar parameters)
scalar_constants：epsilon, pi
scalar_(u)int_sized: is_int
scalar_relational: equal, not_equal (with user defined epsilon values)
scalar_ulp: next_float, prev_float, float_distance, 每个平台的具体实现会不同, 可以考虑拓展 float_distance, 返回具体的距离大小(会准确吗???)
        4.7683716E-7    2.3841858E-7   ...  1.4E-45
	(-4)------------(-3)---------(-2)------(-1)---(0)---(+1)------(+2)---------(+3)------------(+4)

	ULP: unit in the last place or unit of least precision 

	#include <glm/ext/scalar_ulp.hpp> // measure of accuracy in numeric calculations.
	bool test_ulp(float x)
	{
		float const a = glm::next_float(x); // return a float a ULP away from the float argument.
		return float_distance(a, x) == 1; // check both float are a single ULP away.
	}

## vector done

vector_type_n(_precision): 一堆类型别名

vector_common: min/max, fmin/fmax (for 3 to 4 vector parameters)
vector_integer: is_power_of_two, next_power_of_two ... is_multiple, next_multiple ... find_NSB 
vector_relational: equal, not_equal (with user defined epsilon values)
vector_ulp: prev_float, next_float ... float_distance



# GLM/GTC 推荐拓展

ulp: prev_float_n, next_float_n ... float_distance

# GLM/GTX 实验性拓展



# GLM/SIMD

platform.h 平台检测, 类型定义

common.h, exponential.h ... 对应 /glm 下的文件, 存放其 simd 版本 

    GLM_FUNC_QUALIFIER glm_f32vec4 glm_vec4_add(glm_f32vec4 a, glm_f32vec4 b)
    {
        return _mm_add_ps(a, b);
    }

