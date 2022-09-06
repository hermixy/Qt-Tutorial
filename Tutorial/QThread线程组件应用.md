QThread库是QT中提供的跨平台多线程实现方案,使用时需要继承QThread这个基类,并重写实现内部的Run方法,由于该库是基本库,默认依赖于`QtCore.dll`这个基础模块,在使用时无需引入其他模块.

**实现简单多线程:** QThread库提供了跨平台的多线程管理方案,通常一个QThread对象管理一个线程,在使用是需要从QThread类继承并重写内部的Run方法,并在Run方法内部实现多线程代码.
```C
#include <QCoreApplication>
#include <iostream>
#include <QThread>

class MyThread: public QThread
{

protected:
    volatile bool m_to_stop;

protected:
    // 线程函数必须使用Run作为开始
    void run()
    {
        for(int x=0; !m_to_stop && (x <10); x++)
        {
            msleep(1000);
            std::cout << objectName().toStdString() << std::endl;
        }
    }

public:
    MyThread()
    {
        m_to_stop = false;
    }

    // 用于设置结束符号为真
    void stop()
    {
        m_to_stop = true;
    }

    // 输出线程运行状态
    void is_run()
    {
        std::cout << "Thread Running = " << isRunning() << std::endl;
    }

    // 输出线程完成状态(是否结束)
    void is_finish()
    {
        std::cout << "Thread Finished = " << isFinished() << std::endl;
    }

};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 定义线程数组
    MyThread thread[10];

    // 设置线程对象名字
    for(int x=0;x<10;x++)
    {
        thread[x].setObjectName(QString("thread => %1").arg(x));
    }

    // 批量调用run执行
    for(int x=0;x<10;x++)
    {
        thread[x].start();
        thread[x].is_run();
        thread[x].isFinished();
    }

    // 批量调用stop关闭
    for(int x=0;x<10;x++)
    {
        thread[x].wait();
        thread[x].stop();

        thread[x].is_run();
        thread[x].is_finish();
    }

    return a.exec();
}
```

**向线程中传递参数:** 线程在执行前可以通过调用MyThread中的自定义函数,并在函数内实现参数赋值,实现线程传参操作.
```C
#include <QCoreApplication>
#include <iostream>
#include <QThread>

class MyThread: public QThread
{
protected:
    int m_begin;
    int m_end;
    int m_result;

    void run()
    {
        m_result = m_begin + m_end;
    }

public:
    MyThread()
    {
        m_begin = 0;
        m_end = 0;
        m_result = 0;
    }

    // 设置参数给当前线程
    void set_value(int x,int y)
    {
        m_begin = x;
        m_end = y;
    }

    // 获取当前线程名
    void get_object_name()
    {
        std::cout << "this thread name => " << objectName().toStdString() << std::endl;
    }

    // 获取线程返回结果
    int result()
    {
        return m_result;
    }
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    MyThread thread[3];

    // 分别将不同的参数传入到线程函数内
    for(int x=0; x<3; x++)
    {
        thread[x].set_value(1,2);
        thread[x].setObjectName(QString("thread -> %1").arg(x));
        thread[x].start();
    }

    // 等待所有线程执行结束
    for(int x=0; x<3; x++)
    {
        thread[x].get_object_name();
        thread[x].wait();
    }

    // 获取线程返回值并相加
    int result = thread[0].result() + thread[1].result() + thread[2].result();
    std::cout << "sum => " << result << std::endl;

    return a.exec();
}
```

**QMutex 互斥同步线程锁:** QMutex类是基于互斥量的线程同步锁,该锁`lock()`锁定与`unlock()`解锁必须配对使用,线程锁保证线程间的互斥,利用线程锁能够保证临界资源的安全性.

 - 线程锁解决的问题: 多个线程同时操作同一个全局变量,为了防止资源的无序覆盖现象,从而需要增加锁,来实现多线程抢占资源时可以有序执行.
 - 临界资源(Critical Resource): 每次只允许一个线程进行访问 (读/写)的资源.
 - 线程间的互斥(竞争): 多个线程在同一时刻都需要访问临界资源.
 - 一般性原则: 每一个临界资源都需要一个线程锁进行保护.

