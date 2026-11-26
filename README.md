# -
STM32标准库项目
#代码 
### main.c
~~~c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "OLED.h"
#include "LED.h"
#include "Timer.h"
#include "Key.h"
#include "RP.h"
#include "Motor.h"
#include "Encoder.h"
#include "Serial.h"



uint8_t KeyNum;
int16_t PWM;
int16_t Speed;
float Target,Actual,Out;//目标值，实际值，输出值
float Error1,Error0,ErrorInt;

float Kp=0,Ki=0,Kd=0;//比例项，积分项，微分项

int main(void)
{
	Motor_Init();
	Key_Init();
	OLED_Init();
	Timer_Init();
	Encoder_Init();
	Serial_Init();
	LED_Init();
	OLED_Printf(0,0,OLED_8X16,"Speed Control");
	OLED_Update();
	while(1)
	{
		KeyNum=Key_GetNum();	
		if(KeyNum==1)
		{
			Target+=10;
			
		}
		if(KeyNum==2)
		{
			Target-=10;
		
		}
		if(KeyNum==3)
		{
			Target=0;
		}
		OLED_Printf(0,16,OLED_8X16,"Tar:%+04.0f",Target);
		OLED_Printf(0,32,OLED_8X16,"Act:%+04.0f",Actual);
		OLED_Printf(0,48,OLED_8X16,"Out:%+04.0f",Out);

		
		if(Actual>5){		
			Serial_Printf("转速超过阈值,当前值:%f\r\n",Actual);
}



		OLED_Update();
	}	
}
void TIM1_UP_IRQHandler(void)
{
	static uint16_t Count;
	if (TIM_GetITStatus(TIM1, TIM_IT_Update) == SET)
	{
	Key_Tick();
		Count++;
		if(Count>=40)
		{
		Count=0;

			Actual=Encoder_Get();
			Error1=Error0;
			Error0=Target-Actual;
			ErrorInt+=Error0;
			
			Out=Kp*Error0+Ki*ErrorInt+Kd*(Error0-Error1);
			if(Out>100){Out=100;}
			if(Out<-100){Out=-100;}
			Motor_SetPWM(Out);

		}

	TIM_ClearITPendingBit(TIM1, TIM_IT_Update);
	}
}

~~~
[源码](C:\Users\Autumn\Desktop\新建文件夹\PID入门教程资料\PID\测速报警实验)

[[GPIO输出]]
[[DMA+AD多通道]]
[[NVIC和EXTI介绍]]
[[TIM定时器介绍]]

