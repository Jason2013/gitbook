# Google Translate

Vulkan / SPIR-V内存模型建立在C ++内存模型的基础上，但最终在许多地方出现了分歧。

经过多年的发展，GPU编程模型在现代图形API上的工作方式已发生了许多变化，反映了这些API所针对的市场。 自然地，Vulkan / SPIR-V内存模型做出了一些反映这一点的决定。 我们在模型中添加了几个新方面，包括[范围](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_memory_and_execution_scopes) ， [存储类](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_distinct_storage_classes_and_inter_thread_happens_before)以及内存[可用性和可见性操作，](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_availability_and_visibility)以列举一些更为突出的方面。

但是，它也不是严格的超集，并且在某些地方出于相似的原因而省略了某些功能。 例如， [不支持顺序一致性](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_no_sequentially_consistent_semantics) ，并且[向前进度的保证是有限的](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_limited_forward_progress_guarantees) 。

这篇文章旨在对这些差异进行高级别的概述，解释这些差异是**什么** ， **为何**不同，以及C ++概念**如何** （如果有的话）可以映射到Vulkan / SPIR-V内存模型。 它主要针对那些已经熟悉C ++内存模型的人，他们或者想对差异有什么了解，或者对我们为什么选择我们的方向感到好奇的人。

### 差异 <a id="_the_differences"></a>

这里的每个小节都描述了差异是什么，为什么在**理由中**存在差异，并概述了如何在将C ++映射到SPIR-V中将C ++概念映射**到SPIR-V** 。 该博客在顶部列出了基本的区别，最后从C ++内存模型中删除了一些区别。

#### 内存位置和引用 <a id="_memory_locations_and_references"></a>

C ++将单个变量（或整个位域）定义为内存位置-除了将内存用作可用变量之外，没有真正的概念来分配内存。

Vulkan内存模型定义了两个概念，而不是一个：记忆体位置

表示实现分配的基础内存。参考文献

可以通过其访问基础内存的变量或句柄。

**理由**

在Vulkan中，内存分配具有与资源或变量完全不同的生存期，可用于从这些内存分配读取值或将值写入这些内存分配。

**将C ++映射到SPIR-V**

在此内存模型中，C ++变量应同时被视为引用和内存位置，而_不仅仅是_ C ++内存模型中的内存位置。 然后，指针或C ++引用（＆）是对该同一内存位置的附加引用，而指针本身具有一个内存位置并对该新内存位置进行引用。

#### 可用性和可见性 <a id="_availability_and_visibility"></a>

C ++内存模型与Vulkan / SPIR-V的最大区别在于可用性和可见性操作。

可用性操作可确保写入一个线程中的内存位置的值**可**用于其他线程。 可见性操作可确保使**可用**值对线程**可见** ，以确保读取正确的值。 对于支持Vulkan的典型设备，可用性和可见性操作将映射到缓存控制（例如，刷新/无效/旁路）。

为了避免数据争用，除了C ++内存模型中通常需要的同步之外，每种危险类型还需要执行以下操作：写后读

要求首先使写入**可用** ，然后在读取之前**可见** 。写后写

要求在第二次写入之前使第一次写入**可用** 。读后写

不需要可用性或可见性操作。读后读

没有危险。

存储器模型具有表达可用性和可见性操作的能力，这些可用性和可见性操作针对执行特定存储访问时的特定存储访问，或使用同步操作（例如存储屏障）一次表示一组访问。 前者旨在作为实现绕过缓存或使用缓存一致性的实现的指令，并且通常应在仅执行少量小型访问时使用。 后一个选项旨在作为指令来代替执行批量缓存刷新/无效，当在同一线程中执行对相邻内存位置的大量访问时，这可能会更好。

可用性和可见性操作按发布顺序进行排序，并以与存储和加载相同的方式获取语义。

**理由**

C ++是在具有一致的CPU缓存的系统上发展而来的，因此缓存维护操作没有明确公开是很自然的。 当C ++编译到具有非一致性缓存的系统时，可以将适当的缓存维护操作折叠为发布和获取操作。

