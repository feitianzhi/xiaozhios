# 小雉系统

#### 项目地址
官网     http://www.feitianzhi.com/  
github  https://github.com/feitianzhi/xiaozhios  
gitee   https://gitee.com/feitianzhi/xiaozhios   
QQ交流群：[869598376](http://www.feitianzhi.com/ "小雉系统")  
微信号：xiaozhios
 
#### **小雉系统简介**
------

“小雉系统”并非是开发操作系统,而是一套服务于软件供应商的产品升级方案;  


#### **需求分析**
------
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;软件一般都运行在操作系统中,软件的运行依赖系统库、驱动，在实际项目中往往遇到以下问题：

 1. 新版本的软件需要兼容项目上所有运行的系统（如ubuntu 16.04、ubuntu 18.04、centos6、centos7、centos8等），否则无法同步给所有客户或是需要打分支为每一个版本的系统单独编译测试；
 2. 早期项目选定一个系统（centos5）可满足所有需求,而centos5不包含新硬件（以前的硬件可能停产，且新硬件的性能更好价格更便宜）的驱动，而为centos5添加硬件驱动的难度（如新驱动必须要4.*以上的内核）成本都非常高昂，最终只能采用centos8系统，这时需要所有的centos5的机器全部重新部署centos8或是软件增加分支同时维护centos5和centos8的版本；
 3. 实验室测试效果与项目运行结果不一致，比如指定ubuntu18.04为开发及运行环境，而系统安装好后需要增加额外的软件，apt 可能从某云拉取软件包进行安装，但因时间问题拉取到的数据可能不一致（某云的软件仓库在更新，其次不掌握在自己手中的数据怎么谈一样，即使知道不一样打印一句日志问题也没有消失），导致实验室和项目上的系统有差异，最终导致软件运行结果不一致；
 4. 从源码自己编译与维护一套系统可以保证实验室与项目系统完全一致，但必须解决驱动适配，硬件测试的问题，其难度大，成本甚至比您要开发的应用程序成本还高；


#### **思考与假设**
------
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我是一名应用开发者，我希望使用新系统--学习新技术，希望可以选择合适的系统（比如某合作单位提供的库是ubuntu18.04编译的，而选centos6系统作为生产环境）要求方案具备如下特性：

 1. 现场与实验室的系统完全一样，包括内核、驱动、虚根、库、文件系统、日志数据（每次系统启动时，所有机器的日志及临时文件也相同，保证启动时绝对可以正常运行）；
 2. 应用开发完成后只需要在开发环境中测试，不需要在各个生产环境中测试（过多的生产环境增加测试成本，测试人员增多，增加沟通成本）；
 3. 开发环境的系统可以随意修改（比如可以更换内核、增加驱动、修个核心库版本、甚至直接跟换系统发行版本把ubuntu18.04更换成cnetos8.1),生产环境可以自动安全无差别的同步为和开发环境一致的系统；
 4. 系统可以退回历史的任意版本，N年前的项目现升级最新系统后有功能异常了，需要立即退回原来版本已恢复生产环境，实验室也可恢复到项目的历史版本进行测试；

#### **抽象与总结**
------

 1. 系统可以来自任意发行版本，要让系统遵循某一规则是不可能的，方案必须兼容所有系统的规则；
 2. 系统可以升级与降级，且升级与降级可以跨越版本，则要求方案与版本无关，版本无连续性要求；
 3. 系统启动后日志及临时文件相同，则要求系统启动时可以自还原为初始状态；
 4. 系统升级需要安全及完全的一致性，则要求方案具备断电保护机制与数据校验机制；


#### **小雉系统设计**
------
 - “小雉系统”采用事务性更新  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在系统中含有两个root filesystem分区：A区及B区(各含有一个系统image)。当A区启动后，它可以用来更新B区。只有B区更新完整后才可以切换过来到B区，否则永远处于A区。反之亦然，我们可以用同样的办法来更新A区。这对很多需要稳定工作的环境的系统来说非常重要，比如更新一个远在路口的webcam。
 - 更强的应用安全    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;系统image只是可读的，任何应用不可以更改它。这样的好处是不至于由于某个应用的bug导致整个系统重启后不能恢复，保证系统重启后绝对正常运行。系统的只读特性让系统每次启动都会恢复到初始状态，保证重启大法的绝对有效。
 - 更随意的软件测试     
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;系统内的所有软件都具有完整的控制权限，可以任意更改和删除文件系统中的数据，在重启后都会恢复为系统image，适合做对操作系统有破坏的测试。
 - 带权限的软件包设计     
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;系统软件包设计权限位，实现一个升级包可用于多客户。


