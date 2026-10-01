# Spinning 2D Scanner (Part 1)

## The goal
I'm building a spinning 2D scanner: an ultrasonic distance sensor mounted on a motor that rotates, taking a distance reading every few degrees. If you plot those readings, you get a rough outline of the room around the sensor. This is the simple version of a 3D scanner, which we'll build toward later.

## Parts I used
- Arduino Uno
- 28BYJ-48 stepper motor with a ULN2003 driver board
- HC-SR04 ultrasonic distance sensor
- Jumper wires

I used the built-in `Stepper` library that comes with the Arduino IDE.

## How I wired it
- Stepper driver IN1, IN2, IN3, IN4 → pins 2, 3, 4, 5
- Sensor trig → pin 9, echo → pin 10
- Driver board power → 5V and GND

![My scanner setup](scanner.jpg)

## What I tried
First I got the motor spinning on its own, then I got the sensor reading distances on its own, and then I combined them. The scanner now turns 5 degrees, stops, takes a reading, prints the angle and distance to the Serial Monitor, and repeats until it has swept 180 degrees. Then it spins back to the start and does it again.

I print the data as CSV, like this:

```
Angle,Distance(cm)
0,45.23
5,44.87
10,12.04
```

That way I can copy it out of the Serial Monitor and paste it into a spreadsheet later to actually graph the shape of the room.

## What didn't work
**The motor wires don't go in the order you'd expect.** In my code, the pins are listed as `Stepper motor(stepsPerRevolution, 2, 4, 3, 5)`. Notice that's 2, 4, 3, 5, not 2, 3, 4, 5. The `Stepper` library fires the coils in a different order than the ULN2003 board labels them, so if you list the pins in plain order, the motor just buzzes and shakes instead of turning. Swapping the middle two numbers fixed it.

## My code
```cpp
#include <Stepper.h>

const int stepsPerRevolution = 2048;

// ULN2003: IN1, IN3, IN2, IN4 order for Stepper library
Stepper motor(stepsPerRevolution, 2, 4, 3, 5);

const int trigPin = 9;
const int echoPin = 10;

void setup() {
  Serial.begin(9600);

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  motor.setSpeed(10);  // RPM

  Serial.println("Angle,Distance(cm)");
}

float getDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH, 30000);

  if (duration == 0) {
    return -1;
  }

  float distance = duration * 0.0343 / 2;

  return distance;
}

void loop() {

  // Scan from 0 to 180 degrees
  for (int angle = 0; angle <= 180; angle += 5) {

    // 2048 steps = approximately 360 degrees
    int steps = (int)(2048.0 * 5.0 / 360.0);

    motor.step(steps);

    delay(100);

    float distance = getDistance();

    Serial.print(angle);
    Serial.print(",");
    Serial.println(distance);

    delay(100);
  }

  // Return to starting position
  for (int angle = 180; angle > 0; angle -= 5) {

    int steps = (int)(2048.0 * 5.0 / 360.0);

    motor.step(-steps);

    delay(5);
  }

  delay(1000);
}
```

## What's next
- Fix the angle drift. The motor takes 2048 steps for a full turn, so 5 degrees should be 28.4 steps, but the Arduino rounds that down to 28. That means every "5 degrees" is really about 4.9 degrees, so by the end of a 180 degree sweep I'm actually only at about 177 degrees. The sweep out and the sweep back also don't use the same number of steps, so the scanner creeps a little further forward every cycle.
- Mount the sensor better so it doesn't wobble while it turns.
