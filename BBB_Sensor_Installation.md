# BeagleBone Black 장치의 센서 드라이버 설치 가이드

본 챕터는 BeagleBone Black(이하 BBB)에서 동작하는 10종 센서에 대한 설치 방법 및 동작 확인 예제 코드를 제공합니다.

### 1. Sensor Development Environment
1. BeagleBone Black 보드에 NeuroMeka사의 SensorCape KIT을 연결하여, 센서를 부착한다.
![](images/800px-Sensorcape_img.png)

2. BBB사에서 제공하는 기본 커널에서 센서를 구동한다.
3. 각 센서의 드라이버 코드는 dtbo 파일을 제공하고, 이를 커널에 로드한다.
4. 예제 코드는 드라이버 파일이 로드된 것을 가정하고 동작한다.

### 2. Sensor List & Feature

 BBB에서 사용되는 센서 목록을 나열하고, 센서 종류 및 특징을 설명한다.
![](images/10sensor.png)

* **Motion** - 센서 이름 : [PassiveInfraRed], 종류 : 움직임 감지 센서, 특징 : GPIO로 연결되며, 현재 개발 환경에서 GPIO 51번에 연결되어 있다. 평소에는 0값을 리턴하고, 움직임을 감지할 경우 1값을 리턴한다.
* **Temperature** - 센서 이름 : [DS18B20], 종류 : 움직임 감지센서, 특징 : 1-wire로 연결되며, 드라이버 파일이 로드되면, 장치에서 값을 읽을 수 있게 된다. Celsius degree 로 표기되며, 소스점 3째 자리까지 표기된다.
* **Light** - 센서 이름 : [BH1750], 종류 : 조도 센서, 특징 : 디바이스 드라이버를 설치할 필요가 없다. BBB에서 제공하는 I2C-1로 연결되어 동작한다. 조도 값을 리턴하며, 측점 범위는  0~65535lx(럭스) 이다.
* **Humidity** - 센서 이름 : [HTU21D], 종류 : 습도 센서, 특징 : 디바이스 드라이버를 설치할 필요가 없다. BBB에서 제공하는 I2C-1로 연결되어 동작한다. 현재 습도를 측정하며, 값은 0-100% 로 나타낸다.
* **Button** - 센서 이름 : [Push Button Switch], 종류 : 버튼, 특징 : 버튼 센서로 현재 개발 환경에서 GPIO 51번에 연결되며, 평소에는 1값을 리턴하고, 버튼을 누를 경우 0값을 리턴한다.
* **Noise** - 센서 이름 : [FC-04], 종류 : 소리 감지 센서, 특징 : 소리 감지 센서로 현재 개발 환경에서 GPIO 51번에 연결되며, 평소에는 0값을 리턴하고, 특정 크기 이상의 소리를 감지하면 1값을 리턴한다.
* **Dust** - 센서 이름 : [PM1001], 종류 : 먼지 측정 센서, 특징 : 먼저 측정 센서로 현재 먼지 농도를 PCS/L 단위로 나타내며, BBB 에서 기본적으로 제공하는 UART5 Serial Port에 연결된다. (Co2 센서와 동시에 동작할 수 없다)
* **Co2** - 센서 이름 : [CM1101], 종류 : Co2 측정 센서, 특징 : Co2 측정 센서로 현재 Co2 농도를 0～2000ppm로 나타내며, BBB 에서 기본적으로 제공하는 UART5 Serial Port에 연결된다. (먼지 센서와 동시에 동작할 수 없다)
* **Door** - 센서 이름 : [MC38], 종류 : Magnetic(Door 센서), 특징 : Door 센서로 현재 개발 환경에서 GPIO 51번에 연결되며, 평소에는 1값을 리턴하고, Magnetic이 감지될 경우(Door Closed) 0값을 리턴한다.
* **LED** - 센서 이름 : [RGB LED], 종류 : LED 센서, 특징 :  LED 센서로 현재 개발 환경에서 GPIO 5번에 빨간색 발광 다이오드가, GPIO 49번에 초록색 발광 다이오드가, GPIO 115번 파랑색 발광 다이오드가 연결되어 있으며, GPIO 값을 조절하여, 원하는 색이 점등되록 조절할 수 있다.

