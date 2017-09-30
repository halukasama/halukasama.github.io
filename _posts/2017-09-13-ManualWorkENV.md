---
layout:     post
title:      "Manual"
subtitle:   "开发环境搭建"
date:       2017-09-13 12:00:00
author:     "Wjl"
header-img: "img/post_bqmanual.jpg"
catalog: true
tags:
    - DOC
---

> "我刚来的时候,连grep都不会用,如何关防火墙都要搜......感谢峰神不杀之恩"

# 开发环境
  * **平台组开发及维护**
    * **CV5.0下辖C/S**
    * **设备及设备接入层s/p**

## *Linux*  
忘记是第几次重做系统了，基本上每次都要查一些重复的东西，
所以这次把配置流记下来，防止以后浪费时间。  
所有rpm包由yum管理，知道自己装的是什么，自己工作需要的库都知道在哪里，明确哪些库为对应APP的依赖。  
*始终关注情境,尽量保证lib都是有需要才被安装的。*  
以前的库管理混乱，单纯为解决一个问题，乱装，乱放。本次尝试合理规划。

|目的|位置|
|---|----|
|3rd party lib|/usr/local/lib|
|release dependence(32/64)|/usr/lib(/lib64)|
|debug dependence(32/64/arm)|~/bq/cv5/linux_i686(x86_64/arm)|
|work dependence private|~/bqvision|

1. 初始化配置
    * repo,gpg公钥,添加第三方源(非使用时关闭)等等
2. 应用程序
    * chrom浏览器[bug1(set_selinux_off)][bug2(root)_config_addroot]
    * virtualbox 5.10【官网】+ USB等其他扩展【5.10】(此问题只在sl中出现,centos7没有问题)
    ```
      问题： kernel deiver not installed (rc=-1908)
平台： centos7
内核版本：  3.10.0-123.el7.x86_64
gcc			【yum】
make 	      【yum】
patch		  【yum】
dkms 		  【yum】
qt 			【yum】
libgomp  	  【yum】
kernel-headers 【yum】
kernel-devel 首先安装yum中的包
安装完成发现/usr/src/kernels/与uname -a 中的版本号不匹配，
（yum提供的包版本太新，中间我没upgrade 或者update）
下载uname -a 对应的版本【RPM Search】
fontforge	  【yum】
binutils	   【yum】
glibc-headers  【yum】
glibc-devel    【yum】
执行 yum localinstall {Vb.rpm}..
因为VB需要根据kernel的源码去安装对应的模块：
执行 export KERN_DIR=/usr/src/kernels/{kernel}..
或添加上面的环境变量到/etc/profile再source一下   
执行 vboxdrv.sh setup  (默认安装/usr/lib/vitualbox)
测试正常使用。
```
    * VLC 	[yum(nux+epel+sl)]
3. develop
    * qt5.6		 【官网.run包】
    * gcc-c++		ver:4.85  
      依赖： cpp,gcc,libmpc,libstdc++-devel  
      更新： libgcc,libgomp,libstdc++
    * IDE&Edit
      * qtcreator4.1 【Qt集成】
      * VsCode  		 【官网】
      * atom         【官网】
    * 版本控制
      * git	1.8		【yum】
      * svn 1.7		【yum】
      * vsftpd 3.02
4. 其他工具（非yum安装会显示指明，其他均为yum的几个repo）
  * 7zip      忽略编码的解压软件
  * yumex     源包管理客户端
  * htop      进程监视
  * ntfs-3g   支持ntfs格式

***

## *CV5*

### 依赖

当前 ClearVision 5.0 系统主要运行于 NeoKylin 3.x(基于 RHEL 6.x) 系统,
而该系统默认为 ``x86_64``, 所以通常情况下我们优先编译 ``x86_64`` 的版本.

而我们使用的部分第三方提供的 sdk 只有 ``i686`` 版本, 所以我们也需要编译 ``i686`` 的版本.

通常在安装依赖时需要同时安装 ``x86_64`` 与 ``i686`` 的版本.

#### RHEL/CentOS/SL/NeoKylin
```
yum install -y \
    libuuid libuuid-devel \
    libuuid.i686 libuuid-devel.i686
```

#### Ubuntu
***
* **TODO**
***

### 获取源码

