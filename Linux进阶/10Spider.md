# 随便唠唠

网络爬虫又名网络蜘蛛，是一种可以自动化，拓扑采集网络资源的软件程序，爬虫更像一款web应用，爬虫会模拟浏览器的行为访问web服务器，并请求下载资源，使用的也是万维网相关技术

## 浏览器与服务器交互的简易流程

浏览器先从域名解析服务器DNS处通过*url*获取IP地址以及端口。然后通过该IP找到目的的服务器与其Tcp连接

> http的端口号为80，https的端口号为443，https是加密后的http，现在比较通用，更加安全
>
> url：统一资源定位符，全球唯一，表示网络资源的唯一地址，通过该地址可以访问或请求该资源

## 网络资源类型

网络资源一般分为以下几种：

- 文本资源：txt、html、xml
- 二进制资源：jpg、png、bmp、gif
- 音频：mp3
- 视频：rmvb、flv、ovi、mp4

这些都是爬虫可以获取的资源

## 爬虫工作领域

1. 搜索引擎技术（爬虫系统、资料系统、索引系统、分词系统）
2. 数据分析、数据挖掘、数据可视化
3. 爬虫只负责数据采集和准备阶段，后续如何使用数据与爬虫无关

## 爬虫工作流程

假如我们有一个图片的URL：`http://www.pic.com/tmp/cs/20230803/01.jpg`

我们会先对其解析，获取以下信息：

- domain（域名）：www.pic.com
- 文件存储路径：/tmp/cs/20230803/
- 文件名：01.jpg
- IP地址：202.13.6.78，**注意：这个是通过域名在dns处解析获得的**
- 协议类型：http

然后与该IP对应的服务器进行连接

向该服务器发送*请求头*以及*请求体*，其中，请求有两种方式：

1. GET请求：带请求体
2. POST请求：不带请求体

如果我们请求成功的话，将会收到服务器发回的响应头和响应体

一般响应头大小为4k，于是我们会一开始接收8K的内容，其中会包含全部响应头和分隔符`\r\n\r\n`以及部分响应体

然后查找一下空行判断是否获取成功，若没有空行则说明下载失败

若有空行则保存响应头并提取转换响应码，判断响应码是否为200

若响应头也正确，则创建存储文件，将首次读取的部分响应体存储，再循环存储剩余内容

### 请求头格式

**注意：前三个（请求方式、URL地址、http版本）同一行，以空格分隔，剩下的独自一行**

- 请求方式：GET
- 请求的URL地址
- HTTP协议版本：HTTP/1.1

- Accept可以接收的数据类型以及优先级
- User_Agent：浏览器版本信息及兼容性
- Host：服务器域名
- Connection：连接方式（短连接与长连接）

### 响应头格式

- 响应码，响应码比较常见的有这些：
	- 200：OK，响应成功
	- 404：Not Found：网络资源失效
	- 301 302：*资源重定向*
	- 501 502：服务器异常
- 响应大小：return_size
- 服务器版本：apache，Nginx等
- 服务器响应时间
- 服务器缓存位置

其中响应头和响应体通过`\r\n\r\n`分隔，响应体内一般都是请求资源的数据（二进制、文本、视音频等）

**注意：如果响应码非200表示响应失败，响应体为空**

> 资源重定向：请求的URL资源位置发生变更，服务器反馈301，并且反馈新的URL，用户通过新的URL可以访问到资源

# 获取图片url

本实践以下载http图片为例，假如某图的url如下：`https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fc-ssl.duitang.com%2Fuploads%2Fitem%2F202003%2F20%2F20200320134221_uxuek.thumb.1000_0.gif&refer=http%3A%2F%2Fc-ssl.duitang.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1693882827&t=bec7f09c8afc8babd5a3bd7865f57803`

由于是https开头的，我们不能直接解析下载，但是能看得出该图片的原始路径，也就是从`src`指出的路径到`gif`：

`http%3A%2F%2Fc-ssl.duitang.com%2Fuploads%2Fitem%2F202003%2F20%2F20200320134221_uxuek.thumb.1000_0.gif`

然后我们将其中的`%3A`替换成`:`，将`%2F`替换成`/`，得到该图片的原始路径：

