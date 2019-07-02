## Cisco ACI安装和配置
+ 作者： Robbie Shen 
+ 联系： yazshen@cisco.com
+ 日期： 2019/03/01

### 1. 安装前准备工作
#### 1.1 ACI软件版本
ACI软件官方下载链接：[ACI Software Download](https://software.cisco.com/download/home/285968390/type)

+ 软件文件说明

APIC Image for 3.2(4e) Release (aci-apic-dk9.3.2.4e.iso): 版本号为3.2(4e)，用于安装和升级APIC服务器

Cisco Nexus 9000 Series ACI Mode Switch Software Release 13.2(4e) (aci-n9000-dk9.13.2.4e.bin): 版本号为13.2(4e)，用于安装和升级Cisco Nexus 9K交换机

+ 软件版本说明

Cisco官方要求：整个ACI Fabric里面只能运行一个软件版本，也就是说APIC和N9K交换机上版本必须一致

N9K交换机版本号会在ACI版本号前面多一个"1"，例如：13.2(4e)等于ACI 3.2(4e)

#### 1.2 APIC CIMC版本
APIC服务器是Cisco UCS C-Series产品定制，CIMC是UCS服务器上的带外管理系统。

Cisco官方针对每一个ACI软件版本都会有一个推荐CIMC版本，建议用户使用推荐版本。我们可以在Release Notes文档里面找到相关版本信息，例如: [3.2(4e)](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/3-x/release_notes/Cisco-APIC-Release-Notes-324.html)

```

This release supports the following firmware:

—      3.0(4d) CIMC HUU ISO (recommended)

—      For UCS C220/C240 M4: 3.0(3f) CIMC HUU ISO

—      For UCS C220/C240 M3: 3.0(3e) CIMC HUU ISO

—      2.0(13i) CIMC HUU ISO

—      2.0(9c) CIMC HUU ISO

—      2.0(3i) CIMC HUU ISO

```

#### 1.3 推荐版本

从刚才的官方链接我们可以看的，目前ACI有1.x, 2.x, 3.x, 4.x这4个大版本，每个大版本下面都有不少MR版本。那么到底如何选择呢？

+ 1.x 已经EOS，不需要再考虑了

[ACI 1.x EOS](https://www.cisco.com/c/en/us/products/collateral/cloud-systems-management/application-policy-infrastructure-controller-apic/eos-eol-notice-c51-740340.html)

+ 官方推荐2.x, 3.x

[Recommended ACI Software Release](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/recommended-release/b_Recommended_Cisco_ACI_Releases.html)

+ 如果你购买了APIC-SERVER-M3/APIC-SERVER-L3，那么只支持4.x版本

+ 选择好大版本之后，使用最新的MR版本，不要选择已经标注为Deferred Release的版本


### 2. CIMC升级
#### 2.1 查看UCS服务器型号和规格

首选，登录CIMC管理界面，找到Server Summary -> Server Properties
![UCS Type Model](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-01.png)

从BIOS Version中我们可以知道这台APIC-SERVER-M1的型号为Cisco UCS C220M3

从UCS产品页面下载CIMC安装介质：[UCS CIMC Download](https://software.cisco.com/download/home/283612685)

选择"Unified Computing System (UCS) Server Firmware":
![CIMC Firmware](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-02.png)

#### 2.2 使用Virtual KVM更新CIMC
我们可以使用Local KVM with DVD，Virtual KVM方式来重启服务器并用CIMC介质启动，升级CIMC相关固件时候，建议Update ALL

输入默认密码：password

![CIMC Firmware](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-03.png)

选择Cisco vKVM-Mapped vDVD

![CIMC Firmware](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-04.png)

CIMC引导中

![CIMC Firmware](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-05.png)

选择Update All

![CIMC Firmware](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-06.png)

#### 2.3 检查TPM, LLDP状态
在APIC安装之前，我们还要确认TPM LLDP状态：

进入CIMC管理界面，选择"Server -> Inventory -> TPM"，确认状态为"Activated"

![CIMC TPM](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-16.png)

进入CIMC管理界面，选择"Server -> Inventory -> Cisco VIC Adapters"，确认LLDP状态为"Disabled"

![CIMC TPM](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-17.png)

### 3. APIC安装
参考官方文档：

[APIC 2.x Guide](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/2-x/GSG/b_APIC_Getting_Started_Guide_Rel_2_x.html)

[APIC 3.x Guide](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/3-x/getting_started/b_APIC_Getting_Started_Guide_Rel_3_x.html)

[APIC 4.x Guide](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/4-x/getting-started/Cisco-APIC-Getting-Started-Guide-401.html)

[APIC-SERVER-M3/L3 Installation Guide](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/server/M3-L3-server/APIC-M3-L3-Server.html)

注意事项：

+ APIC-SERVER-M3/L3的VIC卡连接方式有特殊要求，请按照文档要求来连接Fabric Link到Leaf Switch
+ Infra Pool和Infra VLAN不要和现有网络冲突，尽可能选用未使用的资源来做Infra
+ Tenant Network也不能和Infra Pool重复
+ Infra Pool至少需要/23 Subnet
+ 建议Infra VLAN: 3967

### 4. N9K交换机发现和注册
#### 4.1 发现交换机
登录APIC管理页面，选择"Fabric -> Inventory -> Fabric Membership"

如果N9K交换机已经运行ACI Firmware并且没有其他ACI Fabric信息，那么我们可以先看到APIC服务器直连的Leaf交换机
![Fabric Membership](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-07.png)

如果N9K交换机还是NX-OS模式，需要先进行转换，参考文档如下：

[NX-OS 6.x](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/6-x/upgrade/guide/b_Cisco_Nexus_9000_Series_NX-OS_Software_Upgrade_and_Downgrade_Guide_Release_6x/b_Cisco_Nexus_9000_Series_NX-OS_Software_Upgrade_and_Downgrade_Guide_Release_6x_chapter_010.html)

[NX-OS 7.x](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/7-x/upgrade/guide/b_Cisco_Nexus_9000_Series_NX-OS_Software_Upgrade_and_Downgrade_Guide_Release_7x/Converting_from_Cisco_NX_OS_to_ACI_Boot_Mode.html)

[NX-OS 9.x](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/9-x/upgrade/guide/b_Cisco_Nexus_9000_Series_NX-OS_Software_Upgrade_and_Downgrade_Guide_9x/b_Cisco_Nexus_9000_Series_NX-OS_Software_Upgrade_and_Downgrade_Guide_9x_chapter_0101.html)

[Clean & Reload](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/1-x/troubleshooting/b_APIC_Troubleshooting/b_APIC_Troubleshooting_chapter_01001.html)

[Leaf Hardware Recovery](https://www.cisco.com/c/en/us/support/docs/cloud-systems-management/application-policy-infrastructure-controller-apic/118865-prosbol-leaf-00.html)

+ 注意：转换到ACI模式后，记得再reload一次，确保交换机可以正常boot到ACI模式

#### 4.2 注册交换机
右键交换机，选择"Register"
![Fabric Membership](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-08.png)

输入Node ID和Node Name，然后点击"Update"
![Fabric Membership](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-09.png)

右键交换机，选择"Commission"
![Fabric Membership](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-10.png)

Commission成功后，我们可以看到交换机状态已经变成"Active"并且分配到Infra IP地址
![Fabric Membership](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-11.png)

同时，我们也看到APIC发现了Spine交换机。按照以上步骤继续完成其他交换机注册。

SSH进入APIC CLI界面，运行如下命令检查ACI Fabric状态：

"show controller"
![Fabric Membership](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-12.png)

"show switch"
![Fabric Membership](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-13.png)

"acidiag fnvread"
![Fabric Membership](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-14.png)

"show version"
![Fabric Membership](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-15.png)

### 5. Fabric Level配置
在创建Tenant之前，我们还需要完成如下Fabric配置
#### 5.1 OOB配置
参考官方配置文档：[Inband and OOB Mgmt](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/kb/b_KB_Configuring_Static_Management_Access.html)

![OOB Mgmt](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-18.png)

![OOB Mgmt](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-19.png)

![OOB Mgmt](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-20.png)


#### 5.2 MP-BGP Route Reflector配置
参考官方配置文档：[MP-BGP Route Reflector](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/2-x/L3_config/b_Cisco_APIC_Layer_3_Configuration_Guide/b_Cisco_APIC_Layer_3_Configuration_Guide_chapter_01001.html)

![MP-BGP Route Reflector](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-21.png)

![MP-BGP Route Reflector](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-22.png)

![MP-BGP Route Reflector](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-23.png)

#### 5.3 Disable Remote EP Learn (on border leaf)配置
参考官方文档：[ACI Endpoint WhitePaper](https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/white-paper-c11-739989.html#_Toc529820938)

![Disable Remote EP Learning](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-24.png)

#### 5.4 IP Aging Policy配置
参考官方文档：[IP Aging Policy](https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/white-paper-c11-739989.html#_Toc529820940)

![IP Aging Policy](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-28.png)

#### 5.5 NTP配置
参考官方配置文档：[NTP](https://www.cisco.com/c/en/us/support/docs/cloud-systems-management/application-policy-infrastructure-controller-apic/200128-Configuring-NTP-in-ACI-Fabric-Solution.html)

![NTP](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-25.png)

![NTP](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-26.png)

![NTP](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-27.png)

#### 5.6 Disable Remote EP Learing on Border Leaf
参考官方配置文档：[Disable Remote EP Learning](https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/white-paper-c11-739989.html#_Toc529820938)

![Disable Remote EP Learning](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-30.png)

#### 5.7 Enforce Subnet Check
参考官方配置文档：[Enforce Subnet Check](https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/white-paper-c11-739989.html#_Toc529820939)

![Enforce Subnet Check](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-31.png)

#### 5.8 Enforce Domain Validation
（注意：一旦启用无法关闭）参考官方配置文档：[Enforce Domain Validation](https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/white-paper-c11-737909.html#_Toc6452930)

![Enforce Domain Validation](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-32.png)

![Enforce Domain Validation](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-33.png)

### 6. 检查System Fault信息

完成Fabric配置后，我们再检查一下系统告警信息，如果没有意外信息，那就可以开始Tenant配置了。

![System Faults](https://github.com/yazshen/cisco-aci-troubleshooting/blob/master/resource/new-installation-29.png)