[git](https://git-scm.com/), 请务必熟悉 ``git`` 的相关概念, 并熟练掌握常用命令.

账号应该在加入平台软件开发组时由管理员创建, 对于未及时创建的, 可以向管理员申请.

#### 源码组织结构

多个项目间可能存在相互引用的内容, 而部分项目中又直接使用相对路径进行引用, 所以建议统一使用如下的源码组织结构:

```
$HOME
`-- work
    `-- cv5
        `-- docs
        `-- tools
        `-- pubsoft
        |   `-- rapidxml_1_13
        |   `-- swift
        |   `-- tinyxpath_1_3_1
        |   `-- ...
        `-- sdks
        |   `-- UDOCA
        |   `-- rpp
        |   `-- ...
        `-- share
        `-- clients
        |   `-- SurveillanceCenter
        |   `-- ...
        `-- servers
            `-- share
            `-- CentralServer
            `-- ...
```


项目说明:
* [docs](): 文档;
* [tools](): 包含编译脚本, sdk 安装脚本等;
* [pubsoft/swift](): 基础库;
* [pubsoft/tinyxpath_1_3_1](): xml 解析库;
* [sdks/UDOCA]():
* [sdks/rpp](): 丢包重传库;
* [share](): 不属于单个项目, 公用, 需要克隆源码使用;
* [clients/SurveillanceCenter](): 实时监控客户端;
* [clients/ManagementCenter](): 服务管理客户端;
* [servers/share](): 不属于单个项目, 公用, 需要克隆源码使用;

*clients 下为中心客户端, clients为中心服务端 根据目录区分项目类型等等..不再一一复述*  

部分项目会使用到环境变量 ``CV5_GIT_ROOT``, 对应本源码组织结构即为: ``CV5_GIT_ROOT=${HOME}/work/cv5``

执行如下命令创建基本工作目录(假设环境变量已正确设置):
```
export CV5_GIT_ROOT=${HOME}/work/cv5
mkdir -p ${CV5_GIT_ROOT}
```

#### 克隆项目源码

**请熟悉 git 相关概念**  
**请找管理员要协作项目权限**  
**使用ssh_keygen受信**

```
git clone git@hostname:file/project.git
```

### 编译项目

#### 安装编译工具

编译工具在项目 [tools]() 下, 需要先克隆该项目, 方法参考上面步骤,
假设已正确克隆该项目到 ``${CV5_GIT_ROOT}/tools`` 下;

安装编译脚本:
```
cd ${CV5_GIT_ROOT}/tools/scripts
./setup.sh
```
``setup.sh`` 脚本默认会把相关编译脚本, 安装到 ${HOME}/bin 下,
也可以通过设置 ``SHK_HOME`` 环境变量以指定脚本安装目录, 具体过程可以直接打开脚本文件 ``setup.sh`` 查看;

请正确设置环境变量 ``PATH`` 以方便执行相关脚本.

#### 安装项目依赖

##### 项目安装脚本

* projadm.py: 项目安装脚本, 可以下载安装已发布的项目;(需要python3)
* _platform_-install.sh: 安装所有项目的 _platform_ 版本;

**注**: 项目安装脚本使用 ``python`` 编写, 依赖于 ``python 3``, 请自行安装 ``python 3``.

##### 项目依赖列表

| Project | Depend on |
|---|---|
| swift| boost_static-1.55.0 |
| rpp | swift |
|uodca| swfit |


#### 编译项目源码

我们的项目通常使用 [cmake](https://cmake.org/) 或 [qmake](https://en.wikipedia.org/wiki/Qmake) 进行管理编译;

##### 编译举例

在上面的[安装项目的例子](#example)中, 如果执行了不带 ``platform`` 参数的命令, 将会发现, 在安装 ``x64_64`` 对应的版本时失败了, 输出类似(注意最后一行):

```
Downloading file ftp://192.168.0.250/release/3rdparty/linux_i686/tinyxpath_1_3_1-linux_i686-release.tar.gz	[OK]
['tar', 'xf', '/tmp/tinyxpath_1_3_1-linux_i686-release.tar.gz', '-C', '/home/wxf/bqvision/clearvision/linux_i686/release', '--strip-components=1']
Downloading file ftp://192.168.0.250/release/3rdparty/linux_arm_imx6/tinyxpath_1_3_1-linux_arm_imx6-release.tar.gz	[OK]
['tar', 'xf', '/tmp/tinyxpath_1_3_1-linux_arm_imx6-release.tar.gz', '-C', '/home/wxf/bqvision/clearvision/linux_arm_imx6/release', '--strip-components=1']
Downloading file ftp://192.168.0.250/release/3rdparty/linux_x86_64/tinyxpath_1_3_1-linux_x86_64-release.tar.gz	[Failed]
```

原因是下载 [tinyxpath_1_3_1]() ``x86_64`` 版本的包时失败了.
实际上我们并没有发布该项目的 ``x86_64`` 版本, 因为没有用到.
但是我们在此不妨就以编译该项目的 ``x86_64`` 版本为例来熟悉编译流程.

克隆项目源码:
```
cd ${WORKING_DIR}
git clone xx
```

进到项目目录可以发现, 该项目是使用 [cmake](https://cmake.org/) 进行管理编译的(因为存在 ``CMakeLists.txt`` 文件),
故使用 ``shk_cmake_build.sh`` 脚本进行编译, 命令如下:
```
cd ${CV5_GIT_ROOT}/pubsoft/tinyxpath_1_3_1
shk_cmake_build.sh -s . -d -i -p linux_x86_64
```
其中:
* ``-s`` 指定项目 ``CMakeLists.txt`` 文件所在路径, 本例为 ``.`` 即当前目录;
* ``-d`` 指编译项目的 ``debug`` 版;
* ``-i`` 指编译完成后执行安装过程, 请自行根据输出日志确定安装目录;
* ``-p`` 指定编译 ``x86_64`` 平台版本;


#### 特殊项目安装说明

* ``rapidxml_1_13``

### 附录
[centos7 cv5编译问题汇总](https://docs.google.com/document/d/1SzohayqvQAE5gLGorRNUMRDeB6RM19LojrZtU_fgboE/edit?usp=sharing)
#### 相关环境变量设置参考

```
export CV5_GIT_ROOT=${HOME}/work/cv5
export CV5_SDK_ROOT=${HOME}/bqvision/clearvision

# 本地发布目录
export CV5_RC_DIR=/home/backup/release/cv5.0
export GITSERVER=192.168.0.250

# 个人路径, 并不推荐
SHK_HOME=${HOME}/dks/shk
SHK_PATH=${SHK_HOME}/bin

export PATH=$PATH:$SHK_PATH
export SHK_QMAKE_PATH=/usr/bin/qmake-qt5
```

***

## DeviceLayer及第三方设备插件调试与测试说明

### 使用的服务和客户端

* DeviceGateWay
   * 所有设备必须添加到某个设备接入服务中
   * 用于 开启/关闭 设备接入服务
   * 将设备插件加载到服务器中
   * 常用于打印调试信息

	安装或编译DeviceLayer中的DeviceGateWay到指定路径  
	具体操作步骤:
	[cv5](#CV5)  
	执行DeviceGateway  
	配置参数: --mode=app

* DeviceToolBox  
   * 查找连接中的设备
   * 获取未知设备的IP地址
   * 可以在Client升级固件,而不用访问Browser  

	安装程序  
	因DeviceToolBox是有界面的应用
	所以根据信息操作即可.

* ManagementCenter  
   * 添加管理设备接入服务
   * 添加管理测试调试设备

	根据工作需要对 设备/服务 进行增删改查等操作

* SurveillanceCenter  
   * 是音视频设备的显示输出
   * 用于修改设备参数

	是衡量音视频设备是否正常工作的标准

### 结构化数据

* DeviceModels.xml 保存设备物理型号说明  
用于描述用户创建设备所需基本参数  

* ExtraData.xml 保存所有设备的拓展数据  
用于描述用户创建设备所需额外参数

* DeviceFamilies.xml 保存设备抽象类型说明  
用于描述设备的能力,供系统内部使用  

* ConnectionParamTemplates.xml 保存设备连接平台说明  
用于描述设备抽象类型在主动和被动连接平台时所需的参数  

* reg_values.xml 保存服务的相关数据  
用于描述平台接入服务的配置信息

### 搭建流程
1. 确保服务与客户端安装正确
2. 配置 reg_values.xml 将设备接入服务加载到对应的服务器中;
3. 运行 DeviceGateWay 开启设备接入服务;
4. 运行 ManagementCenter 服务器当中对设备进行 添加/修改/配置 等操作;
5. 运行 SurveillanceCenter 在其中配置设备,并且根据最终输出确认设备是否正常工作;
6. 设备详细信息请查阅devicetypes中的结构化数据;

### FAQ
* **Q1: 添加新设备显示不在线**
```
      A: 排除设备自身问题之后, 查看DeviceGateWay是否正常运行;
```
* **Q2: 修改插件代码后,重新编译DeviceLayer, 程序运行最终结果无变化**
```
      A: 重启DeviceGateWay确保重新加载修改后的插件.
```
* **Q3: 登陆 ManagementCenter 中没有对应的设备接入服务**
```
      A: reg_values.xml配置的服务器和登陆的服务器是否是同一个服务器
         比如配置中填写的是192.168.6.20,而实际登陆的是192.168.50.250
```
* **Q4: 在运行DeviceGateWay的终端下,没有打印任何调试信息**
```
      A: 首先,确保设备的接入服务选项是我们自己的服务;

          其次,利用pgrep Device 查看是否有且仅有一个服务进程,多进程会导致失效;

          最后,在ManagementCenter中查看登陆的服务器,是否是DeviceGateWay的服务器(同Q3);  

          注:查看运行时的配置参数是否为"--mode=app" 而不是"-mode=app"或者其他输入错误导致的问题;
```
* **Q5: 确认修改了reg_values.xml,但服务还是无法正常上线**
```
    A:  所有服务的配置文件都叫"reg_values.xml"  
      执行pwd查看是否配错服务,如正确配置服务,继续检查配置完成后是否未运行服务
```
[DeviceGateWay说明早期版本](https://docs.google.com/document/d/1-Hzo8tbqplTtsvvXOsj73q0ET669VfIXaQMUjrnRk3A/edit?usp=sharing)

***

## pacemaker+crosync+pcs配置

### 准备

两个在可以互相访问的主机：

VB_Clu_test01：node1 (192.168.0.124)  
VB_Clu_test02：node2 (192.168.0.125)  
VIP：192.168.126

### 系统环境配置
1. 关闭SELinux和防火墙:
```
systemctl disable firewalld
```
将/etc/sysconfig/selinux 中设置 “SELINUX=disabled”
不想reboot可以
```
systemctl stop firealld
setenforce 0
```
2. host  
分别配置两个节点的hostname
```
vim /etc/hostname
node1 (192.168.0.124)
node2 (192.168.0.125)
```
两个节点配置完成后重启动网络
```
systemctl restart network
```
查看是否配置成功
修改/etc/hosts 添加hostname

3. 时间同步
```
yum install ntp
ntpdate 192.168.4.192(公司自己的ntp服务器)
```
4. ssh互信
```
ssh-keygen -t rsa -P ''
ssh-coyp-id -i /root/.ssh/id_rsa.pub root@{nodename}
```
第二次配置没添加公钥也配置成功了。。。
5. pcs
```
systemctl start pcsd.service
systemctl enable pcsd.service
```
### 配置pacemaker管理
1. 创建集群<密码最好一致>
```
passwd hacluster #修改密码
```
2. 集群中所有节点互相认证
```
pcs cluster auth {node_names}
```
输入用户名和密码
3. 启动集群
```
pcs cluster setup --start --name {cluster_name} {node_names}
```
提示 :
```
node1:Succeeded
node1:Starting Cluster...
node2:Succeeded
node2:Starting Cluster...
```
证明成功
可添加自启动：
```
pcs cluster enable --all
```
常用命令行：
```
pcs config 配置
pcs status 状态
pcs cluster start –all 启动
pcs resource show 资源
pcs resource debug-start resource 调试
pcs cluster standby {nodename} 设置主备
```

### 添加资源

***
* **TODO**
***

## 新建设备参数说明(2017.wjl changelog)

### 管理服务中心新建设备所需参数及说明

#### 设备类型(DeviceDef.h)
```
enum CV_DEVICE_TYPE
{
    CV_DEVICE_TYPE_UNKNOWN       = 0,    /**< 未知类型 */
    CV_DEVICE_TYPE_VIDEO_ENCODER =      CV_MAKE_DEVICE_TYPE(CV_DEVICE_CATE_CODEC, 1),   /**< 视频编码器 */
    CV_DEVICE_TYPE_VIDEO_DECODER =      CV_MAKE_DEVICE_TYPE(CV_DEVICE_CATE_CODEC, 2),   /**< 视频解码器 */
    CV_DEVICE_TYPE_VIDEO_CODEC   =      CV_MAKE_DEVICE_TYPE(CV_DEVICE_CATE_CODEC, 3),   /**< 视频编解码器 */
    CV_DEVICE_TYPE_VIDEO_TRANSCODER = CV_MAKE_DEVICE_TYPE(CV_DEVICE_CATE_CODEC, 4),   /**< 视频转码器 */
    CV_DEVICE_TYPE_NETWORK_CAMERA = CV_MAKE_DEVICE_TYPE(CV_DEVICE_CATE_CODEC, 10),   /**< 网络摄像机 */
}

```

#### 视频编码(CV_DEVICE_TYPE_VIDEO_ENCODER)
1. SIP Device
   - Sip-Stream  


|  字段(粗体为必填项)|属性|说明|  
| ----------------|---|----|
|  **Type**       		| 设备类型	| 设备的抽象类型 		|
|  **ModelID**    		| 设备型号	| 设备的物理型号 		|
|  **ID**	    		| 标识		| 通过标识可以找到当前设备	|
|  **Name**    		| 名称		| 设备在客户端中的名称	|
|  Description 		| 描述		| 设备在客户端中的描述	|
|  Location   		| 本地位置	| 设备所在组织结构描述	|
|  AutoCreatePeripheral	| 自动创建外设	| 视频输入对应摄像机	|
|		    		|	   	| 视频输出对应监视器	|
|  **DeviceGatewayID**	| 设备接入服务标识	| 设备所选择的接入服务	|
|  ConnectionModel		| 连接方式 	| 主动(不支持被动连接)	|
|	**IPAddress**  		| SIP服务器地址 	| 设备地址      		|
|  **SipDestServerPort**	| SIP服务器端口 	| 连接设备端口		|
|  **VideoInputPort**	| 视频接收端口  	| 取视频流端口   		|
|  AudioInputPort		| 音频接收端口  	| 取音频流端口  		|
|  MulticastEnabled	| 组播模式	| 设备是否支持组播模式	|
|  StreamTransProtocol	| 输入流传输协议	| 默认       		|
|  TimeSyncMode		| 时间同步	| 0			|
|  NTPServer		| NTP服务器	| NULL			|
|  AutoAllocMulticastAddress|自动分配组播地址	| 是/否			|
|  MulticastAddress	| 组播地址	| 自动分配组播地址为否可用	|

新建示例:  
**上表中粗体字段代表必填参数,其他字段可选填**

```cpp
UDOCA::CreateObjectRequest req = UDOCA::CreateObjectRequest::create();
req.setClassName("CodecDevice");

UDOCA::Object obj = UDOCA::Object::create();
uint32 type = 16777217;
obj.setProperty("Type", type);
obj.setProperty("ModelID", "Sip-Stream");
obj.setProperty("Name", "名称");
obj.setProperty("Description", "描述");
obj.setProperty("Location", "本地位置");
obj.setProperty("AutoCreatePeripheral", "true");
// 获取DeviceGatewayID
UDOCA::QueryObjectsRequest req = UDOCA::QueryObjectsRequest::create();
req.setClassName("Service");
req.setPropertyName("ID");
req.setPropertyName("Name");
req.setPropertyName("Address");
req.setPropertyName("Online");
req.setPropertyName("Enabled");
UDOCA::QueryObjectsResponse resp(mc->getLastSyncResponse());
UDOCA::List list = resp.getObjectList();
UDOCA::Iterator it = list.iterator();
std::list<string> objIdList;
while(it.hasNext()) {
	UDOCA::Object obj = it.next();
	//获取设备接入服务id : obj.getProperty("ID").toString())
	objIdList.append(obj.getProperty("ID").toString()));
}
// 这里在list中拿自己需要的设备接入服务ID
obj.setProperty("DeviceGatewayID", objIdList.first());

