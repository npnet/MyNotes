## Cortex-A7常用汇编指令

### 1.处理器内部数据传输指令

1. MOV指令

   MOV 指令用于将数据从一个寄存器拷贝到另外一个寄存器，或者将一个立即数传递到寄存器里面  

   ```assembly
   MOV R0, R1 @将寄存器 R1 中的数据传递给 R0，即 R0=R1
   MOV R0, #0X12 @将立即数 0X12 传递给 R0 寄存器，即 R0=0X12
   ```

2. MRS指令
   MRS 指令用于将特殊寄存器(如 CPSR 和 SPSR)中的数据传递给通用寄存器，要读取特殊寄存器的数据只能使用 MRS 指令！

   ```assembly
   MRS R0, CPSR @将特殊寄存器 CPSR 里面的数据传递给 R0，即 R0=CPSR
   ```

3. MSR指令MSR 指令用来将普通寄存器的数据传递给特殊寄存器，也就是写特殊寄存器，写特殊寄存器只能使用 MSR  

   ```assembly
   MSR CPSR, R0 @将 R0 中的数据复制到 CPSR 中，即 CPSR=R0
   ```

   

### 2.存储器访问指令

| 指令                   | 描述                                            |
| ---------------------- | ----------------------------------------------- |
| LDR Rd, [Rn , #offset] | 从存储器 Rn+offset 的位置读取数据存放到 Rd 中。 |
| STR Rd, [Rn, #offset]  | 将 Rd 中的数据写入到存储器中的 Rn+offset 位置。 |

1. LDR指令
   LDR 主要用于从存储加载数据到寄存器 Rx 中， LDR 也可以将一个立即数加载到寄存器 Rx中， 
   LDR 加载立即数的时候要使用“=”，而不是“#”。  

   ```assembly
   LDR R0, =0X0209C004 @将寄存器地址 0X0209C004 加载到 R0 中，即 R0=0X0209C004
   LDR R1, [R0] @读取地址 0X0209C004 中的数据到 R1 寄存器中,offset为0
   ```

2. STR指令
   STR 就是将数据写入到存储器中  

   ```assembly
   LDR R0, =0X0209C004 @将寄存器地址 0X0209C004 加载到 R0 中，即 R0=0X0209C004
   LDR R1, =0X20000002 @R1 保存要写入到寄存器的值，即 R1=0X20000002
   STR R1, [R0] @将 R1 中的值写入到 R0 中所保存的地址中
   ```



### 3.压栈和出栈指令

| 指令              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| PUSH \<reg list\> | 将寄存器列表存入栈中。从存储器 Rn+offset 的位置读取数据存放到 Rd 中。 |
| POP \<reg list\>  | 从栈中恢复寄存器列表。将 Rd 中的数据写入到存储器中的 Rn+offset 位置。 |

```assembly
PUSH {R0~R3, R12} @将 R0~R3 和 R12 压栈
PUSH {LR} @将 LR 进行压栈
POP {LR} @先恢复 LR
POP {R0~R3,R12} @在恢复 R0~R3,R12
```

| 指令                             |
| -------------------------------- |
| LDM{cond} mode Rn{!}, reglist{^} |
| STM{cond} mode Rn{!}, reglist{^} |

PUSH 和 POP 的另外一种写法是“STMFD SP！”和“LDMFD SP!”  ,FD为满递减

```assembly
STMFD SP!,{R0~R3, R12} @R0~R3,R12 入栈
STMFD SP!,{LR} @LR 入栈
LDMFD SP!, {LR} @先恢复 LR
LDMFD SP!, {R0~R3, R12} @再恢复 R0~R3, R12
```



### 4.跳转指令

| 指令         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| B \<label\>  | 跳转到 label，如果跳转范围超过了+/-2KB，可以指定 B.W \<label\>使用 32 位版本的跳转指令， <br />这样可以得到较大范围的跳转 |
| BX \<Rm\>    | 间接跳转，跳转到存放于 Rm 中的地址处，并且切换指令集         |
| BL \<label\> | 跳转到标号地址，并将返回地址保存在 LR 中。                   |
| BLX \<Rm\>   | 结合 BX 和 BL 的特点，跳转到 Rm 指定的地址，并将返回地 址保存在 LR 中，切换指令集 |

1. B指令
   B 指令会将 PC 寄存器的值设置为跳转目标地址，一旦执行 B 指令，ARM 处理器就会立即跳转到指定的目标地址，执行后不返回。

2. BL指令
   在跳转之前会在寄存器 LR(R14)中保存当前 PC 寄存器值，
   通过将 LR 寄存器中的值重新加载到 PC 中来继续从跳转之前的代码处运行  。

   ```assembly
   push {r0, r1} @保存 r0,r1
   cps #0x13 @进入 SVC 模式，允许其他中断再次进去
   
   bl system_irqhandler @加载 C 语言中断处理函数到 r2 寄存器中
   
   cps #0x12 @进入 IRQ 模式
   pop {r0, r1}
   str r0, [r1, #0X10] @中断执行完成，写 EOIR
   ```



### 5.算术运算指令

| 指令               | 计算公式                | 备注                         |
| ------------------ | ----------------------- | ---------------------------- |
| ADD Rd, Rn, Rm     | Rd = Rn + Rm            | 加法运算，指令为 ADD         |
| ADD Rd, Rn, #immed | Rd = Rn + #immed        |                              |
| ADC Rd, Rn, Rm     | Rd = Rn + Rm + 进位     | 带进位的加法运算，指令为 ADC |
| ADC Rd, Rn, #immed | Rd = Rn + #immed +进位  |                              |
| SUB Rd, Rn, Rm     | Rd = Rn – Rm            | 减法                         |
| SUB Rd, #immed     | Rd = Rd - #immed        |                              |
| SUB Rd, Rn, #immed | Rd = Rn - #immed        |                              |
| SBC Rd, Rn, #immed | Rd = Rn - #immed – 借位 | 带借位的减法                 |
| SBC Rd, Rn ,Rm     | Rd = Rn – Rm – 借位     |                              |
| MUL Rd, Rn, Rm     | Rd = Rn * Rm            | 乘法(32 位)                  |
| UDIV Rd, Rn, Rm    | Rd = Rn / Rm            | 无符号除法                   |
| SDIV Rd, Rn, Rm    | Rd = Rn / Rm            | 有符号除法                   |



### 6.逻辑运算指令

| 指令               | 计算公式            | 备注     |
| ------------------ | ------------------- | -------- |
| AND Rd, Rn         | Rd = Rd &Rn         | 按位与   |
| AND Rd, Rn, #immed | Rd = Rn &#immed     |          |
| AND Rd, Rn, Rm     | Rd = Rn & Rm        |          |
| ORR Rd, Rn         | Rd = Rd \| Rn       | 按位或   |
| ORR Rd, Rn, #immed | Rd = Rn \| #immed   |          |
| ORR Rd, Rn, Rm     | Rd = Rn \| Rm       |          |
| BIC Rd, Rn         | Rd = Rd & (~Rn)     | 位清除   |
| BIC Rd, Rn, #immed | Rd = Rn & (~#immed) |          |
| BIC Rd, Rn , Rm    | Rd = Rn & (~Rm)     |          |
| ORN Rd, Rn, #immed | Rd = Rn \| (#immed) | 按位或非 |
| ORN Rd, Rn, Rm     | Rd = Rn \| (Rm)     |          |
| EOR Rd, Rn         | Rd = Rd ^ Rn        | 按位异或 |
| EOR Rd, Rn, #immed | Rd = Rn ^ #immed    |          |
| EOR Rd, Rn, Rm     | Rd = Rn ^ Rm        |          |