# lenovo-miix-520-hackintosh
[README](README.md) | [中文文档](README_zh.md)

macos on lenovo miix 520 is almost perfect
支持macos10.14.x、10.15.x、11.0.x。

miix 520 接近完美黑苹果，[点击查看远景贴](http://bbs.pcbeta.com/viewthread-1795824-1-1.html)

#### 目录

1. 配置信息
2. 系统运行截图
3. 正常工作
4. 不正常工作
5. bug与解决方式
	- bug1:触摸板与触摸屏不能同时使用
	- bug2:睡眠唤醒后键盘失效(重新拔插后正常)
6. 更新日志
7. 常见问题解答
	- 1:安装前的准备有哪些？
	- 2:为什么安装后触屏不能用？
	- 3:为什么睡眠唤醒后键盘与触摸板就不能用了？
	- 4:我可以随意更新系统吗？
	- 5:读卡器能驱动吗？
	- 6:无线网卡怎么驱动？
	- 7:为什么转换为以OpenCore更新为主？
	- 8:如何禁用OpenCore开机声音？

## 1.配置信息

- 品牌型号：联想miix 520
- cpu：i5 8250u 
- 显卡：uhd620
- 内存：16G
- 声卡：alc298
- 无线网卡：bcm94352z
- 屏幕大小：12.2寸
- 分辨率：1920x1200
- NVME硬盘：samsung pm961 1TB
- bios：6ncn35ww

## 2.系统运行截图

- OpenCore多系统选择界面：
<div align=center><img src="https://raw.githubusercontent.com/acai66/lenovo-miix-520-hackintosh-CLOVER/master/Resource/images/OpenCoreBootloader.png" width="95%" /></div>

- Mac OS运行图：
<div align=center><img src="https://raw.githubusercontent.com/acai66/lenovo-miix-520-hackintosh-CLOVER/master/Resource/images/about.png" width="95%" /></div>

## 3.正常工作

1. 声显网三卡：OK
2. usb：OK
3. 电量显示：OK
4. 亮度调节：OK
5. 变频：OK
6. 盒盖睡眠 开盖唤醒：OK
7. 触摸屏、手写笔：OK
8. 蓝牙：OK
9. usb键盘、鼠标唤醒：OK
10. SD读卡器：需要者请手动添加，看 [常见问题解答8](https://github.com/acai66/lenovo-miix-520-hackintosh-CLOVER#8读卡器能驱动吗)

## 4.不正常工作

1. I2C的重感、摄像头（无解）
2. ~~SD读卡器（我的是LTE版本的，没有sd模块，这个无法测试，可能有解~~
3. iMessage（有解）
4. 指纹识别（无解）

## 5.bug与解决方式

### bug1:触摸板与触摸屏不能同时使用

这个bug应该来自于VoodooI2CHID.kext驱动，因为它会让触摸板加载这个驱动，而触摸板加载这个驱动后就没法使用了，所以解决这个bug的方法是修改VoodooI2CHID.kext驱动，让它不能识别触摸板，具体修改方法是删掉VoodooI2CHID.kext/Contents/Info.plist里的这一段：

    <key>VoodooI2CHIDDevice Multitouch HID Event Driver</key>
    <dict>
    <key>CFBundleIdentifier</key>
    <string>com.alexandred.VoodooI2CHID</string>
    <key>DeviceUsagePairs</key>
    <array>
    <dict>
    <key>DeviceUsage</key>
    <integer>4</integer>
    <key>DeviceUsagePage</key>
    <integer>13</integer>
    </dict>
    <dict>
    <key>DeviceUsage</key>
    <integer>5</integer>
    <key>DeviceUsagePage</key>
    <integer>13</integer>
    </dict>
    <dict>
    <key>DeviceUsage</key>
    <integer>2</integer>
    <key>DeviceUsagePage</key>
    <integer>13</integer>
    </dict>
    </array>
    <key>IOClass</key>
    <string>VoodooI2CMultitouchHIDEventDriver</string>
    <key>IOProbeScore</key>
    <integer>200</integer>
    <key>IOProviderClass</key>
    <string>IOHIDInterface</string>
    </dict>

我上传的clover里集成的VoodooI2CHID.kext默认已经去掉了这一段代码了，这里介绍这个bug，是为了避免更新VoodooI2C系列驱动时忘记修改驱动而导致触摸板无法使用的问题。


### bug2:睡眠唤醒后键盘失效(重新拔插后正常)

这是个奇葩的bug，可能和键盘硬件有关，经过搜索，发现不少用户遇到了唤醒后鼠标失效、键盘失效的问题，重新拔插后又能正常使用了，针对这个bug，解决办法就是安装sleepwatcher来监控系统的睡眠唤醒，在电脑唤醒时执行一条重连usb设备的命令，该补丁包默认适合miix 520的键盘bug，如果想要用到别的电脑上解决重连usb的问题，需要修改/usr/local/acai/patch，里面的0x14500000是miix 520键盘的usb口的地址。

经过测试，按照如下步骤就能解决miix 520的键盘失效问题：

1. 安装brew，终端执行如下命令


    `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

2. 安装sleepwatcher，终端执行如下命令

	`brew install sleepwatcher`

3. 下载解压补丁包 : 解决唤醒后需要拔插键盘问题的方案.zip
4. 终端进入补丁包目录，执行如下命令

	`sudo sh ./patch.sh`

## 6.更新日志

### v2.5 -- 2020-12-15
- 更新最新OpenCore、驱动，支持最新 macOS 11.0.1.
- 不再支持CLOVER.
- 添加Intel网卡与蓝牙驱动，测试性支持。
- 启用Apple Secure Boot.

### v2.5 beta -- 2020-07-02
- 更新最新自编译OpenCore、驱动，支持最新 macOS 11.0 (10.16).
- 修复OpenCore图形化界面快捷键不可用问题.
- 说明:
  1. 此次为测试版.
  2. 此次未更新CLOVER.

### v2.4 -- 2020-05-22
- 更新最新自编译opencore与各驱动版本，支持最新 macOS 10.15.4
- 图形化opencore界面支持热键与设置默认启动项，默认倒计时依旧5秒。
- 修复opencore引导后设置->节能里缺少电池选项

### v2.3 -- 2020-04-09
- 更新最新自编译opencore与各驱动版本，支持最新 macOS 10.15.4
- 支持图形化opencore界面。

### v2.2 -- 2020-03-13
- 更新最新自编译opencore与各驱动版本，支持最新 macOS 10.15.3
- 支持opencore开机声音。

### v2.1
- 更新最新自编译opencore与各驱动版本，支持最新 macOS 10.15.2
- 大量优化opencore配置：
    + 修复一处acpi中关于电源键的问题
    + opencore引导界面支持上下方向切换光标，回车键选中
    + opencore支持按住ctrl加回车键设置默认启动项，ctrl加系统序号也行
    + 支持mac快捷键，例如启动时按住win键加v实现啰嗦模式，其余快捷键参考mac启动快捷键
    + 设置show picker为false后可以通过在启动时按住Esc显示picker
- 本次未更新clover，下次有时间再更新，同时翻译成英文与发布release。


### v2.0 -- 2020-01-02
- 更新最新自编译clover、opencore与各驱动版本，支持最新 macOS 10.15.2
- 大量优化opencore配置，从此版开始，以后主更新opencore引导
- 修正usb配置，修复ipad充电问题
- 修改wifi 国家为NZL，将支持更多5ghz wifi频段，同时不影响13个2.4ghz频段。
- 更新 解决唤醒后需要拔插键盘问题的方案.zip ，测试支持macOS 10.15
- 说明1: oc启动引导如果没有自动扫描出windows或linux启动项，请手动自定义添加引导配置到oc的config.plist里。
- 说明2: oc里的acpi等补丁会对所有系统生效，所以由oc引导的windows会把机型识别为MacBook pro，并且也将支持原生的macOS的启动磁盘切换(待测试)。
- 说明3: 最近为了hotpatch oc化做了很多修改，我这边目前正常，如有异常bug等请发issue，我是美版miix 520，bios版本6ncn35ww，hotpatch补丁有部份是依赖bios里dsdt表的，所以bios版本最好一致。

### v1.9 -- 2019-08-23
- 更新最新自编译clover与各驱动版本，支持最新 macOS 10.15 beta6
- 更新最新自编译OpenCore引导
- 进一步精简冗余hotpatch补丁
- 添加ssdt-usbx.aml，避免潜在的usb电源问题
- 说明1：更新beta6系统后如果触屏失效，请运行kext utility修复权限 重建缓存
- 说明2：clover与opencore的config.plist文件都添加了brcmfx-country=CN来支持2.4ghz的12和13 wifi频段，但会造成5ghz wifi频段缺少的问题，brcmfx-country=US支持更多的5ghz 频段，但没有12和13频段，各国家wifi频段参考[wiki](https://zh.wikipedia.org/wiki/WLAN信道列表)，实际可用频段请以自己实测为准。

### v1.8 -- 2019-07-16
- 更新clover与各驱动版本，支持最新 macOS 10.15 beta3，OC引导下个版本更新
- 修复新系统下睡眠唤醒键盘失效问题

### v1.7 -- 2019-05-29
 - 移除默认的acpi/windows/dsdt.aml,避免造成无法引导windows的问题
 - 添加英文readme

### v1.6 -- 2019-05-15

- 修正电源管理、声卡layout id补丁，转用ssdt hotpatch或devices属性注入；
- 添加cpufriend，动态注入变频信息，将原先的最低频率1.3ghz调成800mhz，空闲时更加省电；
- 精简部分冗余驱动；
- 更新驱动与clover版本；
- 取消仿冒6代u与核显，据说可以实现显卡硬解；
- 修正电池dsdt补丁逻辑，这个修复感受不到有任何改进；
- 修复上次更新时忘记去掉-v与默认启动选项；
- ACPI/patched/WINDOWS/下提供dsdt.aml，以修复美版miix520在新版bios下无法识别指纹的问题(针对windows系统)；
- 测试添加OpenCore引导，还不完善，不推荐日常使用，仅供研究；

### v1.5 -- 2019-04-05

- 修复触屏多指手势bug，触屏更好用

### v1.4 -- 2019-03-29

- 更新clover版本。
- 更新驱动版本。
- 精简driver64uefi与kexts里的冗余文件。
- 修复显卡补丁与禁用mac i2c原生驱动补丁。

### v1.3 -- 2019-01-26

- 更新clover版本。
- 添加开机声音驱动文件与声音资源文件，如何使用 请看 [常见问题解答7](https://github.com/acai66/lenovo-miix-520-hackintosh-CLOVER#7开机声音怎么开启与关闭)。
- 修改电池型号信息为miix 520，无关紧要的一条更新。
- 默认不集成sd卡驱动，如需要sd卡驱动，请看 [常见问题解答8](https://github.com/acai66/lenovo-miix-520-hackintosh-CLOVER#8读卡器能驱动吗)。


### v1.2 -- 2018-12-30

- 启用VirtualSMC.kext，放弃fakesmc.kext。
- 启用SMCBatteryManager.kext，放弃ACPIBatteryManager.kext。
- 更新clover版本为4831。
- 更新kexts内各驱动版本。
- 显卡仿冒19168086，解决仪表盘添加小部件时水波纹花屏问题。
- CLOVER完全重新从我自用的提取，不是增量更新。
- 测试添加SD读卡器驱动。

### v1.1 -- 2018-10-14

- 亮度调节方式更改为AppleBacklightFixup，[详细信息点击查看](https://bitbucket.org/RehabMan/applebacklightfixup)。
- 设置默认启动上次启动的系统启动项，倒计时为10秒。

### v1.0 -- 2018-10-11

- 初始版本.

## 7.常见问题解答

### 1:安装前的准备有哪些？
1. U盘一个，容量最好大于8G；电脑硬盘预留一个分区，用来装mac；硬盘的esp分区大小大于等于200m，否则安装mac系统时抹盘会失败；
2. bios里磁盘模式为ahci，启动模式为uefi，安全启动关闭。
3. dmg格式的macos10.14.x镜像，百度 黑果小兵 可以找到新版clover镜像。
4. 刻录镜像到u盘的工具，[etcher](https://www.balena.io/etcher/)
5. 刻录完成后，替换u盘esp分区的OC文件夹为该项目的OC。
6. 安装完成后，添加硬盘启动引导。


### 2:为什么安装后触屏不能用？
A: 可能是与系统内置i2c驱动有冲突或者第一次启动时驱动注入不成功有关，安装后完成，请重启1·2次，如果触屏还是不能使用，请手动移除sle里的AppleIntelLpssI2C.kext和AppleIntelLpssI2CController.kext，并运行kext utility来修复权限重建缓存，再重启测试触屏。

### 3:为什么睡眠唤醒后键盘与触摸板就不能用了？
A: 请看readme里 bug与解决方式部分


### 4:我可以随意更新系统吗？
A: 理论上可以直接更新系统.


### 5:读卡器能驱动吗？
A: 默认不再添加读卡器驱动，原因是这个不稳定，会有启动问题或别的问题，而且需求一般不是很明显。该项目的这个Sinetek-rtsx.kext就是读卡器驱动，据说这个不能用或者有别的问题，如果有问题，请自行编译该驱动，[原驱动项目开源地址](https://github.com/sinetek/Sinetek-rtsx)

### 6:无线网卡怎么驱动？
A: v2.5开始添加intel wifi和蓝牙驱动，当前处于测试阶段，为了稳定使用，建议更换bcm94352z联想版。不建议使用usb wifi，因为不稳定，不方便。

### 7:为什么转换为以OpenCore更新为主？
A: 为了完成更加接近原生的体验以及更加完善完美的acpi补丁与hotpatch补丁，选择了oc启动引导，该引导工具会平等的对待各个操作系统，hotpatch补丁、smbios等将会影响它启动的各个系统，包括win与linux，所以对hotpatch补丁的完善提出了更大的要求。该引导还有很多特性，尽管可能目前配置还没完善，但不得不说这是个优秀的引导工具，开机快、支持macOS bless、支持mac原生快捷组合键以及未来可能会支持安全启动。

### 8:如何禁用OpenCore开机声音？
A: v2.2版本开始支持OC开机声音，如需禁用声音，可以通过修改oc配置文件，将AudioSupport设置为false，更多声音设置(声音大小、类型等)请参阅OC官方文档。