obj.setProperty("ConnectionModel", "Active");
// 设置连接参数
UDOCA::Object arg = UDOCA::Object::create();
arg.setProperty("IPAddress", "192.168.0.123");
arg.setProperty("SipDestServerPort", "8000");
arg.setProperty("AudioInputPort", "5060");
arg.setProperty("SipDestServerPort", "0");

obj.setProperty("ConnectionArguments", arg);
obj.setProperty("Address", arg.getProperty("IPAddress"));

//
obj.setProperty("MulticastEnabled", false);
// 当"MulticastEnabled", true)时 添加此属性组播地址
obj.setProperty("MulticastAddress", "239.255.255.255");

// 部分可选参数
obj.setProperty("StreamTransProtocol", 0);
obj.setProperty("OutputStreamTransProtocol", 0);
obj.setProperty("NTPServer", "");
obj.setProperty("TimeSyncMode", 0);
```

***
* **TODO**
***

* Stream Device
   - Stream-TS
   - Stream-TS-RTP
* VIP Device

#### 视频解码
* AIPDevice
* ..
#### 视频编解码
* TEADevice
* ..
#### IPC
* HikDevice
* OnvifDevice
* PanSCDevice
* ..

### 获取设备列表(2016 update)

> 已发notify解释如何配置devicegateway, 不要在bugzilla提交关于这个问题产生的bug了

* 新增设备获取方式
  - **想要获取的设备属性**

请求示例:
```
UDOCA::QueryObjectsRequest req = UDOCA::QueryObjectsRequest::create();
req.setClassName("VIEW_Camera");

