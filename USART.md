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
> 1.串口参数（波特率、停止位）初始化：主要调用`HAL_UART_Init`函数
```c
 UART_HandleTypeDef UART1_Handler;                         //UART串口句柄,全局变量
 void uart_init(u32 bound)                 //下面是对UART1_Handler这个结构体中Instance和Init两个成员变量的填充，
{					   //还有一些很重要的成员未初始化,需要看这个结构体的完整定义。最后初始化
	//UART 初始化设置
	UART1_Handler.Instance=USART1;	      //串口寄存器基地址，其实就是底层文件定义好的寄存器名：USART1。寄存器名和寄存器地址一一对应
	UART1_Handler.Init.BaudRate=bound;		 //波特率
	UART1_Handler.Init.WordLength=UART_WORDLENGTH_8B;   //字长为8位数据格式
	UART1_Handler.Init.StopBits=UART_STOPBITS_1;	    //一个停止位
	UART1_Handler.Init.Parity=UART_PARITY_NONE;		    //无奇偶校验位
	UART1_Handler.Init.HwFlowCtl=UART_HWCONTROL_NONE;   //无硬件流控
	UART1_Handler.Init.Mode=UART_MODE_TX_RX;		    //收发模式
	HAL_UART_Init(&UART1_Handler);			 
```
> 注意：最后的`HAL_UART_Init(&UART1_Handler);`除了会对串口参数初始化之外，还会使能串口（内部调用`__HAL_UART_ENABLE(huart);`）,还会调用回调函数进行和MCU相关的初始化(内部调用`HAL_UART_MspInit(huart);`)。对于`HAL_UART_MspInit(huart);`,我们通常不会使用系统提供的这个weak函数，而是会自己重写，主要重写内容包括：IO口初始化、时钟使能、NVIC配置（这些都和具体MCU有关，而上面的波特率、停止位等等都和MCU无关）。如何重写呢？见第二步



> 2. I/O时钟使能、串口时钟使能、GPIO初始化、I/O口复用、中断配置(使能、优先级设置)
```c
void HAL_UART_MspInit(UART_HandleTypeDef *huart)
{
    //GPIO端口设置
	GPIO_InitTypeDef GPIO_Initure;
	
	if(huart->Instance==USART1)//如果是串口1，进行串口1 MSP初始化
	{
		__HAL_RCC_GPIOA_CLK_ENABLE();			//使能GPIOA时钟
		__HAL_RCC_USART1_CLK_ENABLE();			//使能USART1时钟
	
		GPIO_Initure.Pin=GPIO_PIN_9;			//PA9
		GPIO_Initure.Mode=GPIO_MODE_AF_PP;		//复用推挽输出
		GPIO_Initure.Pull=GPIO_PULLUP;			//上拉
		GPIO_Initure.Speed=GPIO_SPEED_FAST;		//高速
		GPIO_Initure.Alternate=GPIO_AF7_USART1;	//复用为USART1
		HAL_GPIO_Init(GPIOA,&GPIO_Initure);	   	//初始化PA9

		GPIO_Initure.Pin=GPIO_PIN_10;			//PA10
		HAL_GPIO_Init(GPIOA,&GPIO_Initure);	   	//初始化PA10
		
#if EN_USART1                                                //通过一个自己定义的中断“开关”决定是否开启串口中断
		HAL_NVIC_EnableIRQ(USART1_IRQn);	     //使能USART1中断通道
		HAL_NVIC_SetPriority(USART1_IRQn,3,3);	     //抢占优先级3，子优先级3
#endif	
	}

}
```
> 注意，条件编译中是对串口的中断的使能和设置优先级，但如果需要使用串口的发送或接收中断功能，还需要分别开启发送或接收中断。如
  接收完成中断使能：`__HAL_UART_ENABLE_IT(huart, UART_IT_RXNE);` 发送完成中断使能：`__HAL_UART_ENABLE_IT(huart, UART_IT_TC);`



> 3.编写中断服务程序
```c
void USART1_IRQHandler(void)
{

}
```
