# 服务端

服务端设计步骤为：

1. 加载库
2. 创建套接字
3. 绑定IP地址
4. 监听
5. 接收连接
6. 接收数据
7. 发送数据
8. 关闭聊天使用的套接字
9. 关闭套接字，卸载库

## 监听

TCP通信要一对一连接上后再进行通信，于是有一个监听的操作，告诉操作系统收到连接请求则交给程序来判断是否接受连接

监听的函数为`listen`

参数比较少也很简单就直接挂代码：

```c++
// 4、监听
err = listen(sock, 
             10); // 等待连接队列的最大长度
if (SOCKET_ERROR == err) {
    cout << "listen error:" << WSAGetLastError() << endl;
    // 删除套接字
    closesocket(sock);
    // 卸载库
    WSACleanup();
    return 1;
}
else {
    cout << "listen Success!" << endl;
}
```

---

## 接受连接

在有客户端请求连接时要判断是否接收连接

接收连接的函数为`accept`

```c++
// 5、接受连接
sockaddr_in addrClient;
int addrClientSize = sizeof(addrClient);
SOCKET sockssr = accept(sock, (sockaddr*)&addrClient, &addrClientSize);
if (sockssr == INVALID_SOCKET) {
    cout << "accept error:" << WSAGetLastError() << endl;
    break;
}
else {
    cout << "Client ip:" << inet_ntoa(addrClient.sin_addr) << endl;
}
```

该函数会返回一个新的套接字，后续的接收、发送数据就是通过这个新的套接字来进行的

最先创建的套接字在TCP通信中只有监听和接收连接的作用

---

## 发送&接收数据

TCP通信由于是一对一连接后再进行通信，于是在发送和接收数据的时候就不需要传入目标的地址了，所以发送和接收函数也与之前不同，不过用法还是一毛一样的

```c++
// 6、接收数据
recvNum = recv(sockssr, recvBuf, sizeof(recvBuf), 0);
if (recvNum > 0) {
    cout << "Client say:" << recvBuf << endl;
}
else {
    cout << "receive error:" << WSAGetLastError() << endl;
    break;
}

// 7、发送数据
gets_s(sendBuf);
sendNum = send(sockssr, sendBuf, sizeof(sendBuf), 0);
if (sendNum == SOCKET_ERROR) {
    cout << "send error:" << WSAGetLastError() << endl;
    break;
}
```

> 具体的某些参数：
>
> ```c++
> char recvBuf[1024] = "";
> char sendBuf[1024] = "";
> int recvNum = 0;
> int sendNum = 0;
> ```

---

# 客户端

客户端设计步骤为：

1. 加载库
2. 创建套接字
3. 发出连接请求
4. 发送数据
5. 接收数据
6. 关闭套接字，卸载库

## 请求连接

客户端需要发送连接请求，具体就是用`connect`方法

```c++
// 发出连接请求
sockaddr_in addrserver;
addrserver.sin_family = AF_INET;
addrserver.sin_port = htons(12345);
addrserver.sin_addr.S_un.S_addr = inet_addr("192.168.3.233");
err = connect(sock, (sockaddr*)&addrserver, sizeof(addrserver));
if (err == SOCKET_ERROR) {
    cout << "connect error:" << WSAGetLastError() << endl;
    // 删除套接字
    closesocket(sock);
    // 卸载库
    WSACleanup();
    return 1;
}
else {
    cout << "connect Success!" << endl;
}
```

然后其他的操作差不多，改改就行