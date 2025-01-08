int Trig = 12;  //發出聲波腳位
int Echo = 14;  //接收聲波腳位
#include <ESP32Servo.h>
#include <WiFi.h>
#include <PubSubClient.h> //請先安裝PubSubClient程式庫
#include <SimpleDHT.h>
#define FORCE_SENSOR_PIN 36
char ssid[] = "Guaguagua";
char password[] = "00000000";
char* MQTTServer = "mqttgo.io";//免註冊MQTT伺服器
int MQTTPort = 1883;//MQTT Port
char* MQTTUser = "";//不須帳密
char* MQTTPassword = "";//不須帳密
//推播主題1:推播溫度(記得改Topic)
char* MQTTPubTopic1 = "jay0411/class205/force";
long MQTTLastPublishTime;//此變數用來記錄推播時間
long MQTTPublishInterval = 10000;//每10秒推撥一次
WiFiClient WifiClient;
PubSubClient MQTTClient(WifiClient);
Servo myServo;
void setup() {
  Serial.begin(115200);
  pinMode(Trig, OUTPUT);
  pinMode(Echo, INPUT);
  pinMode(15, OUTPUT);  //燈
  pinMode(2, OUTPUT);
  pinMode(4, OUTPUT);
  pinMode(16, OUTPUT);  //喇叭
  //SG90的脈衝寬度500~2400
  WifiConnecte();

  //開始MQTT連線
  MQTTConnecte();
  myServo.attach(4, 500, 2400);
}

void loop() {
  delayMicroseconds(5);
  digitalWrite(Trig, HIGH);  //啟動超音波
  delayMicroseconds(10);
  digitalWrite(Trig, LOW);               //關閉
  float EchoTime = pulseIn(Echo, HIGH);  //計算傳回時間
  float CMValue = EchoTime / 29.4 / 2;   //將時間轉換成距離
  int analogReading = analogRead(FORCE_SENSOR_PIN);  //讀取感測器訊號值
  
  if (CMValue > 100 && CMValue !=0) {
    Serial.println(CMValue);
    digitalWrite(15, HIGH);
    myServo.write(90);  //轉到0度（原點）
    Serial.print("Force sensor reading = ");
    Serial.print(analogReading);
    delay(1000);
    if (analogReading < 50) {  // from 0 to 9  沒有按壓
      Serial.println(" -> no water");
      digitalWrite(15, LOW);
      delay(1000);
    } else {  // from 800 to 1023  高強度
      Serial.println(" -> water");
      digitalWrite(15, LOW);
      digitalWrite(15, HIGH);  //播放一次歡迎詞
      delay(700);             //暫停一秒
      digitalWrite(15, LOW);   //關閉PLAYE，否則會一直播放
      delay(300);
    }
  } else if(CMValue <= 60 && CMValue !=0){
    Serial.println(CMValue);
    digitalWrite(15, LOW);
    myServo.write(0);
  }                                  //轉到90度
  //如果WiFi連線中斷，則重啟WiFi連線
  if (WiFi.status() != WL_CONNECTED) WifiConnecte(); 

  //如果MQTT連線中斷，則重啟MQTT連線
  if (!MQTTClient.connected())  MQTTConnecte(); 

  //如果距離上次傳輸已經超過10秒，則Publish溫溼度
  if ((millis() - MQTTLastPublishTime) >= MQTTPublishInterval ) {
    //讀取溫濕度
    // ------ 將DHT11溫度送到MQTT主題 ------
    MQTTClient.publish(MQTTPubTopic1, String(analogReading).c_str());
    Serial.println("漏水已推播到MQTT Broker");
    MQTTLastPublishTime = millis(); //更新最後傳輸時間
  }
  MQTTClient.loop();//更新訂閱狀態
  delay(50);
}
  //開始WiFi連線
void WifiConnecte() {
  //開始WiFi連線
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("WiFi連線成功");
  Serial.print("IP Address:");
  Serial.println(WiFi.localIP());
}
  //開始MQTT連線
void MQTTConnecte() {
  MQTTClient.setServer(MQTTServer, MQTTPort);
  MQTTClient.setCallback(MQTTCallback);
  while (!MQTTClient.connected()) {
    //以亂數為ClietID
    String  MQTTClientid = "esp32-" + String(random(1000000, 9999999));
    if (MQTTClient.connect(MQTTClientid.c_str(), MQTTUser, MQTTPassword)) {
      //連結成功，顯示「已連線」。
      Serial.println("MQTT已連線");
      //訂閱SubTopic1主題
    } else {
      //若連線不成功，則顯示錯誤訊息，並重新連線
      Serial.print("MQTT連線失敗,狀態碼=");
      Serial.println(MQTTClient.state());
      Serial.println("五秒後重新連線");
      delay(5000);
    }
  }
}
//接收到訂閱時
void MQTTCallback(char* topic, byte* payload, unsigned int length) {
  Serial.print(topic); Serial.print("訂閱通知:");
  String payloadString;//將接收的payload轉成字串
  //顯示訂閱內容
  for (int i = 0; i < length; i++) {
    payloadString = payloadString + (char)payload[i];
  }
  Serial.println(payloadString);
  //比對主題是否為訂閱主題1//stremp=字串比對
}