### 3. Sensor Driver Installation

BBB에서 사용되는 10종 센서는 4가지 연결방식을 사용한다. 1) UART , 2) 1-WIRE , 3) GPIO , 4) I2C 각 연결 방식을 지원하도록 Device Tree Overlay라는 커널 기능을 이용하여, 원하는 포트와 핀을 원하는 형태의 인터페이스로 설정한다. 본 문서에서 설명하는 방법은 NeuroMeka에서 제공하는 SensorCape와 호환이 되는 드라이버에 대한 설명이다. 
미들웨어 패키지가 설치된 상태에서는 드라이버 설치 과정은 무시한다.

##### 1) UART
* BBB로 접속하여, 기본적으로 제공하는 UART5를 동작시킨다.

* **Host PC** : ssh 명령을 이용하여 보드에 접속한다. (ssh [BBB ID]@[BBB IP] )
	```
	# ssh root@192.168.7.2
	= BBB 접속 =
	```
* **BBB** : BB-UART5 를 로드한다. cat을 통해 확인하였을 때, BB-UART5가 보이면 정상이다.

	```
	# echo BB-UART5 > /sys/devices/bone_capemgr.9/slots
	# cat /sys/devices/bone_capemgr.9/slots
	 0: 54:PF--- 
	 1: 55:PF--- 
	 2: 56:PF--- 
	 3: 57:PF--- 
	 4: ff:P-O-L Bone-LT-eMMC-2G,00A0,Texas Instrument,BB-BONE-EMMC-2G
	 5: ff:P-O-- Bone-Black-HDMI,00A0,Texas Instrument,BB-BONELT-HDMI
	 6: ff:P-O-- Bone-Black-HDMIN,00A0,Texas Instrument,BB-BONELT-HDMIN
	 7: ff:P-O-L Override Board Name,00A0,Override Manuf,BB-UART5
	```

* **Error** : BB-UART5가 로드 되지 않을 경우, 다음 명령을 수행한다.

	* 이미 포트와 핀이 사용 중인 경우
	```
	# echo BB-UART5 > /sys/devices/bone_capemgr.9/slots
	-bash: echo: write error: File exists
	```
	* 해결방법 : 포트와 핀이 사용되지 않도록 Disable 한다. 아래 내용을 추가하거나 주석을 해제한다. 재부팅 후 다시 UART5를 로드한다.
	```
	# vi /boot/uEnv.txt
	...
	##Disable HDMI (v3.8.x)
	cape_disable=capemgr.disable_partno=BB-BONELT-HDMI,BB-BONELT-HDMIN
	...
	:wq
	# reboot -f
	```

* UART5/HDMI 형식의 센서 사용시 주의사항

	* BeagleboneBlack에서 HDMI 포트와 UART포트가 겹치는 현상이 있어 둘중에 하나만 사용이 가능하다.
	* 미들웨어 설치시 HDMI포트 자동으로 Disable하고 UART를 Enable 하기 때문에, HDMI를 다시 사용 할 경우 다음과 같은 절차를 따른다.
	```
	# dpkg -r devicemiddleware
	# vi /boot/uEnv.txt

	##Disable HDMI (v3.8.x)
	#cape_disable=capemgr.disable_partno=BB-BONELT-HDMI,BB-BONELT-HDMIN
	```
	> cape_disable 앞에 # 을 추가한 후 reboot 명령어로 재시작하면 HDMI 포트 사용이 가능해짐과 동시에 UART 사용 불가