#### **小雉系统使用**
------

1. 下载开源系统镜像（xiaozhios-vmware.zip）：http://www.mym9.com:16080/files/xiaozhios-vmware.zip    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;开源镜像是使用其中一个版本的升级包制作的vmware镜像（用户名：root，密码：12345），在此版本上应用升级包即可把系统升级或降级为升级包中的系统；<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;开源镜像启动后需要配置ip,ip配置方法(工具最新版本：http://www.mym9.com:16080/files/tools.zip )：http://www.feitianzhi.com/boke/index.php/archives/15/  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从镜像中制作系统安装包的方法：</br>http://www.feitianzhi.com/boke/index.php/archives/50/   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用镜像安装到云服务器（也适用于物理机）的方法：</br>http://www.feitianzhi.com/boke/index.php/archives/11/  

2. 下载升级包源码    
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;升级包源码为cpio.gz压缩包，解压方法：gzid -cd xiaozhios-20221212.gz |cpio -idvm    
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;升级包源码源码，最新版本(http://www.mym9.com:16080/files/xiaozhios-openSource.gz ):   
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;历史版本:2022-12-12 https://download.csdn.net/download/zhangrui_fslib_org/87269355   
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;初始化版本；    
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;制作升级包的方法（制作好后使用升级工具升级即可，升级包中的定制见下节）:   
>    
    mkdir -p /opt/os
    cd /opt/os
    //mv xiaozhios-20221212.gz /opt/os/xiaozhios-openSource.gz
    wget -O xiaozhios-openSource.gz http://www.mym9.com:16080/files/xiaozhios-openSource.gz
    gzip -cd xiaozhios-openSource.gz |cpio -idvm
    cd xiaozhios
    ./clean;./run
  生成升级包（如xiaozhi-4.94.1122.upt.jpg)后，把升级包拷贝到windows下，使用下级的工具升级
  ![制作升级包的方法](__pic/0123.png)
  