// 默认情况下获取设备的所有属性
// 也可以通过以下方式只获取关心的属性
UDOCA::List names = UDOCA::List::create();
names.add("ID");
names.add("Name");
names.add("Type");
names.add("ModelID");
names.add("Online");
names.add("Enabled");
names.add("CodecDeviceName");
names.add("Location");
names.add("Description");
req.setPropertyNameList(names);

// 也可以限制只获取特定 ID(其他属性限制类似) 的摄像机
req.setCondition(UDOCA::EqualExpr::create("ID", "5D50ADB8-AC30-46A7-8238-6DD60E4896E5"));

// 排序可选
req.addOrderBy("Name", true);
```

响应示例,
```
// UDOCA::QueryObjectsResponse resp;
// 需要检查返回是否成功, resp.isFailure(), 假设成功
// 返回 UDOCA::List
UDOCA::List cameraList = resp.getReturn().toList();

UDOCA::Iterator it = cameraList.iterator();

while (it.hasNext()) {
	UDOCA::Object camera = it.next();
	// 最好获取属性前检查是否有该属性
	if (camera.hasProperty("ID")) {
		UDOCA::String id = camera.getProperty("ID").toString();
	}
}

```
该例仅作参考, 请根据实际情况进行修正, 函数名可能有误

***
* TODO
 后续支持,兼容无法更新DeviceGateWay版本的环境(保留原有设计)以及更新<<设备接入>>
***

<<设备接入>>
```
注意事项
1. 定义设备家族(接入插件) (DeviceFamilies.xml)
	定义设备家族名称
	定义设备家族设备连接的参数和其默认值
	定义设备家族设备发现的参数和其默认值

