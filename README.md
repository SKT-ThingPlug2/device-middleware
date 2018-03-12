# ThingPlug 2.0 Device Middleware

## ThingPlug 2.0�� ���� Device �̵���� ��ġ �� ���డ�̵�
�� é�ʹ� SKT ThingPlug Device �̵���� ��ġ �� ���� ����� �����Ѵ�.

#### 1. ThingPlug Device �̵���� ��?
������ �繰���ͳ� �÷��� ThingPlug 2.0�� ���� Device �̵����� ThingPlug ���� ������ ���� ���ְ� �پ��� Device �� ���� ������, Device ������ �� ���� �����ڸ� ���� ����Ʈ�����̴�.
![](images/simple_mw_architect.png)
* �̵����� �� 2���� ���� 5���� ������Ʈ�� �����Ǿ� ������, ������ ������Ʈ�� ���� �繰�� ThingPlug �� ������ �����Ѵ�.
* **Management Agent** �� �̵���� ������ ��� ��� �� ó���� �߽ɿ��� �������� ������ �����Ѵ�. User ���� �ܺ� Interface �� Gateway Portal, ThingPlug �ʹ� Connection Ready Agent �� �����ϸ�, ���� Device/Sensor �ʹ� Service Ready Agent �� ���� �����Ѵ�.
* **Connection Ready Agent** �� Simple Protocol �� �����ϸ�, ThingPlug ������ ����� ����Ѵ�. ����� MQTT(S) ����� ����Ѵ�.
* **Service Ready Agent** �� Sensor Management Agent �� ���� ���޹��� ���� ���� ��������, ������ ��å�� ���� �����͸� �����ϴ� ������ �Ѵ�. ������ ���� ������ Management Agent �� �����Ѵ�.
* **Sensor Management Agent** �� ���� �����͸� �����ϰ�, ���� ��� ����ϸ�, Management Agent �� �����͸� �ְ� �޴´�.
* **Gateway Portal** �� ������/�����ڰ� �̵���� �ý����� �����ϰ�, ���� ������ ��ȸ�� �� �ִ� ����� �������̽��̸�, Node.js ����� ���ø����̼����� �����Ǿ� �ִ�.

#### 2. ThingPlug ���� ���� ����
ThingPlug ���� Protocol �� SimpleAPI�� �����Ѵ�.
![](images/v1_overview.png)

#### 3. ���� ��� �� �ϵ����
* ���� ���
  * Memory : 128 Mb �̻�
  * CPU : 200MHz �̻�
* �׽�Ʈ ȯ��
  * Raspberry Pi2/3, BeagleBone Black �� ARM/Linux ����̽�

#### 4. ȯ�� ����

0. ������ ������� ��� �Ʒ��� URL ���� putty �� �ٿ�޾� ��ġ�Ѵ�.
	* http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html
1. ���ͳ� ������ ���Ͽ� Ethernet(LAN ���̺�)�̳� Wi-Fi USB ������ ��ġ�� �����Ѵ�.
2. �͹̳�(������ PC������ putty)�� ���� �� ��ġ ȯ�濡 ���� ��Ʈ��ũ ȯ���� �����Ѵ�.
3. ó�� �����ϴ� ��ġ�� ������Ʈ �� ���׷��̵� �Ѵ�.

	```
	# apt-get update
	# apt-get upgrade
	```

#### 5. �̵����� ����ϴ� Library �ȳ�
�̵����� ����ϴ� Library ���� ������ ����.
<table>
<thead><tr><th>Part</th><th>Library</th><th>Type</th><th>�뵵</th></tr></thead>
<tbody>
<tr><td rowspan="7">Gateway Portal</td><td>express</td><td>��Ű�� ����</td><td>�����ӿ�ũ</td></tr>
<tr><td>express-session</td><td>��Ű�� ����</td><td>Express �� Session �߰�</td></tr>
<tr><td>body-parser</td><td>��Ű�� ����</td><td>Express �� BodyParser �߰�</td></tr>
<tr><td>request</td><td>��Ű�� ����</td><td>http request ����</td></tr>
<tr><td>xml2js</td><td>��Ű�� ����</td><td>XML �Ľ�</td></tr>
<tr><td>ping</td><td>��Ű�� ����</td><td>Ping üũ</td></tr>
<tr><td>i18n</td><td>��Ű�� ����</td><td>�ٱ��� ����</td></tr>
<tr><td rowspan="4">Management Agent</td><td>libcurl</td><td>��Ű�� ����</td><td>�̵���� ���׷��̵�</td></tr>
<tr><td>libpaho-mqtt3as</td><td>��Ű�� ����</td><td>MQTT TLS ���</td></tr>
<tr><td>libsqlite3</td><td>shared</td><td>������ ����</td></tr>
</tbody>
</table>

#### 6. ��Ű�� ��ġ
0. ����� ��Ű�� ������ �ٿ�ε� �Ѵ�.

	```
	# wget https://github.com/SKT-ThingPlug2/raw/master/device-middleware/pkg/thingplug_dmw_ARM_2.0.1_1803061640.deb
	```

1. ����� ��Ű���� ��ġ�Ѵ�.(�ݵ�� root ������ �̿��ؾ� �Ѵ�.)	

	* �Ϲ������� dpkg ����� ���Ͽ� ��Ű���� ��ġ�Ѵ�.
	```
	# dpkg -i thingplug_dmw_ARM_2.0.0_1712221030.deb
	```
	* Library dependencies ���� ������ �߻��� ��� gdebi �� �̿��Ͽ� ��Ű���� ��ġ�Ѵ�.
	```
	# apt-get install gdebi
	# gdebi thingplug_dmw_ARM_2.0.0_1712221030.deb
	```

#### 7. ��Ű�� ��ġ Ȯ��
* ���������� http://IP-address:8000 ������ �����Ͽ� ������ ���� ȭ��(Gateway Portal)�� ������ ��� ��ġ�� �Ϸ�� ���̴�.  
![](images/gpIntro.png)
> �α��� ȭ�鿡�� ���̵� / ��й�ȣ : thingplugadmin / [mac-address(:����)]

* ���� �α��νÿ� ���� ����� �ʿ��ϴ�.
![](images/v1_gp_changeAccount.png)
> �ȳ������� ���ϴ� ���̵�� ��й�ȣ�� �����Ѵ�.

* ������ ����ϸ� ��ú��带 Ȯ���� �� �ִ�.
![](images/gp_dashboard.png)
> ���� ������ ������ȸ > ���� �޴����� ������ ����̹� �����Ŀ� �����ϴ�.

* ������ ���� ��й�ȣ �����ϱ�
![](images/gpPwd.png)
> ȯ�漳�� > �����ڼ��� �޴����� ���� ��й�ȣ�� ������ �� �ִ�.

#### 8. ��� ���
0. ����

	```
	# service middleware stop
	```

1. ����

	```
	# service middleware star
	```

2. �����

	```
	# service middleware restart
	```

3. ���� Ȯ��

	```
	# ps xl | grep middleware
	```

4. ����
	* ���� �� usr/local/middleware ������ resource_v2.db ����, log ������ ��������, conf ������ ���ϵ��� /tmp/middleware_backup_v2_00 ������ ����Ǿ� ��������.

	```
	# dpkg -r devicemiddleware
	```

#### 9. ��� ���
* [ThingPlug Device �̵���� ��� ��� ���̵�](Simple_Guide.md)

#### 10. ����
* [���� ���� ���̵�](SMA_Guide.md)
* [BeagleBone Black ��ġ�� ���� ����̹� ��ġ ���̵�](BBB_Sensor_Installation.md)

