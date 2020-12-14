# Aulia-Octaviani
Rancangan Bangun Alat Ukur Elastisitas (Modulus Young) pada Kawat Tembaga
#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27, 16, 2);

#include "HX711.h"
HX711 scale(5, 4);

//set debounce
#define tombolStart 2

int buttonState_vUP;             
int lastButtonState_vUP = LOW;   
int buttonStateBot_vUP;             
int lastButtonStateBot_vUP = LOW;
unsigned long lastDebounceTime = 0; 
//end debounce



#include <Stepper.h>
const int stepsPerRevolution = 200;
Stepper myStepper(stepsPerRevolution, 8, 9, 10, 11);

float calibration_factor = 2230; // this calibration factor is adjusted according to my load cell
float units;
float ounces;
float jarak;

void setup() {
  Serial.begin(9600);
  lcd.begin();
  lcd.setCursor(0,0);
  lcd.print("");
  lcd.setCursor(0,1);
  lcd.print("");

  pinMode(12, INPUT);
  pinMode(3, INPUT);
  pinMode(2, INPUT);

  myStepper.setSpeed(10);

  scale.set_scale();
  scale.tare();  //Reset the scale to 0

  long zero_factor = scale.read_average(); //Get a baseline reading
  Serial.print("Zero factor: "); //This can be used to remove the need to tare the scale. Useful in permanent scale projects.
  Serial.println(zero_factor);
  jarak = 0.00;
}

void loop() {
  scale.set_scale(calibration_factor); //Adjust to this calibration factor

  Serial.print("Reading: ");
  units = scale.get_units(), 10;
  if (units < 0)
  {
    units = 0.00;
  }
  ounces = units * 0.035274;
  Serial.print(units);
  Serial.print(" grams"); 
  Serial.print(" calibration_factor: ");
  Serial.print(calibration_factor);
  Serial.println();

  if(Serial.available())
  {
    char temp = Serial.read();
    if(temp == '+' || temp == 'a')
      calibration_factor += 1;
    else if(temp == '-' || temp == 'z')
      calibration_factor -= 1;
  }

  int baca = digitalRead(12);
  int baca2 = digitalRead(3);
  if (baca == HIGH){
    myStepper.step(2);
    jarak = jarak +0.29;
  } else if (baca2 ==HIGH){
    myStepper.step(-2);
    jarak = jarak-0.29;
  }
  
  lcd.setCursor(0,0);
  lcd.print("Jarak:");

  lcd.setCursor(0,1);
  lcd.print("Gaya :"); 
  lcd.setCursor(8,0);
  lcd.print("     mm");
  lcd.setCursor(11,1);
  lcd.print("  N");

  lcd.setCursor(7,0);
  lcd.print(jarak);
  lcd.setCursor(7,1);
  lcd.print(units*0.1);
  button2();
}

void button2(){
          int reading_vUP = digitalRead(tombolStart);
     if (reading_vUP != 0) {
       lastDebounceTime = millis();
     }
  
    if ((millis() - 0) > 50) {
      if (reading_vUP != buttonState_vUP) {
         buttonState_vUP = reading_vUP;
         if (buttonState_vUP == HIGH) {
          jarak = 0;
          }
        }
    }
     lastButtonState_vUP = reading_vUP;
}
