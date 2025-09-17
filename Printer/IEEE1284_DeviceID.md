# 获取符合IEEE1284标准的设备标识符

## IEEE1284介绍

### 并行接口

并行接口（Parallel Port）一般称为Centronics接口，最早由Centronics Data Computer Corporation公司在20世纪60年代中期制定。Centronics公司当初是为点阵行式打印机设计的并行接口，1981年被IBM公司采用，后来成为IBM PC计算机的标准配置。

### IEEE1284

1991年，Lexmark、 IBM、Texas instruments等公司为扩大其应用范围而与其他接口竞争，改进了Centronics接口，使它实现更高速的双向通信，以便能连接磁盘机、磁带机、光盘机、网络设备等计算机外部设备（简称外设），最终形成了 **IEEE1284-1994** 标准，全称为"Standard Signaling Method for a **Bi-directional Parallel Peripheral Interface** for Personal Computers"，数据率从10KB/s提高到可达2MB/s（16Mbit/s）。但事实上这种双向并行通信并没有获得广泛使用，并行接口仍主要用于打印机和绘图仪，其他类型设备应用较少，因此这种接口一般被称为打印接口或LPT接口 。

符合IEEE1284标准的并口，使用 **设备标识符（Device identification sequence）** 来实现即插即用（Plug and Play）配置，使并口更易于使用。计算机从外部设备读取设备标识符后，可判断出设备信息，为后续操作（如加载对应的设备驱动）做好准备。

随着硬件设备的更新换代，连接打印机的并口，由于体积较大及通信速度限制等问题，在2000年代之后逐渐被**USB接口**所取代，但IEEE1284协议标准被保留了下来，仍然可以通过设备通信来获取IEEE1284标准的设备标识符。

## 设备标识符

### 规范

设备标识符（Device ID）由长度域和其后区分大小写的ASCII字符串组成。  
头两个字节表示这个序列的长度（包括这两个字节）。紧随这两个字节的之后的序列由多个关键字（全大写字母）和值组成，其格式为：  

**key: value {, value};** 

其中，**MANUFACTURER**、**COMMAND SET**、**MODEL**为必须的关键字，它们也可以缩写为**MFG**、**CMD**、**MDL**。

### 示例

***HP Color LaserJet 550*** 的设备标识符ASCII字符串如下：

```text
MANUFACTURER:Hewlett-Packard;COMMAND SET:PJL,MLC,PCL,POSTSCRIPT;MODEL:HP Color LaserJet 550;CLASS:PRINTER;COMMENT:HP LaserJet;
```

***Canon G3010 series*** 的设备标识符ASCII字符串如下：

```text
MFG:Canon;CMD:BJRaster3,NCCe,IVEC;SOJ:CHMP;MDL:G3010 series;CLS:PRINTER;DES:Canon G3010 series;VER:1.060;STA:10;PSE:KLMN77353;CID:CA_IVEC1TYPE4_IJP;
```

可以看出，除了必须的关键字，厂商也可以扩展自定义的关键字，例如上面的HP的COMMENT、Canon的 **PSE（设备序列号）**、CID等。


## USB获取设备标识符

### 通过Linux内核驱动usblp获取

