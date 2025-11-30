# stm32-HAL-UARTRXInterrupt
stm32单片机进行UART接口中断，接收电脑发送的串口数据调整板载LED闪烁时间。  
## 硬件原理
![电路接线图](https://github.com/wahtcanisay/stm32-HAL-UARTRXInterruppt/blob/main/image.png)
电路接线图  
通过中断改变延迟时间来实现改变LED闪烁速度  
串口调试中设置相同的参数，发送1/2/3可以更改LED闪烁速度。  
## 软件实现
### cubemx部分
首先在`SYS`中`Debug`选择`Serial Wire`    
将PC13引脚设为初始高电平、输出开漏模式、低速  
在`USART1`设置为异步模式`Asynchronous`,以下参数保持默认(波特率115200、八位无校验、一位停止位)   
在`NVIC settings`选中`usart1 global interrupt`使能USART中断。  
对中断优先级不做更改。  
### keil5部分
**新接口：**
```c
HAL_StatusTypeDef HAL_UART_Receive_IT(UART_Handle *huart,
                                      uint8_t *pData,
                                      uint16_t Size
)
```
作用：使用中断接收一定数量数据。  
参数：  
*huart：串口句柄指针   
*pData：数据缓冲区指针  
Size：接收数据的数量  
```c
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
```
作用：回调函数，所有Size个字节中断接收完成后，单片机调用所写的回调函数    
```c
static uint32_t blinkInterval = 1000; //闪灯间隔
static uint8_t dataRcvd; //接收缓冲区
/*全局变量的定义在Private varibales(约第42行)的BEGIN和END之间*/
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart){
    //回调函数
    if (huart == &huart1){
        //判断是否从huart1传来中断
        if(dataRcvd == '1'){
            blinkInterval = 1000;
        }
        else if(dataRcvd == '2'){
            blinkInterval = 300;
        }
        else if(dataRcvd == '3'){
            blinkInterval = 50;
        }
    }

    HAL_UART_Receive_IT(&huart1, &dataRcvd, 1);//嵌套使用中断接收，可以实现接收不定长的中断数据。
}
/*以上代码写入Private Usercode的BEGIN 0和END 0之间*/
int main(){
    HAL_UART_Receive_IT(&huart1, &dataRcvd, 1); //中断接收
    while(1){
        //闪烁
        HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);

        HAL_Delay(blinkInterval);

        HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_SET);

        HAL_Delay(blinkInterval);
    }
}
```
