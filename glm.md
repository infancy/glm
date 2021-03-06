# GLM 阅读笔记

一系列声明文件, 具体实现在 detail, ext 里

hpp + inc, 相当于**声明+实现**

每个 hpp 的开头会写这个文件提供的功能

## api 

*/glm/doc/api/index.html
*/glm/doc/api/modules.html
*/glm/doc/api/files.html

## 精度限定符

qualifier：精度, low、medium、high；aligned；storage；
	
	typedef qualifier precision;
	
	通过 vec<...>::typename detail::storage<1, T, detail::is_aligned<Q>::value>::type data; 
	来控制对齐, 这与精度无关
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
	
关于 glm 的精度：https://stackoverflow.com/questions/25592975/glm-precision-qualifier , 
OpenGL 和 glm 的精度都没什么用, 只是为了兼容 OpenGL ES, 和精度相关的代码都可以跳过.



# GLM/

- common: 很多常用的标量, 向量函数: abs, sign, floor, ceil, mod, min, max, clamp...实现在 "detail/func_common.inl", "detail/func_common_simd.inl", "simd/common.h" 里
- exponential：pow, exp, log, exp2, log2, sqrt, inversesqrt
- ext：包含 ext/ 下的所有文件
- fwd：一大堆类型定义
- geometric：length、distance、dot、cross、normalize、faceforward、reflect、refract
- glm：包含了 glm/ 下的大部分头文件
- integer：bit 相关
- mat*x*：包含 ext/matrix_float*x*.hpp, ext/matrix_float*x*_precision.hpp
- matrix：transpose, determinant, inverse
- packing:
  - packUnorm2x16: 把两个 float 压缩到 short, 再放入 uint 里
- trigonometric：三角函数, radians、degrees、sin、cos……
- vec*：like mat*x*
- vector_relational：>=, <=, any, all, not



# GLM/DETAIL

##  overview

type_：vec, quat, mat 的具体实现
func_：函数的实现
mat2, mat3, mat4 比别的矩阵多一些方法, 但 fay 里为了统一没有实现这些方法

## misc

\_features: C++特性的提案, 开关宏
\_fixes: 取消 cmath 定义的一些宏
\_noise: 噪声发生器
\_swizzle_func: 实现 swizzle 的一些宏, 使得允许 vec3().xyz() 的操作
\_swizzle: 实现 swizzle 的一些类型, 使得允许 vec3().xyz 的操作
\_vectorize：传入 function 和 vec, 将 vec 的每个元素应用到 function, 简化了实现

compute_common: compute_abs
compute_vector_relational

glm: 一大堆类型的前向声明
setup.hpp：一些通过宏定义设置配置的技巧, 值得借鉴
qualifier: 
	不需要关注 highp/mediump/lowp
	开启 GLM_HAS_ALIGNOF 后, 会对类似 vec3 的类型做对齐
	开启 GLM_ARCH & SSE2 后, 替换实际存储类型为 __m128 等
	genTypeEnum, genTypeTrait, init_gentype

## /func_*(_simd).inl

glm/ 下同名头文件(不包含 func_ 前缀)的具体实现

func_common
func_exponential
func_geometric
func_integer
func_matrix
func_packing
func_trigonometric
func_vector_relational

## type_float

type_float: union float_t<float>, union float_t<double>, union float_t<vec3> 提取指数和尾数, cool~
type_half: float 和 short 的互转(C++ 标准中的提案: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1467r1.html)

## type_*

```C++
struct math_type
{
	// typedef xxx yyy;

	// #if GLM_SILENT_WARNINGS ...

	// data
	// union { ... }

	// #endif GLM_SILENT_WARNINGS ...

	// functions
}
```

### type_quat


### type_vector_n

```C++
struct math_type
{
	// typedef xxx yyy;

	// #if GLM_SILENT_WARNINGS ...

	// data
	// union { ... }

	// #endif GLM_SILENT_WARNINGS ...

	
	// length
	// operator[]

	// 包含隐式, 显式, 截断的构造函数

	// operator=, T&U

	// operator+=, -=, *=, /=, vec1&scalar

	// operator++, --

	// operator%=, &=, |=, ^=, <<=, >>=
}

// operator+, -, ~, unary

// operator+, -, *, /, %, &, |, ^, <<, >>, binary

// operator==, !=, &&, ||
```

只有 vec4 有 simd 版本
看实现和测试代码可能比看声明更容易搞懂在干什么