##### 2) 1-WIRE
* 파일을 로드하고 이를 확인한다. BB-W1이 보이면 정상이다.

	```
	# cp BB-W1-00A0.dtbo /lib/firmware
	# echo BB-W1 > /sys/devices/bone_capemgr.9/slots
	# cat /sys/devices/bone_capemgr.9/slots
	 0: 54:PF--- 
	 1: 55:PF--- 
	 2: 56:PF--- 
	 3: 57:PF--- 
	 4: ff:P-O-L Bone-LT-eMMC-2G,00A0,Texas Instrument,BB-BONE-EMMC-2G
	 5: ff:P-O-- Bone-Black-HDMI,00A0,Texas Instrument,BB-BONELT-HDMI
	 6: ff:P-O-- Bone-Black-HDMIN,00A0,Texas Instrument,BB-BONELT-HDMIN
	 7: ff:P-O-L Override Board Name,00A0,Override Manuf,BB-W1
	```

##### 3) GPIO
* 파일을 로드하고 이를 확인한다. BB-IGOT-GPIO 이 보이면 정상이다.

	```
	# cp BB-IGOT-GPIO-00A0.dtbo /lib/firmware
	# echo BB-IGOT-GPIO > /sys/devices/bone_capemgr.9/slots
	# cat /sys/devices/bone_capemgr.9/slots
	 0: 54:PF--- 
	 1: 55:PF--- 
	 2: 56:PF--- 
	 3: 57:PF--- 
	 4: ff:P-O-L Bone-LT-eMMC-2G,00A0,Texas Instrument,BB-BONE-EMMC-2G
	 5: ff:P-O-- Bone-Black-HDMI,00A0,Texas Instrument,BB-BONELT-HDMI
	 6: ff:P-O-- Bone-Black-HDMIN,00A0,Texas Instrument,BB-BONELT-HDMIN
	 8: ff:P-O-L Override Board Name,00A0,Override Manuf,BB-IGOT-GPIO
	```

#### 4) I2C
* I2C는 BBB에서 기본적으로 포트를 제공한다. 따라서 포트가 동작 중인지 확인한다.

	```
	# ssh root@192.168.7.2
	= BBB 접속 =
	# ls /dev/i2c-1
	/dev/i2c-1
	```

#### 5) 부팅 시 자동 드라이버 로드
* 드라이버 인터페이스는 재부팅 할 때마다 다시 로드해주어야 한다. 따라서 부팅 시 자동으로 로드하도록 하는 작업이 필요하다. /etc/rc.local 파일을 수정하여 재부팅 후에도 자동으로 드라이버가 초기화되도록 설정할 수 있다.
* /etc/rc.local file에 다음 명령어를 추가한다. 재부팅 시 자동으로 드라이버가 로드된다.

	```
	#  vi /etc/rc.local
	...
	echo BB-UART5 > /sys/devices/bone_capemgr.9/slots
	echo BB-W1 > /sys/devices/bone_capemgr.9/slots
	echo BB-IGOT-GPIO > /sys/devices/bone_capemgr.9/slots
	...
	:wq
	# reboot -f
	```

### 4. 부록 (Example Code)
##### 1) Motion | Button | Noise | Door  (GPIO51)
-----
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <signal.h>

#define FILE_PATH "/sys/class/gpio/gpio51/value"
#define ENABLE_GPIO_51 "echo 51 > /sys/class/gpio/export"
#define DATA_MAX_SIZE 2

FILE *gpio_fp = NULL;

void signal_handler(int sig)
{
    printf("GPIO EXIT\n");
    if(gpio_fp != NULL)fclose(gpio_fp);
    exit(1);
}