`http://c-ssl.duitang.com/uploads/item/202003/20/20200320134221_uxuek.thumb.1000_0.gif`

# 解析url

解析这个还是比较简单的，就一步步分析，取出域名，存储路径，文件名，就行了，唯一的新知识就是需要通过新函数用域名获取IP

```c
// 获取ip
struct hostent* ent;
if ((ent = gethostbyname(node->domain)) == NULL) {
    perror("gethostbyname fail");
    return -1;
}
inet_ntop(AF_INET, ent->h_addr_list[0], node->ip, 16);
```

完整的解析代码：

```c
#include <spider.h>

// https://www.baidu.com

int spider_analytical_url(url_t* node) {
	bzero(node->domain, 4096);
	bzero(node->savepath, 1024);
	bzero(node->file, 1024);
	bzero(node->ip, 16);

	char* head[] = {"http://", "https://", NULL};
	int start;
	int size;
	int flag;
	int i;

	// 判定协议类型
	if (strncmp(node->origin, head[0], strlen(head[0])) == 0) {
		start = strlen(head[0]);
		node->http_type = 0;
		node->port = 80;
	}
	else {
		start = strlen(head[1]);
		node->http_type = 1;
		node->port = 443;
	}

	// 取域名
	i = 0;
	for (flag = start; node->origin[flag] != '\0' && node->origin[flag] != '/'; ++flag) {
		node->domain[i] = node->origin[flag];
		++i;
	}

	// 文件名长度
	size = 0;
	for (flag = strlen(node->origin) - 1; node->origin[flag] != '\0' && node->origin[flag] != '/'; --flag, ++size);

	// 取文件名
	i = 0;
	for (flag = strlen(node->origin) - size; node->origin[flag] != '\0'; ++flag) {
		node->file[i] = node ->origin[flag];
		++i;
	}

	// 取路径
	i = 0;
	 for (flag = start + strlen(node->domain); flag < strlen(node->origin) - size; ++flag) {
		 node->savepath[i] = node ->origin[flag];
		 ++i;
	 }

	 // 获取ip
	 struct hostent* ent;
	 if ((ent = gethostbyname(node->domain)) == NULL) {
		 perror("gethostbyname fail");
		 return -1;
	 }
	 inet_ntop(AF_INET, ent->h_addr_list[0], node->ip, 16);
	 printf("url anlay success!\n");
	 printf("domain: %s\npath: %s\nfile: %s\nip: %s\n", node->domain, node->savepath, node->file, node->ip);
	 return 0;
}
```

# 网络初始化&连接

与web服务器交互也需要与之连接，也是采用TCP连接

## 初始化：

```c
#include <spider.h>

int spider_netinit() {
	int sockfd;
	struct sockaddr_in addr;
	bzero(&addr, sizeof(addr));
	addr.sin_family = AF_INET;
	addr.sin_port = htons(8080);
	addr.sin_addr.s_addr = htonl(INADDR_ANY);
	if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
		perror("net init fail");
		return -1;
	}
	printf("net init success\n");
	return sockfd;
}
```

## 连接

```c
#include <spider.h>

int spider_connection(int mysock, url_t node) {
	struct sockaddr_in webaddr;
	bzero(&webaddr, sizeof(webaddr));
	webaddr.sin_family = AF_INET;
	webaddr.sin_port = htons(node.port);
	inet_pton(AF_INET, node.ip, &webaddr.sin_addr.s_addr);

	if ((connect(mysock, (struct sockaddr*)&webaddr, sizeof(webaddr))) == 0) {
		printf("connect success\n");
	}
	else {
		perror("perror fail");
		return -1;
	}
	return 0;
}
```

## 生成请求头

```c
#include <spider.h>

int spider_create_request(char* request, url_t node) {
	bzero(request, 4096);
	sprintf(request, "GET %s HTTP/1.1\r\n"\
					"Accept:text/html,application/shtml+xml;q=0.9,image/webp;q=0.8\r\n"\
					"User-Agent:Mozilla/5.0 (X11; Linux x86_64)\r\n"\
					"Host:%s\r\n"\
					"Connection:close\r\n\r\n", node.origin, node.domain);
	printf("Create Request success:\n%s\n", request);
	return 0;
}
```

