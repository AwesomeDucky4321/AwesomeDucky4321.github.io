# Unit 1 Summative: Ultrasonic Proximity Sound Device

## Overview
My device uses an HC-SR04 ultrasonic distance sensor to detect how close something is, then responds with different LEDs and buzzer sounds depending on the distance. Each distance "zone" has its own LED and its own sound.

## Why I chose this
I drew influence when I say my classmate Cruise building a parking sensor where at each distances a certain color of LED would light up, and at the closest distance a buzzer would beep. I made something similar but with 4 LEDs and each LED played a different song.

## New components & sources
- **Input:** HC-SR04 ultrasonic distance sensor
- **Output:** Piezo buzzer + multiple LEDs
- **Starting tutorial:** "Getting Started with the HC-SR04 Ultrasonic Sensor" by Isaac100 on Arduino Project Hub, which gave me the basic code to measure and print distance.

### Step 1: Getting the sensor working
I started with the Isaac100 tutorial code, which printed the distance to the Serial Monitor. Then I added one LED that turned on when something got closer than 20cm.

### Step 2: Adding sound
I added a piezo buzzer and experimented with different effects: a parking-sensor beep that sped up as you got closer, a "theremin" where distance controlled the pitch, and a proximity alarm with a siren sound.

### Step 3: Multiple zones
I expanded to several LEDs, each representing a distance zone. Only one LED is on at a time, and each zone plays a different sound.

### What didn't work (and what I changed)
- **Two buzzers wouldn't play at once.** I tried adding a second buzzer so one could play a siren while the other screamed a steady tone, but only one ever made sound. I learned that Arduino's `tone()` function uses a single hardware timer, so it can only produce one tone at a time. I went back to one buzzer.
- **Songs couldn't be interrupted.** When a song like Happy Birthday started, I couldn't switch to another zone until it finished, because `delay()` freezes the whole program. I fixed this by restructuring the code to play **one note per pass through `loop()`**, so the sensor gets checked between every note and zones switch instantly.

## Final build
<img width="4284" height="5712" alt="IMG_3247" src="https://github.com/user-attachments/assets/b3c38ce3-a6e8-4525-9aec-50a084c50b60" />

**Wiring:**
- LED 1 → pin 11
- LED 2 → pin 6
- Buzzer → pin 8
- Sensor trig → pin 9
- Sensor echo → pin 10

