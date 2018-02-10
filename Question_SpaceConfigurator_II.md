##STL源码剖析
#空间配置器<二>第一级配置器 __malloc_alloc_template 剖析
```
#if 0
# 	include <new>
#	define __THROW_BAD_ALLOC throw bad_alloc
#elif !defined(__THROW_BAD_ALLOC)
#	include <iostream>
#	define __THROW_BAD_ALLOC cerr << "out of money" << endl;
#endif

//µÚÒ»¼¶ÅäÖÃÆ÷
template <int inst>
 class __malloc_alloc_template{
 	private:
 		static void *oom_malloc(size_t);
 		static void *oom_realloc(void *, size_t);
 		static void (* __malloc_alloc_oom_handler)();
 	public:
 		
 		static void * allocate(size_t n)
 		{
 			void *result = malloc(n);
 			if (0 == result) result = oom_malloc(n);
 			return result;
		}
		static void deallocate(void *p, size_t /* n */)
		{
			free(p);
		}
		static void * reallocate(void *p, size_t /* old_sz */, size_t new_sz)
		{
			void * result = realloc(p, new_sz);
			if (0 == result) result = oom_realloc(p, new_sz);
			return result;
		}
		
		static void (* set_malloc_handler(void (*f)()))()
		{
			void (* old)() = __malloc)alloc_oom_handler;
			__malloc_alloc_oom_handler = f;
			return(old);
		}
 };
 //malloc_alloc out-of-memory handling
 template <int inst>
 void (* __malloc_alloc_template<inst>::__malloc_alloc_oom_hander)() = 0;
 
 template <int inst>
 void * __malloc_alloc_template<inst>::oom_malloc(size_t n)
 {
 	void (* my_malloc_handler)();
 	void *result;
 	
 	for(;;){
 		my_malloc_handler = __malloc_alloc_oom_handler;
 		if (0 == my_malloc_handler) { __THROW_BAD_ALLOC; }
 		(*my_malloc_handler)();
 		result = malloc(n);
 		if (result) return(result);
	 }
 }
 
 template <int inst>
 void * __malloc_alloc_template<inst>::oom_realloc(void *p, size_t n)
 {
 	void (* my_malloc_handler)();
 	void *result;
 	
 	for(;;){
 		my_malloc_handler = __malloc_alloc_oom_handler;
 		if (0 == my_malloc_handler) { __THROW_BAD_ALLOC; }
 		(*my_malloc_handler)();
 		result = realloc(p, n);
 		if (result) return(result);
	 }
 }
 
 typedef __malloc_alloc_template<0> malloc_alloc;
```
---
* C++ new handler机制是，你可以要求系统在内存配置需求无法被满足时，调用一个你所指定的函数。
 换句话说，一旦::operator new无法完成任务，在丢出 std::bad_alloc 异常状态之前，
 会先调用由客端指定的处理例程。该处理例程通常被称为 new-handler.