## 下载函数

### 查找空行

我们像发送普通的Tcp信息一样，发送给web服务器的sock，然后接收传回的数据

为了接收请求头的完整性，我们使用了8K空间大的缓冲区进行读取，一般请求头只有4K

所以8K会包含部分的请求体，也就是说，用来分隔请求头和请求体的空行也会被包括在内，于是我们先判定有没有空行，若不存在空行则说明传回的内容有误

```c
send(webfd, request, strlen(request), 0);
printf("send request success\n");
if ((len = recv(webfd, buffer, sizeof(buffer), 0)) == -1) {
    perror("download recv call fail");
    return -1;
}
if ((pos = strstr(buffer, "\r\n\r\n")) == NULL) {
    printf("download find \\r\\n\\r\\n failed\n");
    return -1;
}
snprintf(response, pos - buffer + 5, "%s", buffer);
printf("get response \n%s", response);
```

> 这里我们调用了strstr函数，该函数会返回子串在主串中第一次出现的位置，若返回了空指针说明没有找到，也就说明响应内容有误
>
> 后面又通过snprintf函数将响应头写入了response中，该函数的后两位就是正常printf的参数，第一个是要写入的缓冲区，第二个是写入长度
>
> **注意：这个写入长度把`\0`也算上了，所以要多算一位才行，也就说长度为2才能写入一个字符（另一个为\0）**

### 写入内容

这里写入内容先写了第一次读的8K中的部分响应体的内容，然后再循环读取剩余部分，直到没有内容

```c
if ((status = spider_get_status(response)) == 200) {
    fd = open(node.file, O_RDWR | O_CREAT, 0664);
    // 部分响应体写入文件
    write(fd, pos + 4, len - (pos - buffer + 4));
    bzero(buffer, sizeof(buffer));
    while ((len = recv(webfd, buffer, sizeof(buffer), 0)) > 0) {
        write(fd, buffer, len);
        bzero(buffer, sizeof(buffer));
    }
    close(fd);
    printf("status code %d download success\n", status);
    return 0;
}
else {
    printf("status code %d download failed\n", status);
    return -1;
}
```

### 完整版

```c
#include <spider.h>

int spider_download(int webfd, char* request, url_t node) {
	int status;
	char buffer[8192];
	char response[4096];
	bzero(buffer, sizeof(buffer));
	bzero(response, sizeof(response));
	int len;
	char* pos = NULL; // 查找空行
	int fd;

	// 发送请求
	send(webfd, request, strlen(request), 0);
	printf("send request success\n");
	if ((len = recv(webfd, buffer, sizeof(buffer), 0)) == -1) {
		perror("download recv call fail");
		return -1;
	}
	if ((pos = strstr(buffer, "\r\n\r\n")) == NULL) {
		printf("download find \\r\\n\\r\\n failed\n");
		return -1;
	}
	snprintf(response, pos - buffer + 5, "%s", buffer);
	printf("get response \n%s", response);
	if ((status = spider_get_status(response)) == 200) {
		fd = open(node.file, O_RDWR | O_CREAT, 0664);
		// 部分响应体写入文件
		write(fd, pos + 4, len - (pos - buffer + 4));
		bzero(buffer, sizeof(buffer));
		while ((len = recv(webfd, buffer, sizeof(buffer), 0)) > 0) {
			write(fd, buffer, len);
			bzero(buffer, sizeof(buffer));
		}
		close(fd);
		printf("status code %d download success\n", status);
		return 0;
	}
	else {
		printf("status code %d download failed\n", status);
		return -1;
	}
	return 0;
}
```

## 获取响应头里的状态信息

一般的响应头如下：

`HTTP/1.1 200 OK\r\n
Content-Type: text/html; charset=UTF-8\r\n
Content-Length: 1234\r\n
\r\n`

以`\r\n`为一行的结束

所以判断的时候可以用正则表达式更方便的判断

