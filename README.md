/*
  Make sure your Firebase project's '.read' and '.write' rules are set to 'true'. 
  Ignoring this will prevent the MCU from communicating with the database. 
  For more details- https://github.com/Rupakpoddar/ESP32Firebase 
*/

#include <ArduinoJson.h>            // https://github.com/bblanchon/ArduinoJson 
#include <ESP32Firebase.h>

#define _SSID "Your WiFi SSID "          // Your WiFi SSID 
#define _PASSWORD "WiFi Password"      // Your WiFi Password 
#define REFERENCE_URL "Your Firebase project reference url "  // Your Firebase project reference url 

//define sound speed in cm/uS
#define SOUND_SPEED 0.034
#define CM_TO_INCH 0.393701
Firebase firebase(REFERENCE_URL);
const int trigPin = 5;
const int echoPin = 18;


long duration;
float distanceCm;
float distanceInch;


void setup() {
  Serial.begin(115200);
  // pinMode(LED_BUILTIN, OUTPUT);
  // digitalWrite(LED_BUILTIN, LOW);
  pinMode(trigPin, OUTPUT); // Sets the trigPin as an Output
  pinMode(echoPin, INPUT); // Sets the echoPin as an Input
  WiFi.mode(WIFI_STA);
  WiFi.disconnect();
  delay(1000);

  // Connect to WiFi
  Serial.println();
  Serial.println();
  Serial.print("Connecting to: ");
  WiFi.begin(_SSID,_PASSWORD );

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print("-");
  }

  Serial.println("");
  Serial.println("WiFi Connected");

  // Print the IP address
  Serial.print("IP Address: ");
  Serial.print("http://");
  Serial.print(WiFi.localIP());
  Serial.println("/");
  // digitalWrite(LED_BUILTIN, HIGH);

//================================================================//
//================================================================//

  // Write some data to the realtime database.
  firebase.setString("Example/setString", "It's Working");
  firebase.setInt("Example/setInt", 123);
  firebase.setFloat("Example/setFloat", 45.32);

  firebase.json(true);              // Make sure to add this line.
  
  String data = firebase.getString("Example");  // Get data from the database.

  // Deserialize the data.
  // Consider using Arduino Json Assistant- https://arduinojson.org/v6/assistant/
  const size_t capacity = JSON_OBJECT_SIZE(3) + 50;
  DynamicJsonDocument doc(capacity);

  deserializeJson(doc, data);

  // Store the deserialized data.
  const char* received_String = doc["setString"]; // "It's Working"
  int received_int = doc["setInt"];               // 123
  float received_float = doc["setFloat"];         // 45.32

  // Print data
  Serial.print("Received String:\t");
  Serial.println(received_String);

  Serial.print("Received Int:\t\t");
  Serial.println(received_int);

  Serial.print("Received Float:\t\t");
  Serial.println(received_float);

  // Delete data from the realtime database.
  firebase.deleteData("Example");
}

void loop() {
  // Nothing
    // Clears the trigPin
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  // Sets the trigPin on HIGH state for 10 micro seconds
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  // Reads the echoPin, returns the sound wave travel time in microseconds
  duration = pulseIn(echoPin, HIGH);
  
  // Calculate the distance
  distanceCm = duration * SOUND_SPEED/2;
  
  // Convert to inches
  distanceInch = distanceCm * CM_TO_INCH;
  
  // Prints the distance in the Serial Monitor
  Serial.print("Distance (cm): ");
  Serial.println(distanceCm);
  Serial.print("Distance (inch): ");
  Serial.println(distanceInch); 
  firebase.setString("Example/setString", "distanceCm");
  firebase.setInt("Example/setInt", distanceCm);
  firebase.setFloat("Example/setFloat", distanceCm);
  
  delay(1000);
}