int main(void)
{
    system(ENABLE_GPIO_51);
    char data[DATA_MAX_SIZE];
    char *ret;

    signal(SIGTERM, signal_handler);
    signal(SIGINT, signal_handler);

    while(1) {
        sleep(1);
        gpio_fp = fopen(FILE_PATH,"r");

        if(gpio_fp == NULL) {
            printf("File open Error!\n");
            return -1; 
        }

        (void)memset(data,0,DATA_MAX_SIZE);
        ret = fgets( data, DATA_MAX_SIZE, gpio_fp);
        printf("=> <%s>\n",data);
        if( ret != NULL) {
            fseek( gpio_fp,  0, SEEK_SET);    
        }
        sleep(1);
        fclose(gpio_fp);
        gpio_fp = NULL;
    }   

}
```
##### 2) LED (GPIO5,GPIO49,GPIO115)
-----
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <signal.h>

#define GPIO5_FILE_PATH      "/sys/class/gpio/gpio5/value"
#define GPIO49_FILE_PATH     "/sys/class/gpio/gpio49/value"
#define GPIO115_FILE_PATH    "/sys/class/gpio/gpio115/value"

#define ENABLE_GPIO_5        "echo 5 > /sys/class/gpio/export"
#define ENABLE_GPIO_49       "echo 49 > /sys/class/gpio/export"
#define ENABLE_GPIO_115      "echo 115 > /sys/class/gpio/export"

#define REDIRECTION_GPIO_5   "echo out > /sys/class/gpio/gpio5/direction"
#define REDIRECTION_GPIO_49  "echo out > /sys/class/gpio/gpio49/direction"
#define REDIRECTION_GPIO_115 "echo out > /sys/class/gpio/gpio115/direction"

#define DATA_MAX_SIZE 2

FILE *gpio5_fp = NULL;
FILE *gpio49_fp = NULL;
FILE *gpio115_fp = NULL;

void signal_handler(int sig)
{
    printf("LED EXIT\n");
    if(gpio5_fp != NULL) {
        fwrite("0", sizeof(char), 1, gpio5_fp);
        fclose(gpio5_fp);
    }   
    if(gpio49_fp != NULL) {
        fwrite("0", sizeof(char), 1, gpio49_fp);
        fclose(gpio49_fp);
    }   
    if(gpio115_fp != NULL) {
        fwrite("0", sizeof(char), 1, gpio115_fp);
        fclose(gpio115_fp);
    }   
    exit(1);
}

int main(void)
{
    char *ret;

    system(ENABLE_GPIO_5);
    system(ENABLE_GPIO_49);
    system(ENABLE_GPIO_115);
    system(REDIRECTION_GPIO_5);
    system(REDIRECTION_GPIO_49);
    system(REDIRECTION_GPIO_115);
    signal(SIGTERM, signal_handler);
    signal(SIGINT, signal_handler);

    while(1) {
        sleep(1);
        gpio5_fp   = fopen(GPIO5_FILE_PATH  ,"w");
        gpio49_fp  = fopen(GPIO49_FILE_PATH ,"w");
        gpio115_fp = fopen(GPIO115_FILE_PATH,"w");

        if(gpio5_fp == NULL) {
            printf("File open Error!\n");
            return -1;
        }
        if(gpio49_fp == NULL) {
            printf("File open Error!\n");
            return -1;
        }
        if(gpio115_fp == NULL) {
            printf("File open Error!\n");
            return -1;
        }

        printf("LED START\n");

        sleep(1);

        printf("LED OFF\n");
        fwrite("0", sizeof(char), 1, gpio5_fp);
        fwrite("0", sizeof(char), 1, gpio49_fp);
        fwrite("0", sizeof(char), 1, gpio115_fp);
        fseek( gpio5_fp  ,  0, SEEK_SET);
        fseek( gpio49_fp ,  0, SEEK_SET);
        fseek( gpio115_fp,  0, SEEK_SET);

        sleep(1);

        printf("LED RED\n");
        fwrite("1", sizeof(char), 1, gpio5_fp);
        fwrite("0", sizeof(char), 1, gpio49_fp);
        fwrite("0", sizeof(char), 1, gpio115_fp);
        fseek( gpio5_fp  ,  0, SEEK_SET);
        fseek( gpio49_fp ,  0, SEEK_SET);
        fseek( gpio115_fp,  0, SEEK_SET);

        sleep(1);

        printf("LED GREEN\n");
        fwrite("0", sizeof(char), 1, gpio5_fp);
        fwrite("1", sizeof(char), 1, gpio49_fp);
        fwrite("0", sizeof(char), 1, gpio115_fp);
        fseek( gpio5_fp  ,  0, SEEK_SET);
        fseek( gpio49_fp ,  0, SEEK_SET);
        fseek( gpio115_fp,  0, SEEK_SET);
        sleep(1);

        fclose(gpio5_fp);
        fclose(gpio49_fp);
        fclose(gpio115_fp);
        gpio5_fp   = NULL;
        gpio49_fp  = NULL;
        gpio115_fp = NULL;
    }

    return 0;
}
```
##### 3) Temperature (1-Wire)
-----
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <signal.h>
#include <sys/stat.h>