2. 定义设备型号 (DeviceModels.xml)
	定义设备型号名称
	定义设备型号所属的设备家族
	定义视频输入、视频输出、音频输入、音频输出、串口输入、串口输出、数字报警输入、数字报警输出端口的个数
	定义视音频媒体流的路数，及相关属性(包格式、压缩算法等)

	这里重点说一下Stream标签中的属性：
	id 即设备接入SDK中的StreamID，理论上每个设备家族可以根据其自身需要自行定义其的取值。但在习惯上，统一将视频编码流(包括视频频混合编码流)定义为1-256，视频解码流(包括视频频混合解码流)定义为257-512，音频编码流定义为513-768，音频解码流定义为769-1024
	type 音视频类型: 可取值Video(视频)、Audio(音频)、VideoAudio(视音频混合)
	videoAlgo 视频编码算法
	audioAlgo 音频编码算法
	transProto 流传输协议
	pktfmt 流封包格式
	关于type、videoAlgo、audioAlgo、transProto、pktfmt几个属性的可取值可以参看DeviceLayer/src/DeviceModelManager.cpp和DeviceLayer/include/StreamDef.h两个文件，如需要增加可取值，也需要修改这两个文件，并重新编译DeviceLayer
	对于音频解码流(即对讲流)，还需指定audioSampleRate、audioChannels、audioBitsPerSample 三个属性来规定音频采样参数。

