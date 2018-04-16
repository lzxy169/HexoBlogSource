---
title: ARC(Automatic Reference Counting)简记
date: 2018-04-13 10:29:09
tags:
categories: [Objective-C]
---
## ARC概述
　　在Objective-C中采用Automatic Reference Counting(ARC)机制，让编译器来进行内存管理。

### 内存管理的思考方式

* 自己生成的对象，自己持有
* 非自己生成的对象，自己也能持有
* 不再需要自己持有的对象时释放
* 非自己持有的对象无法释放

**对象操作与Objective-C方法的对应：**

| 对象操作       | Objective-C方法                   |
|----------------|-----------------------------------|
| 生成并持有对象 | alloc/new/copy/mutableCopy 等方法 |
| 持有对象       | retain方法                        |
| 释放对象       | release方法                       |
| 废弃对象       | dealloc方法                       |

>“自己”：对象的使用环境

**苹果采用散列表(引用计数表)来管理引用计数**

```c++
static struct {
    CFLock_t lock;
    CFBasicHashRef table;
//    uint8_t padding[64 - sizeof(CFBasicHashRef) - sizeof(CFLock_t)];
} __NSRetainCounters[NUM_EXTERN_TABLES];

CF_EXPORT uintptr_t __CFDoExternRefOperation(uintptr_t op, id obj) {
    if (nil == obj) HALT;
    uintptr_t idx = EXTERN_TABLE_IDX(obj);
    uintptr_t disguised = DISGUISE(obj);
    CFLock_t *lock = &__NSRetainCounters[idx].lock;
    CFBasicHashRef table = __NSRetainCounters[idx].table;  // 取得对象对应的散列表
    uintptr_t count;
    switch (op) {
    case 300:   // increment
    case 350:   // increment, no event
        __CFLock(lock);
    CFBasicHashAddValue(table, disguised, disguised);
        __CFUnlock(lock);
        if (__CFOASafe && op != 350) __CFRecordAllocationEvent(__kCFObjectRetainedEvent, obj, 0, 0, NULL);
        return (uintptr_t)obj;
    case 400:   // decrement
        if (__CFOASafe) __CFRecordAllocationEvent(__kCFObjectReleasedEvent, obj, 0, 0, NULL);
    case 450:   // decrement, no event
        __CFLock(lock);
        count = (uintptr_t)CFBasicHashRemoveValue(table, disguised);
        __CFUnlock(lock);
        return 0 == count;
    case 500:
        __CFLock(lock);
        count = (uintptr_t)CFBasicHashGetCountOfKey(table, disguised);
        __CFUnlock(lock);
        return count;
    }
    return 0;
}
```
[CF-1153.18/CFRuntime.c][1]

**CFBasicHashRef**

