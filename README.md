
_项目地址：[Minecraft-Rcon-Manage](https://github.com/pilgrimage233/Minecraft-Rcon-Manage)_
## 前言
笔者最近寻找一款能实现Minecraft服务器RCON远程访问的工具，找到了这个目前正在持续更新、功能丰富的开源项目`Minecraft-Rcon-Manage`，但实际部署过程中发现作者提供的教程博客无法正常访问，Github仓库中的README又过于简陋，不易于理解，故记下自己的部署过程。
## 1. 环境要求
根据作者提供的环境要求：

- JDK 21+
- Maven 3.2+
- MySQL 5.7+
- Redis 5.0+
- Node.js 16（白名单前端需 Node.js 18+）

这里以Linux Mint 22.1（Ubuntu 24.04）系统为例，提供一个简易的环境搭建教程。
\
首先确保安装了java，众所周知你如果机器上有跑MC那肯定有java，终端输入`java --version`查看当前java版本，确保java版本高于21。
![image](https://i-blog.csdnimg.cn/img_convert/1780d83047da55e3ab9a112e76f884d1.png)
在图中这种情况下，发现jdk版本只有17，此情况请卸载当前java并安装21版本的java：
```bash
sudo apt install openjdk-21-jdk
```
如果你有像我一样保留老版本java的需求，请单独下载java21：[下载链接](https://adoptium.net/zh-CN/temurin/releases?version=21)
然后终端输入：
```bash
mkdir ~/jdk21
cd ~/jdk21
tar -xzvf OpenJDK21U-jdk_x64...<version>.tar.gz
```
在下文的教程需要用到java命令时，将`java`命令替换成`~/jdk21/jdk-21.0.x/bin/java`即可。

其他的环境配置基本同理，其中MySQL、Redis、Nodejs可直接用apt安装：
```bash
sudo apt update
sudo apt install mysql-server mysql-client redis nodejs
```
通常安装这些后服务就会自动启动，若服务没有运行，使用`sudo systemctl start mysql redis`运行对应的服务就行了。
Maven的安装需要到[官网](https://maven.apache.org/download.cgi)下载并解压，然后将其添加到环境变量。
```bash
tar -xzvf apache-maven-<version>-bin.tar.gz
```
将解压后的文件夹移动到`/usr/local`：
```bash
mv apache-maven-<version> /usr/local/maven
```
编辑环境变量：
```bash
sudo vim /etc/profile
```
在文件末尾添加以下内容：
```bash
export MAVEN_HOME=/usr/local/maven
export PATH=$MAVEN_HOME/bin:$PATH
```
保存并使配置生效：
```bash
source /etc/profile
```
至此，环境的配置已全部完成。
## 2. 部署流程
1. 克隆项目地址
```bash
cd ~
git clone https://gitclone.com/github.com/pilgrimage233/Minecraft-Rcon-Manage
cd Minecraft-Rcon-Manage/
```
2. 下载Endless Manager后端jar
文件较大，推荐使用aria2多线程下载
```bash
aria2c -s 8 -x 8 https://release-assets.githubusercontent.com/github-production-release-asset/770347872/4d4b8f2f-920d-4011-8355-83a8818fb657   # 以3.8.2版本为例
```
运行`java -jar endless-manager-<version>.jar`，此时会在`/config`下生成配置文件`application-druid.yml` `application.yml`，且进程会报错停止运行一次，为正常现象。

编辑文件`application-druid.yml`
```bash
vim config/application-druid.yml
```
这个文件负责数据库连接。打开它，找到`master`部分，按以下说明修改：
```yaml
spring:
  datasource:
    druid:
      master:
        # 1. 修改地址：将 10.0.0.150:13306 改为 localhost:3306 (如果你没改过MySQL默认端口)
        url: jdbc:mysql://localhost:3306/ruoyi?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false&serverTimezone=Asia/Shanghai
        # 2. 修改账号：填入你本地 MySQL 的用户名（通常是 root）
        username: root
        # 3. 修改密码：填入你安装 MySQL 时设置的密码（没设置过默认就是系统登录密码）
        password: 你的数据库密码
```
进入MySQL命令行：
```bash
sudo mysql -u root -p
```
执行命令：
```mysql
CREATE DATABASE minecraft_manager CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE DATABASE ruoyi DEFAULT CHARACTER SET utf8mb4;
exit
```
再次执行：
```bash
java -jar endless-manager-<version>.jar   # 改为你下载的版本
```

此时后端程序应该已经能正常运行，访问8080端口即可打开后端网页界面。
![image](https://i-blog.csdnimg.cn/img_convert/cd8b40bf08ed721f29284ae2c47431de.png)
![image](https://i-blog.csdnimg.cn/img_convert/d09ec6843c116599c2acd0ea24b36b88.png)

![image](https://i-blog.csdnimg.cn/img_convert/6218d2aec6dd41da675d952e5283a5d0.png)


下面来启动前端程序。
先运行命令安装依赖：
```bash
cd endless-ui
npm install --registry=https://registry.npmmirror.com
```
由于apt默认安装的nodejs版本较新，需要我们修改一下`package.json`以确保兼容性：
```bash
vim package.json
```
找到`scripts`部分，将`dev`进行修改：
```json
"dev": "export NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service serve"
```
然后运行：
```bash
npm run dev
```
此时前端程序也已经运行了起来，默认端口一般是1024，访问1024端口即可看到前端网页，相关截图可以在[原仓库](https://github.com/pilgrimage233/Minecraft-Rcon-Manage?tab=readme-ov-file#-%E7%B3%BB%E7%BB%9F%E6%88%AA%E5%9B%BE)找到。
![image](https://i-blog.csdnimg.cn/img_convert/a90e46719071d19dd1fa6c996c7b4e4b.png)
![image](https://i-blog.csdnimg.cn/img_convert/53e1533a82b83b87f38ceab7e784d1db.png)

## 3. 结语
至此，我们已经可以在网页中使用与MC服务器相关的各种功能。