```c
#include <spider.h>

int spider_get_status(const char* response) {
	regex_t status;
	int code;
	char strcode[10];
	bzero(strcode, 10);
	char* reg_str = "HTTP/1.1 \\([^\r\n]\\+\\?\\)\r\n";
	regmatch_t match[2];
	regcomp(&status, reg_str, 0);
	if ((regexec(&status, response, 2, match, 0)) == 0) {
		snprintf(strcode, match[1].rm_eo - match[1].rm_so + 1, "%s", response + match[1].rm_so);
		printf("strcode = %s\n", strcode);
		sscanf(strcode, "%d", &code);
		return code;
	}
	else {
		printf("spider_get_status failed");
		return -1;
	}
}
```

**注意：这里最后获得的strcode其实是`200 OK`**，然后通过sscanf把这个字符串的数字提取了出来

原理就是和scanf读入差不多，只不过scanf读入的数据源在stdin，而这里的数据源是这个字符串

# https下的下载

## 某些理论

如果目标的协议是https的，那么客户端在与服务器连接后需要完成认证，有两种认证方式：

1. 单向认证：只有浏览器认证网站是否真实
2. 双向认证：在单向认证的基础上，网站也需要认证浏览器是否真实

在认证完毕后，客户端可以与服务端进行正常交互，进行会话

### 非对称与对称密钥

这些密钥都是通过RSA加密算法生成的

一般公钥是加密方会发送给对方的，自己留一份私钥。对方发来的信息会通过公钥加密，然后自己接收到后通过私钥解密就行了

**非对称加密**的精度较高，但不适合加密大段报文，适合加密小段数据

**对称加密**方式，适合加密大段数据报，但是安全性，加密精度略低

一般公钥都会包含CA证书和随机码，CA证书用于证明安全性，随机码用来加密

### https单向认证过程

1. **客户端**向**服务端**发送SSL版本信息，上下文等（发起认证）
2. **服务端**向**客户端**发送RSA非对称公钥及SSL版本信息等
3. **客户端**通过CA证书，校验**服务端**的合法性，如果异常则会抛出警告或错误
4. **客户端**向**服务端**发送本机支持的对称加密算法列表（无需加密）
5. **服务端**选择一种，并明文发送给**客户端**
6. **客户端**生成对称加密密钥，向**服务端**发送对称加密密钥（使用RSA非对称公钥加密）
7. **服务端**得到对称密钥后，使用RSA非对称私钥解密
8. 验证成功，正常通信

## SSL技术

### 安装库与文档

首先要去安装相关的库和文档：

```bash
sudo apt-get install libssl-dev
sudo apt-get install libssl-doc
```

### 包含头文件

要调用这些函数必须包含两个头文件：

```c
#include <openssl/ssl.h>
#include <openssl/err.h>
```

### 编译库文件

```bash
gcc -lssl -lcrypto
```

### 打开SSL库

这里几乎就是个模板的内容：
```c
#include <spider.h>

ssl_t* spider_OpenSSL(int webfd) {
	ssl_t* ssl = NULL;
	if ((ssl = (ssl_t*)malloc(sizeof(ssl_t))) == NULL) {
		perror("spider openssl malloc fail");
		return NULL;
	}
	// 初始化openssl错误处理函数
	SSL_load_error_strings();
	// 初始化openssl库
	SSL_library_init();
	// 初始化算法
	OpenSSL_add_ssl_algorithms();

	// 生成上下文信息
	ssl->sslctx = SSL_CTX_new(SSLv23_method());
	// 使用上下文信息创建安全套接字
	ssl->sslsock = SSL_new(ssl->sslctx);

	// 使用tcpsock设置sslsocket
	SSL_set_fd(ssl->sslsock, webfd);

	// 开始认证
	SSL_connect(ssl->sslsock);

	// 后续通信都用这个ssl
	return ssl;
}
```

其中生成上下文信息的函数可以通过查看手册来看可以传入哪些函数

这里的`ssl_t`结构体是自己定义的：

```c
typedef struct {
	SSL* sslsock;
	SSL_CTX* sslctx;
} ssl_t;
```

### 读写函数

要与https的服务端交互的话要调用特定的函数，而且需要通过安全套接字进行对话

