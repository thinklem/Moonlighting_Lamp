/*
Moonlighting Lamp
A glorified lamp that changes brightness based on proximity not motion
By Nicholas LaJeunesse
*/

// defines 4 pins for 4 LEDs
const int LED1 = 11;
const int LED2 = 10;
const int LED3 = 9;
const int LED4 = 6;
// defines pin for distance sensor power
const int distSensorPower = 13;

boolean state = 0;          // distance sensor off is 0, on is 1
int lightSensor = 0;        // value to store light sensor reading
int brightness = 0;         // brightness of LEDs
int brightnessWant = 0;     // desired brightness of LEDs
int fadeAmount = 5;         // amount to fade LEDs by
int distSensor = 0;         // value to store distance sensor reading
int i;

void setup()  { 
  // set LED pins to OUTPUT
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
  pinMode(LED3, OUTPUT);
  pinMode(LED4, OUTPUT);
  // set distance sensor power pin to OUTPUT
  pinMode(distSensorPower, OUTPUT);
  
  Serial.begin(9600);
} 

void loop()  {    
  // read light sensor if LEDs not active
  if (brightness == 0)
    lightSensor = analogRead(1);
  
  // enable distance sensor only if day
  if (lightSensor > 400) {
    digitalWrite(distSensorPower, LOW);
    state = 0;
  }
  else if (lightSensor <= 400) {  
    digitalWrite(distSensorPower, HIGH);
    state = 1; 
  }
  
  if (state == 1) {
    // take average of distance sensor readings
    for (i=0; i<8; i++) {
      distSensor += analogRead(0);
      delay(5);
    }
    distSensor /= 8;
    
    // map desired brightness to distance sensor reading
    brightnessWant = map(distSensor, 15, 125, 0, 255);
    if (brightnessWant < 0)
      brightnessWant = 0;
    if (brightnessWant > 255)
      brightnessWant = 255;
    
    // set desired brightness inversly proportional to distance from sensor  
    brightnessWant = 255 - brightnessWant;
    
    // set brightness to fade in or out based on desired brightness  
    if (brightness < brightnessWant) {
      brightness = brightness + fadeAmount;
      if (brightness > 255)
        brightness = 255;
    }
    else if (brightness > brightnessWant) {
      brightness = brightness - fadeAmount;
      if (brightness < 0)
        brightness = 0;
    }
    else
      brightness = brightnessWant;
  }
  else if (state == 0) {
    brightness = 0;
  }
  
  // write brightness to LEDs
  analogWrite(LED1, brightness);    
  analogWrite(LED2, brightness);  
  analogWrite(LED3, brightness);  
  analogWrite(LED4, brightness); 
    
  Serial.print("Sensor = ");
  Serial.println(distSensor);
}
