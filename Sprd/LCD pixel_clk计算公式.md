### LCD  pixel_clk计算公式

一.   LCD参数

1. VBP(vertical back porch)：表示在一帧图像开始时，垂直同步信号以后的无效的行数，对应驱动中的upper_margin；

   VFP(vertical front porch)：表示在一帧图像结束后，垂直同步信号以前的无效的行数，对应驱动中的lower_margin；

   VSPW(vertical sync pulse width)：表示垂直同步脉冲的宽度，用行数计算，对应驱动中的vsync_len；

   HBP(horizontal back porch)：表示从水平同步信号开始到一行的有效数据开始之间的VCLK的个数，对应驱动中的left_margin；

   HFP(horizontal front porch)：表示一行的有效数据结束到下一个水平同步信号开始之间的VCLK的个数，对应驱动中的right_margin；

   HSPW(horizontal sync pulse width)：表示水平同步信号的宽度，用VCLK计算，对应驱动中的hsync_len；
   

二.  DOTCLOCK

   dotclock是视频硬件在显示器上绘制像素的速率，公式如下（单位Hz）

```
dotclock = (x向分辨率+左空边+右空边+HSYNC长度)* (y向分辨率+上空边+下空边+YSYNC长度)*整屏的刷新率
```

即

```
dotclock = (Width + HSPW + HBP + HFP)*(Height+VSPW+ VBP + VFP) * Frame
```

   计算出来的dotclock是一个具体值，需要获取源码中给定的近似值，需要将所计算dotclock乘以2，(乘以2是因为mipi为差分信号，双边沿采样，上升沿和下降沿分别可以传输一个数据)，将2*dotclock的值与dotclockdpi_clk_src数组内的值进行比较，选取最近似的值，再除以2即可得到近似的dotclock。

   例如，WIDTH=720，HEIGHT=1280，HFP=130，HBP=64，HSYNC=20，VFP=8，VBP=30，VSYNC=2，dotclockdpi_clk_src数组为

```c
//bsp/kernel/kernel4.14/drivers/gpu/drm/sprd/sharkl3/global_dispc.c （展讯9863A平台）
static const u32 dpi_clk_src [] = {
	128000000,
	153600000,
	192000000
};
```

则dotclock = (720+130+64+20) * (1280+8+30+2) * 60 = 73,972,800 ， 73,972,800 * 2 = 147,945,600 ≈ 153600000 ，

153600000 / 2 = 76,800,000Hz ， 即可计算出dotclock为 76,800,000Hz 。



