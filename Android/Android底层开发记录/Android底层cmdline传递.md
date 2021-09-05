## Android底层cmdline传递

### 目的

在传递cmdline的时候，有一个mrdump，想通过增加这个mrdump，传递多几组数据，让kernel去reserved这些内存。

平台： MT6757

版本： android7.0

### 操作

#### preloader部分

1. 主要文件

   platform\mt6757\src\drivers\platform.c

   platform\mt6757\src\drivers\inc\platform.h

2. 主要数据结构

   platform.h文件添加以下内容：

   ~~~c
   // add by zhanye
   #define BOOT_TAG_RESERVE_MEM    0x8861001A
   
   #define MAX_RESERVED_COUNT 12
   
   typedef struct _MEM_BLOCK{
   	u32  addr;
   	u32  size;
   }RESERVED_BLOCK;
   
   typedef struct _RESERVED_MEM{
   	u32 cnt; //max 12
   	RESERVED_BLOCK block[MAX_RESERVED_COUNT];
   }RESERVED_MEM;
   
   struct boot_tag_reserved_mem {
      RESERVED_MEM dram_reserved_mem;
   };
   typedef struct {
       struct boot_tag_header hdr;
       union {
           struct boot_tag_boot_reason boot_reason;
           struct boot_tag_boot_mode boot_mode;
           struct boot_tag_meta_com meta_com;
           struct boot_tag_log_com log_com;
           struct boot_tag_mem mem;
           struct boot_tag_md_info md_info;
           struct boot_tag_boot_time boot_time;
           struct boot_tag_da_info da_info;
           struct boot_tag_sec_info sec_info;
           struct boot_tag_part_num part_num;
           struct boot_tag_part_info part_info;
           struct boot_tag_eflag eflag;
           struct boot_tag_ddr_reserve ddr_reserve;
           struct boot_tag_dram_buf dram_buf;
           struct boot_tag_sram_info sram_info;
   	    struct boot_tag_ptp ptp_volt;
           struct boot_tag_imgver_info imgver_info;
   	    struct boot_tag_max_cpus max_cpus;
   	    struct boot_tag_bat_info bat_info;
           // add  by zhanye
           struct boot_tag_reserved_mem reserved_mem;
       } u;
   }boot_tag;
   ~~~

3. 主要函数

   platform.c文件添加以下内容：

   ~~~c
   void platform_set_boot_args_by_atag(unsigned *ptr)
   {
       ...
       // add by zhanye
       tags = ptr;
       tags->hdr.size = boot_tag_size(boot_tag_reserved_mem);
       tags->hdr.tag = BOOT_TAG_RESERVE_MEM;
       tags->u.reserved_mem.dram_reserved_mem.cnt = g_reserved_mem.cnt;
       for(i=0;i<tags->u.reserved_mem.dram_reserved_mem.cnt;i++)
       {
           tags->u.reserved_mem.dram_reserved_mem.block[i].addr = g_reserved_mem.block[i].addr;
           tags->u.reserved_mem.dram_reserved_mem.block[i].size = g_reserved_mem.block[i].size;
       }
       ptr += tags->hdr.size;
       ...
       ...
   }
   ~~~

#### LK部分

1. 主要文件

   platform\mt6757\include\platform\boot_mode.h

   platform\mt6757\platform.c

   app\mt_boot\aee\mrdump_rsvmem.c