3. 定义设备型号和设备家族的功能属性 (ExtraData.xml)
	分辨率、码率、帧率的取值范围
	是否支持调节亮度、色度、对比度、饱和度
	是否支持OSD配置，以及可以配置哪些OSD参数
	时间同步模式

4. 编写设备接入插件
	实现IDLPConnector(连接器)、IDLPDeviceControl(设备)、IDLPStreams(媒体流集合)、IDLPStream(媒体流)、IDLPConfig(参数配置)、IDLPUart(串口控制)、IDLPPtzControl2(云台控制)、IDLPAlarms（报警）等接口
	修改plugins.xml将设备家族和设备家族接入插件文件关联

(注：DeviceFamilies.xml、DeviceModels.xml、ExtraData.xml、ConnectionParamTemplates.xml、plugins.xml 这几个文件被修改后，需要重新签名，签名的方法是执行DeviceLayer\build下的signature_devicetypes.bat和signature_devicetypes_linux.bat)

5. 修改Liquid和StreamPlay以及MediaParser，支持解析新设备的码流封装格式(如已支持则不需要修改)
	这里的流封装格式就是前面提到的DeviceModels.xml文件中Stream标签的pktfmt属性的取值
	理论上每个pktfmt都对应一个parser(即实现IMediaParser)来实现对其的解析(但实际的实现有出入，例如对于带有rtp头的流格式，会在StreamPlay层中统一去掉rtp头)
	所以，要支持一种新的流格式，就必须实现一个新的继承自IMediaParser的parser类，该类最重要的两个函数是ParseFrame和ParseFrameHead
	ParseFrame用于从输入的数据中解析出每一帧视频和音频的完整数据，然后将解析后的帧数据送到解码模块进行解码
	ParseFrameHead用于从输入数据中解析出每一个视频帧的帧头位置，以及其帧类型(I帧或P帧)。该函数主要是在MediaParser被调用，而MediaParser主要是在存储服务器录像时和设备接入服务器向解码器转发码流时，定位视频帧帧头时使用。

6. 支持音频对讲所需的编码算法
	如果音频对讲需要支持新的音频编码算法，则需要修改设备接入服务(DeviceGateway)，在音频对讲的实现流程中，是客户端将PCM数据发送到设备接入，然后根据对讲设备的类型，使用相应的音频编码算法进行编码。目前已支持G711A/Mu、G726、MP2、AAC (参见servers/share/StreamRelay/AudioEncoder.cpp)
