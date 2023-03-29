# 创建线程

这里用到了新的创建线程的函数`_beginthreadex`，这个函数几乎可以完全替代之前的`CreateThread`函数，并且提高了一定的安全性，原因如下：

>  这里创建线程就不用之前的`CreateThread`了，因为`CreateThread`和`ExitThread`是一对，如果线程中使用到C++运行时库的函数（例如`strcpy`），会申请空间，然后自己不回收，同时`ExitThread`在结束线程的时候也不会回收这个空间
>
> ` _beginthreadex`和`_endthreadex`是一对，`_endthreadex`会在结束线程的时候先回收这个空间，再调用`ExitThread`退出线程

该函数要包含头文件`#include <process.h>`

然后调用的参数几乎和原先的`CreateThread`方法一样

**注意：最后要把返回值强转成`HANDLE`类型**

```c++
m_handle = (HANDLE)::_beginthreadex(nullptr, 0, &Fun, (void*)this, 0, nullptr);
```

## 线程函数

**这里要注意的是：线程函数要调用类中受保护的方法，于是要把线程函数声明成类的成员函数**

同时，又因为是线程函数，所以传入的参数有限制，如果是普通成员函数，则还需要传入`this`指针，于是这里要设置成***静态成员函数***

```c++
class UdpNet : public INet
{
	//friend unsigned __stdcall Fun(void* lp);
public:
	UdpNet(UdpNetMediator*);
	~UdpNet();
	bool InitNet();
	bool SendData(char* buf, int nlen, long lSend);
	void UnInitNet();

protected:
	SOCKET m_sock;
	HANDLE m_handle;
	bool IsQuit;

protected:
	void RecvData();
	static unsigned __stdcall Fun(void*); // 线程函数
};
```

---

# 接收数据

这里由于接收来的数据要传给中介者，然后再传给上面进行处理，如果只用一个`buf`来存储的话，就有可能因为后续接收到的新数据覆盖掉前面还未来得及处理的数据

于是我们就需要每次`new`一个新的字符串缓冲区用来存储，这样数据就不会覆盖了，在后续处理完后再`delete`即可，也不会造成内存泄漏

**注意：这里的拷贝最好用按内存拷贝，这样能精准的把接收到的数据拷贝到新的缓冲区中**

```c++
void UdpNet::RecvData()
{
	sockaddr_in addr;
	char buf[4096] = "";
	int size = sizeof(addr);
	while (!IsQuit) {
		int num = recvfrom(m_sock, buf, sizeof(buf), 0, (sockaddr*)&addr, &size);
		if (num > 0) {
			// 创建一个新的缓冲区用来保存数据
			char* tmp = new char[num];

			// 按内存拷贝
			memcpy(tmp,	 // 拷贝目标
				buf,  // 源
				num); // 拷贝大小

		 // 把新的数据交给中介者处理
			m_pMed->DealData(tmp, num, addr.sin_addr.S_un.S_addr);
		}
		else {
			cout << "UdpNet::RecvData error:" << WSAGetLastError() << endl;
		}
	}
}
```

---

# QT应用界面制作

## 某些需要注意的点：

### 1. 部分细节：

 1.  创建的窗口类型是`QDialog`

 2.  显示窗口尽量用`showNormal`

	该函数和普通的`show`函数的差别主要在于，普通的`show`如果窗口被最小化了也算作被显示，于是仍然按照最小化显示，用户是看不到的；而`showNormal`可以把窗口恢复到正常大小，即使被最小化了，也可以回复到原先大小

### 2.最后的Kernel类不要忘记写析构函数

```c++
Kernel::~Kernel()
{
    qDebug() << __func__;
    if (m_pChat) {
        m_pChat->hide();
        delete m_pChat;
        m_pChat = nullptr;
    }

    if (m_pMed) {
        m_pMed->CloseNet();
        delete m_pMed;
        m_pMed = nullptr;
    }
}
```

---

### 3. 导入库

使用的网络相关的函数需要导入库，在QT中导入库可以这么操作

先进入`.pro`文件内，然后添加一行代码为`LIBS += -lWs2_32`

结果如下：

```c++
#-------------------------------------------------
#
# Project created by QtCreator 2023-03-21T18:31:03
#
#-------------------------------------------------

QT       += core gui

greaterThan(QT_MAJOR_VERSION, 4): QT += widgets

TARGET = FeiQ
TEMPLATE = app

LIBS += -lWs2_32

INCLUDEPATH += ./net
INCLUDEPATH += ./mediator

SOURCES += main.cpp\
        mychatdialog.cpp \
    mediator/UdpNetMediator.cpp \
    net/UdpNet.cpp \
    kernel.cpp \
    mediator/INetMediator.cpp

HEADERS  += mychatdialog.h \
    mediator/INetMediator.h \
    mediator/UdpNetMediator.h \
    net/Config.h \
    net/INet.h \
    net/UdpNet.h \
    kernel.h

FORMS    += mychatdialog.ui
```

