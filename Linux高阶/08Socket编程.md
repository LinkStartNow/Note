# 网络信息结构体

包含了一些网络信息，可以用于初始化socket

```c
struct sockaddr_in addr;
```

该结构体有以下属性：

- ```c
	adddr.sin_family = AF_INET | AFINET6;
	```

	其中`AF_INET`对应IPV4，另一个对应IPV6

- ```c
	addr.sin_port = 8080;
	```

	写入指定的端口

- ```c
	addr.sin_addr.s_addr = ip;
	```

	写入IP地址

# 大小端转换函数

## 大小端

- 小端：低位存低地址，高位存高地址
- 大端：相反，低位存高地址，高位存低地址

一般网络字节序用的是大端存储，而主机字节序用的是小端存储

小端数据不可以直接大端使用，强制转换存储格式会导致失效，我们可以使用一些函数来进行转换

## 函数

- `htons`：本机端口转网络端口

	```c
	server_addr.sin_port = htons(8080);
	```

	> 绑定端口为8080

- `htonl`：本机IP转网络IP

- `ntohs`：网络端口转本机端口

- `ntohl`：网络IP转本机IP

这些转换都是通过数字完成的，会非常的不好记，也不好用，我们通常用以下的三个函数来替代：

- `inet_addr()`：转大端IP

- ```c
	inet_pton(AF_INET, char* ip, void* p);
	```

	字符串IP转为大端序IP

- ```c
	inet_ntop(AF_INET, void* ip, char* ip, int size);
	```

	大端IP转字符串IP

# socket相关函数

## 创建

Linux下的socket一般都是整形

```c
int sockfd = socket(AF_INET, SOCK_STREAM | SOCK_DGRAM, 0);
```

第二个参数是说明用的是TCP还是UDP，TCP则用STREAM，UDP则用DGRAM

第三个参数一般填0就好了

函数成功会返回socket， 失败则返回-1

## 绑定

```c
bind(int sockfd, struct sockaddr* addr, sizeof(addr));
```

网络信息结构体的内容会绑定在该socket上

成功返回0，失败返回-1

## 监听

```c
listen(sockfd, backlog);
```

参数2一般填128即可，也就是最长等待连接序列

该函数也就TCP用，因为UDP不需要连接

## 连接

```c
int ret = connect(sockfd, struct sockaddr* destaddr, sizeof(destaddr));
```

参数2填的是目标的网络信息结构体，**而不是自己的**

## 接收连接

```c
int newsockfd = accept(mysockfd, struct sockaddr* addr, socklen_t* addrlen);
```

阻塞函数，等待连接，连接成功则返回后续用来与之通信的socket，**服务端自己原先的那个socket只用于连接，而不能用来通信**

参数2并不需要自己写，只需要传入一个未初始化的对象即可，该函数最后会把来连接的客户端的信息存储在这里，也就是说这是一个**输出参数**

最后一个参数要传入一个`int*`类型的长度，也就是先求出长度，然后再取地址

## 发送信息

```c
send(sockfd, char* buf, ssize_t size, int flag);
```

发送数据，成功返回发送的数据量，失败返回-1

这里的size传的是发送字符串的长度，也就是`strlen(buf)`

## 接收信息

```c
recv(sockfd, char* buf, ssize_t size, int flag);
```

成功返回读取的数据量，失败返回-1，如果返回0则表示连接退出

这里的size是接收缓冲区的大小，也就是`sizeof(buf)`