2. 主要数据结构（要与preloader的对应上）

   boot_mode.h中添加以下内容：

   ~~~c
   #define MAX_RESERVED_COUNT 12
   
   typedef struct _MEM_BLOCK{
   	u32  addr;
   	u32  size;
   }RESERVED_BLOCK;
   
   typedef struct _RESERVED_MEM{
   	u32 cnt; //max 12
   	RESERVED_BLOCK block[MAX_RESERVED_COUNT];
   }RESERVED_MEM;
   
   typedef struct {
   	u32	maggic_number;
   	BOOTMODE	boot_mode;
   	u32	e_flag;
   	u32	log_port;
   	u32	log_baudrate;
   	u8	log_enable;
   	u32	log_dynamic_switch;
   	u8	part_num;
   	u8	reserved[2];
   	u32	dram_rank_num;
   	u64	dram_rank_size[4];
   	mblock_info_t	mblock_info;
   	dram_info_t	orig_dram_info;
   	mem_desc_t	lca_reserved_mem;
   	mem_desc_t	tee_reserved_mem;
   	u32	boot_reason;
   	META_COM_TYPE	meta_com_type;
   	u32	meta_com_id;
   	u32	boot_time;
   	da_info_t	da_info;
   	SEC_LIMIT	sec_limit;
   	part_hdr_t	*part_info;
   	u8	md_type[4];
   	u32	ddr_reserve_enable;
   	u32	ddr_reserve_success;
   	u32	ddr_reserve_ready;
   	ptp_info_t	ptp_volt_info;
   	u32	dram_buf_size;
   	u32	meta_uart_port;
   	u32	smc_boot_opt;
   	u32	lk_boot_opt;
   	u32	kernel_boot_opt;
   	u32	non_secure_sram_addr;
   	u32	non_secure_sram_size;
   	char	pl_version[8];
   	u32	pl_imgver_status;
   	u32	max_cpus;
   	u32 boot_voltage;
   	u32 shutdown_time;
   	//add by zhanye
   	RESERVED_MEM dram_reserved_mem;
   } BOOT_ARGUMENT;
   ...
   ...
   // add by zhanye
   #define BOOT_TAG_RESERVE_MEM    0x8861001A
   struct boot_tag_reserved_mem {
      RESERVED_MEM dram_reserved_mem;
   };
   
   struct boot_tag {
   	struct boot_tag_header hdr;
   	union {
   		struct boot_tag_boot_reason boot_reason;
   		struct boot_tag_boot_mode boot_mode;
   		struct boot_tag_meta_com meta_com;
   		struct boot_tag_log_com log_com;
   		struct boot_tag_mem mem;
   		struct boot_tag_md_info md_info;
   		struct boot_tag_boot_time boot_time;
   		struct boot_tag_da_info da_info;
   		struct boot_tag_sec_info sec_info;
   		struct boot_tag_part_num part_num;
   		struct boot_tag_part_info part_info;
   		struct boot_tag_eflag eflag;
   		struct boot_tag_ddr_reserve ddr_reserve;
   		struct boot_tag_dram_buf dram_buf;
   		struct boot_tag_vcore_dvfs vcore_dvfs;
   		struct boot_tag_boot_opt boot_opt;
   		struct boot_tag_sram_info sram_info;
   		struct boot_tag_ptp ptp_volt;
   		struct boot_tag_imgver_info imgver_info;
   		struct boot_tag_max_cpus max_cpus;
   		struct boot_tag_bat_info bat_info;
   		// add  by zhanye
           struct boot_tag_reserved_mem reserved_mem;
   	} u;
   };
   ~~~