```c
// 发送请求
SSL_write(ssl->sslsock, request, strlen(request));
printf("send request success\n");
if ((len = SSL_read(ssl->sslsock, buffer, sizeof(buffer))) == -1) {
    perror("download recv call fail");
    return -1;
}
if ((pos = strstr(buffer, "\r\n\r\n")) == NULL) {
    printf("download find \\r\\n\\r\\n failed\n");
    return -1;
}
snprintf(response, pos - buffer + 5, "%s", buffer);
printf("get response \n%s", response);
if ((status = spider_get_status(response)) == 200) {
    fd = open(node.file, O_RDWR | O_CREAT, 0664);
    // 部分响应体写入文件
    write(fd, pos + 4, len - (pos - buffer + 4));
    bzero(buffer, sizeof(buffer));
    while ((len = SSL_read(ssl->sslsock, buffer, sizeof(buffer))) > 0) {
        write(fd, buffer, len);
        bzero(buffer, sizeof(buffer));
    }
    close(fd);
    close(webfd);
    free(ssl);
    ssl = NULL;
    printf("status code %d download success\n", status);
    return 0;
}
else {
    printf("status code %d download failed\n", status);
    close(webfd);
    free(ssl);
    ssl = NULL;
    return -1;
}
```

当然返回值都是一样的，也就参数个数少一个，所以代码不用改动太大

# 爬虫

要进行爬虫，首先要有一个种子URL（精心挑选的），这个网页中有大量指向新网页的URL地址

## 过程

1. 首先，我们对每个遇到的URL都要进行去重校验

	为了有效去重，我们准备了两个队列，一个用来装未处理的URL队列，另一个用来装已下载已解析的URL

	每个遇到的URL都需要与这两个队列里保存的URL进行比较，若有重复的则直接丢弃，不进行处理

2. 若没有重复，则会被加入未处理URL队列

3. 然后未处理的URL会经历之前http或https那种下载的经过进行处理，随后从未处理队列删除，放入已下载解析队列

## 头文件

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <regex.h>
#include <openssl/ssl.h>
#include <openssl/err.h>
#include <sys/mman.h>

#define Result_Max 10000

typedef struct {
	SSL* sslsock;
	SSL_CTX* sslctx;
} ssl_t;

typedef struct {
	char origin[8192];
	char domain[4096];
	char savepath[1024];
	char file[1024];
	char ip[16];
	int port;
	int http_type; // 0:http, 1:https
} url_t;

typedef struct {
	int front;
	int rear;
	int cur;
	int max;
	url_t* list;
} ct_t;

int Result_fd;
int Result_num;
int analytical_shutdown;

int spider_netinit();
int spider_connection(int mysock, url_t node);

// 传入传出，传入原始的域名origin，传出完整的结构体
int spider_analytical_url(url_t* node);

int spider_create_request(char* request, url_t node);
int spider_get_status(const char* response);
int spder_download(int webfd, char* request, url_t node, ssl_t* sslsock);
ssl_t* spider_OpenSSL(int webfd); // 参数为已连接的socket，返回值为安全套接字
ct_t* spider_container_create(int max);
int spider_get_node(ct_t* ct, url_t* node);
int spider_set_node(ct_t* ct, url_t node);
int spider_remove_duplication(ct_t* ECT, ct_t* PCT, const char* link);
int spider_analytical_html(ct_t* ECT, ct_t* PCT, url_t node);
```

## 主函数

**注意：每取出一个新的url都需要重新创建sock，重新进行连接、验证**

```c
#include <spider.h>

