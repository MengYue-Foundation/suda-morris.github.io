---
layout: post
title: "Nuvoton-M451-Timer"
date: 2015-05-20 11:03:28 +0800
comments: true
categories: [Nuvoton-M451,Study]
---
##新唐CortexM4内核芯片M451学习之----定时器Timer
>* 4组32位定时器TIMER0~TIMER3,带24位向上计数器和一个8位的预分频器
> * 支持两个中断标志：一个是`TIF`，该标志在当定时器计数器值CNT与定时器比价值CMPDAT相匹配时置位，另一个是`CAPIF`标志，该标志在当Tx_ENT管脚的变化与CAPEDGE的设置一致是置位
> * 定时器计数模式：one-shot(单周期),periodic(多周期),toggle-output(翻转输出)和continuous counting(连续计数)计数模式
> * 可以选择Tx管脚上的信号作为计数器CNT的时钟(外部事件计数模式)，输入事件的频率必须小于1/8PCLK。此外，Tx管脚还可作为toggle-output模式下的输出引脚
> * 事件捕捉功能是当检测到Tx_EXT管脚边沿有变化时，当时的CNT值会送到CAPDAT中。Tx_EXT管脚的信号变化率必须小于1/8PCLK。如果CPU不清除CAPIF状态标志，定时器会保持TIMERx_CAP寄存器的值不变，且不会保存新的捕捉值。
> * 注意当使用`事件计数模式`或者`事件捕捉模式`时候，相应定时器的时钟源必须选择为HCLK或者PCLK;CAPFUNCS为0对应事件捕捉，为1对应事件复位，即会复位CNT的值
> * 注意当使用事件计数功能时候，定时器的工作模式不能设置为TIMER_TOGGLE_MODE
> * 超时溢出周期 = (定时器输入时钟周期) * (8-bit PSC + 1) * (24-bit CMPDAT)

##API
* `TIMER_SET_CMP_VALUE(timer,u32Value)`
* `TIMER_SET_PRESCALE_VALUE(timer,u32Value)`
* `TIMER_IS_ACTIVE(timer)`
* `TIMER_SELECT_TOUT_PIN(timer,u32ToutSel)`
* `TIMER_Start(TIMER_T *timer)`
* `TIMER_Stop(TIMER_T *timer)`
* `TIMER_EnableWakeup(TIMER_T *timer)`
* `TIMER_DisableWakeup(TIMER_T *timer)`
* `TIMER_EnableCaptureDebounce(TIMER_T *timer)`
* `TIMER_DisableCaptureDebounce(TIMER_T *timer)`
* `TIMER_EnableEventCounterDebounce(TIMER_T *timer)`
* `TIMER_DisableEventCounterDebounce(TIMER_T *timer)`
* `TIMER_EnableInt(TIMER_T *timer)`
* `TIMER_DisableInt(TIMER_T *timer)`
* `TIMER_EnableCaptureInt(TIMER_T *timer)`
* `TIMER_DisableCaptureInt(TIMER_T *timer)`
* `TIMER_GetIntFlag(TIMER_T *timer)`
* `TIMER_ClearIntFlag(TIMER_T *timer)`
* `TIMER_GetCaptureIntFlag(TIMER_T *timer)`
* `TIMER_ClearCaptureIntFlag(TIMER_T *timer)`
* `TIMER_GetWakeupFlag(TIMER_T *timer)`
* `TIMER_ClearWakeupFlag(TIMER_T *timer)`
* `TIMER_GetCaptureData(TIMER_T *timer)`
* `TIMER_GetCounter(TIMER_T *timer)`
* `TIMER_Open(TIMER_T *timer,uint32_t u32Mode,uint32_t u32Freq)`
* `TIMER_Close(TIMER_T *timer)`
* `TIMER_Delay(TIMER_t *timer)`
* `TIMER_EnableCapture(TIMER_T *timer,uint32_t u32CapMode,uint32_t u32Edge)`
* `TIMER_DisableCapture(TIMER_T *timer)`
* `TIMER_EnableEventCounter(TIMER_T *timer,uint32_t u32Edge)`
* `TIMER_DisableEventCounter(TIMER_T *timer)`
* `TIMER_GetModuleClock(TIMER_T *timer)`

