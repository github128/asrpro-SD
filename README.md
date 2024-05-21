搬用 arduino SD卡程序 使用softspi模拟实现  修改send recv begin 


将 SD2 文件夹放入 D:\天问Block\asrpro\myLib下 
是滞添加 asr_pro_sdk\projects\offline_asr_sample\project_file\source_file.prj 好像可以 不添加
若添加 
source-file: ../myLib/SD2/Sd2Card.cpp
source-file: ../myLib/SD2/SdFile.cpp
source-file: ../myLib/SD2/SdVolume.cpp

主程序代码如下 


#include "asr.h"
extern "C"{ void * __dso_handle = 0 ;}
#include "setup.h"
//#include "SD2Card.h"
#include "../myLib/SD2/SD.h"
#include "HardwareSerial.h"

uint32_t snid;
void ASR_CODE();
Sd2Card card;
SdVolume volume;
SdFile root;

//{speak:小蝶-清新女声,vol:10,speed:10,platform:haohaodada}
//{playid:10001,voice:欢迎使用语音助手，用天问五幺唤醒我。}
//{playid:10002,voice:我退下了，用天问五幺唤醒我}

/*描述该功能...
*/
void ASR_CODE(){
  //本函数是语音识别成功钩子程序
  //运行时间越短越好，复杂控制启动新线程运行
  //唤醒时间设置必须在ASR_CODE中才有效
  set_state_enter_wakeup(10000);
  //用switch分支选择，根据不同的识别成功的ID执行相应动作，点击switch左上齿轮
  //可以增加分支项
  switch (snid) {
   case 1:
    digitalWrite(4,0);
    break;
   case 2:
    digitalWrite(4,1);
    break;
  }
  //除了switch分支选择，也可用如果判断识别ID的值来执行动作
  if((snid) == 3){
    digitalWrite(4,0);
  }
  if((snid) == 4){
    digitalWrite(4,1);
  }
  //采用如果判断模块，可更方便复制

}

int chipSelect = 2;

void hardware_init(){
  //需要操作系统启动后初始化的内容
  //音量范围1-7
  vol_set(1);
  Serial.begin(9600);
  delay(500);

  Serial.print("\nInitializing SD card...");

  // we'll use the initialization code from the utility libraries
  // since we're just testing if the card is working!
  if (!card.init(SPI_HALF_SPEED, chipSelect)) {
    Serial.println("initialization failed. Things to check:");
    Serial.println("* is a card inserted?");
    Serial.println("* is your wiring correct?");
    Serial.println("* did you change the chipSelect pin to match your shield or module?");
    while (1);
  } else {
    Serial.println("Wiring is correct and a card is present.");
  }

  // print the type of card
  Serial.println();
  Serial.print("Card type:         ");
  switch (card.type()) {
    case SD_CARD_TYPE_SD1:
      Serial.println("SD1");
      break;
    case SD_CARD_TYPE_SD2:
      Serial.println("SD2");
      break;
    case SD_CARD_TYPE_SDHC:
      Serial.println("SDHC");
      break;
    default:
      Serial.println("Unknown");
  }
  
  //  Now we will try to open the 'volume'/'partition' - it should be FAT16 or FAT32
  if (!volume.init(card)) {
    Serial.println("Could not find FAT16/FAT32 partition.\nMake sure you've formatted the card");
    while (1);
  }

  Serial.print("Clusters:          ");
  Serial.println(volume.clusterCount());
  Serial.print("Blocks x Cluster:  ");
  Serial.println(volume.blocksPerCluster());

  Serial.print("Total Blocks:      ");
  Serial.println(volume.blocksPerCluster() * volume.clusterCount());
  Serial.println();

  // print the type and size of the first FAT-type volume
  uint32_t volumesize;
  Serial.print("Volume type is:    FAT");
  Serial.println(volume.fatType(), DEC);

  volumesize = volume.blocksPerCluster();    // clusters are collections of blocks
  volumesize *= volume.clusterCount();       // we'll have a lot of clusters
  volumesize /= 2;                           // SD card blocks are always 512 bytes (2 blocks are 1KB)
  Serial.print("Volume size (Kb):  ");
  Serial.println(volumesize);
  Serial.print("Volume size (Mb):  ");
  volumesize /= 1024;
  Serial.println(volumesize);
  Serial.print("Volume size (Gb):  ");
  Serial.println((float)volumesize / 1024.0);

  Serial.println("\nFiles found on the card (name, date and size in bytes): ");
  root.openRoot(volume);

  // list all files in the card with date and size
  root.ls(LS_R | LS_DATE | LS_SIZE);
  
  
  vTaskDelete(NULL);
}

void setup()
{
  //需要操作系统启动前初始化的内容
  //播报音下拉菜单可以选择，合成音量是指TTS生成文件的音量
  //欢迎词指开机提示音，可以为空
  //退出语音是指休眠时提示音，可以为空
  //休眠后用唤醒词唤醒后才能执行命令，唤醒词最多5个。回复语可以空。ID范围为0-9999
  //{ID:0,keyword:"唤醒词",ASR:"天问五幺",ASRTO:"我在"}
  //{ID:1,keyword:"命令词",ASR:"打开灯光",ASRTO:"好的，马上打开灯光"}
  //{ID:2,keyword:"命令词",ASR:"关闭灯光",ASRTO:"好的，马上关闭灯光"}
  //{ID:3,keyword:"命令词",ASR:"灯光打开",ASRTO:"好的，马上打开灯光"}
  //{ID:4,keyword:"命令词",ASR:"灯光关闭",ASRTO:"好的，马上关闭灯光"}
  pinMode(2,output);  //cs
  setPinFun(2,0);
  digitalWrite(2,1);
  
  pinMode(3,output);  //mosi
  setPinFun(3,0);
  digitalWrite(3,1);
  
  pinMode(5,output);  //sck
  setPinFun(5,0);
  digitalWrite(5,1);
  
  pinMode(6,input);  //miso
  dpmu_set_io_pull(pinToFun[6],DPMU_IO_PULL_UP);
  
  
  
}




