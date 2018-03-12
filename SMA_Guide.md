Middleware (+TLS)
===

Sensor Driver
---

1. Sensor Driver 파일

* user-defs.c
* user-defs.h

2. Driver 선언

Driver의 구분은 NAME String을 이용한다.

user-defs.h 에 아래와 같이 정의한다.

```
DRIVER_DEF(${NAME})
```

3. Driver 구현

센서 동작을 위해 구현 할 함수는 다음과 같다.

user-defs.c 에 아래 함수를 구현하면 미들웨어에서 센서가 동작한다.

!주의! 함수 이름 규칙을 준수하여야 한다.

```
int ${NAME}_Create(void)
int ${NAME}_GetData(char *data, int *len)
int ${NAME}_Control(char *in_buf, int in_len, char *out_buf, int *out_len)
int ${NAME}_Destroy(void)
```
```
return 0(성공) 나머지 실패
```

* Create 함수 : 센서 초기화 관련된 함수를 구현한다
* GetData 함수 : 센서에서 값을 읽는 함수를 구현한다
  * data : 읽은 값을 전달한다.
  * len : 읽은 값의 data size를 전달한다.
* Control 함수 : 센서를 제어하기 위한 함수를 구현한다
  * in_buf : 제어해 필요한 값을 입력받는다.
  * len : 입력 받은 값의 크기이다.
  * out_buf : 제어 후 결과를 전달한다.
  * len : 결과 값의 크기이다.
* Destroy 함수 : 센서가 종료된 때 처리되는 함수를 구현한다