---

## 信号与槽

### 使用条件

***类继承自`QObject`，并且类里面有`Q_OBJECT`***

### 使用步骤：

1. 在发送数据的类中定义信号（用signals声明）

	```c++
	class UdpNetMediator : public INetMediator
	{
	    Q_OBJECT
	public:
		UdpNetMediator();
		~UdpNetMediator();
		bool OpenNet();
	    bool SendData(char* buf, int nlen, long lSend);
		void CloseNet();
		void DealData(char* buf, int nlen, long lSend);
	signals:
	    void SIG_ReadyData(char* buf, int nlen, long lSend);
	};
	```

2. 在需要发送数据的地方用 `Q_EMIT`发送信号

	```c++
	void UdpNetMediator::DealData(char* buf, int nlen, long lSend)
	{
	    qDebug() << buf;
	    Q_EMIT SIG_ReadyData(buf, nlen, lSend);
	}
	```

3. 在接受数据的类里定义槽函数（用public slots声明）

	```c++
	class Kernel : public QObject
	{
	    Q_OBJECT
	public:
	    explicit Kernel(QObject *parent = 0);
	    ~Kernel();
	
	public:
	    MyChatDialog* m_pChat;
	    INetMediator* m_pMed;
	
	public:
	    void DealOnlineRq(char *buf, int nlen, long lSend);
	    void DealOnlineRs(char *buf, int nlen, long lSend);
	    void DealChatRq(char *buf, int nlen, long lSend);
	    void DealOfflineRq(char *buf, int nlen, long lSend);
	
	signals:
	
	public slots:
	    void SLOT_ReadyData(char* buf, int nlen, long lSend);
	};
	```

4. 实现槽函数

	```c++
	void Kernel::SLOT_ReadyData(char *buf, int nlen, long lSend)
	{
	    qDebug() << __func__;
	    int type = *(int*)buf;
	    switch (type) {
	    case _UDP_PROTOCOL_ONLINE_RQ:
	        qDebug() << ((STRU_UDP_ONLINE*)buf)->name;
	        DealOnlineRq(buf, nlen, lSend);
	        break;
	    case _UDP_PROTOCOL_ONLINE_RS:
	        DealOnlineRs(buf, nlen, lSend);
	        break;
	    case _UDP_PROTOCOL_CHAT_RQ:
	        DealChatRq(buf, nlen, lSend);
	        break;
	    case _UDP_PROTOCOL_OFFLINE_RQ:
	        DealOfflineRq(buf, nlen, lSend);
	        break;
	    default:
	        break;
	    }
	}
	```

5. 连接信号和槽函数（在接受数据的类里面，发送信号的对象new出来以后）

	```c++
	Kernel::Kernel(QObject *parent) : QObject(parent)
	{
	    // new一个好友界面对象
	    m_pChat = new MyChatDialog;
	    m_pChat->show();
	
	    // new一个中介者对象
	    m_pMed = new UdpNetMediator;
	
	
	    // 打开网络
	    if (!m_pMed->OpenNet()) {
	        QMessageBox::about(m_pChat, "提示", "打开网络失败！");
	        exit(0); // 退出
	    }
	
	    QMetaObject::Connection x = connect(m_pMed, SIGNAL(SIG_ReadyData(char*,int,long)),
	                                        this, SLOT(SLOT_ReadyData(char*,int,long)));
	
	    if (!x) {
	        qDebug() << "connect error";
	    }
	    else {
	        qDebug() << "connect Success!";
	    }
	
	    //
	    STRU_UDP_ONLINE rq;
	    gethostname(rq.name, sizeof(rq.name));
	    //    memcpy(rq.name, "ORZ.", sizeof("ORZ."));
	    m_pMed->SendData((char*)&rq, sizeof(rq), inet_addr("255.255.255.255"));
	}
	```

---

## 信息处理

这里传播的信息直接被打包成了结构体，协议头也定义成了其中的属性