Vulkan和SPIR-V在具有许多不同执行单元的系统上发展，其中每个执行单元的缓存可能与其他缓存不协调，因此公开缓存维护是API /语言的自然组成部分，并且可以作为优化的来源。

如果没有明确的可用性和可见性操作，则不需要某些高速缓存维护操作（例如，写后写入不需要）的危害仍将被约束执行，这将导致性能下降。

**将C ++映射到SPIR-V**

由于C ++没有可用性和可见性操作，因此从C ++转换为Vulkan SPIR-V时，处理这些问题的最简单方法是根据内存顺序自动将`MakeAvailable`和`MakeVisible`添加到同步指令中，如下所示：

| C ++内存顺序 | SPIR-V存储器语义 |
| :--- | :--- |
| `memory_order_relaxed` | 没有 |
| `memory_order_consume` | 升级到`memory_order_acquire` 1 |
| `memory_order_acquire` | `MakeVisible | Acquire` |
| `memory_order_release` | `MakeAvailable | Release` |
| `memory_order_acq_rel` | `MakeAvailable | MakeVisible | AcquireRelease` |
| `memory_order_seq_cst` | 没有可用的地图2 |

1个

SPIR-V中没有'consume'语义，但是出于映射目的将它们提升为`memory_order_acquire`是安全的。 C ++转换程序可能能够移动内存操作，以减少此升级不必要地同步的内存操作的数量。2

参见[没有“顺序一致”的语义](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_no_sequentially_consistent_semantics)

如上所述，由于SPIR-V提供了两个在何处标记可用性和可见性操作的选项，因此C ++编译器还可以选择将可用性和可见性语义分别附加到存储和加载中，但是如上所述，期望通常是批量操作更适合典型的C ++代码。

| 注意 | 如果C ++转换器将外部分配的内存位置（例如，缓冲区对象）重用于另一个对象，则应注意确保适当执行可用性和可见性操作，以避免由此类重用引起的数据竞争。 |
| :--- | :--- |


#### 混叠 <a id="_aliasing"></a>

引用同一基础内存位置的两个或多个引用被称为_别名_ 。 如果已知引用是别名，则编译器将不会通过这些引用对彼此进行重新排序。

默认情况下，假定引用**不是**别名。 可以使用`Aliased`关键字显式地标记引用，其中所有用`Aliased`装饰的引用都被视为彼此别名。 此外，已知在函数内从另一个参考创建的任何参考都会在该函数持续时间内与原始参考混叠。

对于描述符引用（例如，缓冲区或图像资源），其中在它们之间的任何别名都完全在SPIR-V程序之外进行描述（即，它们是不同的描述符），对一个别名进行的写操作不能在另一别名中被其他别名使用或看到。 SPIR-V程序。 但是，访问仍以相同的方式进行排序，因此通过适当使用`Aliased`关键字和`Acquire` / `Release`语义，仍可以减轻WAR危害。

**理由**

默认情况下不假定使用别名，因为着色器通常是关键路径的很大一部分，并且通常在假定没有别名的情况下进行编写。

引用重叠内存位置的描述符不一定共享相同的虚拟地址，因此对于具有虚拟标记的缓存层次结构的设备，访问不同描述符之间的一致性要求刷新到RAM，这在某些架构上可能是不可能的，与其他同类API相比，充其量不过是性能下降。

**将C ++映射到SPIR-V**

如果C ++到SPIR-V转换器也认为内存访问可能会别名，则通常应将内存访问装饰为别名。 此类转换器将需要谨慎处理外部定义的别名，这样，如果这种细微差别没有以某种方式直接暴露给应用程序，则可以完全避免使用外部别名。

#### 不同的存储类和线程间发生之前 <a id="_distinct_storage_classes_and_inter_thread_happens_before"></a>

Vulkan定义了访问着色器中的内存的多种不同方式，例如缓冲区和图像-在Vulkan规范中称为资源和描述符。