```C
#include <QCoreApplication>
#include <iostream>
#include <QThread>
#include <QMutex>

static QMutex g_mutex;      // 线程锁
static QString g_store;     // 定义全局变量

class Producer : public QThread
{
protected:
    void run()
    {
        int count = 0;

        while(true)
        {
            // 加锁
            g_mutex.lock();

            g_store.append(QString::number((count++) % 10));
            std::cout << "Producer -> "<< g_store.toStdString() << std::endl;

            // 释放锁
            g_mutex.unlock();
            msleep(900);
        }
    }
};

class Customer : public QThread
{
protected:
    void run()
    {
        while( true )
        {
            g_mutex.lock();
            if( g_store != "" )
            {
                g_store.remove(0, 1);
                std::cout << "Curstomer -> "<< g_store.toStdString() << std::endl;
            }

            g_mutex.unlock();
            msleep(1000);
        }
    }
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    Producer p;
    Customer c;

    p.setObjectName("producer");
    c.setObjectName("curstomer");

    p.start();
    c.start();

    return a.exec();
}
```
QMutexLocker是在QMutex基础上简化版的线程锁,QMutexLocker会保护加锁区域,并自动实现互斥量的锁定和解锁操作,可以将其理解为是智能版的QMutex锁,该锁只需要在上方代码中稍加修改即可.
```C
#include <QMutex>
#include <QMutexLocker>

static QMutex g_mutex;      // 线程锁
static QString g_store;     // 定义全局变量

class Producer : public QThread
{
protected:
    void run()
    {
        int count = 0;

        while(true)
        {
			// 增加智能线程锁
            QMutexLocker Locker(&g_mutex);

            g_store.append(QString::number((count++) % 10));
            std::cout << "Producer -> "<< g_store.toStdString() << std::endl;

            msleep(900);
        }
    }
};
```
互斥锁存在一个问题,每次只能有一个线程获得互斥量的权限,如果在程序中有多个线程来同时读取某个变量,那么使用互斥量必须排队,效率上会大打折扣,基于`QReadWriteLock`读写模式进行代码段锁定,即可解决互斥锁存在的问题.

**QReadWriteLock 读写同步线程锁:** 该锁允许用户以同步读`lockForRead()`或同步写`lockForWrite()`两种方式实现保护资源,但只要有一个线程在以写的方式操作资源,其他线程也会等待写入操作结束后才可继续读资源.
```C
#include <QCoreApplication>
#include <iostream>
#include <QThread>
#include <QMutex>
#include <QReadWriteLock>

static QReadWriteLock g_mutex;      // 线程锁
static QString g_store;             // 定义全局变量

class Producer : public QThread
{
protected:
    void run()
    {
        int count = 0;

        while(true)
        {
            // 以写入方式锁定资源
            g_mutex.lockForWrite();

            g_store.append(QString::number((count++) % 10));

            // 写入后解锁资源
            g_mutex.unlock();

            msleep(900);
        }
    }
};

class Customer : public QThread
{
protected:
    void run()
    {
        while( true )
        {
            // 以读取方式写入资源
            g_mutex.lockForRead();
            if( g_store != "" )
            {
                std::cout << "Curstomer -> "<< g_store.toStdString() << std::endl;
            }

            // 读取到后解锁资源
            g_mutex.unlock();
            msleep(1000);
        }
    }
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    Producer p1,p2;
    Customer c1,c2;

    p1.setObjectName("producer 1");
    p2.setObjectName("producer 2");

    c1.setObjectName("curstomer 1");
    c2.setObjectName("curstomer 2");

    p1.start();
    p2.start();

    c1.start();
    c2.start();

    return a.exec();
}
```

**QSemaphore 基于信号线程锁:** 信号量是特殊的线程锁,信号量允许N个线程同时访问临界资源,通过`acquire()`获取到指定资源,`release()`释放指定资源.
```C
#include <QCoreApplication>
#include <iostream>
#include <QThread>
#include <QSemaphore>

const int SIZE = 5;
unsigned char g_buff[SIZE] = {0};

QSemaphore g_sem_free(SIZE); // 5个可生产资源
QSemaphore g_sem_used(0);    // 0个可消费资源

// 生产者生产产品
class Producer : public QThread
{
protected:
    void run()
    {
        while( true )
        {
            int value = qrand() % 256;

            // 若无法获得可生产资源，阻塞在这里
            g_sem_free.acquire();

            for(int i=0; i<SIZE; i++)
            {
                if( !g_buff[i] )
                {
                    g_buff[i] = value;
                    std::cout << objectName().toStdString() << " --> " << value << std::endl;
                    break;
                }
            }

            // 可消费资源数+1
            g_sem_used.release();

            sleep(2);
        }
    }
};

// 消费者消费产品
class Customer : public QThread
{
protected:
    void run()
    {
        while( true )
        {
            // 若无法获得可消费资源，阻塞在这里
            g_sem_used.acquire();

            for(int i=0; i<SIZE; i++)
            {
                if( g_buff[i] )
                {
                    int value = g_buff[i];

                    g_buff[i] = 0;
                    std::cout << objectName().toStdString() << " --> " << value << std::endl;
                    break;
                }
            }

            // 可生产资源数+1
            g_sem_free.release();

            sleep(1);
        }
    }
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    Producer p1;
    Customer c1;

    p1.setObjectName("producer");
    c1.setObjectName("curstomer");

    p1.start();
    c1.start();

    return a.exec();
}
```