#define BUFF_SIZE  128 
#define MASTER_FILENAME  "cat /sys/bus/w1/devices/w1_bus_master1/w1_master_slaves"

static int gDS18B20Fd = -1; 
char slavePath[256];

FILE *gW1MasterFd;

int getTemperatureValue(char *data)
{
    int value = 0;
    if(data == NULL) {
        return 0;
    }   
    char *ptr;
    //Extract temperature value
    ptr = strtok(data,"=");
    ptr = strtok(NULL,"=");
    ptr = strtok(NULL,"=");
    value = atoi(ptr);
    return value;
}

void signal_handler(int sig)
{
    printf("TEMPERATURE EXIT\n");
    if(gDS18B20Fd > 0)close(gDS18B20Fd);
    exit(1);
}

int main(void)
{
    char slaveName[256];
    float value;
    int  readByte = 0;
    char buf[BUFF_SIZE];

    signal(SIGTERM, signal_handler);
    signal(SIGINT, signal_handler);

    gW1MasterFd = popen(MASTER_FILENAME, "r");
    if(gW1MasterFd == NULL) {   
        return -1; 
    }   
    (void)memset(slaveName,0,256);
    fgets(slaveName,255,gW1MasterFd);
    slaveName[strlen(slaveName)-1] = '\0';
    sprintf(slavePath,"/sys/bus/w1/devices/");
    strcat(slavePath,slaveName);
    strcat(slavePath,"/w1_slave");
    fclose(gW1MasterFd);


    while(1) {
        memset(buf, 0, BUFF_SIZE);

        gDS18B20Fd = open(slavePath,O_RDONLY, S_IREAD);
        if(gDS18B20Fd < 0) {
            printf("File open Error!\n");
            return -1;
        }

        readByte = read(gDS18B20Fd, buf, BUFF_SIZE);
        if(readByte <= 0) {
            close(gDS18B20Fd);
            return -1;
        }


        value = (float)getTemperatureValue(buf);
        value = value / 1000;

        printf(" temperature = <%f>\n",value);

        close(gDS18B20Fd);
        sleep(2);
    }

    return 0;
}
```
##### 4) Light (I2C)
-----
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/stat.h>
#include <fcntl.h> 
#include <string.h>
#include <sys/ioctl.h>
#include <linux/i2c.h>
#include <linux/i2c-dev.h>
#include <signal.h>

#define LIGHT_ADDRESS 0x23
#define BUFF_SIZE           32
#define ADAPTER_NUMBER    1
#define DEV_I2C_NUMBER    "/dev/i2c-%d"

static int gLightFd = -1;

void signal_handler(int sig)
{
    printf("LIGHT EXIT\n");
    if(gLightFd > 0)close(gLightFd);
    exit(1);
}

int main(void)
{
    char fileName[20];
    char  buf[BUFF_SIZE];
    int   sensorData[3];
    int   readByte  = 0;
    int   writeByte = 0;
    float value     = 0;

    signal(SIGTERM, signal_handler);
    signal(SIGINT, signal_handler);

    memset(buf,0,BUFF_SIZE);
    memset(fileName, 0, 20);
    snprintf(fileName, 19, DEV_I2C_NUMBER, ADAPTER_NUMBER);

    while(1) {
        gLightFd = open(fileName, O_RDWR);
        if(gLightFd < 0) {
            printf("File open Error!\n");
            return -1;
        }
        if(ioctl(gLightFd, I2C_SLAVE, LIGHT_ADDRESS) < 0)
        {
            printf("ioctl Error!\n");
            close(gLightFd);
            return -1;
        }
        buf[0] = 0x11;
        writeByte = write(gLightFd,buf,1);
        if(writeByte < 0) {
            printf("Device light reg write Error\n");
            close(gLightFd);
            return -1;
        }
        (void)memset(buf,0,BUFF_SIZE);
        readByte = read(gLightFd, buf, BUFF_SIZE);
        if(readByte <= 0) {
            printf("read Error!\n");
            return -1;
        }

        sensorData[0] = (int)buf[0];
        sensorData[1] = (int)buf[1];

        value = (((256*sensorData[0]) + sensorData[1]) / 1.2);

        printf(" light = <%f>\n", value);

        close(gLightFd);
        sleep(2);
    }
    return 0;
}
```
##### 5) Humidity (I2C)
-----
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/stat.h>
#include <fcntl.h> 
#include <string.h>
#include <sys/ioctl.h>
#include <linux/i2c.h>
#include <linux/i2c-dev.h>
#include <signal.h>

