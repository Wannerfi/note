## emm，看看就好，认识一下坑，注意别踩，下面采用boost库的方式更好
[参考网站](http://c.biancheng.net/cpp/html/3029.html)
这种方法不能跨平台，函数inet_addr会报错
```cpp
#include <iostream>
#include <string>
#include <fstream>
#include <winsock2.h>

#pragma comment(lib, "Ws2_32.lib")

#pragma warning (disable: 4996)

using namespace std;

/*
 * 传输方式
 * SOCK_STREAM 可靠传输，有校对
 * SOCK_DGRAM 不可靠传输，无校对
 */
//服务端
int main() {
    //init DLL
    WSADATA wsaData;
    WSAStartup(MAKEWORD(2, 3), &wsaData);

    //创建套接字句柄
    SOCKET servSock = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);

    //绑定套接字
    sockaddr_in sockAddr;
    memset(&sockAddr, 0, sizeof(sockAddr));
    sockAddr.sin_family = PF_INET;//IPv4
    sockAddr.sin_addr.s_addr = inet_addr("127.0.0.1");//具体IP地址
    sockAddr.sin_port = htons(12345);//port
	//套接字句柄，套接字
    bind(servSock, (SOCKADDR*)&sockAddr, sizeof(SOCKADDR));

    //监听，套接字， 请求队列的最大长度
    listen(servSock, 20);

    //接收客户端请求
    SOCKADDR clntAddr;
    int nSize = sizeof(SOCKADDR);
    SOCKET clntSock = accept(servSock, (SOCKADDR*)&clntAddr, &nSize);

    //向客户端发送数据
    const char* str = "Hello World!";
    send(clntSock, str, strlen(str) + sizeof(char), NULL);

    //关闭套接字
    closesocket(clntSock);
    closesocket(servSock);

    //终止DLL的使用
    WSACleanup();
    return 0;
}
```
```cpp
// ConsoleApplication2.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//

#include <iostream>
#include <stdlib.h>
#include <WinSock2.h>

#pragma comment(lib, "ws2_32.lib")
#pragma warning (disable: 4996)

using namespace std;

//客户端
int main()
{
    //init dll
    WSADATA wsaData;
    WSAStartup(MAKEWORD(2, 2), &wsaData);

    //build socket
    SOCKET sock = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);

    //发起请求
    sockaddr_in sockAddr;
    memset(&sockAddr, 0, sizeof(sockAddr));
    sockAddr.sin_family = PF_INET;
    sockAddr.sin_addr.s_addr = inet_addr("127.0.0.1");
    sockAddr.sin_port = htons(12345);
    connect(sock, (SOCKADDR*)&sockAddr, sizeof(SOCKADDR));

    //接收服务区传回的数据
    char szBuffer[MAXBYTE] = { 0 };
    recv(sock, szBuffer, MAXBYTE, NULL);

    cout << szBuffer << endl;

    closesocket(sock);
    WSACleanup();
    return 0;
}
```

## 采用boost库
[参考Boost.Asio C++ 网络编程](https://mmoaay.gitbooks.io/boost-asio-cpp-network-programming-chinese/content/)

### 1 Boost.Asio入门
**同步和异步**
在同步编程中，每个操作都是阻塞的，为了不影响主线程，同步的服务端/客户端通常是多线程的。
异步编程是事件驱动的。启动一个操作后，它只是提供一个回调，当操作结束时，会调用这个API，返回操作结果。所以只需要一个线程。
```cpp
//基础同步客户端
using namespace boost::asio;
io_service service;
ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
ip::tcp::socket sock(service);
sock.connect(ep);
//建立连接后即可通信
```
Boost.Asio使用io_service同操作系统的IO进行交互。然后创建连接的地址和端口，再建立socket进行连接。  

```cpp
//简单服务端
#define DATALEN 512
typedef boost::shared_ptr<ip::tcp::socket> socket_ptr;
void client_session(socket_ptr sock) {
	while (true) {
		char data[DATALEN];
		size_t len = sock->read_some(buffer(data));
		if (len) write(*sock, buffer("ok", 2));
	}
}
int main()
{
	io_service service;
	ip::tcp::endpoint ep(ip::tcp::v4(), 2001);//listen on 2001
	ip::tcp::acceptor acc(service, ep);
	while (true) {
		socket_ptr sock(new ip::tcp::socket(service));
		acc.accept(*sock);
		boost::thread(boost::bind(client_session, sock));
	}
	return 0;
}
```
需要至少一个io_service实例，然后指定想监听的端口，再创建一个接收客户端连接对象的接收器。在循环中创建socket来等待客户端的连接。当连接被建立时，创建一个线程来处理这个连接。  

```cpp
void connect_handler(const boost::system::error_code& ec) {
	std::cout << "success" << std::endl;
}
int main()
{
	//异步客户端
	using namespace boost::asio;
	io_service service;
	ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
	ip::tcp::socket sock(service);
	sock.async_connect(ep, connect_handler);
	service.run();
	return 0;
}
```
需要至少一个io_service，指定连接的地址以及创建socket。连接完成时，即异步连接到指定的地址和端口，会调用connect_handler。
connect_handler被调用时会检查错误代码ec，成功就可以向服务端进行异步写入
```cpp
#include <boost/lexical_cast.hpp>     
#include <iostream>   
#include <vector>
#include <boost/asio.hpp>

void connect_handler(const boost::system::error_code& ec) {
	//连接被拒绝是应为目标服务器上无对应监听套接字
	//https://blog.csdn.net/test1280/article/details/80642847
    //完成服务端代码即可连接成功
	std::cout << ec << " " << ec.message() << std::endl;
}
int main()
{
	//异步客户端
	using namespace boost::asio;
	io_service service;
	ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 12500);
	ip::tcp::socket sock(service);
	sock.async_connect(ep, connect_handler);
	service.run();
	return 0;
}
```
基本的异步服务端
```cpp
#include <boost/lexical_cast.hpp>     
#include <iostream>   
#include <boost/asio.hpp>
#include <boost/thread.hpp>
#include <Windows.h>

using namespace std;
using namespace boost::asio;
typedef boost::shared_ptr<ip::tcp::socket> socket_ptr;
io_service service;
ip::tcp::endpoint ep(ip::tcp::v4(), 2001);//监听端口2001
ip::tcp::acceptor acc(service, ep);

void handle_accept(socket_ptr sock, const boost::system::error_code& err);
void start_accept() {
	socket_ptr sock(new ip::tcp::socket(service));
	acc.async_accept(*sock, boost::bind(handle_accept, sock, std::placeholders::_1));
}

//只要该函数的压栈不在start_accept上面，就不会死循环
void handle_accept(socket_ptr sock, const boost::system::error_code& err) {
	if (err) return;
	//开始socket读写
	std::cout << "halder_accept" << " " << sock->remote_endpoint().address() 
								 << " " << sock->remote_endpoint().port() << "   ";
	start_accept();//再次启动异步
}

int main() {
	start_accept();
	service.run();
	return 0;
}
```
**异常处理和错误代码**
```cpp
int main()
{
	using namespace boost::asio;
	io_service service;
	ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
	ip::tcp::socket sock(service);
	sock.connect(ep);//抛出错误
	boost::system::error_code err;
	sock.connect(ep, err);//返回错误码
	service.run();
	return 0;
}
```
```cpp
	try {
		sock.connect(ep);
	}
	catch (boost::system::system_error e){
		std::cout << e.code() << std::endl;
	}
	//同上
	boost::system::error_code err;
	sock.connect(ep, err);
	if (err) std::cout << err << std::endl;
```
**Boost.Asio中的线程**
- io_service是线程安全的，可以多个线程同时调用io_service::run()。当一个线程调用run()，所有回调都会被调用，并且阻塞其他io_service::run()的线程
- socket类不是线程安全的

**io_service类**
io_service负责等待所有异步操作结束，然后为每个异步操作调用其完成处理程序
- 一个io_service处理多个线程
```cpp
void connect_handler(const boost::system::error_code& ec) {
	//连接被拒绝是应为目标服务器上无对应监听套接字
	//https://blog.csdn.net/test1280/article/details/80642847
	std::cout << ec << " " << ec.message() << std::endl;
}
int main()
{
	using namespace boost::asio;
	io_service service;
	ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
	ip::tcp::socket sock1(service), sock2(service);
	sock1.async_connect(ep, connect_handler);
	sock2.async_connect(ep, connect_handler);
	for (int i = 0; i < 3; ++i) {
		boost::thread([&service] {
			service.run();
		});
	};
	system("pause");
	return 0;
}
```
- 多个io_service处理多线程
```cpp
#include <boost/lexical_cast.hpp>     
#include <iostream>   
#include <vector>
#include <boost/asio.hpp>
#include <boost/thread.hpp>

void connect_handler(const boost::system::error_code& ec) {
	//连接被拒绝是应为目标服务器上无对应监听套接字
	//https://blog.csdn.net/test1280/article/details/80642847
	std::cout << ec << " " << ec.message() << std::endl;
}
int main()
{
	using namespace boost::asio;
	io_service service[2];
	ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
	ip::tcp::socket sock1(service[0]), sock2(service[1]);
	sock1.async_connect(ep, connect_handler);
	sock2.async_connect(ep, connect_handler);
	for (int i = 0; i < 2; ++i) {
		io_service& s = service[i];
		boost::thread([&s] {
			s.run();
			});
		//t.join();
	};
	system("pause");
	return 0;
}
```
### 2 Boost.Asio基本原理
**Boost.Asio命名空间**
- boost::asio 核心类和函数所在。io_service, streambuf, read, read_at, read_until
- boost::asio::ip 网络通信部分所在。address, endpoint, tcp, udp, icmp, connect, async_connect
- boost::asio::error 调用I/O历程时返回的错误码
- boost::asio::ssl SSL处理类的命名空间
- boost::asio::local 包含POSIX特性的类
- boost::asio::windows 包含Windows特性的类

**IP地址**
- ip::address(v4_or_v6_address) 把v4_v6地址转换成ip::address
- ip::address::from_string(str) str为IP地址
- ip::address::to_string() 看名字
- ip::address_v4::broadcast([addr, mask]) 创建一个广播地址
- ip::address_v4::any() 返回能表示任何地址的地址
- ip::address_v4::loopback(), ip::address_v6::loopback() 返回环路地址
- ip::host_name() 返回当前主机的stirng类型

```cpp
ip::address addr = ip::address::from_string("127.0.0.1");
```
**端点**
端点有ip::tcp::endpoint，ip::udp::endpoint,ip::icmp::endpoint
```cpp
//连接本机80端口
ip::tcp::endpoint ep( ip::address::from_string("127.0.0.1"), 80);

//endpoint() 有时用来创建UDP/ICMP socket
ip::tcp::endpoint ep1;
//endpoint(protocol, port) 创建接受新连接的服务器端socket
ip::tcp::endpoint ep2(ip::tcp::v4(), 80);
//endpoint(addr, port) 创建连接到某个地址和端口
ip::tcp::endpoint ep3(ip::address:from_string("127.0.0.1"), 80);

//通过域名查询端点
io_service service;
ip::tcp::resolver resolver(service);
ip::tcp::resolver::query que("www.XXX.com", "80");
ip::tcp::resolver::iterator iter = resolver.resolve(que);
ip::tcp::endpoint ep = *iter;

ep.address().to_string();
```
**套接字**
socket总在构造的时候传入io_service实例
```cpp
io_service service;
ip::udp::socket sock(service);
```
**同步错误码**
所有的同步函数都有抛出异常或者返回错误码的重载
**socket成员方法**
- 连接相关
- 1. open(protocol) 用给定IP协议打开socket
- 2. bind(endpoint) 绑定一个地址
- 3. connect(endpoint) 用同步的方式连接一个地址
- 4. async_connect(endpoint) 用异步的方式连接一个地址
- 5. is_open() 如果套接字已经打开则返回true
- 6. close() 关闭套接字。调用时关闭套接字上任何异步操作，返回error::operation_aborted错误
- 7. shutdown(type_of_shutdown) 使send和receive操作失效
- 8. cancel() 结束套接字上所有的异步操作，返回error::operation_aborted错误

```cpp
	ip::tcp::socket sock(service);
	sock.open(ip::tcp::v4());
	sock.close();
```
- 读写函数
- 1. async_receive(buffer, [flags,] handler) 从套接字异步接收数据
- 2. async_read_some(buffer,handler) 同async_receive
- 3. async_receive_from(buffer, endpoint[, flags], handler) 从指定端点异步接收数据
- 4. async_write_some(buffer, handler) 同async_receive_from
- 5. async_send_to(buffer, endpoint, handler) 异步send缓冲区数据到指定端点
- 6. receive(buffer [, flags]) 异步读取缓冲区数据。完成前阻塞
- 7. read_some(buffer) 同receive
- 8. send(buffer [, flags]) 同步发送缓冲区数据。完成前阻塞
- 9. write_some(buffer) 同send
- 10. send_to(buffer, endpoint [, flags]) 同步发送缓冲区数据。完成前阻塞
- 11. available() 无阻塞读取函数返回的数据

标记，默认为0
- ip::socket_type::socket::message_peek 检测并返回消息
- ip::socket_type::socket::message_out_of_band 处理带外（OOB）数据
- ip::socket_type::socket::message_do_not_route 指定数据不使用路由表发送
- ip::socket_type::socket::message_end_of_record 指定的数据表示了记录的结束。不支持Windows

```cpp
using namespace boost::asio;
int main() {
	//tcp同步读写
	io_service service;
	ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
	ip::tcp::socket sock(service);
	sock.connect(ep);
	sock.write_some(buffer("Get I am client"));
	char buff[1024];
	size_t read = sock.read_some(buffer(buff));
	std::cout << buff << std::endl;
	return 0;
}
```
```cpp
#include <boost/asio.hpp>
#include <boost/thread.hpp>

using namespace boost::asio;

int main() {
	//udp同步读写
	io_service service;
	ip::udp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
	ip::udp::socket sock(service);
	sock.open(ip::udp::v4());
	sock.send_to(buffer("I am client"), ep);
	char buff[1024];
	size_t read = sock.receive(buffer(buff));
	std::cout << buff << std::endl;
	return 0;
}
```
- 套接字控制
- 1. get_io_service() 返回构造时的io_service实例
- 2. get_option(option) 返回一个套接字的属性
- 3. set_option(option) 设置一个套接字的属性
- 4. io_control(cmd) 在套接字上执行一个IO指令

opetion `broadcast, debug, do_not_route, enable_connection_aborted, keep_alive, linger, receive_buffer_size, receive_low_watemark, reuse_address, send_buffer_size, send_low_watermark, ip::v6_only`
```cpp
ip::tcp::endpoint ep( ip::address::from_string("127.0.0.1"), 2001);
ip::tcp::socket sock(service);
sock.connect(ep);
// 重用地址
ip::tcp::socket::reuse_address ra(true);
sock.set_option(ra);
```
**套接字实例不能被拷贝**
**套接字缓冲区**
缓冲区buffer必须在receive和send时都存在。
Boost.Asio会给完成处理句柄保留一个拷贝，当操作完成时会调用这个完成处理句柄，所以可以用共享指针来处理缓冲区
**缓冲区封装函数buffer()**
buffer()参数类型 
`const char[], void *, std::string, const POD[], vector<pod>, boost::array, std::array<pod>`
**异步编程**
~~这本书关于多线程同步编程的编写思路和数据冲突有一定误导，这种情况加锁就能解决。同步编程的弊端在于阻塞，这对于高并发服务端是致命的~~
本书这部分代码编写有问题，异步编程参照前面第一章
**异步run(), run_one(), poll(), poll_one()**
先给出服务端和客户端应答的例子
```cpp
//同步客户端
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <boost/asio.hpp>
#include <boost/thread.hpp>
#include <Windows.h>

using namespace boost::asio;

char buff[256];
void sendData(ip::tcp::socket& sock) {
	sock.send(buffer("I am Client 2"));
};
void reciveData(ip::tcp::socket& sock) {
	sock.receive(buffer(buff));
	std::cout << buff << std::endl;
};
bool isConnected(const boost::system::error_code& err) {
	if (err) {
		std::cout << err << std::endl;
		return false;
	}
	return true;
}

int main() {
	io_service service;
	ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
	ip::tcp::socket sock(service);
	boost::system::error_code err;
	sock.connect(ep, err);
	if (!isConnected(err)) return 0;
	sendData(sock);
	reciveData(sock);
	sock.close();
	return 0;
}
```
```cpp
//异步服务端
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <boost/asio.hpp>
#include <boost/thread.hpp>
#include <Windows.h>

using namespace std;
using namespace boost::asio;
#define DATALEN 524

typedef boost::shared_ptr<ip::tcp::socket> socket_ptr;
io_service service;
ip::tcp::endpoint ep(ip::tcp::v4(), 2001);
ip::tcp::acceptor acc(service, ep);
char readBuff[DATALEN];

void acceptHande(socket_ptr, const boost::system::error_code&);

void start_accept() {
	socket_ptr sock(new ip::tcp::socket(service));
	acc.async_accept(*sock, boost::bind(acceptHande, sock, std::placeholders::_1));
}
void acceptHande(socket_ptr sock, const boost::system::error_code& err) {
	if (err) {
		std::cout << err << std::endl;
		return;
	}
	std::cout << sock->remote_endpoint().address() << " " << sock->remote_endpoint().port() << std::endl;
	if (sock->available()) {
		sock->read_some(buffer(readBuff));
		std::cout << readBuff << std::endl;
		sock->write_some(buffer("Hi, I am Servive"));
	}
	start_accept();
}

int main() {
	start_accept();
	service.run();
	return 0;
}
```
run_one()方法最多执行和分发一个异步操作。实验时将服务端的run()改为run_one()即可
poll_one()方法以非阻塞的方式最多运行一个准备好的等待操作，若没有等待的操作，立刻返回0，否则运行它并返回1
操作正在等待并准备以非阻塞方式运行，通常有如下情况
- 计时器过期了，并且有async_wait待处理
- IO操作完成了，需要调用handler
- 之前被加入io_services实例队列中的自定义handler
```cpp
io_service service;
while ( true) {
    // 运行所有完成了IO操作的handler
    while ( service.poll_one()) ;
    // ... 在这里做其他的事情 …
}

//同上
io_service service;
service.poll();
```
**异步工作**
使用service.post()异步调用自定义方法
```cpp
io_service service;
void func(int i) {
	std::cout << "func called, i= " << i << std::endl;
}

int main(int argc, char* argv[]) {
	for (int i = 0; i < 10; ++i)
		service.post(boost::bind(func, i));
	boost::thread threads([&] {service.run();});
	threads.join();
	return 0;
}
```
io_service::strand让异步方法按顺序调用
```cpp
io_service service;
void func(int i) {
	std::cout << "func called, i= " << i << " " << std::this_thread::get_id() << std::endl;
}

int main(int argc, char* argv[]) {
	io_service::strand strand_one(service), strand_two(service);
	for (int i = 0; i < 5; ++i)
		service.post(strand_one.wrap(boost::bind(func, i)));
	for (int i = 5; i < 10; ++i)
		service.post(strand_two.wrap(boost::bind(func, i)));
	boost::thread threads([&] {service.run(); });
	threads.join();
	return 0;
}
```
**回显服务端/客户端**
回显应用就是一个把客户端发过来的任何内容回显给其本身，然后关闭连接的的服务端。（这个我好像上一章实现了，接着看看吧）
**TCP同步客户端**
规定每个消息都以换行符结束。书中这个样例用拆包的方式去接收读到的信息，同步客户端在上文已展示，这里小改一下，使用拆包的方式去实现一遍
```cpp
using namespace boost::asio;
using namespace std;

//结束返回0，非0就继续读
size_t read_complete(char* buff, const boost::system::error_code& err, size_t bytes) {
	if (err) return 0;
	return std::find(buff, buff + bytes, '\n') >= buff + bytes;
}
void sendData(ip::tcp::socket& sock) {
	sock.send(buffer("I am Client 2"));
};
void reciveData(ip::tcp::socket& sock) {
	char buff[1024];
	size_t bytes = read(sock, buffer(buff), boost::bind(read_complete, buff, std::placeholders::_1, std::placeholders::_2));
	std::cout << string(buff, buff + bytes) << "$$" << std::endl;
};
bool isConnected(const boost::system::error_code& err) {
	if (err) {
		std::cout << err << std::endl;
		return false;
	}
	return true;
}

int main() {
	io_service service;
	ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
	ip::tcp::socket sock(service);
	boost::system::error_code err;
	sock.connect(ep, err);
	if (!isConnected(err)) return 0;
	sendData(sock);
	reciveData(sock);
	sock.close();
	return 0;
}
```
**TCP同步服务端**
同步无须调用service.run()，会阻塞
```cpp
using namespace std;
using namespace boost::asio;
#define DATALEN 524

int main() {
	io_service service;
	ip::tcp::endpoint ep(ip::tcp::v4(), 2001);
	ip::tcp::acceptor acc(service, ep);
	char buff[DATALEN];
	while (true) {
		boost::system::error_code err;
		ip::tcp::socket sock(service);
		acc.accept(sock);
		if (err) {
			std::cout << err << " " << err.message() << std::endl;
		}
		if (sock.available()) {
			sock.read_some(buffer(buff));
			std::cout << buff << std::endl;
			sock.write_some(buffer(buff));
		}
		sock.close();
	}
	return 0;
}
```
**TCP异步客户端**
这里实现一下发送10次消息的异步客户端。书里实现的离谱在于用多线程共用一个buffer，访问同一份内存还不加锁。
```cpp
#include <boost/lexical_cast.hpp>     
#include <iostream>   
#include <boost/asio.hpp>
#include <boost/thread.hpp>
#include <Windows.h>

using namespace std;
using namespace boost::asio;
typedef boost::shared_ptr<ip::tcp::socket> socket_ptr;
io_service service;
ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
int step = 0;

void handle_accept(socket_ptr sock, const boost::system::error_code& err);
void start_accept() {
	socket_ptr sock(new ip::tcp::socket(service));
	sock->async_connect(ep, boost::bind(handle_accept, sock, std::placeholders::_1));
}

void handle_accept(socket_ptr sock, const boost::system::error_code& err) {
	if (err) {
		std::cout << err << err.message() << std::endl;
		return;
	}
	//开始socket读写
	string s = "I am Client " + to_string(step);
	++step;
	sock->write_some(buffer(s));
	sock->read_some(buffer(s));
	std::cout << "recive  " << s << "  %%" << std::endl;
	boost::this_thread::sleep(boost::posix_time::millisec(1000));
	sock->close();
	start_accept();//再次启动异步
}

int main() {
	start_accept();
	int times = 10;
	while (times--) {
		service.run_one();
	}

	return 0;
}
```
**TCP异步服务端**
这一部分在上文已经实现过了，可以做的优化是把方法写进类里面，再用多线程调用类。值得注意的是使用智能指针以保证socket的正确释放。
**UDP回显服务/客户端**
```cpp
//UDP客户端
using namespace std;
using namespace boost::asio;
io_service service;
int step = 0;
void run() {
	ip::udp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
	ip::udp::endpoint myEp(ip::address::from_string("127.0.0.2"), 2002);
	ip::udp::socket sock(service, myEp);
	boost::system::error_code err;
	std::string s = "I am Client " + to_string(step);
	sock.send_to(buffer(s, s.size()), ep);
	//recive
	char buff[1024];
	ip::udp::endpoint sender_ep;
	if (sock.available()) {
		size_t bytes = sock.receive_from(buffer(buff), sender_ep);
		std::cout << string(buff, buff + bytes) << "  $$" << std::endl;
	}
	boost::this_thread::sleep(boost::posix_time::millisec(10));
	sock.close();
	++step;
}
int main() {
	int times = 5;
	while (times--)
		run();
	return 0;
}
```
```cpp
//UDP服务端
using namespace std;
using namespace boost::asio;
#define DATALEN 1024
io_service service;

int main() {
	ip::udp::socket sock(service, ip::udp::endpoint(ip::udp::v4(), 2001));
	while (true) {
		char buff[DATALEN];
		ip::udp::endpoint sender_ep;
		if (!sock.available()) continue;
		size_t bytes = sock.receive_from(buffer(buff), sender_ep);
		std::cout << sender_ep.address() << " " << sender_ep.port();
		std::cout << " " << string(buff, buff + bytes) << std::endl;
		std::string msg(buff, bytes);
		sock.send_to(buffer("I am Service " + msg), sender_ep);
	}
	return 0;
}
```
### 客户端和服务端
这一章节给了一个场景并实现，emm，由于我已经不信任这本书的代码逻辑会好，甚至书中的代码只提供了片段，所以这章略过
### 同步和异步
在前面代码的实现中，出现过在异步中使用同步的方法，这会使异步线程处于挂起的状态。
这一章提出基本的服务端推送模型。客户端定时ping服务端（轮询），维持一个长连接。服务端收到消息后根据对应的ID及其对应的通道进行消息下发。
这一部分实现，只要将前面异步客户端的发送最后一行加延时即可实现，当然，最好将其中的同步方法替换成异步的。
不变的是这章还是只展示了代码片段。这章说到底还是同步，异步，多线程套着说，提出奇奇怪怪的组合，而且跟前面一个通病，就是多线程使用同一段buff，导致加锁操作也变得复杂起来。这种问题在多个实例类上解决轻而易举。
**代理实现**
代理一般在客户端和服务端之间，接收客户端的请求，可能对请求进行修改，然后把请求发送到服务端。对于服务端的结果类推。
这里只实现了客户端-代理-服务端
```cpp
#include <iostream>
#include <boost/asio.hpp>
#include <boost/thread.hpp>
#include <boost/lexical_cast.hpp>
#include <boost/thread.hpp>

using namespace std;
using namespace boost::asio;
class Proxy {
	typedef boost::shared_ptr<ip::tcp::socket> socket_ptr;
#define DATALEN 1024
public:
	//面向客户端IO，面向服务器IO(同步，无须run)，代理端点，服务器端点
	explicit Proxy(io_service& clientService, io_service& serviceService, ip::tcp::endpoint& myEP, ip::tcp::endpoint& serviceEP) :
		clientService(&clientService), serviceService(&serviceService), acc(clientService, myEP), serviceEP(&serviceEP) {

		startAccept();
	};
private:
	bool isConnect(const boost::system::error_code& err) {
		if (err) {
			std::cout << err << std::endl;
			return false;
		}
		return true;
	}
	void startAccept() {
		socket_ptr clientSock(new ip::tcp::socket(*clientService));
		acc.async_accept(*clientSock, //作为服务器接收客户端
			boost::bind(&Proxy::acceptHandle, this, std::placeholders::_1, clientSock));
	}
	void acceptHandle(const boost::system::error_code& err, socket_ptr clientSock) {
		if (!isConnect(err)) return;

		std::cout << clientSock->remote_endpoint().address() << " " << clientSock->remote_endpoint().port() << "  ";
		if (clientSock->available()) {
			clientSock->async_read_some(buffer(clientBuff, DATALEN),
				boost::bind(&Proxy::readClientHandle, this, std::placeholders::_1, std::placeholders::_2));
		}
		startAccept();
	}
	void readClientHandle(const boost::system::error_code& err, const size_t bytes) {
		if (!isConnect(err)) return;

		clientMutex.lock(); //保护客户数据直到被转发
		std::cout << __FUNCTION__ << "  " << string(clientBuff, bytes) << std::endl;

		//发送数据
		startSend();
	}
	void startSend() {
		socket_ptr serviceSock(new ip::tcp::socket(*serviceService));
		serviceSock->async_connect(*serviceEP, //作为客户端连接服务器
			boost::bind(&Proxy::sendHandle, this, std::placeholders::_1, serviceSock));
	}
	void sendHandle(const boost::system::error_code& err, socket_ptr serviceSock) {
		if (!isConnect(err)) return;
		serviceSock->send(buffer(clientBuff, buffLen));
	}

	boost::mutex clientMutex; //保护客户端数据
	boost::mutex serviceMutex; //保护服务器数据
	char clientBuff[DATALEN];
	size_t buffLen = 0;
	io_service* clientService; //面向客户端
	io_service* serviceService; //面向服务端
	ip::tcp::endpoint* serviceEP; //服务器端点
	ip::tcp::acceptor acc;
};

int main() {
	io_service clientService, serviceService;
	ip::tcp::endpoint myEP(ip::tcp::v4(), 2001);
	ip::tcp::endpoint serviceEP(ip::address::from_string("127.0.0.1"), 2002);
	Proxy p(clientService, serviceService, myEP, serviceEP);
	clientService.run();
	return 0;
}
```