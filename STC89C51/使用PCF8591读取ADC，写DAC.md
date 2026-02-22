# 使用PCF8591读取ADC，写DAC

AD/DA芯片电路图：

![](https://i-blog.csdnimg.cn/direct/e3abae0ddd7e4e3495271aab15a89884.jpeg)

[](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

编辑

通过PCF8591芯片，可以读取旋转电位器和光敏电阻的电压值 ，如图的左边第一部分，就是光敏电阻的电路图，第二部分就是旋转电位器的电路原理图

![](https://i-blog.csdnimg.cn/direct/81b839361fee44a4a115cbfc1a8ee713.jpeg)

[](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

编辑

**注意：**

**1.需要将读到的电压（取值范围是0~255) 转换为实际电压（0~5.00V）**

**2.读取ADC和写DAC需要基于IIC协议，即要包含iic.c源文件，文件我分享到百度网盘里面了，大家需要的话可以自取**

（通过网盘分享的文件：iic.c

链接: https://pan.baidu.com/s/1oQTP0uyjfqiRZ4PnahcX-A?pwd=dqtf 提取码: dqtf）

**3.光敏电阻的读取通道是(0x41),旋转电位器的读取通道是（0x43)。即传入ADC_Read_process()的dat的取值为0x41/0x43**

---

**1.读取PCF8591函数：（需要放在iic.c文件中）**

```cpp
unsigned char ADC_Read_process(unsigned char dat){
	unsigned char adc_val;
	IIC_Start();
	IIC_SendByte(0x90);
	IIC_WaitAck();
	IIC_SendByte(dat);
	IIC_WaitAck();

	IIC_Start();
	IIC_SendByte(0x91);
	IIC_WaitAck();
	adc_val=IIC_RecByte();
	IIC_SendAck(1);
	IIC_Stop();
	return adc_val;
}
```

[](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**2. AD读取程序示例：**

```cpp
unsigned char adc_val; //储存的是ADC的值，范围是0~255
unsigned int adc_volt; //储存的是电压值*100，范围是0~500
unsigned char cnt_adc;
void ADC_process(void){
   if(cnt_adc<=50) //每50ms读取一次
  {
    cnt_adc=0;
    adc_val=ADC_Read_process(0x43); //读取旋转电位器的ADC的值
    adc_volt=(adc_val*100)/51; //计算得到旋转电位器的电压值
   }
}
```

[](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

解释：

- 要将ADC的值转换为电压值，根据255/5=51可知，需要在得到的adc_val后除以51
- 将adc_val*100是为了在数码管上显示两位小数，详细解释可以看看我的这篇博客：[DS18B20扩展：在数码管上显示温度时包含小数部分-CSDN博客](https://blog.csdn.net/Libra_pro/article/details/149611930?sharetype=blogdetail&sharerId=149611930&sharerefer=PC&sharesource=Libra_pro&sharefrom=mp_from_link)

**3.同时读取光敏电阻和旋转电位器的程序示例：**

```cpp
unsigned char cnt_adc;
unsigned char channal1,channal2;
void ADC_process(void){
  if(cnt_adc>=10)
  {
   cnt_adc=0;
   ADC_Read_process(0x41);  //读取光敏电阻的AD值
   channal1= ADC_Read_process(0x41);

   ADC_Read_process(0x43);
   channal2= ADC_Read_process(0x43);  //读取旋转电位器的AD值
  }
}
```

[](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

解释：

这里为什么要调用两次ADC_Read_process()函数呢？因为ADC_Read_process()函数返回的是**上一次读取的数值**，如果直接写成：

```cpp
 channal1= ADC_Read_process(0x41);
 channal2= ADC_Read_process(0x43);
```

[](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

那么本应该读取旋转电位器的AD值时读到的确实光敏电阻的AD值，**所以为了避免这一问题 ，我们可以调用两次ADC_Read_process()函数。**

---

**1.写PCF8591函数：（同样的，也需要放在iic.c文件中）**

```cpp
//写DAC函数
void vWrite_DAC(unsigned char dat){
	IIC_Start();
	IIC_SendByte(0x90);
	IIC_WaitAck();

	IIC_SendByte(0x40);
	IIC_WaitAck();

	IIC_SendByte(dat);
	IIC_WaitAck();

	IIC_Stop();
}
```

[](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

注意：这个函数与ADC_Read_process()函数不同，传入函数的参数的取值范围为（0~255），转换为电压值就是（0~5.00)

**2.DAC程序示例：**

```cpp
unsigned char cnt_dac;
void DAC_process(){
 if(cnt_dac>=10)  //每10ms写入一次
 {
  cnt_dac=0;
  vWrite_DAC(51*2);
  }
}
```

[](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)
