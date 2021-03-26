# My-Possibilities-Art-Table
Arduino Code for Motorized Art Table Control // Art Table Nunchuck 2.0

#include "Nunchuk.h"
#include <Wire.h>


const int pulsePin = 8; //Pulses vertical motors(green)
const int pulsePinRot = 2; //Pulses rotational motion

const int dirPin = 9; //Sets direction of vertical motion (blue)
const int dirPinRot = 11; //Sets direction of rotational motion

const int enblPinL = 10; // enables left motor (orange)
const int enblPinR = 11; //enables right motor (orange)
const int enblPinRot = 1; //enables rotational motor

const int brakePinTopL = 5; // (white)
const int brakePinTopR = 3; // (white)
const int brakePinBot = 6; // (white) 
//***Need a way to have rotational brakes implemented***

const int solenoid = 13; //Controls solonoid relay

const int lightSwitch = 4; // digital pin to control the light (purple)
const int relay = 7; // controls the light relay (no wire required due to shield)
int tracker = LOW;

int yPos;
int xPos;

bool isVertical; //Boolean flag to check if the user is inputting vertical motion
bool trackTracker = false;

bool rotBrake;

void setup()
{
  Wire.begin();
  pinMode (pulsePin, OUTPUT);
  pinMode (pulsePinRot, OUTPUT);
  
  pinMode (dirPin, OUTPUT);
  pinMode (dirPinRot, OUTPUT);
  
  pinMode (enblPinL, OUTPUT);
  pinMode (enblPinR, OUTPUT);
  pinMode (enblPinRot, OUTPUT);
  
  pinMode (brakePinTopL, INPUT);
  pinMode (brakePinTopR, INPUT);
  pinMode (brakePinBot, INPUT);
 
  pinMode(lightSwitch,INPUT);
  pinMode(relay,OUTPUT);

  pinMode(solenoid,OUTPUT);
  
  Serial.begin(115200);
  nunchuk_init(); 

  digitalWrite(pulsePin, LOW);
  digitalWrite(dirPin, LOW);
}

void loop()
{
  //Read in nunchuk data
  nunchuk_read();
  
  yPos = nunchuk_joystickY();
  xPos = nunchuk_joystickX();

  int topBrakeL = digitalRead(brakePinTopL);
  int topBrakeR = digitalRead(brakePinTopR);
  int botBrake = digitalRead(brakePinBot);

  //Check to enable or disable motors 
  verticalBrake(topBrakeL, topBrakeR, botBrake);

  //Set the vertical direction based off the joystick
  setVerticalDirection();

  //Pulse vertical motors
  pulseVertical();

  //If the user did not input any vertical motion
  if(!isVertical)
  {
 
    //Check to enable or disable brakes
    rotationalBrake();

    //Determine the direction of rotation based off of user input
    setRotationalDirection();

    //If the rotational brakes were triggered
    if(rotBrake)
    {
      //Keep the solonoid locked
      digitalWrite(solenoid, LOW);
      delay(50);
    }
    //If the table will be rotating
    else
    {
      //Move the solonoid out of the way
      digitalWrite(solenoid, HIGH);
      delay(50);
    }

    //Pulse horizontal motors
    pulseHorizontal();
  }


  //If the button to switch the light is pressed
  if(nunchuk_buttonZ() == 1)
  {
    //Make sure it wasn't already pressed
    if(!trackTracker)
    {
      //Toggle the light
      toggleLight();
    }
  }
  //If it isn't pressed
  else
  {
    //Reset the repeated press tracker
    trackTracker = false;
  }

  //If the light should be on, turn it on
  if(tracker)
  {
    digitalWrite(relay,HIGH);
  }
  //If not, turn it off
  else
  {
    digitalWrite(relay,LOW);
  }
}

//-----------------------------------------------------------------------------------------------------------------------------
//This function looks at all the vertical brakes to determine whether to enable
//or disable the motor
void verticalBrake(int topBrakeL, int topBrakeR, int botBrake)
{
  //If y controller is in deadzone, disable motors
  if( (yPos < 70) && (yPos > -70) )
  {
     digitalWrite(enblPinL, HIGH); 
     digitalWrite(enblPinR, HIGH);
     isVertical = false;
  }
  //If they aren't in the deadzone, enable the motors
  else
  {
     digitalWrite(enblPinL, LOW); 
     digitalWrite(enblPinR, LOW);
     isVertical = true;
  }

  //If the user is trying to make the table go up (position above 0)
  //and the top left brake is tripped, disable the left motor
  if( (yPos > 0) && (topBrakeL == 1) )
  {
     digitalWrite(enblPinL, HIGH);
  }
  
  //If the user is trying to make the table go up (position above 512)
  //and the top right brake is tripped, disable the right motor
  if( (yPos > 0) && (topBrakeR == 1) )
  {
     digitalWrite(enblPinR, HIGH);
  }

  //If the user is trying to make the table go down (position below 512)
  //and the bottom brake is tripped, disable both motors
  if( (yPos < 0) && (botBrake == 1) )
  {
     digitalWrite(enblPinR, HIGH);
     digitalWrite(enblPinL, HIGH);
  }
}
//-------------------------------------------------------------------------------------------------
void setVerticalDirection()
{
  //Set direction of motor input based on joystick
  if(yPos < 0)
  {
    digitalWrite(dirPin, HIGH); // set direction based on user input
  }
  else 
  {
    digitalWrite(dirPin, LOW);
  }

}
//---------------------------------------------------------------------------------------------------
void pulseVertical()
{
  for(int x=0; x<50; x++)
  {
    digitalWrite(pulsePin, HIGH); // move 50 pulses
    delay(1);
    digitalWrite(pulsePin, LOW);
    delay(1);
  }

}
//----------------------------------------------------------------------------------------------------
void toggleLight()
{
  tracker = !tracker;
  trackTracker = true;
}
//----------------------------------------------------------------------------------------------------
//Checks to see if the rotational motion needs to be braked
void rotationalBrake()
{
  if( (xPos < 60) && (xPos > -60) )
  {
      digitalWrite(enblPinRot, HIGH);
      rotBrake = true;
  }
  else
  {
      digitalWrite(enblPinRot, LOW);
      rotBrake = false;
  }

  //****Need to add conditions for when rotational limit switches are tripped**
}
//---------------------------------------------------------------------------------------------------
void setRotationalDirection()
{
  //Set direction of rotation based off of user input
  if(xPos < 512)
  {
     digitalWrite(dirPinRot, HIGH);
  }
  else
  {
     digitalWrite(dirPinRot, LOW);
  }
}
//-----------------------------------------------------------------------------------------------------
void pulseHorizontal()
{
  for(int x=0; x<50; x++)
  {
     digitalWrite(pulsePinRot, HIGH); 
     delay(1);
     digitalWrite(pulsePinRot, LOW);
     delay(1);
  }
}



