采用boost中的thread
### 线程管理
```cpp
#include <iostream>
#include <boost/thread.hpp>

void wait(int seconds) {
    boost::this_thread::sleep(boost::posix_time::seconds(seconds));
}

void thread() {
    for (int i = 0; i < 5; ++i) {
        wait(1);
        std::cout << i << std::endl;
    }
}
int main()
{
    boost::thread t(thread);//被创建时就执行函数
    t.join();//阻塞当前线程直到t线程运行结束
    system("pause");
    return 0;
}
```
一个特定的线程（thread）可以通过诸如变量t来访问，通过方法join()等待线程结束。但是即使t越界或者析构了，该线程也将继续执行。一开始线程绑定boost::thread类型的变量，它们的关系是有关联但不依赖。方法detach()允许boost::thread的变量从它对应的线程里分离，即变量不再代表有效线程，也无法调用join()之类的成员函数。

```cpp
#include <iostream>
#include <boost/thread.hpp>

void wait(int seconds) {
    boost::this_thread::sleep(boost::posix_time::seconds(seconds));
}

void thread() {
    try {
        for (int i = 0; i < 5; ++i) {
            wait(1);
            std::cout << i << std::endl;
        }
    }
    catch (boost::thread_interrupted&) {
        std::cout << "interrupted" << std::endl;
    }
}
int main()
{
    boost::thread t(thread);//被创建时就执行函数
    wait(3);
    //到达中断点才会中断线程，抛出boost::thread_interrupted的异常
    //如果不包含任何中断点，interrupt()不会起作用
    //Boost.Thread定义了一系列中断点，如sleep()函数
    t.interrupt();
    //终端结束后
    t.join();
    system("pause");
    return 0;
}
```

```cpp
int main()
{
    std::cout << boost::this_thread::get_id() << std::endl;
    std::cout << boost::thread::hardware_concurrency() << std::endl;//返回基于CPU数目或者CPU内核数目此刻在同时在物理机器上运行的线程数
    return 0;
}
```

### 同步
互斥锁，保证一次只有一个线程访问被锁的资源，直到unlock()
```cpp
#include <iostream>
#include <boost/thread.hpp>

void wait(int seconds) {
    boost::this_thread::sleep(boost::posix_time::seconds(seconds));
}

boost::mutex mutex;//互斥锁

int a = 0;

void thread() {
    wait(1);
    mutex.lock();
    std::cout << a << " " << boost::this_thread::get_id() << " " << std::endl;
    ++a;
    mutex.unlock();
} 

int main()
{
    for (int i = 0; i < 4; ++i) {
        boost::thread t(thread);
    }
    system("pause");
    std::cout << a << std::endl;
    return 0;
}
```
使用boost::lockguard，内部构造和析构函数分别自动调用lock()和unlock()，使用RAII机制
```cpp
#include <iostream>
#include <boost/thread.hpp>

void wait(int seconds) {
    boost::this_thread::sleep(boost::posix_time::seconds(seconds));
}

boost::mutex mutex;//互斥锁

int a = 0;

void thread() {
    wait(1);
    boost::lock_guard<boost::mutex> lock(mutex);
    std::cout << a << " " << boost::this_thread::get_id() << " " << std::endl;
    ++a;
} 

int main()
{
    for (int i = 0; i < 4; ++i) {
        boost::thread t(thread);
    }
    system("pause");
    std::cout << a << std::endl;
    return 0;
}
```
独占锁
```cpp
#include <iostream>
#include <boost/thread.hpp>

void wait(int seconds) {
    boost::this_thread::sleep(boost::posix_time::seconds(seconds));
}

boost::timed_mutex mutex;//互斥锁

int a = 0;

void thread() {
    wait(1);
    //独占锁
    //通过多个构造函数来提供不同的方式获得互斥体，这种方式调用了lock()方法直到获得互斥体
    //如果传入boost::try_to_lock类型的值，构造函数会调用try_lock()方法并立即返回，在获得互斥体之前不会被阻塞
    boost::unique_lock<boost::timed_mutex> lock(mutex, boost::try_to_lock);
    //owns_lock()可以检查是否可获得互斥体，不能返回false
    if (!lock.owns_lock()) {
        //等待一定时间以获得互斥体
        lock.timed_lock(boost::get_system_time() + boost::posix_time::seconds(1));
    }

    std::cout << a++ << " " << boost::this_thread::get_id() << " " << std::endl;
    
    //boost::unique_lock的析构函数会自动释放互斥量，也可以手动释放
    //release()解除unique_lock和互斥量之间的关联
    boost::timed_mutex* m = lock.release();
    //此时必须手动释放互斥量
    m->unlock();
} 

int main()
{
    for (int i = 0; i < 4; ++i) {
        boost::thread t(thread);
    }
    system("pause");
    std::cout << a << std::endl;
    return 0;
}
```
共享锁
只对某个资源读访问的时候使用
```cpp
//count操作可能数组越界
#include <iostream>
#include "boost/thread.hpp"
#include <vector>

void wait(int seconds) {
    boost::this_thread::sleep(boost::posix_time::seconds(seconds));
}

boost::shared_mutex mutex;
std::vector<int> random_numbers;

void fill() {
    std::srand(static_cast<unsigned>(std::time(0)));
    for (int i = 0; i < 4; ++i) {
        boost::unique_lock<boost::shared_mutex> lock(mutex);
        random_numbers.push_back(std::rand() % 10);
        lock.unlock();
        wait(1);
    }
}
void print() {
    for (int i = 0; i < 4; ++i) {
        wait(1);
        boost::shared_lock<boost::shared_mutex> lock(mutex);
        std::cout << "print: " << random_numbers.back() << std::endl;
    }
}
int sum = 0;
void count() {
    for (int i = 0; i < 4; ++i) {
        wait(1);
        boost::shared_lock<boost::shared_mutex> lock(mutex);
        sum += random_numbers.back();
    }
}

int main()
{
    boost::thread t1(fill);
    boost::thread t2(print);
    boost::thread t3(count);
    t1.join();
    t2.join();
    t3.join();
    std::cout << sum << std::endl;
    return 0;
}
```