```c++
CF_PRIVATE CFBasicHashRef CFBasicHashCreate(CFAllocatorRef allocator, CFOptionFlags flags, const CFBasicHashCallbacks *cb) {
    size_t size = sizeof(struct __CFBasicHash) - sizeof(CFRuntimeBase);
    if (flags & kCFBasicHashHasKeys) size += sizeof(CFBasicHashValue *); // keys
    if (flags & kCFBasicHashHasCounts) size += sizeof(void *); // counts
    if (flags & kCFBasicHashHasHashCache) size += sizeof(uintptr_t *); // hashes
    CFBasicHashRef ht = (CFBasicHashRef)_CFRuntimeCreateInstance(allocator, CFBasicHashGetTypeID(), size, NULL);
    if (NULL == ht) return NULL;

    ht->bits.finalized = 0;
    ht->bits.hash_style = (flags >> 13) & 0x3;
    ht->bits.fast_grow = (flags & kCFBasicHashAggressiveGrowth) ? 1 : 0;
    ht->bits.counts_width = 0;
    ht->bits.strong_values = (flags & kCFBasicHashStrongValues) ? 1 : 0;
    ht->bits.strong_keys = (flags & kCFBasicHashStrongKeys) ? 1 : 0;
    ht->bits.weak_values = (flags & kCFBasicHashWeakValues) ? 1 : 0;
    ht->bits.weak_keys = (flags & kCFBasicHashWeakKeys) ? 1 : 0;
    ht->bits.int_values = (flags & kCFBasicHashIntegerValues) ? 1 : 0;
    ht->bits.int_keys = (flags & kCFBasicHashIntegerKeys) ? 1 : 0;
    ht->bits.indirect_keys = (flags & kCFBasicHashIndirectKeys) ? 1 : 0;
    ht->bits.num_buckets_idx = 0;
    ht->bits.used_buckets = 0;
    ht->bits.deleted = 0;
    ht->bits.mutations = 1;

    if (ht->bits.strong_values && ht->bits.weak_values) HALT;
    if (ht->bits.strong_values && ht->bits.int_values) HALT;
    if (ht->bits.strong_keys && ht->bits.weak_keys) HALT;
    if (ht->bits.strong_keys && ht->bits.int_keys) HALT;
    if (ht->bits.weak_values && ht->bits.int_values) HALT;
    if (ht->bits.weak_keys && ht->bits.int_keys) HALT;
    if (ht->bits.indirect_keys && ht->bits.strong_keys) HALT;
    if (ht->bits.indirect_keys && ht->bits.weak_keys) HALT;
    if (ht->bits.indirect_keys && ht->bits.int_keys) HALT;

    uint64_t offset = 1;
    ht->bits.keys_offset = (flags & kCFBasicHashHasKeys) ? offset++ : 0;
    ht->bits.counts_offset = (flags & kCFBasicHashHasCounts) ? offset++ : 0;
    ht->bits.hashes_offset = (flags & kCFBasicHashHasHashCache) ? offset++ : 0;

#if DEPLOYMENT_TARGET_EMBEDDED || DEPLOYMENT_TARGET_EMBEDDED_MINI
    ht->bits.hashes_offset = 0;
    ht->bits.strong_values = 0;
    ht->bits.strong_keys = 0;
    ht->bits.weak_values = 0;
    ht->bits.weak_keys = 0;
#endif

    ht->bits.__kret = CFBasicHashGetPtrIndex((void *)cb->retainKey);
    ht->bits.__vret = CFBasicHashGetPtrIndex((void *)cb->retainValue);
    ht->bits.__krel = CFBasicHashGetPtrIndex((void *)cb->releaseKey);
    ht->bits.__vrel = CFBasicHashGetPtrIndex((void *)cb->releaseValue);
    ht->bits.__kdes = CFBasicHashGetPtrIndex((void *)cb->copyKeyDescription);
    ht->bits.__vdes = CFBasicHashGetPtrIndex((void *)cb->copyValueDescription);
    ht->bits.__kequ = CFBasicHashGetPtrIndex((void *)cb->equateKeys);
    ht->bits.__vequ = CFBasicHashGetPtrIndex((void *)cb->equateValues);
    ht->bits.__khas = CFBasicHashGetPtrIndex((void *)cb->hashKey);
    ht->bits.__kget = CFBasicHashGetPtrIndex((void *)cb->getIndirectKey);

    for (CFIndex idx = 0; idx < offset; idx++) {
        ht->pointers[idx] = NULL;
    }

#if ENABLE_MEMORY_COUNTERS
    int64_t size_now = OSAtomicAdd64Barrier((int64_t) CFBasicHashGetSize(ht, true), & __CFBasicHashTotalSize);
    while (__CFBasicHashPeakSize < size_now && !OSAtomicCompareAndSwap64Barrier(__CFBasicHashPeakSize, size_now, & __CFBasicHashPeakSize));
    int64_t count_now = OSAtomicAdd64Barrier(1, & __CFBasicHashTotalCount);
    while (__CFBasicHashPeakCount < count_now && !OSAtomicCompareAndSwap64Barrier(__CFBasicHashPeakCount, count_now, & __CFBasicHashPeakCount));
    OSAtomicAdd32Barrier(1, &__CFBasicHashSizes[ht->bits.num_buckets_idx]);
#endif

    return ht;
}
```
[CF-1153.18/CFBasicHash.c][1]

