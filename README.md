# STM32-RTOS-USB-HowToFix
Some useful tips to survive on STM32 using RTOS and USB CDC device 

STM32CubeIDE and STM32CubeMX generates code for USB management that has **malloc** inside interrupt (bad practive) this can generate HEAP corrution, also some versions of FREERTOS have a buggy version of ```heap_4.c```

1) If you don't use USB and you have problems with FREERTOS\
**Solution** : try to use ```heap_useNewlib.c``` by Dave Nadler instead of ```\Middlewares\Third_Party\FreeRTOS\Source\portable\MemMang\heap_4.c"heap_4.c```
also take a look at http://www.nadler.com/embedded/newlibAndFreeRTOS.html

2) Inside ```sysmem.c``` there is a function ```caddr_t _sbrk(int incr)``` that should be used for **malloc** but is never actually called. Need to be investigated

3) USB code uses **malloc** inside interrupt, best is to replace malloc with a static struct. Look PDF **UM1734 STM32Cube™ USB device library** at point 6.7 Library footprint optimization, this will be a fist step to impruve the code.

4) On Windows 10 there are many problems opening a VCP, this is because the driver returns a bad parameters of VCP configuration\
In file ***usbd_cdc_if.c***, inside Private Variables section, declare\
```static uint8_t uartcfg[7] = {0,0,0,0,0,0,0};```\
then inside function **CDC_Control_FS**

```
    case CDC_SET_LINE_CODING:
      	uartcfg[0] = pbuf[0];
      	uartcfg[1] = pbuf[1];
      	uartcfg[2] = pbuf[2];
      	uartcfg[3] = pbuf[3];
      	uartcfg[4] = pbuf[4];
      	uartcfg[5] = pbuf[5];
      	uartcfg[6] = pbuf[6];
    break;

    case CDC_GET_LINE_CODING:
      	pbuf[0] = uartcfg[0];
      	pbuf[1] = uartcfg[1];
      	pbuf[2] = uartcfg[2];
      	pbuf[3] = uartcfg[3];
      	pbuf[4] = uartcfg[4];
      	pbuf[5] = uartcfg[5];
      	pbuf[6] = uartcfg[6];
    break;
```
this will prevent Windows 10 to take error opening CDC Virtual Com Port