```cpp
#include <iostream>
#include "boost/thread.hpp"
#include <vector>

boost::mutex mutex;
boost::condition_variable_any cond;
std::vector<int> random_numbers;

void fill() {
	std::srand(static_cast<unsigned int>(std::time(0)));
	for (int i = 0; i < 3; ++i) {
		boost::unique_lock<boost::mutex> lock(mutex);
		random_numbers.push_back(std::rand());
		cond.notify_all();//唤醒wait()线程
		cond.wait(mutex);//阻止当前执行线程，并将其添加到等待的线程列表中
	}
}
void print() {
	std::size_t next_size = 1;
	for (int i = 0; i < 3; ++i) {
		boost::unique_lock<boost::mutex> lock(mutex);
		while (random_numbers.size() != next_size) {
			cond.wait(mutex);
		}
		std::cout << random_numbers.back() << std::endl;
		++next_size;
		cond.notify_all();
	}
}
int main() {
	boost::thread t1(fill);
	boost::thread t2(print);
	t1.join();
	t2.join();
	return 0;
}
```
线程本地存储（TLS）是一个只能由一个线程访问的专门的存储区域。 TLS的变量可以被看作是一个只对某个特定线程而非整个程序可见的全局变量。
```cpp
#include <iostream>
#include "boost/thread.hpp"
#include <vector>

void init_number_generator() {
	static boost::thread_specific_ptr<bool> tls;//支队对应的线程可见可用
	if (!tls.get())//检查是否保存new到的地址
		tls.reset(new bool(false));
	if (!*tls) {
		*tls = true;
		std::srand(static_cast<unsigned>(std::time(0)));//各个线程都要初始化
	}
}
boost::mutex mutex;
void random_number_generator() {
	init_number_generator();
	int i = std::rand();
	boost::lock_guard<boost::mutex> lock(mutex);
	std::cout << i << std::endl;
}
int main() {
	boost::thread t[3];
	for (int i = 0; i < 3; ++i)
		t[i] = boost::thread(random_number_generator);
	for (int i = 0; i < 3; ++i)
		t[i].join();
	return 0;
}
```