int main() {
	url_t node;
	url_t tmpnode;
	int mysock;
	char request[4096];
	ssl_t* ssl = NULL;
	Result_fd = open("Result.html", O_RDWR | O_CREAT, 0664);
	// 写入html头部信息
	spider_save_result(NULL, NULL, NULL);
	ct_t* ECT = spider_container_create(2000);
	printf("ECT = %p\n", ECT);
	ct_t* PCT = spider_container_create(Result_Max);
	printf("PCT = %p\n", PCT);
	char* url = "https://baike.baidu.com/item/ChatGPT/62446358";
	strcpy(node.origin, url);

	if (spider_remove_duplication(ECT, PCT, node.origin)) {
		printf("put %s into ECT\n", node.origin);
		spider_set_node(ECT, node);
	}
	while (ECT->cur > 0 && PCT->cur < Result_Max) {
		if (ECT->cur >= 500) {
			analytical_shutdown = 0;
			printf("close analytical_shutdown\n");
		}
		else if (ECT->cur <= 30) {
			analytical_shutdown = 1;
			printf("Open analytical_shutdown\n");
		}
		spider_get_node(ECT, &tmpnode);
		mysock = spider_netinit();
		spider_analytical_url(&tmpnode);
		spider_connection(mysock, tmpnode);
		spider_create_request(request, tmpnode);
		if (tmpnode.http_type)
			ssl = spider_OpenSSL(mysock);
		if ((spider_download(mysock, request, tmpnode, ssl)) == -1)
			continue;
		spider_set_node(PCT, tmpnode);
		spider_analytical_html(ECT, PCT, tmpnode);
		printf("analytical html %s success\n", tmpnode.origin);
	}
	printf("spider success, exiting\n");
	return 0;
}
```

## 保存结果

我们需要将爬虫下载到的标题、描述、链接给保存下来，可以选择用数据库保存，或者磁盘保存（一般采用html绘制表格的方式）

下面以html表格方式为例

```c
#include <spider.h>

int spider_save_result(const char* title, const char* desc, const char* link) {
	char result[20000]; // 结果数组
	bzero(result, sizeof(result));
	const char* start = "<html>\r\n<head>\r\n<meta http-equiv=\"Content-Type\" content=\"text/html;charset=utf-8\">\r\n</head>\r\n<table border=\"1\"cellspacing=\"1\" cellpadding=\"1\" width=\"1920\">\r\n<caption>爬虫采集数据集（百度百科）</caption>\r\n<tr><th>ID</th><th>词条标题</th><th>词条描述字段</th><th>词条链接</th></tr>\r\n";
	const char* end = "</table>\r\n</html>\r\n";
	if (Result_num == 0) { // 判定未抓取插入html首部信息
		write(Result_fd, start, strlen(start));
		return 0;
	}
	sprintf(result, "<tr><td>%d</td><td>%s<ts><td>%s</td><td><a href=\"%s\">%s</a></td><tr>\r\n", Result_num, title, desc, link, link);
	write(Result_fd, result, strlen(result));
	if (Result_Max == Result_num) {
		write(Result_fd, end, strlen(end));
		close(Result_fd);
	}
	return 0;
}
```

先插入头，最后等爬虫完成后再插入尾部，传入的参数是需要保存的三个数据的字符串，因为刚开始就要插入头部，而且头部只插入一次，所以这里巧妙的用结果数目来做条件判断是否开始添加数据

具体html的插入相关格式，就去看html相关的笔记吧

## 创建容器

在写入头以后，就可以创建容器了（其实他俩没有明确的前后关系）

```c
#include <spider.h>

ct_t* spider_container_create(int Max) {
	ct_t* ct = NULL;
	if ((ct = (ct_t*)malloc(sizeof(ct_t))) == NULL) {
		perror("spider_container_create failed");
		return NULL;
	}
	if ((ct->list = (url_t*)malloc(sizeof(url_t) * Max)) == NULL) {
		perror("spider_container_create failed");
		return NULL;
	}
	ct->front = 0;
	ct->rear = 0;
	ct->cur = 0;
	ct->max = Max;
	printf("create container success, container %p\n", ct);
	return ct;
}
```

## 添加入队列

在进入循环前，需要先把种子url添加入队列，作为第一个url开始解析下载处理

```c
#include <spider.h>

int spider_set_node(ct_t* ct, url_t node) {
	if (ct->cur == ct->max) {
		printf("container %p is full\n",ct);
		return -1;
	}
	ct->list[ct->front] = node;
	++ct->cur;
	ct->front = (ct->front + 1) % ct->max;
	return 0;
}
```

## 从队列取值

处理的第一步是从队列中取出一个url

```c
#include <spider.h>