Linux内核驱动 [drivers/usb/class/usblp.c](https://elixir.bootlin.com/linux/latest/source/drivers/usb/class/usblp.c#L67) 中，已经定义了用于取得USB设备标识符的宏 **LPIOC_GET_DEVICE_ID** 相关代码如下：

```c
/* Get device_id string: */
#define LPIOC_GET_DEVICE_ID(len) _IOC(_IOC_READ, 'P', IOCNR_GET_DEVICE_ID, len)
```

调用方法可参照 **Debian/Ubuntu** 系统中 **printer-driver-foo2zjs** 包中 **usb_printerid** 命令的实现（[usb_printerid.c](https://salsa.debian.org/printing-team/foo2zjs/-/blob/debian/main/usb_printerid.c)）：

```c
int main (int argc, char *argv[])
{
    int			fd;
    unsigned char	argp[1024];
    int			length;

    --argc;
    ++argv;

    if (argc != 1)

    fd = open(argv[0], O_RDWR);
    if (fd < 0)

    if (ioctl(fd, LPIOC_GET_DEVICE_ID(sizeof(argp)), argp) < 0)

    length = (argp[0] << 8) + argp[1] - 2;
    printf("GET_DEVICE_ID string:\n");
    fwrite(argp + 2, 1, length, stdout);
    printf("\n");

    close(fd);
    exit(0);
}
```

在 **Debian/Ubuntu** 系统中安装CUPS时，已默认安装了 printer-driver-foo2zjs 包，可直接运行 **usb_printerid** 命令获得设备标识符：
```shell
sudo usb_printerid /dev/usb/lp0
```
例如：USB连接 **Canon TS3480** 喷墨打印机后，运行该命令时返回的字符串如下：
```txt
GET_DEVICE_ID string:
MFG:Canon;CMD:IVEC;SOJ:CHMP,CHMPu,CLS;MDL:TS3400 series;CLS:PRINTER;DES:Canon TS3400 series;VER:1.010;STA:10;PSE:AGFC39872;CID:CA_IVEC1TYPE1_IJP;
```

在 **CUPS beckend** 中也有类似调用：[ieee1284.c](https://github.com/OpenPrinting/cups/blob/master/backend/ieee1284.c) 的 *backendGetDeviceID()* 函数，在调用 **/usr/lib/cups/backend/usb** 命令时，也会有含有设备标识符的DEBUG信息输出。

### 通过libusb函数获取

 **CUPS beckend** 中也有通过 *libusb_control_transfer* 函数获取设备标识符的相关实现：[usb-libusb.c](https://github.com/OpenPrinting/cups/blob/master/backend/usb-libusb.c)
```c
/*
 * 'get_device_id()' - Get the IEEE-1284 device ID for the printer.
 */

static int				/* O - 0 on success, -1 on error */
get_device_id(usb_printer_t *printer,	/* I - Printer */
              char          *buffer,	/* I - String buffer */
              size_t        bufsize)	/* I - Number of bytes in buffer */
{
  int	length;				/* Length of device ID */


  if (libusb_control_transfer(printer->handle,
			      LIBUSB_REQUEST_TYPE_CLASS | LIBUSB_ENDPOINT_IN |
			      LIBUSB_RECIPIENT_INTERFACE,
			      0, printer->conf,
			      (printer->iface << 8) | printer->altset,
			      (unsigned char *)buffer, bufsize, 5000) < 0)
  {
    *buffer = '\0';
    return (-1);
  }

```

理论上，该操作也可以移植到 Android 平台上，使用 [UsbConnection 类的 controlTransfer()](https://developer.android.google.cn/reference/android/hardware/usb/UsbDeviceConnection#controlTransfer(int,%20int,%20int,%20int,%20byte[],%20int,%20int,%20int)) 方法来实现。
```java
  byte[] buffer = new byte[4096];
  int transferred = connection.controlTransfer(
    0xa1, // REQUEST_TYPE_OF_GET_DEVICE_ID
    0, // REQUEST_OF_GET_DEVICE_ID
    0, // value
    (usbInterface.getId() << 8) | usbInterface.getAlternateSetting(), // index
    buffer,
    buffer.length,
    500 // Timeout
    );
```


## MIB获取设备标识符

对于支持MIB的网络打印机，可以通过 oid **1.3.6.1.4.1.2699.1.2.1.2.1.1.3.1**  [ppmPrinterIEEE1284DeviceId](https://oidref.com/1.3.6.1.4.1.2699.1.2.1.2.1.1) 来获取 IEEE1284 设备标识符。

## 参考资料

百度百科词条：[并行接口](https://baike.baidu.com/item/%E5%B9%B6%E8%A1%8C%E6%8E%A5%E5%8F%A3)

豆丁网文章：[IEE1284标准简介](https://www.docin.com/p-875342530.html)

豆丁网文章：[IEEE1284并行接口的设计与实现](https://www.docin.com/p-1519508508.html)

豆丁网文章：[Linux打印机驱动源码分析](https://www.docin.com/mobile/detail.do?id=441082105)

Linux and UNIX Man Pages：[usb_printerid(1)](https://www.unix.com/man-page/debian/1/usb_printerid)

Ubuntu manuals：[usb_printerid](https://manpages.ubuntu.com/manpages/xenial/man1/usb_printerid.1.html)

Microsoft文档：[连接到 LPT 端口的打印机](https://learn.microsoft.com/zh-cn/windows-hardware/drivers/print/printer-connected-to-an-lpt-port)

Microsoft文档：[IOCTL_USBPRINT_GET_1284_ID IOCTL (usbprint.h)](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/usbprint/ni-usbprint-ioctl_usbprint_get_1284_id)