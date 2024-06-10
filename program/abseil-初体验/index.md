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



使用brew安装之后， 可能还是有个问题，那就是可能会出现链接或者编译错误， 如果出现了这个错误， 把自己的项目也改成c&#43;&#43;17版本就可以了。  

```cmake
# specify the C&#43;&#43; standard
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
  &#34;stacktrace&#34;
  &#34;symbolize&#34;
  &#34;examine_stack&#34;
  &#34;failure_signal_handler&#34;
  &#34;debugging_internal&#34;
  &#34;demangle_internal&#34;
  &#34;leak_check&#34;
  &#34;leak_check_disable&#34;
  &#34;stack_consumption&#34;
  &#34;debugging&#34;
  &#34;hash&#34;
  &#34;spy_hash_state&#34;
  &#34;city&#34;
  &#34;memory&#34;
  &#34;strings&#34;
  &#34;strings_internal&#34;
  &#34;cord&#34;
  &#34;str_format&#34;
  &#34;str_format_internal&#34;
  &#34;pow10_helper&#34;
  &#34;int128&#34;
  &#34;numeric&#34;
  &#34;utility&#34;
  &#34;any&#34;
  &#34;bad_any_cast&#34;
  &#34;bad_any_cast_impl&#34;
  &#34;span&#34;
  &#34;optional&#34;
  &#34;bad_optional_access&#34;
  &#34;bad_variant_access&#34;
  &#34;variant&#34;
  &#34;compare&#34;
  &#34;algorithm&#34;
  &#34;algorithm_container&#34;
  &#34;graphcycles_internal&#34;
  &#34;kernel_timeout_internal&#34;
  &#34;synchronization&#34;
  &#34;thread_pool&#34;
  &#34;bind_front&#34;
  &#34;function_ref&#34;
  &#34;atomic_hook&#34;
  &#34;log_severity&#34;
  &#34;raw_logging_internal&#34;
  &#34;spinlock_wait&#34;
  &#34;config&#34;
  &#34;dynamic_annotations&#34;
  &#34;core_headers&#34;
  &#34;malloc_internal&#34;
  &#34;base_internal&#34;
  &#34;base&#34;
  &#34;throw_delegate&#34;
  &#34;pretty_function&#34;
  &#34;endian&#34;
  &#34;bits&#34;
  &#34;exponential_biased&#34;
  &#34;periodic_sampler&#34;
  &#34;scoped_set_env&#34;
  &#34;type_traits&#34;
  &#34;meta&#34;
  &#34;random_random&#34;
  &#34;random_bit_gen_ref&#34;
  &#34;random_distributions&#34;
  &#34;random_seed_gen_exception&#34;
  &#34;random_seed_sequences&#34;
  &#34;random_internal_traits&#34;
  &#34;random_internal_distribution_caller&#34;
  &#34;random_internal_distributions&#34;
  &#34;random_internal_fast_uniform_bits&#34;
  &#34;random_internal_seed_material&#34;
  &#34;random_internal_pool_urbg&#34;
  &#34;random_internal_explicit_seed_seq&#34;
  &#34;random_internal_sequence_urbg&#34;
  &#34;random_internal_salted_seed_seq&#34;
  &#34;random_internal_iostream_state_saver&#34;
  &#34;random_internal_generate_real&#34;
  &#34;random_internal_wide_multiply&#34;
  &#34;random_internal_fastmath&#34;
  &#34;random_internal_nonsecure_base&#34;
  &#34;random_internal_pcg_engine&#34;
  &#34;random_internal_randen_engine&#34;
  &#34;random_internal_platform&#34;
  &#34;random_internal_randen&#34;
  &#34;random_internal_randen_slow&#34;
  &#34;random_internal_randen_hwaes&#34;
  &#34;random_internal_randen_hwaes_impl&#34;
  &#34;random_internal_uniform_helper&#34;
  &#34;status&#34;
  &#34;time&#34;
  &#34;civil_time&#34;
  &#34;time_zone&#34;
  &#34;container&#34;
  &#34;btree&#34;
  &#34;compressed_tuple&#34;
  &#34;fixed_array&#34;
  &#34;inlined_vector_internal&#34;
  &#34;inlined_vector&#34;
  &#34;counting_allocator&#34;
  &#34;flat_hash_map&#34;
  &#34;flat_hash_set&#34;
  &#34;node_hash_map&#34;
  &#34;node_hash_set&#34;
  &#34;container_memory&#34;
  &#34;hash_function_defaults&#34;
  &#34;hash_policy_traits&#34;
  &#34;hashtablez_sampler&#34;
  &#34;hashtable_debug&#34;
  &#34;hashtable_debug_hooks&#34;
  &#34;have_sse&#34;
  &#34;node_hash_policy&#34;
  &#34;raw_hash_map&#34;
  &#34;container_common&#34;
  &#34;raw_hash_set&#34;
  &#34;layout&#34;
  &#34;tracked&#34;
)
```

*在使用的时候别忘记添加`absl::`前缀。*





---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/abseil-%E5%88%9D%E4%BD%93%E9%AA%8C/  

