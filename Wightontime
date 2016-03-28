/*    
 Arduino nano and ESP8266 checking a parsed version of the BBC shipping forecast 
 then displaying it on an LCD.
 Also gets next high tide time
 tomd@v3power.co.uk December 2015
   
*/    
    #include <SoftwareSerial.h>
    #include <LiquidCrystal.h>
    #define DEBUG true
    #define SERIAL_BUFFER_SIZE 200
    #define RESET 19 // esp RST==A5
    SoftwareSerial esp8266(14,15);  //espRX==A0 espTX==A1
    LiquidCrystal lcd(12, 11, 5, 4, 3, 2);
    unsigned long sensorPreviousTime = 0; //used for reset timing
    unsigned long internetPreviousTime = 0;
    int sensorTimeout=60000;
    int internetTimeout=60000;
    int messageLength = 0;
    boolean flag=true;
    boolean tide=false;
    String APIOne;

void setup()
    
   {

    //Some thingspeak http api's to display on our screen
    
    APIOne+="EOW9CJ9QB2VO1V61";
 
    Serial.begin(9600);
    esp8266.begin(9600); 
    lcd.begin(16, 1);  //LCD with 16x1 pixels
    lcd.print("Wight on time");
    delay(2000);
    pinMode(RESET,OUTPUT);
   // hardwareReset();
    lcd.setCursor(0,0);
    lcd.print("Booting      ");
    Serial.println("connecting");
    setupwifi();
    connectWiFi();
    lcd.clear();
    lcd.print("Connected!");
    Serial.println("connected");
    //lcd.setCursor(0,0);
    //lcd.print("Connected!");
    sensorPreviousTime = millis(); //Timer since last connected to sensor
    internetPreviousTime = millis(); //Timer since last connected to internet
   }
     
    void loop()
{  
 checkSensors();
  checkInternet();


              getData(APIOne); 
              delay(200);

}    //end of void loop!
          


String sendData(String command, const int timeout, boolean debug)
{
 String response = "";
 esp8266.print(command); // send the read character to the esp8266
 long int time = millis();
 while ((time+timeout) > millis())
          
       {
         if (esp8266.find("+IPD"))
            {
              //delay(10);
             esp8266.find("WIGHT"); // advance cursor to "Wight"
             esp8266.find("WIGHT"); // advance to next instance of Wight - incase there is a gale warning at the start of the message
             delay(20);
             while (esp8266.available())   // The esp has data so display its output to the serial window
                   {                        
                    char c = esp8266.read(); // read the next character.
                   if (c=='\n')
                       {
                        char c = esp8266.read();
                        if (c=='\n')
                           {
                            break;
                           }
                    response+=c;
                       }
                    else
                    {
                     response+=c;              
                    }
                   
                    }
                if (debug)
       {
         
        Serial.print(response);

     }
       String firstSixtyfour=response.substring(0,63);
       String secondSixtyfour=response.substring(63,127);       
       String thirdSixtyfour=response.substring(127,191);   
              // Serial.print(response);
               lcd.clear();
                
              
                 lcd.print(firstSixtyfour);              
                 delay(1000);
                   
                   for (int positionCounter = 0; positionCounter <= (63-16); positionCounter++) 
                   {
                     // scroll one position right:
                     lcd.scrollDisplayLeft();
                     delay(300);
                   }              
                 lcd.print(secondSixtyfour);
                 for (int positionCounter = 0; positionCounter <= (63-16); positionCounter++) 
                   {
                     // scroll one position right:
                     lcd.scrollDisplayLeft();
                     delay(300);
                   }     
                 lcd.print(thirdSixtyfour);
                 for (int positionCounter = 0; positionCounter <= (response.length()-113); positionCounter++) 
                   {
                     // scroll one position right:
                     lcd.scrollDisplayLeft();
                     delay(300);
                   }     
                   

           
                 //sensorPreviousTime=millis();
                 //internetPreviousTime=millis();
           }
           return response;
          }
    
}
int getData(String amp)
{
  sendData("AT+CIPMUX=0\r\n",1000,DEBUG);
  String cipStatus= sendData("AT+CIPSTART=\"TCP\",\"api.thingspeak.com\",80\r\n",1000,tide);
  Serial.println(cipStatus);
  char charBuf1[100];
  cipStatus.toCharArray(charBuf1, 50);
  String OKStatus;
         OKStatus += charBuf1[39];
     if (1==1)
        {
         delay(50);

         String cmd="GET /apps/thinghttp/send_request?api_key=";
         cmd+=amp;
         cmd+=" \r\n\r\n\r\n";
sendData("AT+CIPSEND=68\r\n",2000,DEBUG);
         
delay(100);
esp8266.print(cmd);
delay(20);
                            
     sendData("AT+CIPCLOSE\r\n",1000,DEBUG);
     delay(100);
     sendData("AT+CIPMUX=1\r\n",1000,DEBUG);
}
}

void setupwifi()
{
  Serial.println("setting up wifi");
  lcd.clear();
  lcd.print("Connecting");
  
   //setting up the esp using AT commands
    sendData("AT+RST\r\n",2000,DEBUG); // reset module
    sendData("AT+CWMODE=1\r\n",1000,DEBUG); // configure as client and server
    Serial.println("halfwaythru");
    sendData("AT+CIFSR\r\n",3000,DEBUG); // get ip address(es)
internetPreviousTime = millis(); //reset (the) reset timer!

 
}

void hardwareReset()
{
  digitalWrite(RESET,LOW);

  Serial.println("ESP Hardware reset");
  
  delay(100);
  digitalWrite(RESET,HIGH);
  delay(500);

}

boolean connectWiFi()
{
     
String connectStatus= sendData("AT+CWJAP=\"GavinAndSarah\",\"Cheltenham\"\r\n",5000,false);
//Serial.println(connectStatus);
char charBuf[50];
connectStatus.toCharArray(charBuf, 50);
String OKStatus;
       OKStatus += charBuf[39];
       
if (OKStatus=="O")
   {
     
         Serial.println("Woohoo - we're connected to the WiFi.");
         return true;
      
   }
else
{
Serial.println("connecting to the WiFi.");
return false;
}
}

void checkSensors()
{
if ((millis() - sensorPreviousTime) >= 60000)
       {
        Serial.println("a minute has passed with no received signal - lets reset!");
        lcd.setCursor(0,0);
        lcd.print("No sensor data");
        sensorPreviousTime = millis();
      //  hardwareReset();
        delay(1000);
        setupwifi();
        connectWiFi();
        
       }
}

void checkInternet()
{
  if ((millis() - internetPreviousTime) >=90000)
        {
          Serial.println("InternetTimer");
        //  hardwareReset();
          setupwifi();
          connectWiFi();
          internetPreviousTime = millis();
          Serial.println("Internet Timer Reset");
        }
}

