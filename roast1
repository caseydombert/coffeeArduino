#include <PID_v1.h>
#define RelayPin 7
#include "max6675.h"

int thermoDO = 4;
int thermoCS = 5;
int thermoCLK = 6;

MAX6675 thermocouple(thermoCLK, thermoCS, thermoDO);

int thermo2DO = 8;
int thermo2CS = 9;
int thermo2CLK = 10;

MAX6675 thermocouple2(thermo2CLK, thermo2CS, thermo2DO);

const int fanPin = 3;
int fanSpeed = 205;

const double a = -2.1;
const double b = 50;
const double c = 180;
double setPointCalc;
double nowMinutesCalc;
double nowMillisCalc;
double calcOffset;

bool manual=true;

int nowMinutes, nowSeconds;
int val;

double Setpoint, Input, Output;
PID myPID(&Input, &Output, &Setpoint, 6, .75, 1, DIRECT, P_ON_M);

int WindowSize = 1000;
unsigned long windowStartTime;

void setup(){
  Serial.begin(9600);
  pinMode(fanPin,OUTPUT);
  pinMode(RelayPin, OUTPUT);

  windowStartTime = millis();

  Setpoint = 90;
  calcOffset = 0;

  myPID.SetOutputLimits(0, WindowSize);
  myPID.SetSampleTime(1000);
  myPID.SetMode(MANUAL);
  delay(1400);
}

void loop(){
  
  if (Serial.available()){
    val = Serial.read();
  }
  analogWrite(fanPin,fanSpeed);

  myPID.Compute();
  unsigned long now = millis();

  if (now - windowStartTime > WindowSize){
    windowStartTime += WindowSize;
    
    Input = thermocouple.readFarenheit();
    
    nowMinutes = now/60000;
    nowSeconds = now/1000-nowMinutes*60;
    
    nowMillisCalc=now;
    nowMinutesCalc=calcOffset+nowMillisCalc/60000;
    setPointCalc = a*nowMinutesCalc*nowMinutesCalc+b*nowMinutesCalc+c;
    Setpoint = setPointCalc;

    Serial.print(thermocouple2.readFarenheit(), 0);Serial.print(',');
    Serial.print(Input, 0);Serial.print(',');
    Serial.print(nowMinutes);
    Serial.print(':');
    if (nowSeconds <= 9){Serial.print('0');}
    Serial.print(nowSeconds);Serial.print(',');
    Serial.print(Setpoint, 0);Serial.print(',');
    Serial.print(Output, 0);Serial.print(',');
    Serial.print(fanSpeed); Serial.print(',');
    if (manual == true) {Serial.print('0');}
      else {Serial.print('1');}
    Serial.println();
  }
  if (Output > now - windowStartTime) digitalWrite(RelayPin, HIGH);
  else digitalWrite(RelayPin, LOW);

  if (val == 'm') {fanSpeed = 0;}
  if (val == '/') {fanSpeed = 200;}
  if (val ==  ',') {fanSpeed = max(fanSpeed - 5, 0);}
  if (val ==  '.') {fanSpeed = min(fanSpeed + 5, 250);}


  if (manual == true){
    if (val == '\'') {Output = 0;}
    if (val == '=') {Output = 1000;}
    if (val ==  '[') {Output = max(Output - 10, 0);} 
    if (val ==  ']') {Output = min(Output + 10, 1000);} 
  }

  if (val == 'h'){
    manual = true;
    myPID.SetMode(MANUAL);
  }

  if (val == 'j'){
    manual = false;
    myPID.SetMode(AUTOMATIC);
  }

  if (val == 'y') {calcOffset = calcOffset - .25;}
  if (val == 'u') {calcOffset = calcOffset + .25;}

  /*
  //  Serial.print(Output, 0);Serial.print(',');
  Serial.print(myPID.Compute());Serial.print(',');
    
  Serial.print(Input, 0);Serial.print(',');


  Serial.print(Setpoint, 0);Serial.print(',');

  Serial.print(Output, 0);Serial.print(',');
//  Serial.print(fanSpeed);
  
  Serial.print(',');
  Serial.print(myPID.GetMode());
  
  Serial.println();
  */
    val = '~';
}
