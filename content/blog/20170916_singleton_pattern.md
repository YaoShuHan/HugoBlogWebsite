+++
title = "初识golang—单例模式"
date = 2017-09-16T11:36:57+08:00
categories = ["Development", "Golang"]
trags = ["Development", "Golang"]
type = "post"
+++

单例模式是设计模式里最为常用的一种，应用该设计模式的目的是保证类仅有一个全局的对象实例。

## 懒汉模式(Lazy Loading)

懒汉模式是最为简单常见的，最大的缺点是非线程安全的。

    type singleton struct {
        //...
    }
    //private
    var instance *singleton
    //public
    func GetInstance() *singleton {
        if instance == nil {
            instance = &singleton{}
        }
        return instance
    }

## 带锁的单例模式

sync包提供两种锁类型：Mutex和RWMutex。Mutex是我们常见的锁类型，当一个goroutine获得Mutex之后，其它的goroutine只能进行等待直到该Mutex被释放。而RWMutex则是较为友好的单读多写模型，写锁会阻止其它所任何goroutine访问，而读锁则只会阻止写而不阻止读。

这里使用了Go的sync.Mutex，解决了懒汉模式的线程安全问题，但是在已生成实例后每次访问还都会调用锁，造成不必要的开销。

    type singleton sturct {
    }

    var instance *singleton
    var mu sync.Mutex

    func GetInstance() *singleton {
        mu.Lock()
        defer mu.Unlock()

        if instance == nil {
            instance = &singleton{}
        }
        return instance
    }

## 带检查锁的单例模式

通过再封加一层判断，可以解决不必要开销的问题。

    if instance == nil {
        mu.Lock()
        defer mu.Unlock()
        
        if instance == nil {
            instance = &singleton{}
        }
        return instance
    }

通过sync/atomic包中的原子操作，还可以写成

    import "sync"
    import "sync/atomic"

    var initialized uint32
    ...
    func GetInstance() *singleton {
        
        if atomic.LoadUInt32(&initialized) ==1 {
            return instance
        }
        
        mu.Lock()
        defer mu.Unlock()

        if initialized == 0 {
            instance = &singleton{}
            atomic.StoreUint32(&initialized, 1)
        }

        return instance
    }

## sync.Once

Go语言中提供了一个sync.Once类型解决全局唯一性操作问题，Once.Do()方法可以保证全局范围内只调用指定的函数一次，而且其他goroutine调用到该语句时都将被阻塞，直到全局唯一的Once.Do()被调用后才执行。

    import "sync"

    type singleton struct {
    }

    var instance *singleton
    var once sync.Once

    func GetInstance() *singleton {
        once.Do(func() {
            instance = &singleton{}
        })
        return instance
    }