#### type_vec4.inl

开头 namespace detail 里一大堆的 compute_vec4_operator 简化操作, 并允许特化版本<\del如果作者愿意用宏的话, 还可以再简化一层\del>

但目前这种啰嗦的写法, 让读(懂)代码方便了很多

构造函数占了四百多行

先实现类内的函数, 剩下的函数基本通过 return vec<4, T, Q>(0) OP= v; 实现

如果开启了 GLM_CONFIG_SIMD, 就引入 type_vec4_simd.inl

#### type_vec4_simd.inl

- 对 type_vec4.inl 中 namespace detail 里一大堆的 compute_vec4_operator 的特化实现
- simd 版本的构造函数


### type_matrix_m_n

```C++
struct math_type
{
	// typedef xxx yyy;

	// data
	
	// length
	// operator[]

	// 构造函数

	// conversions

	// operator=, T&U

	// operator+=, -=, mat&scalar
	// operator*=, /=, scalar

	// operator++, --
}

// operator+, -, unary

// 括号内为 matNxN 版本才有的函数

// operator+, -, scalar(, mat)

// operator*, scalar, row/col_type, mat_type

// operator/, scalar(, row/col_type, mat_type)

// operator==, !=
```

type_matrix_m_n 的实现基本没看头



# GLM/EXT 稳定拓展

glm 里各种便利类型的实现
matrix/scalar/vector_type_n 都是从 detail 里实例化模板生成的一堆类型, 没什么看头

## matrix done

matrix_type_n(_precision): 一堆(带精度的)类型别名

matrix_clip_space：根据左右手、z轴深度分成好几种：
	ortho(left, right, bottom, top/*, near, far*/), 
	frustum,
	perspective, 
	perspectiveFov, 
	infinitePerspective, 
	tweakedInfinitePerspective（实现一部分就行了）
matrix_common：mix, 类似 lerp？？？
matrix_projection:
	projectZO(zero-one), projectNO(Negative-one): 从对象坐标系映射到窗口坐标系, 实现有一点意思. 
	unProjectZO 进行反转
	pickMatrix 的操作就让人看不懂了
matrix_relational: equal, not_equal
matrix_transform:
	identity
	translate
	rotate
	scale
	lookat

## quaternion todo

## scalar done

scalar_common: min/max, fmin/fmax (for 3 to 4 scalar parameters)
scalar_constants：epsilon, pi
scalar_(u)int_sized: (u)int_n 定义, is_int
scalar_integer: is_power_of_two, next_power_of_two ... is_multiple, next_multiple ... find_NSB 
scalar_relational: equal, not_equal (with user defined epsilon values)
scalar_ulp: next_float, prev_float, 
	float_distance, 每个平台的具体实现会不同, 可以考虑拓展 float_distance, 返回具体的距离大小(会准确吗???)

	ULP: unit in the last place or unit of least precision 

        4.7683716E-7    2.3841858E-7   ...  1.4E-45
	(-4)------------(-3)---------(-2)------(-1)---(0)---(+1)------(+2)---------(+3)------------(+4)

	#include <glm/ext/scalar_ulp.hpp> // measure of accuracy in numeric calculations.
	bool test_ulp(float x)
	{
		float const a = glm::next_float(x); // return a float a ULP away from the float argument.
		return float_distance(a, x) == 1; // check both float are a single ULP away.
	}

## vector done

vector_type_n(_precision): 一堆(带精度的)类型别名

vector_common: min/max, fmin/fmax (for 3 to 4 vector parameters)
vector_integer: is_power_of_two, next_power_of_two ... is_multiple, next_multiple ... find_NSB 
vector_relational: equal, not_equal (with user defined epsilon values)
vector_ulp: prev_float, next_float ... float_distance



# GLM/GTC 推荐拓展

mask:
color_space: linear/sRGB
integer: log2, iround, uround
matrix_inverse: affineInverse, inverseTranspose
noise: perlin, simplex
quaternion: roll, pitch, yaw
ulp: prev_float_n, next_float_n ... float_distance



# GLM/GTX 实验性拓展



# GLM/SIMD

platform.h 平台检测, 类型定义

common.h, exponential.h ... 对应 /glm 下的文件, 存放其 simd 版本 

    GLM_FUNC_QUALIFIER glm_f32vec4 glm_vec4_add(glm_f32vec4 a, glm_f32vec4 b)
    {
        return _mm_add_ps(a, b);
    }