#define HUMIDITY_ADDRESS  0x40
#define DEVICE_REG_READ   0xE7
#define DEVICE_REG_HUMD   0xE5
#define BUFF_SIZE         3
#define ADAPTER_NUMBER    1

#define DEV_I2C_NUMBER    "/dev/i2c-%d"

static int gHTU21DFd = -1;

void signal_handler(int sig)
{
    printf("HUMIDITY EXIT\n");
    if(gHTU21DFd > 0)close(gHTU21DFd);
    exit(1);
}

int main(void)
{
    char fileName[20];
    char  buf[BUFF_SIZE];
    int   sensorData[3];
    int   readByte  = 0;
    int   writeByte = 0;
    float value     = 0;

    signal(SIGTERM, signal_handler);
    signal(SIGINT, signal_handler);

    memset(buf,0,BUFF_SIZE);
    memset(fileName, 0, 20);
    snprintf(fileName, 19, DEV_I2C_NUMBER, ADAPTER_NUMBER);

    while(1) {
        gHTU21DFd = open(fileName, O_RDWR);
        if(gHTU21DFd < 0) {
            printf("File open Error!\n");
            return -1;
        }

        if(ioctl(gHTU21DFd, I2C_SLAVE, HUMIDITY_ADDRESS) < 0)
        {
            printf("ioctl Error!\n");
            close(gHTU21DFd);
            return -1;
        }

        buf[0] = DEVICE_REG_READ;
        writeByte = write(gHTU21DFd,buf,1);
        if(writeByte < 0) {
            printf("Device read reg write Error\n");
            close(gHTU21DFd);
            return -1;
        }

        buf[0] = DEVICE_REG_HUMD;
        writeByte = write(gHTU21DFd,buf,1);
        if(writeByte < 0) {
            printf("Device humidity reg write Error\n");
            close(gHTU21DFd);
            return -1;
        }

        memset(buf,0,BUFF_SIZE);
        readByte = read(gHTU21DFd, buf, BUFF_SIZE);
        if(readByte <= 0) {
            printf("read Error\n");
            return -1;
        }

        memset(sensorData, 0, 3);
        sensorData[0] = (int)buf[0];
        sensorData[1] = (int)buf[1];

        value = (((125.0*(((256*sensorData[0]) + sensorData[1])&0xFFFC))/65536) - 6);
        printf(" value = <%f> \n",value);

        close(gHTU21DFd);
        sleep(2);
    }

    return 0;
}
```
##### 6) Dust (UART)
-----
```c
#include <string.h> 
#include <stdlib.h> 
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h> 
#include <termios.h>
#include <signal.h>

int tty_fd = -1;

void signal_handler(int sig)
{
    printf("DUST EXIT\n");
    if(tty_fd > 0)close(tty_fd);
    exit(1);
}

