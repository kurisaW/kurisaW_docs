注意事项：

1.pins_arduino中的引脚定义（使用引脚生成工具）对于外设模式的使用需要使用`小写`

2.如果存在cubemx中不同的版本，需要使用migrate以适配最新版

## PWM

```cpp
#include <Servo.h>

Servo myservo;

void setup() {
  myservo.attach(D11);
}

void loop() {
  for (int pos = 0; pos <= 180; pos += 1) {
    myservo.write(pos);
    delay(15);
  }
  for (int pos = 180; pos >= 0; pos -= 1) {
    myservo.write(pos);
    delay(15);
  }
}
```

## ADC

```cpp
#include <Arduino.h>

void setup(void)
{
}

void loop(void)
{
    Serial.println(analogRead(A0));
    delay(500);
}
```

## UART

```
```

## I2C

```
```