SPIR-V将这些不同的访问表示为`Storage Classes` ，并且同步指令在其操作的`Semantics`定义了`Storage Classes`的列表。 内存访问不受未在其语义中指定存储类的屏障的影响，以同样的方式， [私有内存访问](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_private_memory_access)不受任何屏障的影响。

但是，由于**某些**障碍可能会影响对某些存储类的访问，而不会影响其他存储类的访问，因此内存模型需要一种描述这些障碍之间相互作用的方法。 这导致线程间发生之前的“模板化”定义，其中一组存储类是该定义的一部分，线程间发生在与不同组存储类之间的关系不交互以形成关系之前。

OpenCL面临类似的问题，并最终导致之前的本地局部不相交，之前的全局局部不相交以及图像访问不相交。 模板化线程间线程在关系处理相同问题之前发生，但是能够表达不同存储类中的操作之间的顺序。

**理由**

C ++分配在很大程度上是同质的-与通过原始变量访问变量相比，指向变量的指针不会改变以任何方式访问该变量的方式。

由于每种资源的访问方法都不同，因此历史上一直使用独特的缓存层次结构来创建图形设备，其中不同的存储类具有自己的缓存和访问硬件。 图形和以GPU为中心的计算API通常具有相同的区别，从而导致这些设备在用于这些设备的编程语言中以某种方式被调用。

**将C ++映射到SPIR-V**

由于C ++并没有积极地为大多数资源类型提供SPIR-V公开的大多数概念，因此主机设备通信仅需要直接支持对`StorageBuffer`存储类中资源的使用，从而避免了此问题。

#### 内存和执行范围 <a id="_memory_and_execution_scopes"></a>

SPIR-V定义了`Scope`枚举，该枚举对应于从一小部分线程子集（ `Subgroup` ）到整个设备（ `Device` ）执行的所有线程的几个嵌套线程`Subgroup` 。

所有同步指令（原子，内存屏障，控制屏障）都包含一个`Scope`操作数，该`Scope`定义了任何同步应该与执行线程相距多远。

例如，具有`Subgroup`范围的具有可用语义的存储/发行版对于不同子组中的加载/获取而言，无论该加载/获取的`Scope`如何，都无法看到。

每个`Scope`对应于一个不同的嵌套[内存域](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_memory_domains) 。

**理由**

Vulkan定位通常“大规模并行”的设备，因为它们能够同时处理大量并发线程。

驱动它们的执行单元通常按层次划分，这样，较小的线程组可以比较大的组更有效地进行通信。 因此，需要跨较小的线程组执行同步操作以实现更高的性能。

**将C ++映射到SPIR-V**

C ++没有作用域，因此默认情况下，编译应选择在任何地方使用`QueueFamily` （或`Device`如果支持）） `Scope` 。 但是，请注意，这可能会对性能产生重大影响，如果可以在编译时推断出较小范围的使用是安全的，则应首选它们。

#### 记忆域 <a id="_memory_domains"></a>

[可用性和可见性](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_availability_and_visibility)操作仅在单个内存域中运行-为了在内存域之间进行数据通信，需要一个**内存域操作** ，这保证了一个内存域中**可用的**值**可**用于另一个内存域。

与[可用性和可见性](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_availability_and_visibility)操作类似，这是一个显式的缓存管理概念，其中每个内存域都映射到Vulkan实现使用的缓存层次结构的不同部分。 内存域操作将适当地执行缓存维护，以确保可以跨这些缓存层次结构正确访问数据。

主机和设备是Vulkan中的两个关键内存域，每种着色器类型都有额外的着色器内存域集，对应于各种“ [内存”和“执行范围”](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_memory_and_execution_scopes) 。 通过使用具有各种作用域的API屏障或着色器屏障，可以使写入可用于不同的存储域。

**理由**

诸如GPU之类的处理器可能包含具有复杂缓存层次结构的几个不同的执行单元。 内存域操作以及[可用性和可见性](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_availability_and_visibility)操作为实现提供了正确管理这些缓存所需的信息。