```

## Windows

### VS2005 用于维护与阅读较早的windows客户端(MFC代码)
  1.  下载地址：  
  VS2005：[Visual Studio 2005 Professional Edition Dvd.iso]()  
  VS助手：[VA_X_Setup]() ，[VA_X.dll]()

  2.  安装
   * 解压VS2005的ISO文件，执行Setup.exe 根据提示安装

   * 安装VA_X_Setup 选择pre-2010versions of Microsoft Visual Studio

  ![p1](https://lh3.googleusercontent.com/ApLZLe8CgOrkq_xtlOBrrQqQJtyHXqMK2QCyQIpX4EJBAZH_8uMhwPMpLF0vrbLqtkFmV4ukxeEhSzuUM-MinJ2WtMyM2W_RitK7aLJwcI-kgXAvhXkCiB5oDXHqwk0UYm4mxI2p4LrNTYBdqgAvDduUlLhgaPU6RcfVSBD-uSNLMIHQUZQRrGqRzISjIpzUj4WUkJs2nyL4EYN5m1jJYMeNiR3-MVroVKI7BbYK6Y8n69vkC2Y7r5hSCCqEzwHGv8l_ZdWtK20vJ5OkKm88HqfpdUj8bnbhmqlx_WEESA05xIUyS3RSQ8sBYnteFj5kJ_lZsJwwXvtgMiNl3YP2KpIWJqsE5F6WigDpVmOpvRRdwA225w_DUr1J9KOgZEqllF14H7bi-vuG4QMYUK-1cnKPQzl20uTWRyJKyAxodDR1dCMXn_n5JQ330sw5Ae6lq7nVGw60Se4Yzz4zuzY3USWDtP46ERab-HhiQB-MAIwfdecQ64SRBj9wdB52L5hCmOU66wHSymeRq2H_ooItNXzjBGwTDcl0N2iGxfPS51gf3rSmNXxp3lEthP-4njj_B6ubQqTI331kVW6luXZgpwrdEmNnWsTXHoSVpS5sCn5HbNU=w521-h365-no)  

   * 完成后将用先前下载的VA_X.dll替换掉安装目录中的VA_X.dll

  ![p2](https://lh3.googleusercontent.com/S66Kq1llnyBwHS4baeD_LEvr41_BWnTJMnBDTDtWRrz5-1Ytzmg9NmFFeh26PvwxrGVbhYmTv4vLqKiDf5UhxD0ys4EuZ5auBBvZdndr4vm1q0WYOziJW2abqatygf5SGX30b18WNPMSW7ElnV5xMZZvcyY7RkU2pPRKFR2nIQLDyKL04sqVLEVksZH5jtTzMJgvwVT4Oo7iRrFglK3UC0TZ1nfDz9dSid7EJxJOemlDguw-kvRTMSqMmoPEJhp3kqCahbHiAjaBshFG2NeTBNdpjz3SyAxi6qnYbCHoX9EnqefbFm61KglIVGUqPbG2U6cByOcLmamkRQrY6V1yERCZDJXkH5A8O3iOBQI-Q4xP8aFVt4_m5Hl7XoRFO3NztIwiZUXDUcgMzDJ6rzqQY-mRTMw9rIxcNFCiDpee5LCfHqv3GMX6XPs-ZaYAaHHyhtN2xELwQppIYR5t-8OXaKb_pWlXn4rRdLD6w6xP6oAsdObQ16LTdFQ6KgmhgS4plDc7Tbcs8LOQI4KiJ9N26_GtwqBMb6RkALBF-CojRcmPxPCaUurKDJgN6q2w9GPHzfhjgJsrK9Ir3ssDtiJKE0ecabGkGOUlNMMnuvZydkCDU-M=w530-h345-no)
  * 进入VS2005 如下图，说明安装成功  

  ![p3](https://lh3.googleusercontent.com/xdpPNSL7xGqgMgr7PEhefBZFg0NBAYd9yw0LrR_PS9W_9OF841sEWt1hIHIF5fopCsKsRXpQyDjInU9JcBNPiw-OG9b2bdg5gnfCDYt9vBCPBxe_dHTYhA_WGsoGi5Ot31qUBnDjDW0l2cCqI8IgO_5q2G9wETjOUZyLnzfM1Ja-VO6VmhoLWhgJmFh8CUx0pvrU7hz53gakvQHm_w20z7V8LCIa1YWb2DYGCkWIlm4k5d9TLKTqzHuPEJX86GALcXsRuo4lO71lb9VGj4RohCVXIxkL9RWO5zGOyo7jq3HMENqQie9SM30YtYbL93M71ioe2f5XqhncHmFeXM6frvi1QuGkVSIN2MENzI2B_OSbWkD7U23D1tiB9xxhyLRZsBRzN799mtObDwPa4i7BLZWPS2RINgNYDIbs7rZbHyhlK6W-Ju_UopDpmu3fcMUpFJOkitnw0PU1HaYZwMGiN8OKll59T815AoGZftebT9tuWDO5pMlyjxMpnevKh0oYs0g8h7JUnbSp4Km-fA8E4LGnLzwgcCKmlymawXPqqhia6Nj9pBNnvvJYddcJGfpXkZMKK0a6A5_XTM1Yuy0HnXdQFNLnKirlrgTaKuz1o7rcAMU=w363-h205-no)

### Cygwin

 1.  下载Cygwin  
   根据操作系统选择对应的版本 **[[下载地址]](https://cygwin.com/install.html)**

 2.  安装Cygwin和git
  *  运行Cygwin-setup  
  选择安装路径，配置软件下载源地址等
  *  在Select Packages中搜索git  
  点击skip(skip会变成版本号)，选择下载源码包或二进制包

![select_package](https://lh3.googleusercontent.com/D73giFFiG8D-3LcupFrHmf8MC9oaGqfNQDlwlhX8mlgaJ9-bembYR0PD8nOZwd-_OBTYnXBJqLEcY7rOmoUGtesVpo6a1iZwbjeR6W8MCcgAfI6jcb7NqcdLQMQwbNPxh6m89tXHzY8JNI0gEmKsa9tcOGZCN4YylZn47sm0hQCf3g8LhfUwc0-9odM1m8jXwumEHpBcgTtmZ1oSws8ovtgc5BQNbqe3U__p9w9pT1c4avy1sXft35vCMWZ5txTl-OFmT6MsbSrP9JYTFBTTsn2_nOqLzeriXnfVcbsIK6Hy7sopJBX-du1EiuewhuyYY_EudSwvi8vP5lab2w8ROTSclIcUkHFkrajXpj4Cx4rbgmRm3FaBl3boHpawKRoYdJmYsVriVzn-cSnJzdGdM5tIQ7NCGAYDDk4jk2tboZyVLW8SEJUjWKmojrfAL-FnKEc3godXDXMkhzQW7E6WFua25oKdlpYfRSH_0YybYKkJSAWUXNO3aUyB6pxgvSVX1FSzKOerPWkiJg5fNWYuMBFh2PyqKj-39xYtTRbpCjS_NEvVPOqX9wVTm1ZKY6GOrFTtQr5qBnJBMBl5De83GzotSIuNFhdjj2kYJs9RUC2UQFVlrIE=w538-h388-no)

 3.  配置git  
  *  git全局配置  
```  
  git config --global user.name "your name"  
  git config --global user.email "your email"
