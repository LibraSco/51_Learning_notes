# 使用PWM波调节LED灯的亮度

**基础知识：**

**PWM(**脉冲宽度调制):是一种通过调整信号的脉冲宽度来控制电力输出的技术。在一个PWM信号周期内，信号会在高电平和低电平之间切换，通过改变高电平持续的时间（即脉冲宽度），可以模拟出不同的电压水平。

**占空比：**就是高电平所占整个周期的时间

例如一个PWM波，周期为10ms,高点平的时间为4ms,那么它的占空比就是40%

---

**编程原理：**

**1.LED调光的频率最好是1kHz（即周期为1ms),太小的频率会导致LED灯闪烁**

**2.由于人的视觉暂留效应，调节占空比就可以调节LED灯的亮度，注意，由于LED灯是共阳极接法，所以占空比越高，LED的亮度反而越低**

**部分程序示例：**

```cpp
unsigned char cnt_pwm;
void PWM_process(){
  cnt_pwm=cnt_pwm % 10 +1; //这样就确保cnt+pwm的取值范围为1~10，即定义了pwm的周期为1ms
  if(cnt_pwm<=5)
  {
   P34=1; //0~0.5ms P34是高电平，LED灯熄灭
  }
  else
  {
   P34=0; //0.5~1ms P34是低电平，LED灯点亮
  }
}

int main(){
  Timer2_Init();
  while(1)
 {
  PWM_process();
 }
 return 0;
}
//中断函数是0.1ms中断
void Timer2_ISR(void)  interrupt 12{
 cnt_pwm++;
}
```

[](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

在该程序中LED的亮度是普通亮度的一半。
