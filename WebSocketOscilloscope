/*
 * Based on project  "WebSocket осциллограф на базе ESP8266" by Sergei Belotserkovskii
 * https://www.youtube.com/watch?v=8BGtgbOYLUU
 * 
 * libraries:
 * WebSockets for Arduino (Server & Client) by Markus Sattler v.2.3.4.
 * Available at Arduino IDE Library Manager
 * or here https://github.com/Links2004/arduinoWebSockets/tree/master/src
 * PageBuilder v.1.4.2 
 * ESP8266WiFi at version 1.0
 * esp8266 v.2.7.4
 * 
 * height could be adjusted by replacing "1024" 
 * 
 * Added some functionalty 
 * by CHE77
 */







#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>
#include <WebSocketsServer.h>

char* ssidAP = "Oscilloscope";
char* ssid = "My Wi-Fi";// adjust
char* password = "MyWi-FiPassword";//adjust
String ADC;
uint8_t timeout = 0;

IPAddress apIP(192, 168, 4, 1);
ESP8266WebServer server(80);
WebSocketsServer webSocket = WebSocketsServer(81);


char webpage[] PROGMEM = R"=====(
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>WebSocket Oscilloscope</title>
<style>
body {background: #2e2e2e; color: #a0bbbb; font-family: Helvetica; }
::selection {background: #a0bbbb;}
::-moz-selection {background: #a0bbbb;}
input, textarea, button, select, select:focus {outline: none;}

#c1 {width: 512px; height: 1024px;  margin: 0 0 12px 12px; background: #2e2e2e; border: 1px solid #949494; border-radius: 4px;
  background-image: url(data:image/svg+xml;base64,ICA8c3ZnIHdpZHRoPSI1MTJweCIgaGVpZ2h0PSIyNTZweCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4gICAgPGRlZnM+ICAgICAgPHBhdHRlcm4gaWQ9ImdyaWQiIHdpZHRoPSI2NCIgaGVpZ2h0PSI2NCIgcGF0dGVyblVuaXRzPSJ1c2VyU3BhY2VPblVzZSI+ICAgICAgICA8cGF0aCBkPSJNIDY0IDAgTCAwIDAgMCA2NCIgZmlsbD0ibm9uZSIgc3Ryb2tlPSIjOTQ5NDk0IiBzdHJva2Utd2lkdGg9IjEiIHN0cm9rZS1kYXNoYXJyYXk9IjIiLz4gICAgICA8L3BhdHRlcm4+ICAgIDwvZGVmcz4gICAgPHJlY3Qgd2lkdGg9IjEwMCUiIGhlaWdodD0iMTAwJSIgZmlsbD0idXJsKCNncmlkKSIgLz4gIDwvc3ZnPg==);
}
#rx1, #rx2 {margin: 0 0 0 12px; font-size: 18px; font-weight:normal;}
#rx3 {margin: 0 0 0 12px; font-size: 26px; font-weight:normal;}
.btn{width: 82px; height: 40px; margin: -4px 0 -2px 0; background: #3e5050; color:#a0bbbb; border: 1px solid #949494; border-radius: 4px; outline: none; font-family: Helvetica; font-size: 18px;}
.btn:active {background: linear-gradient(#3e5050, #949494); border: 1px solid #FFF; color:#FFF;}

</style>
</head>
<body>
<body onload="init();">

<p><a id='rx1'></a></p>
<p><a id='rx2'></a></p>
<p><a id='rx3'></a></p>
<canvas id="c1" width="512" height="1024"></canvas>
<div>
<input type='submit' class='btn' value='Start' style='margin: 0 0 0 12px;' onclick="Socket.send('1'); next = true;">
<input type='submit' class='btn' value='Stop' onclick="Socket.send('2'); next = false;">
<input type='submit' class='btn' value='Reset' onclick="Socket.send('3'); ">
<input type='submit' class='btn' value='Adjust ^' onclick="dY = dY + 1;">
<input type='submit' class='btn' value='Adjust v' onclick="dY = dY - 1;">
</div>

<div>
V0: <input type="text" id="fieldV0" value=' 0' style='margin: 12px 0 0 12px;'>
R1: <input type="text" id="fieldR1" value=' 0'>
R2: <input type="text" id="fieldR2" value=' 0'>
</div>

<script type='text/javascript'>
rx1.innerHTML = 'WebSocket Oscilloscope';
rx2.innerHTML = 'data.length: 0000 | y: 0000 | dY: 0 | x: 0000';
var Socket;
var x = 0;
var m = 4;
var maxX = 512;
var next = true;
var data = [];
var ctx = c1.getContext('2d');
ctx.strokeStyle = "#FFF";
ctx.beginPath(); 
ctx.moveTo(0,128);
var voltage = 0;
baseVoltage = 3.3; // initial value
voltageDividerRatio = 11;
var r1 = 10000;// initial value, adjust
var r2 = 100000;// initial value, adjust
var maxVoltage = 0;
var dY = 0;// adjustment, initial value
document.getElementById("fieldV0").value = baseVoltage;
document.getElementById("fieldR1").value = r1;
document.getElementById("fieldR2").value = r2;


function init() {
  Socket = new WebSocket('ws://' + window.location.hostname + ':81/');
  Socket.onmessage = function(event){
   data.push(event.data);
   baseVoltage = document.getElementById("fieldV0").value; 
   r1 = document.getElementById("fieldR1").value; 
   r2 = document.getElementById("fieldR2").value;  
   voltageDividerRatio = (Number(r2) + Number(r1))/r1;
   voltageDividerRatio = voltageDividerRatio.toFixed(2);
   maxVoltage = baseVoltage * voltageDividerRatio;
   maxVoltage =maxVoltage.toFixed(2);
   voltage = (Number(event.data) + dY) /1024 * baseVoltage * voltageDividerRatio;
   voltage = voltage.toFixed(2);

   rx2.innerHTML = 'data.length: ' + data.length + '  | y: ' + event.data + '  | dY: ' + dY + '  | x: ' + x ;
   rx3.innerHTML = 'VoltDivRatio: ' + voltageDividerRatio + '  | MaxVoltage: ' + maxVoltage + ' | Voltage: ' + voltage ;
  
    if (next) {
    if (x++ < maxX) {
        ctx.lineTo(x*m,1024 - dY - data[x-1]);///
        ctx.stroke();///
    } else {
        ctx.beginPath();
        ctx.clearRect(0,0,512,1024);
        for (var i = 0; i < maxX; i++) {
            var y = 1024 - dY - data[x - maxX + i];///
            ctx.lineTo(i*m,y);
        }
        ctx.stroke();
    }       
    }
    if (data.length > 2047){
      data.splice(0,data.length-maxX);
      x = maxX;
      if (next) {Socket.send('1');}
    }
  }
}

</script>
</body>
</html>
)=====";

//================================================================================
void webSocketEvent(uint8_t num, WStype_t type, uint8_t * payload, size_t length){
if(type == WStype_TEXT){
  if(payload[0] == '1'){  //Start    
  for(uint16_t i=0; i<2048; i++){
    ADC = String(analogRead(A0));
    Serial.print("ADC = ");
    Serial.println(ADC);
    webSocket.broadcastTXT(ADC);  
    delayMicroseconds(24);   //1000ms/41100Hz = 24us
    } 
  }
  if(payload[0] == '2'){  //Stop
    //...
  }  
  if(payload[0] == '3'){  //Reset
    WiFi.disconnect();
    ESP.restart();  
  }  
}
}

//================================================================================
void setup()
{
  pinMode(2, OUTPUT);
  Serial.begin(115200);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid,password);
  while(WiFi.status() != WL_CONNECTED and timeout < 60) //timeout 60/0.5 = 30 sec
  {
    timeout++;
    delay(250);
    Serial.print(".");
    digitalWrite(2, LOW);
    delay(250);
    digitalWrite(2, HIGH);
  }
  if (timeout < 60) {
    Serial.println("");
    Serial.print("IP Address: ");
    Serial.println(WiFi.localIP());
  }
  else { 
    Serial.println("");
    Serial.println("WIFI_AP: 192.168.4.1");
    WiFi.disconnect();        
    WiFi.mode(WIFI_AP);                   //переключение модуля в режим точки доступа
    WiFi.softAPConfig(apIP, apIP, IPAddress(255, 255, 255, 0));
    WiFi.softAP(ssidAP);
  }

  server.on("/",[](){
    server.send_P(200, "text/html", webpage);  
  });
  server.begin();
  webSocket.begin();
  webSocket.onEvent(webSocketEvent);
}

//================================================================================
void loop()
{
  server.handleClient();
  webSocket.loop();
  delay(1);
}
