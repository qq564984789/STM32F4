## 以串口1位例
- 串口时钟使能：APB2ENR寄存器的第4位，其他串口的时钟使能再APB1ENR
- 波特率设置：USART_BRR寄存器
- 串口配置：一般只用到USART_CR1寄存器。这里面的各位要重点关注！
  - 过采样率(OVER8)：16位
  - 串口使能(UE)：
  - 字长
  - TCIE:发送完成中断使能位，(软件)设置该位为1时，当USART_SR中的TC位为1时，将产生串口中断。（TC位见下面）
  - RXNEIE:接收缓冲区非空中断使能为，(软件)设置该位为1，当USART_SR中的ORE或者RXNE位为1时，将产生串口中断。（RXNE见下面）
  - TE:发送使能位，设置为1，将开启串口的发送功能。
  - RE:接收使能位，设置为1，将开启串口的接收功能。
- 数据发送与接收：通过USART_DR实现,这是一个双寄存器（TDR,RDR），发送接收数据都用这个。
- 串口状态：USART_SR
  - RXNE（读数据寄存器非空）：当该位被（硬件）置 1 的时候就是提示已经有数据被接收到了，并且可以读出来了。通过读 USART_DR 可以将该位清零，也可以向该位写 0，直接清除。 
  - TC （发送完成）：当该位被（硬件）置位的时候，表示 USART_DR 内的数据已经被发送完成了。该位也有两种清零方式：1）读 USART_SR，写USART_DR。2）直接向该位写 0。
  
- 中断的产生：
  -TCIE为
  
  ## 使用HAL库编写串口程序
  > 串口参数（波特率、停止位）初始化：主要调用`HAL_UART_Init`函数
  ```c
  void uart_init(u32 bound)
{	
	//UART 初始化设置
	UART1_Handler.Instance=USART1;					    //USART1
	UART1_Handler.Init.BaudRate=bound;				    //波特率
	UART1_Handler.Init.WordLength=UART_WORDLENGTH_8B;   //字长为8位数据格式
	UART1_Handler.Init.StopBits=UART_STOPBITS_1;	    //一个停止位
	UART1_Handler.Init.Parity=UART_PARITY_NONE;		    //无奇偶校验位
	UART1_Handler.Init.HwFlowCtl=UART_HWCONTROL_NONE;   //无硬件流控
	UART1_Handler.Init.Mode=UART_MODE_TX_RX;		    //收发模式
	HAL_UART_Init(&UART1_Handler);					    //HAL_UART_Init()会使能UART1

  ```