```
  *  配置域名对应key  
  在.ssh 目录下新建config文件
```
  host wjl.bqvision.com
        user [name]
        hostname 192.168.0.123
        port 40747
        identityfile [ssh-key]
```

  *  其他配置根据个人需求自行添加

4.  添加ssh_keygen  
 *  进入"/home"目录下的.ssh文件夹，没有则创建；
 *  生成key
 ```
 ssh-keygen -t rsa -C "your account"  
 ```
 *   连续3次回车,创建默认key，默认密码为空。  
     例: 创建默认名称，没有密码的key  

 ```
 $ ssh-keygen.exe -t rsa -C "you account"
 Generating public/private rsa key pair.
 Enter file in which to save the key (/home/usr/.ssh/id_rsa):
 Enter passphrase (empty for no passphrase):
 Enter same passphrase again:
 Your identification has been saved in id_rsa.
 Your public key has been saved in id_rsa.pub.
 The key fingerprint is:
 SHA256:ukDdvaOw30UZNq1sEHkOFP48c6M/dTxkSx7FbBwq35E  
 you account
 The key's randomart image is:
 +---[RSA 2048]----+
 |         .+o   =.|
 |         .o.... B|
 |          o=+..E.|
 |     . . . *+=.=.|
 |    . . S . @.Boo|
 |   .   .   + = =+|
 |    . o   o o  .o|
 |     . + o o ..  |
 |      o.o .   .. |
 +----[SHA256]-----+
 ```

5.  向gitlab添加公钥  
 * 登录gitlab
 * 进入ProFileSetting
 * 进入SSH Keys
 * 将先前获得的id_rsa.pub中的内容粘贴
 * 填写标题
 * 点击Add key按钮  

![gitlab_ssh_key](https://lh3.googleusercontent.com/HqFRttr0U0HrxUiMKscy_mnwuqllG6lnjKuo7aXlgM43-jB8YWKiMQn-lSiesrYOhXgIoygcifD88xqssIo6l-oxTm4AwWemBasPFgeIdNREqw_Fh653yObw3Tt1BovRzmv8-knbZqPMX0w2l41C1AR0_JsovpyPhM6eAApZmWT5i-AsXxRQSdNTWs7-sxHNp8Yd0vaXhXHQVCBn7JQ3PBaSjCe_Pg4sOxESOgnM_5BYdqPNBOE2TdHBq1tRoOfAYIRe43USmN6m9H_xXbMMDFxxjAfyxmNvDWp_08ynkV-q83J-RF8Pk7yiJtI8W4Z3LUKVXRMEZNp7gnuQ-KTFjbuwn2fd-_e52HEB1aFOknbkzJD0-ZBcKm5X2QWxU6d99uZ0XPYhCLd-3E6hO6FHF6_W5drJIerFzLGWXbWPi2rm_8Q0PT7cD7IQ2c14iyFgLAGq7fkGqbpYLX7XmKEQCo7g3m3PNuPGR5J92lzErGZcZZABiVI-G7FGNgRM-LdgkfwBeZSOA5rlYuB1-uBksgTLFhm044Gj4M0kVlwBD1s1gIqFmsS3DtbtL9ZBTSdXnMfYyy-QP5VrweY37MNJ_BvNZdlkLN7ObZSgxeQiE6BfPTWfrVk=w1443-h545-no)

 * 检测是否添加成功：  
例：用户向gitlab服务器192.168.0.122:8080添加公钥后测试是否成功  
执行：
```
$ ssh -T git@wjl.bqvision.com
```

如成功则打印如下：

```
Welcome to GitLab, user!
```

## 协议
> 开始之前应该重新阅读一次相关``RFC``或其他官方guide.
  确定有章可循,而不是``想当然``

### HTTP(CW)

### RTSP(TD)
* RTP
* RTCP
* SDP

### SOAP(CW)
* web_server
* how to use gsoap

### ONVIF(DF)

### CGI(CW)

### Sip(新)
* SDP (这个要不要单拿出来啊?)
