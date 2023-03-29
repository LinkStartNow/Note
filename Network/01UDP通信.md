# 服务端

**注意：别忘记要导入依赖库**

```c++
// 导入依赖库
#pragma comment(lib, "Ws2_32.lib")
```

## 1. 加载库

加载库需要使用`WSAStartup`函数

这里因为需要传入一个`WORD`类型的数来表示版本号，于是要用`MAKEWORD`函数来创造

```c++
WORD word = MAKEWORD(2, // 低位
                     2); // 高位
```

最终会返回一个`int`值来表示是否加载成功

具体代码如下：

```c++
// 1、加载库
WORD word = MAKEWORD(2, // 低位
                     2); // 高位
WSADATA wsaData;
int err = WSAStartup(word, // word用来存储版本号
                     &wsaData);
if (err) {
    cout << "WSAStartup error:" << WSAGetLastError() << endl;
    return 1;
}
else {
    cout << "WSAStartup success" << endl;
}
```

> 其中的`WSAGetLastError`方法用来返回错误的编码

加载成功库后变量`wsaData`中就存储了相关信息，我们可以通过该变量来判断库的版本是否正确

```c++
// 判断一下库是不是2.2版本
if (LOBYTE(wsaData.wVersion) != 2 || HIBYTE(wsaData.wVersion) != 2) {
    cout << "version error" << endl;

    // 卸载库
    WSACleanup();
    return 1;
}
```

---

## 2. 创建套接字

创建套接字需要使用函数`socket`

我们想要创建基于`UDP`协议和`IPV4`的套接字，于是可以查找相关的帮助文档进行传参

```c++
// 2、创建套接字
SOCKET sock = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
if (sock == INVALID_SOCKET) {
    cout << "socket error:" << WSAGetLastError() << endl;

    // 卸载库
    WSACleanup();
    return 1;
}
else {
    cout << "socket success" << endl;
}
```

---

## 3. 绑定IP地址

> 这里绑定IP地址的目的是告诉操作系统，发送给（某个网卡+某个端口）的数据是当前进程要接收的数据，交由该程序处理
>
> 因为服务器端一般都是先收信息，所以不绑定的话，操作系统不知道要将哪些信息交由该服务器处理，也就是说程序运行的过程中将永远接收不到信息

绑定IP地址需要使用`bind`方法，查询了相关帮助文档后我们知道，需要传入三个参数：

1. 套接字：大概就是用来判断基于什么协议传输吧

	> 套接字之前已经创建过了，而且只有一个，于是这里无脑丢进去就行了

2. 一个比较复杂的结构体指针：包含了本地的信息，包括端口、网卡（差不多也就是IP地址）等

	> 这个比较复杂，后面细说

3. 传入那个复杂结构体指针的大小：也不知道啥用，直接用`sizeof`计算一下就行了

### 某个复杂的结构体

我们选择一种比较好赋值的这个结构体的替代品来进行赋值，之后要记得强转就行

它的类型是`sockaddr_in`

```c++
sockaddr_in addrServer;
```

先要对该变量赋值之前的`IPV4`家族（姑且先这么称呼）

```c++
addrServer.sin_family = AF_INET;
```

其次，配置端口，这里使用了`htons`方法，可以把数转换为网络字节序（也就是大端存储）

```c++
addrServer.sin_port = htons(12345); // 转换成网络字节序（大端存储）
```

然后，配置网卡

> 一台电脑有很多网卡，我们这里不用写死，所以可以写成任意网卡，让操作系统自动判断，只要有网卡接收到信息通通算作它的

```c++
addrServer.sin_addr.S_un.S_addr = INADDR_ANY; // 绑定任意网卡（关注所有网卡接收到的数据）
```

最后愉快绑定就可以了：

```c++
	// 3、绑定IP地址（告诉操作系统，发给某个网卡+某个端口的数据是当前进程要接收的数据）
	// 192.168.230--十进制四等分字符串类型的ip地址
	// ulong类型的ip地址
	sockaddr_in addrServer;
	addrServer.sin_family = AF_INET;
	addrServer.sin_port = htons(12345); // 转换成网络字节序（大端存储）
	addrServer.sin_addr.S_un.S_addr = INADDR_ANY; // 绑定任意网卡（关注所有网卡接收到的数据）
	//addrServer.sin_addr.S_un.S_addr = inet_addr("192.168.3.233"); // 把字符串类型的IP地址转换成ulong类型的IP地址
	
	err = bind(sock,  (sockaddr*)&addrServer, sizeof(addrServer));
	if (err == SOCKET_ERROR) {
		cout << "bind error:" << WSAGetLastError() << endl;

		// 关闭套接字
		closesocket(sock);

		// 卸载库
		WSACleanup();
		return 1;
	}
	else {
		cout << "bind success" << endl;
	}
```

---

## 4. 接收数据

接收数据要使用`recvfrom`函数

```c++
// 4、接收数据
nRecvNum = recvfrom(sock, // 套接字
                    recvBuf, // 缓冲区用来存储数据
                    sizeof(recvBuf), // 缓冲区大小
                    0, // 判断标记
                    (sockaddr*)&addrClient, // 跟前面一样的复杂的结构体
                    &addrClientSize);
if (nRecvNum > 0) {
    cout << "client ip:" << inet_ntoa(addrClient.sin_addr) << " buf:" << recvBuf << endl;
}
else {
    cout << "recv error:" << WSAGetLastError() << endl;
    break;
}
```