### autorelease

　　类似于C语言中的自动变量(局部变量)的特性。若某自动变量超出其作用域，改自动变量将被自动废弃。

**autorelease 的具体用法：**

* 生成并持有 NSAutoreleasePool 对象
* 调用已分配对象的 autorelease 实例方法
* 废弃 NSAutoreleasePool 对象
```objective-c
/* ARC无效 */
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
// 等同于 objc_autoreleasePoolPush();

id obj = [[NSObject alloc] init];
[obj autorelease];
// 等同于 objc_autorelease(obj);

[pool drain];
// 等同于 objc_autoreleasePoolPop(pool);

/* ARC有效 */
@autoreleasepool { // 显式
    id __autoreleasing obj = [[NSObject alloc] init];
}

@autoreleasepool { // 非显式
    // 非自己生成并持有的对象
    id __strong obj = [NSMutableArray array];
}
```
　　ARC有效时，指定“@autoreleasepool 块”来代替“NSAutoreleasePool 类对象生成，持有以及废弃”。通过将对象赋值给附加了\_\_autoreleasing 修饰符的变量来代替调用 autorelease 方法。对象赋值给附有 \_\_autoreleasing 修饰符的变量等价于在ARC无效时调用对象的 autorelease 方法，即对象被注册到 autoreleasepool 。

　　非显式地使用\_\_autoreleasing也可以。这是由于编译器会检查方法名是否以 alloc/new/copy/mutableCopy 开始，如果不是则自动将返回的对象注册到 autoreleasepool 。

```objective-c
+ (id)array {
    id obj = [[NSMutableArray alloc] init];
    return obj;
}
```
　　以上为取得非自己生成并持有的对象时被调用方法的源代码示例。因为没有显式指定所有权修饰符所以 id obj 同附有 \_\_strong 修饰符的 id \_\_strong obj 是完全一样的。由于 return 使得对象变量超出其作用域，所以该强引用对应的自己持有的对象会被自动释放，但该对象作为函数的返回值，编译器会自动将其注册到 autoreleasepool 中。

id 的指针(id *obj)或对象的指针(NSObject **obj)在没有显式指定时会被附加上 \_\_autoreleasing 修饰符。



　　在访问附有 \_\_weak 修饰符的变量时，实际上必定要访问注册到 autoreleasepool 的对象。为什么？
```objective-c
id __strong obj0 = [[NSObject alloc] init];
id __weak obj1 = obj0;
id __autoreleasing tmp = obj1;
```
　　这是因为 __weak 修饰符只持有对象的弱引用，而在访问引用对象的过程中，该对象有可能被废弃。如果把要访问的对象注册到 autoreleasepool 中，那么在 @autoreleasepool 块结束之前都能确保该对象存在。

　　在Cocoa框架中，相当于主循环的 NSRunLoop 或者在其他程序可运行的地方，对 NSAutoreleasePool 对象进行生成，持有和废弃处理。

　　Cocoa框架中有很多类方法用于返回 autorelease 的对象。比如：
```objective-c
id array = [NSMutableArray arrayWithCapacity:1];
id array2 = [[[NSMutableArray alloc] initWithCapacity:1] autorelease];
```



**autorelease 的实现**

