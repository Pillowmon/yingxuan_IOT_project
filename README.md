# DIY counter using ESP32-C3-DevKitM-1 & Google Firebase
### A project by ying xuan (1006960)

<img align="right" src="https://github.com/Pillowmon/yingxuan_IOT_project/assets/160840085/da82462a-1d83-424f-999f-cf353e0ef945" width="840" height="650">




| Components Required |
| ------       |
| ESP32-C3-DevKitM-1 x1|
| Breadboard x1 |
|5161AS Seven Segment LED display x1|
|1k ohm resistor x1|
|Button x1|
|Male to Male Jumper Wire x15|

# Story
## Why Counter?

### General story/background
Counters can be more frequently used in our daily lives than one may notice. From bigger scale usage like stock taking at a warehouse and head counting at events, counters frequently employed in excercise and fitness such as counting number of rope skips done or flight of stairs climbed. Recentlty, there has also been an uprise of the use of "wooden fish" as a form of meditation. Digital versions have also developed to aid the process and deduce the accumulated knocks on the "wooden fish". 

![small_2022111634629384](https://github.com/Pillowmon/yingxuan_IOT_project/assets/160840085/67624975-296d-4d76-8bba-f5a61c3c362f)





### My personal story/problem
I have recently picked up crocheting as a hobby, there is a need to track the number of stitches as I go along, but it is rather common that I lose track of counting or miscount, resulting in incorrect product 😢.Digital counters are certainly an option, but they can cause more distractions.

PLUS! Why spend money on an analog counter when I have the chance to make my own personal 101% original counter in this project! 🔥

<img src="https://github.com/Pillowmon/yingxuan_IOT_project/assets/160840085/b53d56c8-e40e-4976-a7bd-4966a9833198" width="360" height="360">



# Set up:
## library required
- Arduino
- WiFi
- ESP8266WiFi
- Firebase_ESP_Client
- SevSeg

## Circuit 
![image](https://github.com/Pillowmon/yingxuan_IOT_project/assets/160840085/8fd2df76-29ba-4b98-956c-a1fa2bb9ca80)

-Drawn with Wokwi, with reference to https://www.circuitbasics.com/arduino-7-segment-display-tutorial/

## Setting up firebase
Some pointers to take note of:
![image](https://github.com/Pillowmon/yingxuan_IOT_project/assets/160840085/de4eb104-330a-4bd9-8f0c-7947823b96ac)
![image](https://github.com/Pillowmon/yingxuan_IOT_project/assets/160840085/69b13bce-ea68-4020-aa30-82375a064152)

## Code
Take note to edit and include network credentials(WIFI_SSID & WIFI_PASSWORD), API_KEY and DATABASE_URL for it to work

``` C
#include "SevSeg.h"
SevSeg sevseg;

const int  buttonPin = 10;    // the pin that the pushbutton is attached to
int buttonState = 0;          // current state of the button
int lastButtonState = LOW;    // previous state of the button
int buttonPushCounter = 0;    // counter for the number of button presses
long counter = 0;
long max_long_val = 2147483647L;


#include <Arduino.h>
#if defined(ESP32)
  #include <WiFi.h>
#elif defined(ESP8266)
  #include <ESP8266WiFi.h>
#endif
#include <Firebase_ESP_Client.h>

//Provide the token generation process info.
#include "addons/TokenHelper.h"
//Provide the RTDB payload printing info and other helper functions.
#include "addons/RTDBHelper.h"

// Insert your network credentials
#define WIFI_SSID "INSERT_PERSONAL_WIFI_SSID"
#define WIFI_PASSWORD "INSERT_PERSONAL_WIFI_PASSWORD"

// Insert Firebase project API Key
#define API_KEY "INSERT_PERSONAL_API_KEY"

// Insert RTDB URLefine the RTDB URL */
#define DATABASE_URL "INSER_PERSONAL_DATABASE_URL" 

//Define Firebase Data object
FirebaseData fbdo;

FirebaseAuth auth;
FirebaseConfig config;

unsigned long sendDataPrevMillis = 0;
int count = 0;
bool signupOK = false;

void setup(){

  {
  byte numDigits = 1;
  byte digitPins[] = {};
  byte segmentPins[] = {6, 5, 2 , 3, 4 , 7, 8, 9};
  bool resistorsOnSegments = true; 
  int count = 0;
  

  byte hardwareConfig = COMMON_CATHODE;
  sevseg.begin(hardwareConfig, numDigits, digitPins, segmentPins, resistorsOnSegments);
  sevseg.setBrightness(90);

  pinMode(buttonPin, INPUT_PULLUP);
  Serial.begin(115200);
  lastButtonState = LOW;
}

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to Wi-Fi");
  while (WiFi.status() != WL_CONNECTED){
    Serial.print(".");
    delay(300);
  }
  Serial.println();
  Serial.print("Connected with IP: ");
  Serial.println(WiFi.localIP());
  Serial.println();

  /* Assign the api key (required) */
  config.api_key = API_KEY;

  /* Assign the RTDB URL (required) */
  config.database_url = DATABASE_URL;

  /* Sign up */
  if (Firebase.signUp(&config, &auth, "", "")){
    Serial.println("ok");
    signupOK = true;
  }
  else{
    Serial.printf("%s\n", config.signer.signupError.message.c_str());
  }

  /* Assign the callback function for the long running token generation task */
  config.token_status_callback = tokenStatusCallback; //see addons/TokenHelper.h
  
  Firebase.begin(&config, &auth);
  Firebase.reconnectWiFi(true);
  sevseg.setNumber(count);
    sevseg.refreshDisplay();
  (Firebase.RTDB.setInt(&fbdo, "Count", count));
}

void loop(){
  buttonState = digitalRead(buttonPin);
  (Firebase.RTDB.setInt(&fbdo, "Button state", buttonState));

  {
  int display;
  Serial.println(count);
  
if (buttonState == HIGH && lastButtonState==LOW){
    Serial.println("button was clicked");
    count++;
    display=count % 10;
    sevseg.setNumber(display,display%2);
    sevseg.refreshDisplay();
    (Firebase.RTDB.setInt(&fbdo, "Count", count));}
  }
  lastButtonState = buttonState;
}

```

## How it works:
When clicked, display shows numbers looping from 0-9. Press and hold has also been accounted for and will only be counted once.
![VID1 GIF](https://github.com/Pillowmon/yingxuan_IOT_project/assets/160840085/417f1ad3-f6cf-4e01-81cd-cda83d3dc9bf)

While the physical display only shows 0-9, these are actually only the ones digit. We can see the entire value in the serial monitor. When button has been pressed, it is also shown there.
![VID_2](https://github.com/Pillowmon/yingxuan_IOT_project/assets/160840085/182e4411-c5e1-4577-8fdf-d24cc677af95)

Last but not least, the realtime database! There we can see the total accumulated clicks. Button state indicates whether button was clicked, this also updates in real time.
![vid3 gif](https://github.com/Pillowmon/yingxuan_IOT_project/assets/160840085/0f3456da-3c46-4350-941a-5fdf2bb60c1e)



## Reference 🙏🙏🙏🙏
- https://randomnerdtutorials.com/esp32-firebase-realtime-database/
- https://www.circuitbasics.com/arduino-7-segment-display-tutorial/
- other resouces provided by dear prof ❤️

