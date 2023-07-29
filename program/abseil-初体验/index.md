# Abseil 初体验


这是一个谷歌开源的cpp库， 包括了一些容器，字符串工具等。

Quickstart:  https://abseil.io/docs/cpp/quickstart-cmake.html

Github: https://github.com/abseil/abseil-cpp



这个库大概主要包括下面几个内容。

- 基础库  base
- 算法库 algorithm
- 容器库  container
- 哈希库 hash
- 字符串库 strings
- 工具库  utility 
- 等等



## 笔者的使用记录

### 编译和链接

笔者刚开始是把 `abseil`当做一个子项目 放在自己的项目里面的， 并使用cmake添加它们。

这样使用的时候， 笔者发现了一个问题，那就是会增加很多个cmake的 target， 编译起来的输出令笔者感觉到不适，而且编译速度应该是有所降低。  所以笔者就改用了 `Homebrew`进行安装。 

使用`brew install abseil`命令进行安装，之后在自己的cmake项目文件里面这样添加即可。

```cmake
# abseil
find_package(absl REQUIRED)
target_link_libraries(
        sight-util PUBLIC
        absl::base
        absl::strings
        absl::hash
        absl::flat_hash_map
        absl::flat_hash_set
        absl::btree
)
```



使用brew安装之后， 可能还是有个问题，那就是可能会出现链接或者编译错误， 如果出现了这个错误， 把自己的项目也改成c++17版本就可以了。  

```cmake
# specify the C++ standard
set(CMAKE_CXX_STANDARD 17)
```

或者， 可以考虑把 abseil的版本切换成自己使用的cpp版本。 

这个方法笔者没有切身实际的去操作过， 所以仅仅提供一些思路。 

- 找到本地Homebrew的 `abseil.rb` 文件， 修改 17为11
- 强制让 homebrew 重新编译和安装
- Homebrew 的文件链接： https://github.com/Homebrew/homebrew-core/blob/master/Formula/abseil.rb



### 使用

在使用的时候， 笔者发现 abseil的cmake target 很乱， 基本上没有太多规则。 如果没有指示的话， 可能很难找到正确的target name。

不过好在， 笔者后来发现了这个文件`CMake/AbseilDll.cmake` 这个文件里面记录了所有的target， 如果需要什么，只需要在这个文件里面搜索一下即可。

这里附上 当前的所有target.

```cmake
set(ABSL_INTERNAL_DLL_TARGETS
  "stacktrace"
  "symbolize"
  "examine_stack"
  "failure_signal_handler"
  "debugging_internal"
  "demangle_internal"
  "leak_check"
  "leak_check_disable"
  "stack_consumption"
  "debugging"
  "hash"
  "spy_hash_state"
  "city"
  "memory"
  "strings"
  "strings_internal"
  "cord"
  "str_format"
  "str_format_internal"
  "pow10_helper"
  "int128"
  "numeric"
  "utility"
  "any"
  "bad_any_cast"
  "bad_any_cast_impl"
  "span"
  "optional"
  "bad_optional_access"
  "bad_variant_access"
  "variant"
  "compare"
  "algorithm"
  "algorithm_container"
  "graphcycles_internal"
  "kernel_timeout_internal"
  "synchronization"
  "thread_pool"
  "bind_front"
  "function_ref"
  "atomic_hook"
  "log_severity"
  "raw_logging_internal"
  "spinlock_wait"
  "config"
  "dynamic_annotations"
  "core_headers"
  "malloc_internal"
  "base_internal"
  "base"
  "throw_delegate"
  "pretty_function"
  "endian"
  "bits"
  "exponential_biased"
  "periodic_sampler"
  "scoped_set_env"
  "type_traits"
  "meta"
  "random_random"
  "random_bit_gen_ref"
  "random_distributions"
  "random_seed_gen_exception"
  "random_seed_sequences"
  "random_internal_traits"
  "random_internal_distribution_caller"
  "random_internal_distributions"
  "random_internal_fast_uniform_bits"
  "random_internal_seed_material"
  "random_internal_pool_urbg"
  "random_internal_explicit_seed_seq"
  "random_internal_sequence_urbg"
  "random_internal_salted_seed_seq"
  "random_internal_iostream_state_saver"
  "random_internal_generate_real"
  "random_internal_wide_multiply"
  "random_internal_fastmath"
  "random_internal_nonsecure_base"
  "random_internal_pcg_engine"
  "random_internal_randen_engine"
  "random_internal_platform"
  "random_internal_randen"
  "random_internal_randen_slow"
  "random_internal_randen_hwaes"
  "random_internal_randen_hwaes_impl"
  "random_internal_uniform_helper"
  "status"
  "time"
  "civil_time"
  "time_zone"
  "container"
  "btree"
  "compressed_tuple"
  "fixed_array"
  "inlined_vector_internal"
  "inlined_vector"
  "counting_allocator"
  "flat_hash_map"
  "flat_hash_set"
  "node_hash_map"
  "node_hash_set"
  "container_memory"
  "hash_function_defaults"
  "hash_policy_traits"
  "hashtablez_sampler"
  "hashtable_debug"
  "hashtable_debug_hooks"
  "have_sse"
  "node_hash_policy"
  "raw_hash_map"
  "container_common"
  "raw_hash_set"
  "layout"
  "tracked"
)
```

*在使用的时候别忘记添加`absl::`前缀。*





---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/abseil-%E5%88%9D%E4%BD%93%E9%AA%8C/  