```c++
class AutoreleasePoolPage 
{
    id *add(id obj)
    {
        id *ret = next;  // faster than `return next-1` because of aliasing
        *next++ = obj;
        return ret;
    }

    void releaseAll() 
    {
        releaseUntil(begin());
    }

    static inline id autorelease(id obj)
    {
        assert(obj);
        assert(!obj->isTaggedPointer());
        id *dest __unused = autoreleaseFast(obj);
        assert(!dest  ||  dest == EMPTY_POOL_PLACEHOLDER  ||  *dest == obj);
        return obj;
    }

    static inline void *push() 
    {
        id *dest;
        if (DebugPoolAllocation) {
            // Each autorelease pool starts on a new pool page.
            dest = autoreleaseNewPage(POOL_BOUNDARY);
        } else {
            dest = autoreleaseFast(POOL_BOUNDARY);
        }
        assert(dest == EMPTY_POOL_PLACEHOLDER || *dest == POOL_BOUNDARY);
        return dest;
    }

    static inline void pop(void *token) 
    {
        AutoreleasePoolPage *page;
        id *stop;

        if (token == (void*)EMPTY_POOL_PLACEHOLDER) {
            // Popping the top-level placeholder pool.
            if (hotPage()) {
                // Pool was used. Pop its contents normally.
                // Pool pages remain allocated for re-use as usual.
                pop(coldPage()->begin());
            } else {
                // Pool was never used. Clear the placeholder.
                setHotPage(nil);
            }
            return;
        }

        page = pageForPointer(token);
        stop = (id *)token;
        if (*stop != POOL_BOUNDARY) {
            if (stop == page->begin()  &&  !page->parent) {
                // Start of coldest page may correctly not be POOL_BOUNDARY:
                // 1. top-level pool is popped, leaving the cold page in place
                // 2. an object is autoreleased with no pool
            } else {
                // Error. For bincompat purposes this is not 
                // fatal in executables built with old SDKs.
                return badPop(token);
            }
        }

        if (PrintPoolHiwat) printHiwat();

        page->releaseUntil(stop);

        // memory: delete empty children
        if (DebugPoolAllocation  &&  page->empty()) {
            // special case: delete everything during page-per-pool debugging
            AutoreleasePoolPage *parent = page->parent;
            page->kill();
            setHotPage(parent);
        } else if (DebugMissingPools  &&  page->empty()  &&  !page->parent) {
            // special case: delete everything for pop(top) 
            // when debugging missing autorelease pools
            page->kill();
            setHotPage(nil);
        } 
        else if (page->child) {
            // hysteresis: keep one empty child if page is more than half full
            if (page->lessThanHalfFull()) {
                page->child->kill();
            }
            else if (page->child->child) {
                page->child->child->kill();
            }
        }
    }

    static void init()
    {
        int r __unused = pthread_key_init_np(AutoreleasePoolPage::key, 
                                             AutoreleasePoolPage::tls_dealloc);
        assert(r == 0);
    }
}
```

### 所有权修饰符
　　Objective-C 中为了处理对象，将类型定义为 id 类型或各种对象类型
　　id 类型用于隐藏类型的类名，相当于C语言中的 void *。

* **__strong：** 表示对对象的“强引用”。持有强引用的变量，在超出其作用域时强引用失效，所以自动地释放自己持有的对象，对象的所有者不存在，因此废弃该对象。该修饰符是 id 类型和对象类型默认的所有权修饰符。
* **__weak：** 表示对对象的“弱引用”。持有弱引用的变量，在超出其作用域时，对象即被释放。在持有某对象的弱引用时，若该对象被废弃，则此弱引用将自动失效且被赋值为 nil (空弱引用)。
* **__unsafe_unretained：** 附有该修饰符的变量不属于编译器的内存管理对象。既不持有对象的强引用也不持有对象的弱引用，只是表示对象，若该对象被废弃，其为悬垂指针。
* **__autoreleasing：** 



>内存泄漏就是应当废弃的对象在超出其生存周期后继续存在。相互引用(循环引用)容易发生内存泄漏。

>野指针是指向“垃圾”内存（不可用内存）的指针。不是NULL指针。

>悬垂指针是指指向曾经存在的对象，但该对象已经不再存在了。


## ARC规则

## ARC实现




[1]: https://opensource.apple.com/tarballs/CF/