**Code:**
```cpp
/*
 * 4-Stage Proximity Sound Alarm (Non-blocking)
 * 
 * Zone 1 (65-85cm): LED1 (pin 6)  + "Twinkle Twinkle Little Star"
 * Zone 2 (45-65cm): LED2 (pin 12) + "Happy Birthday"
 * Zone 3 (25-45cm): LED3 (pin 11) + alarm siren
 * Zone 4 (<25cm):   LED4 (pin 13) + SOS in Morse code
 */

const int trigPin = 9;
const int echoPin = 10;
const int led1Pin = 6;
const int led2Pin = 12;
const int led3Pin = 11;
const int led4Pin = 13;
const int buzzerPin = 8;

int veryFarDistance = 85;
int farDistance = 65;
int mediumDistance = 45;
int closeDistance = 25;

#define NOTE_C4  262
#define NOTE_D4  294
#define NOTE_E4  330
#define NOTE_F4  349
#define NOTE_G4  392
#define NOTE_A4  440
#define NOTE_C5  523

int twinkleMelody[] = {
  NOTE_C4, NOTE_C4, NOTE_G4, NOTE_G4, NOTE_A4, NOTE_A4, NOTE_G4,
  NOTE_F4, NOTE_F4, NOTE_E4, NOTE_E4, NOTE_D4, NOTE_D4, NOTE_C4
};
int twinkleDurations[] = {
  400, 400, 400, 400, 400, 400, 800,
  400, 400, 400, 400, 400, 400, 800
};
int twinkleLength = 14;

int hbMelody[] = {
  NOTE_C4, NOTE_C4, NOTE_D4, NOTE_C4, NOTE_F4, NOTE_E4,
  NOTE_C4, NOTE_C4, NOTE_D4, NOTE_C4, NOTE_G4, NOTE_F4
};
int hbDurations[] = {
  300, 100, 400, 400, 400, 800,
  300, 100, 400, 400, 400, 800
};
int hbLength = 12;

// SOS: 3 dots, 3 dashes, 3 dots (true = dash)
bool sosPattern[] = {false,false,false, true,true,true, false,false,false};
int sosLength = 9;

int currentZone = 0;   // which zone we were in last pass
int noteIndex = 0;     // which note/symbol we're on

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(led1Pin, OUTPUT);
  pinMode(led2Pin, OUTPUT);
  pinMode(led3Pin, OUTPUT);
  pinMode(led4Pin, OUTPUT);
  pinMode(buzzerPin, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  float distance = getDistance();

  // Figure out which zone we're in
  int zone = 0;
  if (distance < closeDistance)        zone = 4;
  else if (distance < mediumDistance)  zone = 3;
  else if (distance < farDistance)     zone = 2;
  else if (distance < veryFarDistance) zone = 1;

  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.print("  Zone: ");
  Serial.println(zone);

  // If we changed zones, restart the sound from the beginning
  if (zone != currentZone) {
    currentZone = zone;
    noteIndex = 0;
    noTone(buzzerPin);
  }

  // Set LEDs - only the active zone's LED is on
  digitalWrite(led1Pin, zone == 1 ? HIGH : LOW);
  digitalWrite(led2Pin, zone == 2 ? HIGH : LOW);
  digitalWrite(led3Pin, zone == 3 ? HIGH : LOW);
  digitalWrite(led4Pin, zone == 4 ? HIGH : LOW);

  // Play just ONE note/beat this pass, then let loop() run again
  if (zone == 1) {
    playOneNote(twinkleMelody, twinkleDurations, twinkleLength);
  }
  else if (zone == 2) {
    playOneNote(hbMelody, hbDurations, hbLength);
  }
  else if (zone == 3) {
    playAlarmBeat();
  }
  else if (zone == 4) {
    playOneMorseSymbol();
  }
  else {
    noTone(buzzerPin);
  }
}

// Plays a single note from a melody, then advances to the next one
void playOneNote(int melody[], int durations[], int length) {
  tone(buzzerPin, melody[noteIndex], durations[noteIndex]);
  delay(durations[noteIndex] * 1.1);
  noteIndex++;
  if (noteIndex >= length) noteIndex = 0;  // loop the song
}

// One WEE WOO sweep
void playAlarmBeat() {
  for (int freq = 500; freq <= 1500; freq += 40) {
    tone(buzzerPin, freq);
    delay(4);
  }
  for (int freq = 1500; freq >= 500; freq -= 40) {
    tone(buzzerPin, freq);
    delay(4);
  }
  noTone(buzzerPin);
}

// One dot or dash of the SOS pattern
void playOneMorseSymbol() {
  int dur = sosPattern[noteIndex] ? 600 : 200;
  tone(buzzerPin, 1000, dur);
  delay(dur + 200);

  noteIndex++;
  if (noteIndex >= sosLength) {
    noteIndex = 0;
    delay(700);  // pause before repeating SOS
  }
}

float getDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH, 25000);
  if (duration == 0) return 999;

  return duration * 0.0343 / 2;
}
```

## Technical Tidbit: How the HC-SR04 measures distance

The sensor works like a bat using echolocation. It sends out a sound that's too high for people to hear, and the sound bounces off an object and comes back. The Arduino measures how long that takes. Sound travels about 0.0343 cm every microsecond, and it goes to the object and back, so the code divides by 2:

distance = time × 0.0343 ÷ 2

## Peer Support
My classmate and friend Dylan noticed that I mixed up some of my wiring and had the pins wrong in the code. He pointed it out to me and taught me how to fix it.

## Use-Case Reflection
**Problem & who it helps:** 

An alert for people if something gets too close. Like a car parking and once the sensor detects it about to hit something it starts beeping to warn the driver.

**What would need to change:** 

A better sensor so it can detect further. The sensor I used for this project only consistently works for under 1 meter distance. Past that, it becomes stuttery and can't accurately measure the distance.

**Skill I'd rely on most:** Debugging. Almost every feature I added broke something at first, and using the Serial Monitor to print distance and zone values was how I figured out what was going wrong.
