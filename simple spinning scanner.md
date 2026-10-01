# Spinning 2D Scanner (Part 1)

## The goal
I'm building a spinning 2D scanner: an ultrasonic distance sensor mounted on a motor that rotates, taking a distance reading every few degrees. If you plot those readings, you get a rough outline of the room around the sensor. This is the simple version of a 3D scanner, which we'll build toward later.

## Parts I used
- Arduino Uno
- 28BYJ-48 stepper motor with a ULN2003 driver board
- HC-SR04 ultrasonic distance sensor
- Jumper wires

I used the built-in `Stepper` library that comes with the Arduino IDE. [Add the tutorial or page you used for wiring the 28BYJ-48 here.]

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

[Add anything else that went wrong: the sensor falling off the motor, loose wires, readings that made no sense, etc.]

## My code
```cpp
[Paste your code here]
```

## What's next
- Graph the data. Right now I just get a list of numbers, so the next step is plotting angle and distance on a polar graph so it actually looks like a scan.
- Fix the angle drift. The motor takes 2048 steps for a full turn, so 5 degrees should be 28.4 steps, but the Arduino rounds that down to 28. That means every "5 degrees" is really about 4.9 degrees, so by the end of a 180 degree sweep I'm actually only at about 177 degrees. The sweep out and the sweep back also don't use the same number of steps, so the scanner creeps a little further forward every cycle.
- Mount the sensor better so it doesn't wobble while it turns.