int main(int argc,char** argv)
{
    char Tx_Data;
    char Rx_Data;
    struct termios tio;
    int i = 0;
    int udet = 200;
    int readbytes = 0;
    int writebytes = 0;
    char cs = 0;
    int dust_val = 0;
    int retry_cnt = 0;

    signal(SIGTERM, signal_handler);
    signal(SIGINT, signal_handler);

    memset(&tio,0,sizeof(tio));
    tio.c_iflag=0;
    tio.c_oflag=0;
    tio.c_cflag=CS8|CREAD|CLOCAL;
    tio.c_lflag=0;
    tio.c_cc[VMIN]=1;
    tio.c_cc[VTIME]=0;

    while(1) {

        tty_fd=open("/dev/ttyO5",O_RDWR|O_NONBLOCK);      
        if(tty_fd < 0) {
            printf("file open error!\n");
            return -1;
        }
        cfsetospeed(&tio,B9600);
        tcsetattr(tty_fd,TCSANOW,&tio);

        sleep(1);
        printf("file write -> ");
        Tx_Data = 0x11;
        writebytes = 0;
        writebytes = write(tty_fd,&Tx_Data,1);
        while(writebytes <= 0) {
            usleep(udet);
            writebytes = write(tty_fd,&Tx_Data,1);
        }
        printf("%02x ",Tx_Data);
        usleep(udet);
        Tx_Data = 0x01;
        writebytes = 0;
        writebytes = write(tty_fd,&Tx_Data,1);
        while(writebytes <= 0) {
            usleep(udet);
            writebytes = write(tty_fd,&Tx_Data,1);
        }
        printf("%02x ",Tx_Data);
        usleep(udet);
        Tx_Data = 0x01;
        writebytes = 0;
        writebytes = write(tty_fd,&Tx_Data,1);
        while(writebytes <= 0) {
            usleep(udet);
            writebytes = write(tty_fd,&Tx_Data,1);
        }
        printf("%02x ",Tx_Data);
        usleep(udet);
        Tx_Data = 0xED;
        writebytes = 0;
        writebytes = write(tty_fd,&Tx_Data,1);
        while(writebytes <= 0) {
            usleep(udet);
            writebytes = write(tty_fd,&Tx_Data,1);
        }
        printf("%02x\n",Tx_Data);
        usleep(udet);

        printf("file read\n => ");

        for(i = 0;i < 16; i++) {
            Rx_Data = 0x00;
            readbytes = read(tty_fd,&Rx_Data,1);
            if(readbytes > 0) {
                if( i == 0 && Rx_Data != 0x16 ) {
                    printf("read result error!\n");
                    break;
                }
                if( i == 1 && Rx_Data != 0x0D ) {
                    printf("read length error!\n");
                    break;
                }
                if(i != 15)cs += Rx_Data;
                if(i == 11)printf("(");
                if(i == 3) {
                    dust_val = (Rx_Data << 24);
                    printf("data( ");
                } else if(i == 4) {
                    dust_val = (Rx_Data << 16);
                } else if(i == 5) {
                    dust_val = (Rx_Data << 8);
                }
                printf("%02x ",Rx_Data);
                if(i == 6) {
                    dust_val += Rx_Data;
                    printf(")");
                }
            } else {
                if(retry_cnt == 1000)
                    break;
                retry_cnt++;
                i--;
                usleep(udet*2);
                continue;
            }
        }

        printf("[%02x])<<checksum | DUST = <%d>\n",0x100-cs,dust_val);
        cs = 0;
        dust_val = 0;
        retry_cnt = 0;

        close(tty_fd);
        tty_fd = -1;
        sleep(1);
    }
    return 0;
}
```
##### 7) Co2 (UART)
-----
```c
#include <string.h> 
#include <stdlib.h> 
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h> 
#include <termios.h>
#include <signal.h>

#define UART5_ENABLE "echo BB-UART5 > /sys/devices/bone_capemgr.9/slots"
int tty_fd = -1;

