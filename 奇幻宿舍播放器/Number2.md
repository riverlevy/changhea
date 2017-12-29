#include <IRremote.h>
int RECV_PIN = 6;//红外接收引脚，
IRrecv irrecv(RECV_PIN);
decode_results results;//构造接受的红外码放在results里
#define music_num_MAX 49 //我们TF卡里有49首歌
#include <SoftwareSerial.h>
#include "audio.h"   //"audio.h"是控制音频文件
#include "U8glib.h"
//-------字体设置，大、中、小
#define setFont_L u8g.setFont(u8g_font_7x13)
#define setFont_M u8g.setFont(u8g_font_fixed_v0r)
#define setFont_S u8g.setFont(u8g_font_fixed_v0r)
//屏幕类型--------
U8GLIB_SSD1306_128X64 u8g(U8G_I2C_OPT_NONE);
#define init_draw 500  //主界面刷新时间
int MENU_FONT = 1;  //初始化字体大小 0：小，1：中，2：大
boolean music_status = false; //歌曲播放状态
int music_num = 1;    //歌曲序号
int music_vol = 20;             //音量0~30
int inChar;
void setup()        //创建无返回值函数
{
  Serial.begin(9600);    //初始化串口通信，并将波特率设置为9600
 audio_init(DEVICE_TF, MODE_loopOne, music_vol);   //初始化mp3模块
  irrecv.enableIRIn(); // Start the receiver
}
void loop()            //无返回值Loop函数
{
 if (irrecv.decode(&results))
 {
  inChar=results.value;
  
  if (inChar==4335) 
    {
      Serial.println("play");   //串口输出 “play”（工作
      audio_play(); //音频工作
      music_status=true;
      inChar=0;//更新inChar的值。
    }
  else if (inChar==18615)                  
  {
    Serial.println("pause");   
      audio_pause();  
      music_status=false;
      inChar=0;   //更新inChar的值。
  }
  else if (inChar==-28561)         
  {
      music_num++; 
      if (music_num > music_num_MAX)
      {
        music_num = 1;
      }
      audio_choose(music_num);
      audio_play();
      music_status = true; 
      inChar=0;//更新inChar的值。
    }
    else if (inChar==-2041) 
    {
      music_status = true;       
      music_vol++;               
      if (music_vol > 30) music_vol = 30;
      audio_vol(music_vol);
      audio_play();
      inChar=0;   //更新inChar的值。
    }
  else if (inChar==-8161)     
  {
      music_num--; 
      if (music_num < 1) 
      {
        music_num = music_num_MAX; 
      }
      audio_choose(music_num);    
      audio_play();                
      music_status = true;  
      inChar=0;    //更新inChar的值。
    }
    else if (inChar==28815)  
    {
      music_status = true;         
      music_vol--;                 
      if (music_vol < 1) music_vol = 0; 
      audio_vol(music_vol);
      audio_play();
      inChar=0;      //更新inChar的值。    
    }
     else if(inChar==-20401)
    {
      music_vol=0;
      audio_vol(music_vol);
      audio_pause();
      music_status=true;
      inChar=0;//更新inChar的值。
      } 
      irrecv.resume();}
u8g.firstPage();
    do {
      draw();
    }
    while ( u8g.nextPage() );
}
//主界面，可自由定义
void draw()
{
  setFont_L;
  u8g.setPrintPos(4, 16);
  u8g.print("Music_sta:");
  u8g.print(music_status ? "play" : "pause");
  u8g.setPrintPos(4, 16 * 2);
  u8g.print("Music_vol:");
  u8g.print(music_vol);
  u8g.print("/30");
  u8g.setPrintPos(4, 16 * 3);
  u8g.print("Music_num:");
  u8g.print(music_num);
  u8g.print("/");
  u8g.print(music_num_MAX);
  u8g.setPrintPos(4, 16 * 4);
  u8g.print(".......318.......");
  //u8g.print(rtc.formatTime(RTCC_TIME_HMS));
}