3. 下载升级工具(工具最新版本：http://www.mym9.com:16080/files/tools.zip )：http://www.feitianzhi.com/boke/index.php/ziyuanxiazai.html    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;升级工具是“小雉系统工具”包中的一个小工具，使用方法可以参考:</br>http://www.feitianzhi.com/boke/index.php/archives/14/        
![小雉系统升级](__pic/0013.png)




#### **小雉系统升级包源码定制**
------

1. xiaozhios/clean：清理制作升级包时产生的临时文件及升级包；    
2. xiaozhios/run：制作升级包的脚本（此脚本要求输入授权码，开源用户输入free）； 
3. xiaozhios/updateFilePackage：打包工具，在run脚本中调用，此工具可以设置密码（默认为12345）,生成的升级包会显示系统的sysKey，升级包在升级时会校验sysKey,不匹配则不能升级，即各用户可以使用不同的密码使自己做出的系统只能应用到自己的系统中；
![sysKey](__pic/0124.png)
4. xiaozhios/platform/x86_64：储存x86_64系统的目录（platform下可以储存多个系统，即可以把多个系统做在一个升级包中）；
5. xiaozhios/platform/x86_64/mask.txt：系统包掩码的定义，升级包按如下规则应用掩码
> 假设系统掩码为m,包掩码为b:    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最高位-表示掩码校验方式(0-表示(m&(b&0x5FFFFF))==(b&0x5FFFFF)时包被需要,1-表示(m&(b&0x5FFFFF))!=0时包被需要);    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;次高位-表示包除按标准方式校验外,如(m&b&0x400000)|((m|b)&0x200000)==0x400000时,包需要进行额外的头判断(即包的前缀与系统的前缀相同);    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;次次高位-强制包的额外头判断恒为真，如(m|b)&0x200000!=0,则包的额外头判断结果为真;  
6. xiaozhios/platform/x86_64/boot：存放kernel、initrd及引导；    
7. xiaozhios/platform/x86_64/base：存放基础软件包，该目录中的软件包是一次性解压后再启动；   
8. xiaozhios/platform/x86_64/extra：存放扩展软件包，在基础软件包解压并启动系统后，再逐个解压进行启动；  
9. 软件包的命名规则,例如zfs-1.0.8.1-7.2-000000    
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;000000:表示权限信息(系统掩码为m,包掩码为b)    
            &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最高位-表示掩码校验方式(0-表示(m&(b&0x5FFFFF))==(b&0x5FFFFF)时包被需要,1-表示(m&(b&0x5FFFFF))!=0时包被需要)    
            &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;次高位-表示包除按标准方式校验外,如(m&b&0x400000)|((m|b)&0x200000)==0x400000时,包需要进行额外的头判断(即包的前缀与系统的前缀相同)    
            &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;次次高位-强制包的额外头判断恒为真，如(m|b)&0x200000!=0,则包的额外头判断结果为真    
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2:表示第二个版本,升级包的版本号是所有包此值的和    
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;7:包的序号,序号小的先解压先启动,序号相同时按名称逆序排序    
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;8.1:表示基于8.1系统制作   
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.0:表示程序的主版本号为1.0 
10. 软件包内部结构，该方案要求软件包内的文件名及目录名不能以"__"打头，内部建议按原版本系统组织（即直接拷贝原系统中文件按原系统目录结构储存）   
>  ![小雉系统软件包内部结构](__pic/0125.png)    
sysKey软件包在/tmp/sysinfo/sysKey文件存放的数据为制作升级包时生成的sysKey数据："c345b45a4ec0a241134c5cefa0dd4aef"
ftp软件包是拷贝原版系统的ftp文件做的一个软件包

#### **小雉系统版权**
------

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"小雉系统"是基于开源linux发行版本定制,未修改linux发行版本源码,能否自由使用遵循对应发行版本的规则;  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"小雉系统"添加的执行程序（升级工具，打包工具，引导程序）版权归作者所有，个人可以免费使用；  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"小雉系统"提供的收费版本为支持服务,其区别如下;


<table>
<thead>
<tr>
<th style="text-align:center;"></th>
<th style="text-align:center;">开源版本</th>
<th style="text-align:center;">收费版本</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center;">升级包大小</td>
<td style="text-align:center;">无限制</td>
<td style="text-align:center;"> 无限制</td>
</tr>
<tr>
<td style="text-align:center;">支持的发行版本</td>
<td style="text-align:center;">所有linux发行版本</td>
<td style="text-align:center;"> 所有linux发行版</td>
</tr>
<tr>
<td style="text-align:center;">是否支持私有升级包制作</td>
<td style="text-align:center;">支持(./updateFilePackage -p 私有密码)</td>
<td style="text-align:center;">支持(./updateFilePackage -p 私有密码)</td>
</tr>
<tr>
<td style="text-align:center;">是否支持差分升级</td>
<td style="text-align:center;">支持</td>
<td style="text-align:center;">支持</td>
</tr>
<tr>
<td style="text-align:center;">是否支持版本回滚</td>
<td style="text-align:center;">支持(24小时内回滚)</td>
<td style="text-align:center;"> 支持(0-100年回滚)</td>
</tr>
<tr>
<td style="text-align:center;">技术服务</td>
<td style="text-align:center;">论坛 </td>
<td style="text-align:center;">技术培训与电话远程支持</td>
</tr>
<tr>
<td style="text-align:center;">技术支持</td>
<td style="text-align:center;">qq群:869598376</td>
<td style="text-align:center;">微信号:xiaozhios</td>
</tr>
</tbody>
</table>

#### **小雉系统集成应用概览**
------

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本集成应用是基于”小雉系统“集成多种应用做成的一套服务于视频应用的视频系统，总大小约230M，相关网址：http://www.feitianzhi.com/boke/index.php/ziyuanxiazai.html  
<table>
<thead>
<tr>
<th style="text-align:center;">应用类别</th>
<th style="text-align:center;">应用名</th>
<th style="text-align:center;">应用描述</th>
<th style="text-align:center;">大小</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:center;">引导</td>
<td>grub2</td>
<td>引导</td>
<td style="text-align:center;">3.1M</td>
</tr>
<tr>
<td style="text-align:center;">内核</td>
<td>kernel</td>
<td >内核</td>
<td style="text-align:center;">7.8M</td>
</tr>
<tr>
<td style="text-align:center;">虚根</td>
<td>initrd</td>
<td>虚根</td>
<td style="text-align:center;">14M</td>
</tr>
<tr>
<td rowspan="13" style="text-align:center;">基础包(本类全部解压完成后启动系统)</td>
<td>system</td>
<td>基础系统</td>
<td style="text-align:center;">15.4M</td>
</tr>
<tr>
<td>libmnl</td>
<td>NAT相关</td>
<td style="text-align:center;">0.12M</td>
</tr>
<tr>
<td>driver-wireless</td>
<td>无线驱动</td>
<td style="text-align:center;">0.32M</td>
</tr>
<tr>
<td>driver</td>
<td>基础驱动</td>
<td style="text-align:center;">3.2M</td>
</tr>
<td>udev</td>
<td>设备自动发现</td>
<td style="text-align:center;">1.1M</td>
</tr>
<tr>
<td>udev</td>
<td>设备自动发现</td>
<td style="text-align:center;">1.1M</td>
</tr>
<tr>
<td>nvidia</td>
<td>nvidia驱动</td>
<td style="text-align:center;">43M</td>
</tr>
<tr>
<td>fsServer</td>
<td>升级程序的服务端，含日志管理，watchdog</td>
<td style="text-align:center;">5.4M</td>
</tr>
<tr>
<td>telnet</td>
<td>telnet服务端和客户端</td>
<td style="text-align:center;">0.08M</td>
</tr>
<tr>
<td>ssh</td>
<td>ssh服务端</td>
<td style="text-align:center;">0.08M</td>
</tr>
<tr>
<td>network</td>
<td>网路小工具,比如网桥工具</td>
<td style="text-align:center;">0.02M</td>
</tr>
<tr>
<td>iscsiclient</td>
<td>iscsi硬盘挂载工具</td>
<td style="text-align:center;">0.57M</td>
</tr>
<tr>
<td>zabbix</td>
<td>zabbix采集程序</td>
<td style="text-align:center;">1.1M</td>
</tr>
<tr>
<td rowspan="48" style="text-align:center;">扩展包(解压一个启动一个)</td>
<td>vsftpd</td>
<td>ftp服务器</td>
<td style="text-align:center;">0.08M</td>
</tr>
<tr>
<td>zfs</td>
<td>zfs文件系统</td>
<td style="text-align:center;">1.5M</td>
</tr>
<tr>
<td>smb</td>
<td>smb服务器</td>
<td style="text-align:center;">8.1M</td>
</tr>
<tr>
<td>nginx</td>
<td>nginx服务器</td>
<td style="text-align:center;">0.58M</td>
</tr>
<tr>
<td>phpext</td>
<td>php扩展</td>
<td style="text-align:center;">0.63M</td>
</tr>
<tr>
<td>php</td>
<td>php服务器</td>
<td style="text-align:center;">4.4M</td>
</tr>
<tr>
<td>mariadb</td>
<td>mariadb服务器</td>
<td style="text-align:center;">6.9M</td>
</tr>
<tr>
<td>git</td>
<td>git服务器</td>
<td style="text-align:center;">3.4M</td>
</tr>
<tr>
<td>virtual</td>
<td>虚拟服务器(kvm虚拟化),可在系统内跑其他linux或windows虚拟机</td>
<td style="text-align:center;">3.4M</td>
</tr>
<tr>
<td>ossfs</td>
<td>使用阿里云的对象储存</td>
<td style="text-align:center;">2.2M</td>
</tr>
<tr>
<td>lvs</td>
<td>lvs文集系统管理工具</td>
<td style="text-align:center;">0.1M</td>
</tr>
<tr>
<td>iptables</td>
<td>防火墙</td>
<td style="text-align:center;">0.29M</td>
</tr>
<tr>
<td>pppd</td>
<td>pptp、l2tp 服务器需要的组件</td>
<td style="text-align:center;">0.21M</td>
</tr>
<tr>
<td>pptpd</td>
<td>pptpd服务器</td>
<td style="text-align:center;">0.05M</td>
</tr>
<tr>
<td>ipsec</td>
<td>l2tp 服务器需要的组件</td>
<td style="text-align:center;">4.6M</td>
</tr>
<tr>
<td>xl2tpd</td>
<td>l2tp服务器</td>
<td style="text-align:center;">0.09M</td>
</tr>
<tr>
<td>jpegipp</td>
<td>intel的jpg编解码库</td>
<td style="text-align:center;">13M</td>
</tr>
<tr>
<td>ffmpeg</td>
<td>ffmpeg库，拥有h264,h265解码</td>
<td style="text-align:center;">13M</td>
</tr>
<tr>
<td>zos</td>
<td>视频软件，用于rtsp、rtmp、hls、gb28181协议的直播、储存、回放，并带ai分析</td>
<td style="text-align:center;">8.6M</td>
</tr>
<tr>
<td>mail</td>
<td>邮件服务器,用于收发邮件，并提供web邮箱，web域名管理</td>
<td style="text-align:center;">18M</td>
</tr>
<tr>
<td>x265</td>
<td>h265编码库</td>
<td style="text-align:center;">0.79M</td>
</tr>
<tr>
<td>x264</td>
<td>h264编码库</td>
<td style="text-align:center;">0.81M</td>
</tr>
<tr>
<td>wireshark</td>
<td>网路抓包与分析工具</td>
<td style="text-align:center;">20M</td>
</tr>
<tr>
<td>valgrind</td>
<td>内存调试，程序bug分析工具</td>
<td style="text-align:center;">1.2M</td>
</tr>
<tr>
<td>tools-oem</td>
<td>客户定制的特殊工具</td>
<td style="text-align:center;">0.44M</td>
</tr>
<tr>
<td>tools</td>
<td>常用工具，如ping、xdd、arp等</td>
<td style="text-align:center;">0.49M</td>
</tr>
<tr>
<td>tc</td>
<td>流控程序</td>
<td style="text-align:center;">0.22M</td>
</tr>
<tr>
<td>rpcapd</td>
<td>流量镜像程序，允许在windows下使用wireshark对linux系统远程抓包</td>
<td style="text-align:center;">0.10M</td>
</tr>
<tr>
<td>qemu-nbd</td>
<td>虚拟磁盘，如vmdk,qcow的挂载程序</td>
<td style="text-align:center;">0.64M</td>
</tr>
<tr>
<td>nvidia-tool</td>
<td>nvidia工具，用于管理与查看gpu</td>
<td style="text-align:center;">0.83M</td>
</tr>
<tr>
<td>ntp</td>
<td>ntp服务器</td>
<td style="text-align:center;">1.8M</td>
</tr>
<tr>
<td>ntfs-3g</td>
<td>ntfs文件系统管理程序</td>
<td style="text-align:center;">0.29M</td>
</tr>
<tr>
<td>nmap</td>
<td>网络扫描程序</td>
<td style="text-align:center;">0.91M</td>
</tr>
<tr>
<td>nft</td>
<td>取代iptable的网络框架</td>
<td style="text-align:center;">0.25M</td>
</tr>
<tr>
<td>ncat</td>
<td>tcp端口代理程序</td>
<td style="text-align:center;">0.2M</td>
</tr>
<tr>
<td>mysql-upgrade</td>
<td>mariadb数据库升级工具，如centos7建立的数据库，在centos8可能需要升级</td>
<td style="text-align:center;">1.8M</td>
</tr>
<tr>
<td>mariadbtool</td>
<td>mmariadb的工具，比如命令行连接数据库工具</td>
<td style="text-align:center;">1.1M</td>
</tr>
<tr>
<td>journalctl</td>
<td>systemd的日志查看工具</td>
<td style="text-align:center;">0.03M</td>
</tr>
<tr>
<td>ipcs</td>
<td>共享内存管理工具</td>
<td style="text-align:center;">0.03M</td>
</tr>
<tr>
<td>iopp</td>
<td>磁盘io监测工具</td>
<td style="text-align:center;">0.01M</td>
</tr>
<tr>
<td>iopp</td>
<td>磁盘io监测工具</td>
<td style="text-align:center;">0.01M</td>
</tr>
<tr>
<td>opencv</td>
<td>opencv库</td>
<td style="text-align:center;">5.0M</td>
</tr>
<tr>
<td>iftop</td>
<td>网路流量监控</td>
<td style="text-align:center;">0.08M</td>
</tr>
<tr>
<td>ifstat</td>
<td>网路接口流量监控</td>
<td style="text-align:center;">0.03M</td>
</tr>
<tr>
<td>htop</td>
<td>进程监控程序</td>
<td style="text-align:center;">0.08M</td>
</tr>
<tr>
<td>gitweb</td>
<td>web呈现本机git服务器的项目</td>
<td style="text-align:center;">0.94M</td>
</tr>
<tr>
<td>ftp</td>
<td>ftp客户端</td>
<td style="text-align:center;">0.04M</td>
</tr>
<tr>
<td>dhsdk</td>
<td>大华的sdk开发包</td>
<td style="text-align:center;">9.2M</td>
</tr>
</tbody>
</table>