3. 主要函数

   platform.c中dram_init函数接收参数保存到BOOT_ARGUMENT中的dram_reserved_mem

   ~~~c
   int dram_init(void)
   {
       ...
       ...
       case BOOT_TAG_RESERVE_MEM:
   		g_boot_arg->dram_reserved_mem.cnt = tags->u.reserved_mem.dram_reserved_mem.cnt;
   		for(i=0;i<g_boot_arg->dram_reserved_mem.cnt;i++){
   			g_boot_arg->dram_reserved_mem.block[i].addr = tags->u.reserved_mem.dram_reserved_mem.block[i].addr;
   			g_boot_arg->dram_reserved_mem.block[i].size = tags->u.reserved_mem.dram_reserved_mem.block[i].size;
   			// printf("g_boot_arg->reserved_mem[i][j]\n"); 打印信息绝对不可以加
   		}		
   	break;
       ...
       ...
   ~~~

   mrdump_rsvmem.c中mrdump_reserve_memory把参数构造成cmdline，传递到内核

   ~~~c
   void mrdump_reserve_memory(void)
   {
       ...
       ...
       cnt = g_boot_arg->dram_reserved_mem.cnt;
   		for(i=0;i<cnt;i++){
   			memset(mrdump_rsv_tmp, 0, RSV_MEM_LEN);
   			snprintf(mrdump_rsv_tmp,RSV_MEM_LEN,",0x%x,0x%x", g_boot_arg->dram_reserved_mem.block[i].addr,g_boot_arg->dram_reserved_mem.block[i].size);
   			strncat(mrdump_rsv_mem, mrdump_rsv_tmp, RSV_MEM_LEN);
   			printf("zzy lk g_boot_arg->dram_reserved_mem.block[%d].addr = 0x%x\n", i, g_boot_arg->dram_reserved_mem.block[i].addr);
   			printf("zzy lk g_boot_arg->dram_reserved_mem.block[%d].size = 0x%x\n", i, g_boot_arg->dram_reserved_mem.block[i].size);
   		}
   
   	cmdline_append(mrdump_rsv_mem);
       ...
       ...
   }
   ~~~

#### Kernel部分

1. 主要文件

   drivers\misc\mediatek\aee\mrdump\mrdump_control.c

2. 主要函数

   ~~~c
   static int __init early_mrdump_rsvmem(char *p)
   {
   	unsigned long start_addr, size;
   	int ret;
   	char *tmp_p = p;
   	int i;
   	for (i = 0; i < 16; i++) {
   		ret = sscanf(tmp_p, "0x%lx,0x%lx", &start_addr, &size);
   		if (ret != 2) {
   			pr_alert("%s:%s reserve failed ret=%d\n", __func__, p, ret);
   			return 0;
   		}
   		rsvmem_block[i].start_addr = start_addr;
   		rsvmem_block[i].size = size;
   		tmp_p = find_next_mrdump_rsvmem(tmp_p, strlen(tmp_p));
   		if (!tmp_p){
                // add by zhanye
   			reserved_cnt = i + 1;
   			break;
   		}			
   	}
   	return 0;
   }
   __init void mrdump_rsvmem(void)
   {
   	int i;
   	printk("zzy reserved_cnt = %ld\n", reserved_cnt);
   	for (i = 0; i < 4; i++) {
   		if (rsvmem_block[i].start_addr) {
   			if (!memblock_is_region_reserved(rsvmem_block[i].start_addr, rsvmem_block[i].size)){
   				memblock_reserve(rsvmem_block[i].start_addr, rsvmem_block[i].size);
   				
   				printk("zzy mrdump:i = %d,addr = 0x%llx,size = 0x%llx\n", i,(unsigned long long)rsvmem_block[i].start_addr, (unsigned long long)rsvmem_block[i].size);
   			}
   			else{
   				/*
   				 * even conflict , we still enable MRDUMP for temp
   				 * because MINI DUMP will reserve the memory in DTSI now
   				 */
   #if 0
   				mrdump_rsv_conflict = 1;
   				mrdump_enable = 0;
   #endif
   				printk(" mrdump region start = %pa size =%pa is reserved already\n",
   						&rsvmem_block[i].start_addr, &rsvmem_block[i].size);
   			}
   		}
   	}
   	// add by zhanye
   	for(i = 0; i < (reserved_cnt - 4); i++)
   	{
   		if (rsvmem_block[i+4].start_addr) {
   			if (memblock_is_region_reserved(rsvmem_block[i+4].start_addr, rsvmem_block[i+4].size)){
   				memblock_remove(rsvmem_block[i+4].start_addr, rsvmem_block[i+4].size);
   				// memblock_reserve(rsvmem_block[i+4].start_addr, rsvmem_block[i+4].size);
   				printk("zzy remove:i = %d,addr = 0x%llx,size = 0x%llx\n", i+4,(unsigned long long)rsvmem_block[i+4].start_addr, (unsigned long long)rsvmem_block[i+4].size);
   			}
   			else{
   				memblock_reserve(rsvmem_block[i+4].start_addr, rsvmem_block[i+4].size);
   				printk("zzy reserved:i = %d,addr = 0x%llx,size = 0x%llx\n", i+4,(unsigned long long)rsvmem_block[i+4].start_addr, (unsigned long long)rsvmem_block[i+4].size);
   			}
   		}
   	}
   }
   ~~~