这里客户端那个指针的数据就不用配置了，因为是接收，接收完直接就会把改指针的值给赋值了

也就是接收完后，指针里存的是发送方的信息

> 这里后面又用了`inet_ntoa`是将前面`ulong`类型的ip地址转换成我们平时见到的十进制4等份IP地址，方便我们观察

---

## 5. 发送数据

发送数据使用的方法是`sendto`，参数和接收数据的几乎一样，只不过这里的指针是要有值的，不然不知道你是发给谁的

```c++
// 5、发送数据
gets_s(sendBuf);
nSendNum = sendto(sock, sendBuf, sizeof(sendBuf), 0, (sockaddr*)&addrClient, addrClientSize);
if (SOCKET_ERROR == nSendNum) {
    cout << "sendto error:" << WSAGetLastError() << endl;
    break;
}
```

---

## 6. 关闭套接字&卸载库

```c++
// 关闭套接字
closesocket(sock);

// 卸载库
WSACleanup()
```

---

# 客户端

客户端设计步骤为：

1. 加载库
2. 创建套接字
3. 发送数据
4. 接收数据
5. 关闭套接字，卸载库

## 绑定IP地址

客户端首先是发信息的，于是可以**不用绑定IP地址**而直接发送，这样发送信息的时候操作系统会自动选择合适的IP分配给它

但由于要发送给对应的服务端，所以必须要配置服务端的IP地址，对应的函数为`inet_addr`

，该方法用来将我们平常用的十进制四等分字符转换成`ulong`类型的IP地址，用于赋值

```c++
addrServer.sin_addr.S_un.S_addr = inet_addr("10.56.22.62"); // 绑定指定的IP地址
```

**注意：这个IP地址的点千万别写成逗号了，之前莫名出bug好久才发现的**

## 广播

广播可以采用直接广播和有限广播

### 直接广播

>  直接广播就是对某个指定网段内发出广播

我们都知道某个网段的最后主机号全为1是广播的IP地址，于是就可以以此来确定要广播的网段

使用起来也很方便，就是把绑定的目标发送IP地址设定为该网段的广播IP地址即可

例如：

```c++
// 直接广播
addrServer.sin_addr.S_un.S_addr = inet_addr("192.168.3.255");
```

### 有限广播

> 有限广播就是对当前主机处于的网段发出广播

有限广播的IP地址为全1，但是要申请广播权限

**注意：这里传入的bool类型的变量一定要赋值为`true`，之前瞎赋值吃过苦头，找了半天bug**

```c++
// 有限广播
addrServer.sin_addr.S_un.S_addr = inet_addr("255, 255, 255, 255");

//申请广播权限
bool val = true;
err = setsockopt(sock, SOL_SOCKET, SO_BROADCAST, (char*)&val, sizeof(val));
if (err == SOCKET_ERROR) {
    cout << "setsockopt error:" << WSAGetLastError() << endl;
    return 1;
}
else {
    cout << "setsockopt success" << endl;
}
```

----

# 阻塞&非阻塞

> 阻塞：必须等待一件事完成之后才能去做另一件事
>
> 非阻塞：不一定要一直等待某一件事有没有完成，可以一边做其他事

socket默认是阻塞的，我们可以将socket设置为非阻塞模式来实现接收的非阻塞

```c++
// 把套接字设置成非阻塞
u_long iMode = 1;
ioctlsocket(sock, FIONBIO, &iMode); 
```

## 缓冲区

缓冲区指操作系统给每个进程分配4G虚拟空间，0~2G用户控件，2G~4G内核控件

我们可以用`getsockopt`来获取缓冲区大小

```c++
int nRecvBufSize = 0;
int nSendBufSize = 0;
int n = sizeof(int);
getsockopt(sock, SOL_SOCKET, 
           SO_RCVBUF, 
           (char*)&nRecvBufSize, 
           &n);
getsockopt(sock, SOL_SOCKET,
           SO_SNDBUF,
           (char*)&nSendBufSize,
           &n);
cout << "nRecvBufSize = " << nRecvBufSize << ", nSendBufSize:" << nSendBufSize << endl;
```

前面的发送和接收方法实际上都是对缓冲区的复制

> `sendto`阻塞：当发送缓冲区控件不够的时候，等待发送缓冲区空间足够大以后再发送
>
> `sendto`非阻塞：当发送缓冲区空间不够的时候，当前缓冲区有多大空间，就往缓冲区里面拷贝多少数据，剩下的数据不拷贝

---

# UDP相关

`UDP`是数据报协议，当需要拷贝的数据大于目前有的空间时，则会只传输对应空间的数据，丢弃剩余的数据

## UDP特点：

1. 面向非连接，接收数据的时候，可以接受任何客户端发送的数据，可以使一对一，也可以是一对多（组播、广播）
2. 数据报文的通讯方式，数据包不可拆分
3. 传输效率高（与TCP对比）
4. 会产生丢包，没有校验，可能会出现乱序（也就是后发的信息先被收到）