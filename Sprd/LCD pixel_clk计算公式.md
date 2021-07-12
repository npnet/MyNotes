### LCD  pixel_clk计算公式

#### 一.   LCD参数

1. VBP(vertical back porch)：表示在一帧图像开始时，垂直同步信号以后的无效的行数，对应驱动中的upper_margin；

   VFP(vertical front porch)：表示在一帧图像结束后，垂直同步信号以前的无效的行数，对应驱动中的lower_margin；

   VSPW(vertical sync pulse width)：表示垂直同步脉冲的宽度，用行数计算，对应驱动中的vsync_len；

   HBP(horizontal back porch)：表示从水平同步信号开始到一行的有效数据开始之间的VCLK的个数，对应驱动中的left_margin；

   HFP(horizontal front porch)：表示一行的有效数据结束到下一个水平同步信号开始之间的VCLK的个数，对应驱动中的right_margin；

   HSPW(horizontal sync pulse width)：表示水平同步信号的宽度，用VCLK计算，对应驱动中的hsync_len；
   

#### 二.  pixel clock计算

如果配置是时钟源是dpi_clk_src中一个，直接使用，然后根据ROUND(a, b)计算分配分配系数

```c
//bsp/kernel/kernel4.14/drivers/gpu/drm/sprd/sharkl3/global_dispc.c （展讯9863A平台）
static const u32 dpi_clk_src [] = {
	128000000,
	153600000,
	192000000
};
```

```c
static const u32 dpi_clk_src[] = {
	96000000,
	128000000,
	153600000,
	192000000
};

static u32 calc_dpi_clk_src(u32 pclk)
{
	int i;

	for (i = 0; i < ARRAY_SIZE(dpi_clk_src); i++) {
		if ((dpi_clk_src[i] % pclk) == 0)
			return dpi_clk_src[i];
	}

	pr_err("calc DPI_CLK_SRC failed, use default\n");
	return 96000000;
}
#define ROUND(a, b) (((a) + (b) / 2) / (b))  

static int calc_dpi_clk(struct sprd_dispc *dispc,
			       u32 *new_pclk, u32 pclk_src,
			       u32 new_val, int type)
{
	switch (type) {
	case SPRD_FORCE_FPS:
	case SPRD_DYNAMIC_FPS:
		if (new_val < LCD_MIN_FPS || new_val > LCD_MAX_FPS) {
			pr_err
			    ("Unsupported FPS. fps range should be [%d, %d]\n",
			     LCD_MIN_FPS, LCD_MAX_FPS);
			return -EINVAL;
		}
		pclk = hpixels * vlines * new_val;
		divider = ROUND(pclk_src, pclk);
		*new_pclk = pclk_src / divider;
		if (pclk_src % divider)
			*new_pclk += 1;
		panel->fps = new_val;
		break;

}
```



#### 三.fps计算

```c
static ssize_t actual_fps_show(struct device *dev,
                        struct device_attribute *attr,
                        char *buf)
{
        struct sprd_dpu *dpu = dev_get_drvdata(dev);
        struct videomode vm = dpu->ctx.vm;
        uint32_t act_fps_int, act_fps_frac;
        uint32_t total_pixels;
        int ret;

        total_pixels = (vm.hsync_len + vm.hback_porch +
                        vm.hfront_porch + vm.hactive) *
                        (vm.vsync_len + vm.vback_porch +
                        vm.vfront_porch + vm.vactive);

        act_fps_int = vm.pixelclock / total_pixels;
        act_fps_frac = vm.pixelclock % total_pixels;
        act_fps_frac = act_fps_frac * 100 / total_pixels;

        ret = snprintf(buf, PAGE_SIZE, "%u.%u\n", act_fps_int, act_fps_frac);

        return ret;
}
static DEVICE_ATTR_RO(actual_fps);
```

如参数：
1206*2138*60=154705680
ROUND(a, b)=1,分频系数为1,那时钟就是153600000
fps≈153600000/(1206*2138)≈60,满足要求

如这组参数,
1547*688*60=63860160
ROUND(a, b)=2,频率就是153600000/2=76800000，
fps≈76800000/(1547*688)≈72fps，不满足要求
但更换64000000时钟源，
ROUND(a, b)=2,频率就是128000000/2=64000000，
fps≈64000000/(1547*688)≈60fps,满足要求

   need_clock是视频硬件在显示器上绘制像素的速率，公式如下（单位Hz）

```
need_clock  = (x向分辨率+左空边+右空边+HSYNC长度)* (y向分辨率+上空边+下空边+YSYNC长度)*整屏的刷新率fps
```

公式如下：

```
1.need_clock = (Width + HSPW + HBP + HFP)*(Height+VSPW+ VBP + VFP) * fps
2.dividor = dpi_clock_src/need_clock;
3.if((dpi_clock_src - dividor*need_clock) > (need_clock/2)){
    dividor += 1;
}
```

计算出来的need_clock是一个具体值，需要获取源码中给定的近似值，需要将所计算need_clock乘以2，(乘以2是因为mipi为差分信号，双边沿采样，上升沿和下降沿分别可以传输一个数据)，将2*need_clock的值与dotclockdpi_clk_src数组内的值进行比较，选取最近似的值，再除以2即可得到近似的need_clock。

#### 四.phy_freq的计算

(DPI_CLK_SRC/DIVIDOR)x 3x 8 bit < phy_feq x lane_num x 0.9
考虑到MIPI传输过程中可能会出现传输错误，实际设置的值可以再加大10%

对于下面屏的配置算出来的是
(153600000*3*8)/(4*0.9)*1.1=1126400000,表格算处理的是1105*1000kbs