int spider_get_node(ct_t* ct, url_t* node) {
	if (ct->cur == 0) {
		printf("container %p is empty\n", ct);
		return -1;
	}
	*node = ct->list[ct->rear];
	--ct->cur;
	ct->rear = (ct->rear + 1) % ct->max;
	return 0;
}
```

## 分析html

我们最终收到的响应体中的内容是html文件，我们需要通过正则表达式提取我们要的内容，这个正则表达式的规则需要用户自己去根据爬的网站进行分析

```c
#include <spider.h>

int spider_analytical_html(ct_t* ECT, ct_t* PCT, url_t node) {
	int fd;
	int filesize;
	char* ptr = NULL; // 映射地址
	char* jptr = NULL; // 地址备份
	char h1[1024];
	char desc[4096];
	char link[8192];
	url_t tmpnode;
	// 正则准备
	regex_t hreg, dreg, lreg;
	regmatch_t hmatch[2], dmatch[2], lmatch[2];

	// 打开源码
	if ((fd = open(node.file, O_RDONLY)) == -1) {
		perror("spider_analytical_html failed, open html error");
		return -1;
	}
	filesize = lseek(fd, 0, SEEK_END);
	// 映射
	if ((ptr = mmap(NULL, filesize, PROT_READ, MAP_PRIVATE, fd, 0)) == MAP_FAILED) {
		perror("spider_analytical_html failed, mmap html error");
		return -1;
	}
	jptr = ptr;
	close(fd);
	// 创建正则类型
	regcomp(&hreg, "<h1 >\\([^<]\\+\\?\\)</h1>", 0);
	regcomp(&dreg, "<meta name=\"description\" content=\"\\([^\"]\\+\\?\\)\">", 0);
	regcomp(&lreg, "<a[^>]\\+\\?href=\"\\(/item/[^\"]\\+\\?\\)\"[^>]\\+\\?>[^<]\\+\\?</a>", 0);
	bzero(h1, sizeof(h1)), bzero(desc, sizeof(desc)), bzero(link, sizeof(link));
	if ((regexec(&hreg, ptr, 2, hmatch, 0)) == 0)
		snprintf(h1, hmatch[1].rm_eo - hmatch[1].rm_so + 1, "%s", ptr + hmatch[1].rm_so);
	if ((regexec(&dreg, ptr, 2, dmatch, 0)) == 0)
		snprintf(desc, dmatch[1].rm_eo - dmatch[1].rm_so + 1, "%s", ptr + dmatch[1].rm_so);
	// 存储结果
	++Result_num;
	spider_save_result(h1, desc, node.origin);
	printf("**************************\n");
	printf("save h1 = %s\n", h1);
	printf("save descrption = %s\n", desc);
	printf("save url = %s\n", node.origin);
	printf("**************************\n");
	if (analytical_shutdown) {
		// 循环处理新连接
		while ((regexec(&lreg, ptr, 2, lmatch, 0)) == 0) {
			snprintf(link, lmatch[1].rm_eo - lmatch[1].rm_so + 24, "https://baike.baidu.com%s", ptr + lmatch[1].rm_so);
			if (spider_remove_duplication(ECT, PCT, link)) {
				strcpy(tmpnode.origin, link);
				spider_set_node(ECT, tmpnode);
			}
			bzero(link, sizeof(link));
			ptr += lmatch[0].rm_eo;
			if (ECT->cur + 1 == ECT->max) {
				printf("ECT is full, break\n");
				break;
			}
		}
	}
	munmap(jptr, filesize);
	regfree(&hreg), regfree(&dreg), regfree(&lreg);
	unlink(node.file); // 删除源码
	return 0;
}
```

## 判断重复

判断重复只需要拿着url在两个队列中都比较一下就行了

```c
#include <spider.h>

int spider_remove_duplication(ct_t* ECT, ct_t* PCT, const char* link) {
	int flag;
	flag = ECT->rear;
	while (flag % ECT->max != ECT->front) {
		if ((strncmp(ECT->list[flag].origin, link, strlen(link))) == 0)
			return 0;
		++flag;
	}
	flag = PCT->rear;
	while (flag % PCT->max != PCT->front) {
		if ((strncmp(PCT->list[flag].origin, link, strlen(link))) == 0)
			return 0;
		++flag;
	}
	return 1;
}
```

