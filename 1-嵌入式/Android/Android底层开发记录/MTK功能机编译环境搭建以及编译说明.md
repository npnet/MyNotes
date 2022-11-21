### MTK功能机编译环境搭建以及编译说明

#### 1.准备阶段

- 需要windowXP系统
- ADS1.2
- RVCT3.1
- rvds4.crack（用来RVCT破解）
- Perl(版本适合winxp,我用的[5.20版](http://www.elmonton.net/programacion/programas/perl/activeperl-v5-20-2-2001/2645/))
- excel
- 7-zip
- armar(1021版本)

#### 2. 安装教程

注意第一步需要先装ADS1.2，然后再装RVCT3.1.

安装RVCT3.1参考这篇：[Windows 7 安装RVCT 3.1 ](http://segon.win/2017/09/30/rvct-for-windows7/) 然后使用armar(1021版)破解就OK

其他的正常安装就好了。

#### 3.编译说明

使用平台为MT6276，目前只试过全编译，单独编译没能成功，没测试过。

- 解压代码后进入make目录，查看主make文件，比如为：CKT76_11B_HSPA.mak

- 敲命令：make CKT76_11B HSPA new 进行全编译