void signal_handler(int sig)
{
    printf("co2 EXIT\n");
    if(tty_fd > 0)close(tty_fd);
    exit(1);
}

int main(int argc,char** argv)
{
    char Tx_Data;
    char Rx_Data;
    struct termios tio;
    int i = 0;
    int udet = 200;
    int readbytes = 0;
    int writebytes = 0;
    char cs = 0;
    int co2_val = 0;
    int retry_cnt = 0;

    system(UART5_ENABLE);

    signal(SIGTERM, signal_handler);
    signal(SIGINT, signal_handler);

    memset(&tio,0,sizeof(tio));
    tio.c_iflag=0;
    tio.c_oflag=0;
    tio.c_cflag=CS8|CREAD|CLOCAL;
    tio.c_lflag=0;
    tio.c_cc[VMIN]=1;
    tio.c_cc[VTIME]=0;

    while(1) {

        tty_fd=open("/dev/ttyO5",O_RDWR|O_NONBLOCK);      
        if(tty_fd < 0) {
            printf("file open error!\n");
            return -1;
        }
        cfsetospeed(&tio,B9600);
        tcsetattr(tty_fd,TCSANOW,&tio);

        sleep(1);
        printf("file write -> ");
        Tx_Data = 0x11;
        writebytes = 0;
        writebytes = write(tty_fd,&Tx_Data,1);
        while(writebytes <= 0) {
            usleep(udet);
            writebytes = write(tty_fd,&Tx_Data,1);
        }
        printf("%02x ",Tx_Data);
        usleep(udet);
        Tx_Data = 0x01;
        writebytes = 0;
        writebytes = write(tty_fd,&Tx_Data,1);
        while(writebytes <= 0) {
            usleep(udet);
            writebytes = write(tty_fd,&Tx_Data,1);
        }
        printf("%02x ",Tx_Data);
        usleep(udet);
        Tx_Data = 0x01;
        writebytes = 0;
        writebytes = write(tty_fd,&Tx_Data,1);
        while(writebytes <= 0) {
            usleep(udet);
            writebytes = write(tty_fd,&Tx_Data,1);
        }
        printf("%02x ",Tx_Data);
        usleep(udet);
        Tx_Data = 0x01;
        writebytes = 0;
        writebytes = write(tty_fd,&Tx_Data,1);
        while(writebytes <= 0) {
            usleep(udet);
            writebytes = write(tty_fd,&Tx_Data,1);
        }
        printf("%02x ",Tx_Data);
        usleep(udet);
        Tx_Data = 0xED;
        writebytes = 0;
        writebytes = write(tty_fd,&Tx_Data,1);
        while(writebytes <= 0) {
            usleep(udet);
            writebytes = write(tty_fd,&Tx_Data,1);
        }
        printf("%02x\n",Tx_Data);
        usleep(udet);

        printf("file read\n => ");

        for(i = 0;i < 12; i++) {
            Rx_Data = 0x00;
            readbytes = read(tty_fd,&Rx_Data,1);
            if(readbytes > 0) {
                if( i == 0 && Rx_Data != 0x16 ) {
                    printf("read result error!\n");
                    break;
                }
                if( i == 1 && Rx_Data != 0x09 ) {
                    printf("read length error!\n");
                    break;
                }
                if(i != 11)cs += Rx_Data;
                if(i == 11)printf("(");
                if(i == 3) {
                    co2_val = (Rx_Data << 8);
                    printf("data( ");
                }
                printf("%02x ",Rx_Data);
                if(i == 4) {
                    co2_val += Rx_Data;
                    printf(")");
                }
            } else {
                if(retry_cnt == 1000)
                    break;
                retry_cnt++;
                i--;
                usleep(udet*2);
                continue;
            }
        }

        printf("[%02x])<<checksum | CO2 = <%d>\n",0x100-cs,co2_val);
        cs = 0;
        co2_val = 0;
        retry_cnt = 0;

        close(tty_fd);
        tty_fd = -1;
        sleep(1);
    }
    return 0;
}
```