> * TIMER_T *timer的取值有`TIMER0`,`TIMER1`,`TIMER2`,`TIMER3`
> * u32ToutSel的取值有`TIMER_TOUT_PIN_FROM_TX`,`TIMER_TOUT_PIN_FROM_TX_EXT`
> * u32CapMode的取值有`TIMER_CAPTURE_FREE_COUNTING_MODE`,`TIMER_CAPTURE_COUNTER_RESET_MODE`
> * 捕获模式中u32Edge的取值有`TIMER_CAPTURE_FALLING_EDGE`,`TIMER_CAPTURE_RISING_EDGE`,`TIMER_CAPTURE_FALLING_AND_RISING_EDGE`
> * 事件计数模式中u32Edge的取值有`TIMER_COUNTER_FALLING_EDGE`,`TIMER_COUNTER_RISING_EDGE`
> * u32Mode的取值有`TIMER_ONESHOT_MODE`,`TIMER_PERIODIC_MODE`,`TIMER_TOGGLE_MODE`,`TIMER_CONTINUOUS_MODE`

##定时器TMR0的配置程序
```C 配置TMR0:toggle-output模式，频率1000Hz(PD4实际产生的方波频率500Hz)，关闭中断 http://suda-morris.github.io author
	/*Enable TMR0 ClK Source with PCLK0*/
	CLK_EnableModuleClock(TMR0_MODULE);
	CLK_SetModuleClock(TMR0_MODULE,CLK_CLKSEL1_TMR0SEL_PCLK0,0);
	/*Reset TMR0 Module*/
	SYS_ResetModule(TMR0_MODULE);
	/*Open TMR0 Module, working at toggle mode, with Freq=1000Hz*/
	uint32_t real_timer0_freq = TIMER_Open(TIMER0,TIMER_TOGGLE_MODE,1000);
	/*Set PD4[PIN18] Multifuntion Pin as T0 for Toggle Output*/
	SYS->GPD_MFPL = (SYS->GPD_MFPL & ~SYS_GPD_MFPL_PD4MFP_Msk) | SYS_GPD_MFPL_PD4MFP_T0;
#if 0
	/*Enable Int for TMR0 Module*/
	TIMER_EnableInt(TIMER0);
	NVIC_EnableIRQ(TMR0_IRQn);
#endif
	/*Start TMR0 Module*/
	TIMER_Start(TIMER0);
	printf("Real TMR0 Freq=%dHz\n",real_timer0_freq);
```

##定时器TMR3的配置程序
```C 配置TMR3：toggle-output模式，频率2Hz(PD11实际产生的方波频率2Hz)，关闭中断 http://suda-morris.github.io author
	/*Enable TMR3 ClK Source with PCLK1*/
	CLK_EnableModuleClock(TMR3_MODULE);
	CLK_SetModuleClock(TMR3_MODULE,CLK_CLKSEL1_TMR3SEL_PCLK1,0);
	/*Reset TMR3 Module*/
	SYS_ResetModule(TMR3_MODULE);
	/*Open TMR3 Module, working at toggle mode, with Freq=2Hz*/
	uint32_t real_timer3_freq = TIMER_Open(TIMER3,TIMER_TOGGLE_MODE,2);
	/*Set PD11[PIN27] Multifuntion Pin as T3 for Toggle Output*/
	SYS->GPD_MFPH = (SYS->GPD_MFPH & ~SYS_GPD_MFPH_PD11MFP_Msk) | SYS_GPD_MFPH_PD11MFP_T3;
#if 0
	/*Enable Int for TMR3 Module*/
	TIMER_EnableInt(TIMER3);
	NVIC_EnableIRQ(TMR3_IRQn);
#endif
	/*Start TMR3 Module*/
	TIMER_Start(TIMER3);
	printf("Real TMR3 Freq=%dHz\n",real_timer3_freq);
```

