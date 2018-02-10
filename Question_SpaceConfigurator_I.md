##学习STL源码剖析时遇到的问题
#函数
* std::cerr是ISO C++标准错误输出流，对应于ISO C标准库的stderr.与std::cout不同，ISO C++要求当cerr被初始化后，
  cerr.flags() & unitbuf非零（保证流在每次输出操作后被刷新），且cerr.tie()返回&cout。即cerr默认和cout同步但无缓冲。
* stl_construct.h 提供对象构造操作::construct(), 对象析构操作::destroy()
* stl_alloc.h 提供内存配置基本操作::operator new(), 提供内存释放基本操作::operator delete()。这两个函数相当于C的malloc()和free()函数。
* 考虑到小型区块可能造成的内存破碎问题，SGI设计了双层级配置器，第一级配置器直接使用malloc()和free()，第二级配置器采用复杂的memory pool 整理方式。
* alloc并不接受任何的template型别参数
```
# ifdef __USE_MALLOC
...
typedef __malloc_alloc_template<0> malloc_alloc;
typedef malloc_alloc alloc;
# else
...
typedef __default_alloc_template<__NODE_ALLOCATOR_THREADS, 0> alloc;
# endif /* ! __USE_MALLOC */

template <class T, class Alloc>
class simple_alloc {
	public:
		static T *allocate(size_t n)
			{ return 0 == n ? 0 : (T*)Alloc::allocate(n * sizeof(T)); }
		static T *allocate(void)
			{ return (T*) Alloc::allocate(sizeof(T)); }
		static void deallocate(T *p, size_t n)
			{ if (0 != n) Alloc::deallocate(p, n * sizeof (T)); }
		static void deallocate(T *p)
			{ Alloc::deallocate(p, sizeof(T)); }
};

template <class T, class Alloc = alloc>
class vector{
	protected:
		typedef simple_alloc<value_type, Alloc> data_allocator;
		
		void deallocate(){
			if(...)
				data_allocator::deallocate(start, end_of_storage - start);
		}
	...
};
```