**将C ++映射到SPIR-V**

如果仅考虑计算着色器（通常用于将C ++代码编译到GPU），则可以忽略外部存储域，仅关注由不同的[Memory和Execution Scope](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_memory_and_execution_scopes)提供的存储域。

在CPU线程和着色器调用之间共享数据时，主要需要显式的内存域操作，因此，在何处完全取决于工作的分配方式。

#### 没有“顺序一致”的语义 <a id="_no_sequentially_consistent_semantics"></a>

顺序一致的语义不包括在Vulkan内存模型中。

**理由**

C ++中顺序一致的语义在缓存操作上设置了严格的顺序保证，这在某种程度上与Vulkan具有显式缓存管理的需求背道而驰，并且在面对多个不同的[内存域](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_memory_domains)时尤其难以保证。 GPU通常在设计时假设内存顺序很弱，并且鉴于顺序一致性在某些CPU架构（POWER）上已显示出问题，因此可以合理地假设在GPU上也会存在类似的问题。

另外，我们没有对顺序一致性的支持，因此，现在添加它似乎并没有带来明显的好处。 其他语言（例如C ++ AMP）选择放弃顺序一致性，这增加了我们对包含该功能的怀疑。

最后，如何将顺序一致性与我们对模型所做的其他一些概括协调起来还不是很明显。

考虑到所有这些，Vulkan内存模型小组选择不包含这种颇有争议的语义，因为得出的结论是，这将给实现带来不必要的负担，而几乎没有实际好处。

**将C ++映射到SPIR-V**

对于Vulkan / SPIR-V中的顺序一致，没有任何琐碎的替换。

在许多情况下，将这些语义降级为`AcquireRelease`可能就足够了，但不是1：1匹配，因此应用程序可能需要在编译时选择加入。

#### 原子操作与原子对象 <a id="_atomic_operations_vs_atomic_objects"></a>

C ++原子通过专用原子对象公开-这在某种程度上简化了内存模型； 对相同内存位置的原子访问和非原子访问之间没有交互。 相反，SPIR-V具有可对常规变量执行的原子操作。

对更广泛的内存模型的实际影响很小，但这确实意味着原子在某些情况下必须像非原子访问一样被同步。 这导致了模型的两个差异：