##定时器TMR2的配置程序
```C 配置TMR2: continuous模式，开启事件计数器功能(PD3)，开启捕获功能(PA5)，开启捕获中断 http://suda-morris.github.io author
	/*Enable TMR2 Clk Source with PCLK1*/
	CLK_EnableModuleClock(TMR2_MODULE);
	CLK_SetModuleClock(TMR2_MODULE,CLK_CLKSEL1_TMR2SEL_PCLK1,0);
	/*Reset TMR2 Module*/
	SYS_ResetModule(TMR2_MODULE);
	/*Open TMR2 Module,working at continuous mode,with Freq=1Hz*/
	TIMER_Open(TIMER2,TIMER_CONTINUOUS_MODE,1);
	/*Enable Event Counter for TMR2, Enable Event Counter Debounce*/
	TIMER_EnableEventCounter(TIMER2,TIMER_COUNTER_FALLING_EDGE);
	TIMER_EnableEventCounterDebounce(TIMER2);
	TIMER_SET_PRESCALE_VALUE(TIMER2,0);
	TIMER_SET_CMP_VALUE(TIMER2,0xFFFFFF);
	/*Enable Capture Counting for TMR2, Enable Capture Debounce*/
	TIMER_EnableCapture(TIMER2,TIMER_CAPTURE_FREE_COUNTING_MODE,TIMER_CAPTURE_FALLING_EDGE);
	TIMER_EnableCaptureDebounce(TIMER2);
	/*Set PD3[PIN17] Multifuntion Pin as T2 for Event Counter*/
	SYS->GPD_MFPL = (SYS->GPD_MFPL & ~SYS_GPD_MFPL_PD3MFP_Msk) | SYS_GPD_MFPL_PD3MFP_T2;
	/*Set PA5[PIN61] Multifuntion Pin as T2_EXT for Capture*/
	SYS->GPA_MFPL = (SYS->GPA_MFPL & ~SYS_GPA_MFPL_PA5MFP_Msk) | SYS_GPA_MFPL_PA5MFP_T2_EXT;
	/*Enable Int for TMR2 Module*/
	TIMER_EnableInt(TIMER2);
	TIMER_EnableCaptureInt(TIMER2);
	NVIC_EnableIRQ(TMR2_IRQn);
	/*Start TMR2 Module*/
	TIMER_Start(TIMER2);
```

##定时器TMR2中断程序
```C TMR2中断程序 http://suda-morris.github.io author
/**
 * @brief       Timer2 IRQ
 *
 * @param       None
 *
 * @return      None
 *
 * @details     The Timer2 default IRQ, declared in startup_M451Series.s.
 */
uint32_t capCount = 0;
void TMR2_IRQHandler(void)
{
    if(TIMER_GetIntFlag(TIMER2) == 1)
    {
        /* Clear Timer2 time-out interrupt flag */
        TIMER_ClearIntFlag(TIMER2);
		printf("TMR2 Time-Out Interrupt occured\n");
    }
	if(TIMER_GetCaptureIntFlag(TIMER2) == 1)
    {
        /* Clear Timer2 capture trigger interrupt flag */
        TIMER_ClearCaptureIntFlag(TIMER2);
		capCount++;
    }
}
```

##主函数测试代码
```C main函数主要测试代码,测试结果为500 http://suda-morris.github.io author
	uint32_t capValue[2] = {0,0};
	uint32_t initCount = 0;
	while(capCount <= 10)
	{
		if(capCount%2 != initCount)
		{
			capValue[0] = capValue[1];
			capValue[1] = TIMER_GetCaptureData(TIMER2);
			printf("diff=%d\n",capValue[1]-capValue[0]);
			initCount = capCount%2;
		}
	}
```

![suda-morris](http://i.imgur.com/Nn7Krru.gif)