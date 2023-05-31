# github

## Explore 探索标签

- Topics分类标签：可以在这里找到专栏

- Trending推送：可以看到最近比较火的项目

	> 可以自己加语言限制，编程语言限制和时间范围限制

## 仓库

- Code标签：表示所有工程资源
- ISSUES问答标签：使用者提交问题，制作者或者其他人解答
- Readme.md文件：阅读文档，快速帮助用户了解工程和使用方式
- LICENSE：许可证，用来说明当前项目的权限

> 三个高权限许可证，给使用者最大的权限以及最少的限制：
>
> - GPL 3.0
> - MIT许可证
> - APACHE 2.0

## 搜索项目

可以通过特定的关键字搜索网站内容：

- `python awesome`：搜索和python相关的非常棒的项目
- `java sample`：查找和java技术相关的样例
- `go tutorial`：搜索go语言相关资料

# github账号绑定本地git

1. 首先打开git bash使用命令`ssh-keygen -t rsa -C 邮箱`，就可以生成本地RSA密钥串

2. 按照生成的位置，打开id_rsa.pub文件，复制其中的密码字符串

3. 进入github -> 头像下拉列表 -> Settings -> SSH and GPG key -> New ssh key，然后粘贴刚刚的密钥

4. 查看配置文件：`git config --list`

5. 若没有user.name和user.email或者不正确则修改：

	调用`git config --global user.name '用户名'`，`git config --global user.email '邮箱'`

	**注意：这里的用户名是github的用户名，email也是对应的邮箱**

6. 为云端仓库的地址起别名：

	1. 从网站中拷贝云端仓库的SSH地址
	2. `git remote add 别名 "SSH地址"`
	3. 删除别名：`git remote remove 别名`