1. 如果原子在相同的[范围内](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#Memory%20and%20Execution%20Scope)并且通过相同的引用构成，则访问同一内存位置的原子只有相互排序。
   * 如果两个原子不在彼此的`Scope` ，则必须将它们排序为非原子操作
   * 该顺序称为“ **范围修改顺序”** ，而不是C ++的“ **总修改顺序”**
2. 原子执行的访问是常规访问的超集，具有以下区别
   * 可用性和可见性操作会自动针对原子访问的存储位置（分别用于存储和加载）执行。
   * 原子访问及其可用性和可见性操作是作为原子指令的一部分“以原子方式”执行的，即，不会促进数据竞争。

**理由**

语言的发展导致了这种情况，我们已经定义了它的工作方式。

**将C ++映射到SPIR-V**

由于C ++不会公开原子和非原子对象的别名，因此实际上不需要映射-仅对原子对象使用原子操作，对常规对象使用常规操作。

#### 私有内存访问 <a id="_private_memory_access"></a>

假定对C ++中的变量的所有访问都参与线程间的排序关系。

在Vulkan内存模型中，我们引入了“私有”与“非私有”访问的概念，其中仅保证非私有访问的结果会影响其他线程中的访问或受其影响。 私有访问不受获取和释放语义的影响。

这些访问对于在计算结束时是只读或仅写入一次的事物来说是典型的。 实际上，仅针对访问其值的计算以及程序的开始或结束对它们进行排序。 它们最终将在API级别同步。

**理由**

通常情况下，读取或写入的资源需要在不同的SPIR-V程序实例之间传递，而实际上并不需要在单个调用中进行同步。 这样，私有内存访问无需与障碍交互，并且将其表达给实现可以通过重新排序这些访问来实现常见的，有时甚至是实质性的加速。

**将C ++映射到SPIR-V**

由于C ++没有这个概念，因此通常应使用`NonPrivate`语义修饰加载和存储。 唯一的例外是const内存的定义，如果没有const强制转换并且不存在可变别名，则可以避免语义。

#### 程序顺序 <a id="_program_order"></a>

Vulkan / SPIR-V内存模型定义了一个简单的术语“程序顺序”，而不是C ++中更复杂的“顺序排序”。 大致上，这是在SPIR-V程序中指定指令的顺序。

**理由**

SPIR-V是一种IR，其中每个基本块都是一个简单的指令有序列表，而不是更复杂的高级语言。 这导致对“程序顺序”的定义简单明了，与C ++更复杂的“先排序”顺序形成鲜明对比。

**将C ++映射到SPIR-V**

尽管在C ++中，某些评估不确定地排序，但执行中的任何两个SPIR-V指令均按程序顺序关联。 将C ++编译为SPIR-V时，由C ++编译器选择不确定顺序的求值顺序。

#### 控制壁垒 <a id="_control_barriers"></a>

Vulkan内存模型提供的另一项功能是包含控制障碍。 控制屏障可确保指定[**范围内的**](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_memory_and_execution_scopes)所有线程都达到相同的执行点。 在SPIR-V程序中，使用控制屏障是唯一明确定义的方法。

**理由**

考虑到Vulkan SPIR-V的[有限转发进度保证](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_limited_forward_progress_guarantees) ，这是一种相当必要的机制，它是安全地阻塞线程以进行线程间通信的唯一方法。

**将C ++映射到SPIR-V**

没有用于控制障碍的直接映射到C ++

#### 有限的前进进度保证 <a id="_limited_forward_progress_guarantees"></a>

没有规范甚至没有建议，当一个或多个线程在用户定义的条件下（例如，自旋锁）被阻塞时，单个执行单元中的所有未阻塞线程最终都会前进。 [控制屏障](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com&sl=en&sp=nmt4&tl=zh-CN&u=https://www.khronos.org/blog/comparing-the-vulkan-spir-v-memory-model-to-cs&xid=25657,15700022,15700186,15700191,15700256,15700259,15700262,15700265,15700271,15700283&usg=ALkJrhitE24NzZEyLQoLKspYwZUorf1oZA#_control_barriers)是SPIR-V中唯一可用于同步多个线程的明确定义的机制。

整个Vulkan API确实可以提供向前的进度保证，这允许应用程序在API级别执行粗略同步以驱动所需的排序。

**理由**

当前的GPU硬件没有这些调度保证。

**将C ++映射到SPIR-V**

C ++仅推荐这种向前的进展，而不需要这种进展-因此，绝对不需要进行任何处理。 但是，在支持C ++的通用平台上，做出了这些保证，并因此被认为是真实的-因此，这可能会使习惯于多线程CPU编码的程序员感到惊讶。

针对SPIR-V的C ++编译器应确保非常清楚地表明用户定义的阻止操作是未定义的行为。 同步功能的库实现可以利用API级别的同步来确保所需的前向进度，但要以效率为代价。

### 摘要 <a id="_summary"></a>

尽管差异列表并不是特别短，并且列表并不详尽，但总体而言，差异并不是很大。 如果您在使用C ++内存模型的背景下阅读了这篇文章，希望这篇文章可以帮助您回答一些问题，并使Vulkan / SPIR-V内存模型更易于使用！Khronos，EGL，glTF，NNEF，OpenVG，OpenVX，OpenXR，SPIR，SPIR-V，SYCL，Vulkan和WebGL是Khronos Group Inc.的商标或注册商标。OpenCL是Apple Inc.的商标，并且OpenGL和OpenML已注册。商标以及OpenGL ES和OpenGL SC徽标是Hewlett Packard Enterprise的商标，由Khronos许可使用。 所有其他产品名称，商标和/或公司名称仅用于标识，并且属于其各自所有者。**由** AMD Tobias Hector **撰写**

