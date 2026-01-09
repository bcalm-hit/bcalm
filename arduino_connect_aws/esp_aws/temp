#include <Arduino_BuiltIn.h>
#include "utils.h"
#include <PubSubClient.h>

const byte sensorPin = 34; 
// הגדרת פין החיישן 
volatile long pulseCount = 0;
//מונה את מספר הפולסים מהחיישן.
float flowRate = 0.0;  //משתנה לאחסון קצב הזרימה הנוכחי (בליטרים לדקה).
unsigned int flowMilliLitres = 0;
unsigned long totalMilliLitres = 0;
unsigned long oldTime = 0;   //שומר את נקודת הזמן האחרונה שבה ביצענו 

// פונקציית הפסיקה- כל פעם שנשלח אות חשמלי מקדם מונה ב1
void IRAM_ATTR pulseCounter() {
  pulseCount++;
}

void setup() {
    Serial.begin(115200);  //פתיחת תקשורת טורית במהירות גבוהה
    
  
    pinMode(sensorPin, INPUT);  //מגדיר את הפין כפין קלט כלומר תצפה לקבל אות חשמלי מפין זה

    attachInterrupt(digitalPinToInterrupt(sensorPin), pulseCounter, RISING);
    //תקשיב לפין 34 ומתי שאתה מזהה קפיצה תריץ את הפונקציה שרמשנו למעלה שמעלה את המונה ב 1
    connectAWS(); //מפעיל את הפונקציה שנמצאת בתייקית utlish עוזר לחבר לוויפי
    oldTime = millis();  //מעדכן את נקודת הזמן לחישוב הבא
}

void loop() {
  if ((millis() - oldTime) > 1000) {    //מכריח את הקוד לחשב בקצב של לפחות שנייה-1000 מילי שניות
    
    detachInterrupt(digitalPinToInterrupt(sensorPin)); //עצירה זמנית לצורך חישוב ועל מנת שלא נקבל פולס באמצע
    
    // חישוב קצב זרימה (לפי היחס של 14.5 שסיפקת)
    flowRate = ((1000.0 / (millis() - oldTime)) * pulseCount) / 14.5;    
    //רוב המכישירם מוציאים 14.5 פוליסים על כל ליטר
    //  תחילה מנרמלים את הזמן כלומר אם לקח 1.1 שניות הופכים לשנייה אחת

    oldTime = millis();
    
    flowMilliLitres = (flowRate / 60) * 1000;   // הופך קצב לדקה לכמות מיליליטרים שעברה בשנייה הבודדת הזו
    totalMilliLitres += flowMilliLitres;       //מעדכן את הסכום למונה הכללי

    
    // הדפסה למסך הסריאלי
    Serial.printf("Flow rate: %.2f L/min | Total: %lu mL\n", flowRate, totalMilliLitres);

    // שליחת הנתונים האמיתיים ל-AWS
    publishMessage(flowRate, totalMilliLitres);

    // איפוס מונה והחזרת האינטראפט
    pulseCount = 0;
    attachInterrupt(digitalPinToInterrupt(sensorPin), pulseCounter, RISING);
  }

  client.loop(); // מעדכן את השרת שהקוד עדיין רץ ושלא יסגר החיבור לשרת
}
