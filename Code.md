#include <LiquidCrystal_I2C.h>
#include <Wire.h>

LiquidCrystal_I2C lcd(0x27,16,2);

const int pingPin = 13; // Trigger Pin of Ultrasonic Sensor
const int echoPin = 12; // Echo Pin of Ultrasonic Sensor
long duration, inches, cm;
int A = 9;        // dark green
int B = 5;        // red wire
int speed;
char input;       // to store input character received via BT.


void setup()
{
  Serial.begin(9600);
  
    pinMode(pingPin, OUTPUT);
    pinMode(echoPin, INPUT);

  pinMode(4, OUTPUT);
  pinMode(6, OUTPUT);
  pinMode(10, OUTPUT);
  pinMode(11, OUTPUT);

  pinMode(A0, INPUT);
  pinMode(9, OUTPUT);
  pinMode(5, OUTPUT);
  

  lcd.begin();
  lcd.setCursor(0, 0);
  lcd.print("Bluetooth RC Car");
  lcd.setCursor(0, 1);
  lcd.print("Ready for input");
}

void loop()
{
  speed = analogRead(A0);
  speed = speed * 0.249;
  analogWrite(A, speed);
  analogWrite(B, speed);

    
  distance();
   if (cm >40) // check if there is no obstacle within 40 cm
  {
    // read input from Bluetooth module
    if (Serial.available())
    {
      // wait a bit for the entire message to arrive
      delay(100);
      // clear the screen
      lcd.clear();
      
      // read all the available characters
      while (Serial.available())
      {
        input = Serial.read();
        if (input == 'F')
        {
          forward();
          lcd.setCursor(0, 0);
          lcd.print("Forward");
          lcd.setCursor(0, 1);
          lcd.print("speed =");
          lcd.print(speed);
        }
        else if (input == 'R')
        {
          right();
          lcd.setCursor(0, 0);
          lcd.print("Right");
          lcd.setCursor(0, 1);
          lcd.print("speed =");
          lcd.print(speed);
        }
        else if (input == 'L')
        {
          left();
          lcd.setCursor(0, 0);
          lcd.print("left");
          lcd.setCursor(0, 1);
          lcd.print("speed =");
          lcd.print(speed);
        }
        else if (input == 'G')
        {
          backward();
          lcd.setCursor(0, 0);
          lcd.print("backward");
          lcd.setCursor(0, 1);
          lcd.print("speed =");
          lcd.print(speed);
        }
        else if (input == 'S')
        {
          stop();
          lcd.print("Car Stoped");
          
        }
      }
    }
  }
  else if(cm<=40 && Serial.available()) // if there is an obstacle within 40 cm, stop the car
  {
    stop();
    lcd.clear();
    lcd.print("Obstacle Detected");
    stop();
    delay(1000);
    backward();
    delay(800);
  }

  else if(cm<=40 && !Serial.available())
  {
    stop();
    lcd.clear();
    lcd.print("no input ");
    stop();
  }
  
}

long microsecondsToInches(long microseconds) {
   return microseconds / 74 / 2;
}

long microsecondsToCentimeters(long microseconds) {
   return microseconds / 29 / 2;
}

void distance()
{
      digitalWrite(pingPin, LOW);
      delayMicroseconds(2);
      digitalWrite(pingPin, HIGH);
      delayMicroseconds(10);
      digitalWrite(pingPin, LOW);
      
      duration = pulseIn(echoPin, HIGH);
      inches = microsecondsToInches(duration);
      cm = microsecondsToCentimeters(duration);
      Serial.print(inches);
      Serial.print("in, ");
      Serial.print(cm);
      Serial.print("cm");
      Serial.println();
      delay(100);

}

void forward()
{
  digitalWrite(6, LOW);
  digitalWrite(4, HIGH);
  digitalWrite(10, HIGH);
  digitalWrite(11, LOW);
  
}

void stop()
{
  digitalWrite(6, LOW);
  digitalWrite(4, LOW);
  digitalWrite(10, LOW);
  digitalWrite(11, LOW);
}

void backward()
{
  digitalWrite(6, HIGH);
  digitalWrite(4, LOW);
  digitalWrite(10, LOW);
  digitalWrite(11, HIGH);
}

void right()
{
  digitalWrite(6, HIGH);
  digitalWrite(4, LOW);
  digitalWrite(10, HIGH);
  digitalWrite(11, LOW);
  
}

void left()
{
  digitalWrite(6, LOW);
  digitalWrite(4, HIGH);
  digitalWrite(10, LOW);
  digitalWrite(11, HIGH);
}