```c++
#pragma once

#define UDP_PORT (666666)

// 名称长度
#define _UDP_NAME_MAX (100)

// 聊天内容大小
#define _UDP_CONTENT_SIZE (1024)

// 协议头基准
#define _UDP_PROTOCOL_BASE (1000)

// 协议头
// 上线请求
#define _UDP_PROTOCOL_ONLINE_RQ (_UDP_PROTOCOL_BASE + 1)

// 上线回复
#define _UDP_PROTOCOL_ONLINE_RS (_UDP_PROTOCOL_BASE + 2)

// 聊天请求
#define _UDP_PROTOCOL_CHAT_RQ (_UDP_PROTOCOL_BASE + 3)

// 下线请求
#define _UDP_PROTOCOL_OFFLINE_RQ (_UDP_PROTOCOL_BASE + 4)



// 请求结构体
// 上线请求：协议头、昵称
// 上线回复：协议头、昵称
struct STRU_UDP_ONLINE
{
    STRU_UDP_ONLINE() : type(_UDP_PROTOCOL_ONLINE_RQ)
    {
        memset(name, 0, _UDP_NAME_MAX);
    }

    int type;
    char name[_UDP_NAME_MAX];
};

// 聊天请求：协议头、聊天内容
struct STRU_UDP_CHAT_RQ
{
    STRU_UDP_CHAT_RQ() : type(_UDP_PROTOCOL_CHAT_RQ)
    {
        memset(content, 0, _UDP_CONTENT_SIZE);
    }

    int type;
    char content[_UDP_CONTENT_SIZE];
};

// 下线请求：协议头
struct STRU_UDP_OFFLINE_RQ
{
    STRU_UDP_OFFLINE_RQ() : type(_UDP_PROTOCOL_OFFLINE_RQ)
    {

    }

    int type;
};
```

---

## 字符串相关

我们目前接触了`QString`、`std::string`(也就是C++中的标准字符串类)和`char*`

`QString`和`std::string`都是封装的类，`QString`是`QT`中的类，`std::string`是纯C++的类

`char*`和`char[]`是一样的，是原始数据类型

他们之间的转换关系为：

1. `char*`为基础类型，所以可以无条件转换为另外两个类

	`QString = char*;`、`std::string = char*;`

2. `std::string`可以通过自身的`c_str`方法转换为`char*`

	`std::string.c_str() => char*`

3. `QString`可以通过`toStdString`转化为`std::string`

	`QString.toStdString() => std::string`

### IP地址转换为`QString`

在该项目中我们需要将IP地址转换为`QString`来显示

因为参数中的IP地址为`ulong`类型的，于是我们可以使用`inet_ntoa`将它转换为我们能看懂的4段char*类型的IP地址

该函数要传入一个`in_addr`类型的参数，于是我们借助结构体`sockaddr_in`来传参

具体代码如下：

```c++
static string GetIpString(long ip)
{
    sockaddr_in addr;
    addr.sin_addr.S_un.S_addr = ip;
    return inet_ntoa(addr.sin_addr);
}
```

---

## 获取一些信息

1. 本地主机的名字

	可以使用函数`gethostname`，通过一个`char`数组即可接收

	```c++
	STRU_UDP_ONLINE rq;
	gethostname(rq.name, sizeof(rq.name)); // 获取主机名
	```

2. 获取本地的IP

	可以使用`gethostbyname`，会返回一个结构体指针

	```c++
	hostent* remoteHost = gethostbyname(name);
	int i = 0;
	while (remoteHost->h_addr_list[i] != 0) {
	    s.insert(*(u_long *) remoteHost->h_addr_list[i++]);
	}
	```


---

# 将QT最终项目发布

## 1. 从Debug模式切换到Release模式

![image-20230327203342579](C:\Users\MAX\AppData\Roaming\Typora\typora-user-images\image-20230327203342579.png)

## 2.构建项目

构建完成后会生成一个`.exe`文件，这是最终要运行的文件，我们可以根据编译输出的路径寻找到它

![image-20230327203609387](C:\Users\MAX\AppData\Roaming\Typora\typora-user-images\image-20230327203609387.png)

## 3.生成相应的动态链接库

直接运行可执行文件会显示缺少库文件，我们只需要执行一个简单的指令即可实现把所有需要用到的库文件直接装进来

这里我们要用到命令框来操作这里我们有两种方式便捷的进入对应的路径：

1. （最为简单，但不一定能用）

	可以利用搜索找到![image-20230327214329627](C:\Users\MAX\AppData\Roaming\Typora\typora-user-images\image-20230327214329627.png)

	直接打开即可

2. 从QT的安装路径一路摸索过去（通用，但略麻烦）
	在本机中的路径是这样的：`C:\Qt\Qt5.6.2\5.6\mingw49_32`

	进入到最后的这个文件夹中即可，然后从当前目录打开cmd就行了

打开命令行之后就很简单了，只需要一句命令：

`windeployqt` + `目的的可执行文件的绝对路径`

例如：`windeployqt F:\A_University\CPlusPlus\QT\FeiQ\build-FeiQ-Desktop_Qt_5_6_2_MinGW_32bit-Release\release\FeiQ.exe`

这里获得绝对路径可以用一种很简单的方法，那就是直接把exe文件拖进命令行

## 4. 最终发布

最终只需要把`release`文件夹改个名整个打包走就行了，使用的时候打开`.exe`文件就可以直接用了
