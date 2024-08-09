#include <Adafruit_Arcada.h>
#include <CircularBuffer.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_LSM6DS33.h>
#include <Adafruit_LIS3MDL.h>
#include <Adafruit_SHT31.h>
#include <Adafruit_APDS9960.h>
#include <Adafruit_BMP280.h>
#include <PDM.h>
#include <Arcada_GifDecoder.h>
#include <Adafruit_AMG88xx.h> //8x8 thermal camera 

// Thermal camera settings ---------------------------------------
Adafruit_AMG88xx amg;
float px[AMG88xx_PIXEL_ARRAY_SIZE];
const uint16_t dpw = 240/8; //, dph = 120/8; // box width in pixels?
const byte dpw18 = dpw/1.8; // scale down by roughly 2x
byte MAXTEMP[6]={30, 25, 32, 40, 50, 80 };
byte MINTEMP[6]={25, 20, 15, 10, 0, 0 }; 
byte tempScale = 2; //0-4 for various fixed ranges; 5=auto ranging




Adafruit_Arcada arcada;
Adafruit_LSM6DS33 lsm6ds33;
Adafruit_LIS3MDL lis3mdl;
Adafruit_SHT31 sht30;
Adafruit_APDS9960 apds9960;
Adafruit_BMP280 bmp280;
extern PDMClass PDM;
GifDecoder<ARCADA_TFT_WIDTH, ARCADA_TFT_HEIGHT, 12> decoder;
File file;


#define WHITE_LED 43

// Color definitions
#define BACKGROUND_COLOR __builtin_bswap16(ARCADA_BLACK)
#define BORDER_COLOR __builtin_bswap16(ARCADA_BLUE)
#define PLOT_COLOR_1 __builtin_bswap16(ARCADA_PINK)
#define PLOT_COLOR_2 __builtin_bswap16(ARCADA_GREENYELLOW)
#define PLOT_COLOR_3 __builtin_bswap16(ARCADA_CYAN)
#define TITLE_COLOR __builtin_bswap16(ARCADA_WHITE)
#define TICKTEXT_COLOR __builtin_bswap16(ARCADA_WHITE)
#define TICKLINE_COLOR __builtin_bswap16(ARCADA_DARKGREY)


const uint16_t camColors[] = {
0x0011,0x0031,0x00B2,0x0112,0x0192,0x01F3,0x0273,0x02F3,
0x03F4,0x0514,0x0572,0x058F,0x05AD,0x05C9, 0x05E5,0x0621,
0x2E40,0x4E80,0x76A0,0x9EC0,0xCEE0,0xE660,0xE520,0xEBE0,
0xEB40,0xEAA0,0xF200,0xF140,0xF100,0xF0C0,0xF080,0xF040,0xF800};


// Buffers surrounding the plot area
#define PLOT_TOPBUFFER 20
#define PLOT_LEFTBUFFER 40
#define PLOT_BOTTOMBUFFER 20
#define PLOT_W (ARCADA_TFT_WIDTH - PLOT_LEFTBUFFER)
#define PLOT_H (ARCADA_TFT_HEIGHT - PLOT_BOTTOMBUFFER - PLOT_TOPBUFFER)

// millisecond delay between samples
#define DELAY_PER_SAMPLE 50
void plotBuffer(GFXcanvas16 *_canvas, const char *title, 
                CircularBuffer<float, PLOT_W> &buffer1, 
                CircularBuffer<float, PLOT_W> &buffer2, 
                CircularBuffer<float, PLOT_W> &buffer3);

// Buffer for our plot data
CircularBuffer<float, PLOT_W> data_buffer;
CircularBuffer<float, PLOT_W> data_buffer2;
CircularBuffer<float, PLOT_W> data_buffer3;

int8_t sensornum = 0;

void setup(void) {
  Serial.begin(115200);
  Serial.print("Hello! Arcada CLUE sensor plotter");
  //while (!Serial) yield();
  amg.begin();

  decoder.setScreenClearCallback(screenClearCallback);
  decoder.setUpdateScreenCallback(updateScreenCallback);
  decoder.setDrawPixelCallback(drawPixelCallback);
  decoder.setDrawLineCallback(drawLineCallback);
  decoder.setFileSeekCallback(fileSeekCallback);
  decoder.setFilePositionCallback(filePositionCallback);
  decoder.setFileReadCallback(fileReadCallback);
  decoder.setFileReadBlockCallback(fileReadBlockCallback);


  // Start TFT and fill black
  if (!arcada.arcadaBegin()) {
    Serial.print("Failed to begin");
    while (1) delay(10);
  }
  arcada.displayBegin();

  // Turn on backlight
  arcada.setBacklight(255);

  arcada.filesysBeginMSD();  
  arcada.filesysBegin();



  if (! arcada.createFrameBuffer(ARCADA_TFT_WIDTH, ARCADA_TFT_HEIGHT)) {
    Serial.print("Failed to allocate framebuffer");
    while (1);
  }
  if (!apds9960.begin() || !lsm6ds33.begin_I2C() || !lis3mdl.begin_I2C() || 
      !sht30.begin(0x44) || !bmp280.begin()) {
      Serial.println("Failed to find CLUE sensors!");
      arcada.haltBox("Failed to init CLUE sensors");
  }
  /********** Check MIC */
  PDM.onReceive(onPDMdata);
  if (!PDM.begin(1, 16000)) {
    Serial.println("**Failed to start PDM!");
  }

  data_buffer.clear();
  data_buffer2.clear();
  data_buffer3.clear();
  pinMode(WHITE_LED, OUTPUT);
  digitalWrite(WHITE_LED, LOW);

  file = arcada.openFileByIndex("/gifs", 0, O_READ, "GIF");
  arcada.display->dmaWait();
  arcada.display->endWrite();   // End transaction from any prior callback
  decoder.startDecoding();

}

uint32_t timestamp = 0;

void loop() {
  timestamp = millis();

  arcada.readButtons();
  uint8_t justPressed = arcada.justPressedButtons();
  uint8_t justReleased = arcada.justReleasedButtons();
  if (justReleased & ARCADA_BUTTONMASK_LEFT) {
    sensornum--;
    data_buffer.clear();
    data_buffer2.clear();
    data_buffer3.clear();
    digitalWrite(WHITE_LED, LOW);
    arcada.display->fillScreen(BACKGROUND_COLOR);
  }
  if (justReleased & ARCADA_BUTTONMASK_RIGHT) {
    sensornum++;
    data_buffer.clear();
    data_buffer2.clear();
    data_buffer3.clear();
    digitalWrite(WHITE_LED, LOW);
    arcada.display->fillScreen(BACKGROUND_COLOR);
  }

  if (sensornum == 0) {
    decoder.decodeFrame();
  } 
  else if (sensornum == 1) {
    float t = bmp280.readTemperature();
    data_buffer.push(t);
    Serial.printf("Temp: %f\n", t);
    plotBuffer(arcada.getCanvas(), "T(C)",
               data_buffer, data_buffer2, data_buffer3);
  } 
  else if (sensornum == 2) {
    float p = bmp280.readPressure();
    data_buffer.push(p);
    Serial.printf("Pressure: %f Pa\n", p);
    plotBuffer(arcada.getCanvas(), "P", 
               data_buffer, data_buffer2, data_buffer3);
  } 
  else if (sensornum == 3) {
    float h = sht30.readHumidity();
    data_buffer.push(h);
    Serial.printf("Humid: %f %\n", h);
    plotBuffer(arcada.getCanvas(),"RH",
               data_buffer, data_buffer2, data_buffer3);
  }
  else if (sensornum == 4) {
    uint16_t r, g, b, c;
    apds9960.enableColor(true);
    //wait for color data to be ready
    while(! apds9960.colorDataReady()) {
      delay(5);
    }
    apds9960.getColorData(&r, &g, &b, &c);
    data_buffer.push(c);
    Serial.printf("Light: %d\n", c);
    plotBuffer(arcada.getCanvas(),"I",
               data_buffer, data_buffer2, data_buffer3);
  }
  else if (sensornum == 5) {
    uint16_t r, g, b, c;
    digitalWrite(WHITE_LED, HIGH);
    apds9960.enableColor(true);
    //wait for color data to be ready
    while(! apds9960.colorDataReady()) {
      delay(5);
    }
    apds9960.getColorData(&r, &g, &b, &c);
    data_buffer.push(r);
    data_buffer2.push(g);
    data_buffer3.push(b);
    Serial.printf("Color: %d %d %d\n", r, g, b);
    plotBuffer(arcada.getCanvas(),"λ",
               data_buffer, data_buffer2, data_buffer3);
  }

   else if (sensornum == 6) {
    uint32_t pdm_vol = getPDMwave(256);
    data_buffer.push(pdm_vol);
    Serial.print("PDM volume: "); Serial.println(pdm_vol);
    plotBuffer(arcada.getCanvas(), "Phon",
               data_buffer, data_buffer2, data_buffer3);
  }  
  else if (sensornum == 7) {
    sensors_event_t accel;
    lsm6ds33.getEvent(&accel, NULL, NULL);
    float x = accel.acceleration.x;
    float y = accel.acceleration.y;
    float z = accel.acceleration.z;
    data_buffer.push(x);
    data_buffer2.push(y);
    data_buffer3.push(z);
    Serial.printf("Accel: %f %f %f\n", x, y, z);
    plotBuffer(arcada.getCanvas(), "a",
               data_buffer, data_buffer2, data_buffer3);
  }   
  else if (sensornum == 8) {
    sensors_event_t gyro;
    lsm6ds33.getEvent(NULL, &gyro, NULL);
    float x = gyro.gyro.x * SENSORS_RADS_TO_DPS;
    float y = gyro.gyro.y * SENSORS_RADS_TO_DPS;
    float z = gyro.gyro.z * SENSORS_RADS_TO_DPS;
    data_buffer.push(x);
    data_buffer2.push(y);
    data_buffer3.push(z);
    Serial.printf("Gyro: %f %f %f\n", x, y, z);
    plotBuffer(arcada.getCanvas(), "φ",
               data_buffer, data_buffer2, data_buffer3);
  } 
  else if (sensornum == 9) {
    sensors_event_t mag;
    lis3mdl.getEvent(&mag);
    float x = mag.magnetic.x;
    float y = mag.magnetic.y;
    float z = mag.magnetic.z;
    data_buffer.push(x);
    data_buffer2.push(y);
    data_buffer3.push(z);
    Serial.printf("Mag: %f %f %f\n", x, y, z);
    plotBuffer(arcada.getCanvas(), "B",
               data_buffer, data_buffer2, data_buffer3);
  } 
  else if (sensornum == 10) { // test arcada plotting functions

      amg.readPixels(px);
      if ((tempScale % 6)==5){ //auto range
        MINTEMP[5]=px[0];
        MAXTEMP[5]=px[0];
        for(int i=1; i<(AMG88xx_PIXEL_ARRAY_SIZE); i++){
          //find min and max temps within frame
          if (px[i]<MINTEMP[5]){MINTEMP[5]=px[i];}
          if (px[i]>MAXTEMP[5]){MAXTEMP[5]=px[i];}// keep at least a 1 deg diff
        }
      };
      
      for(int i=0; i<(AMG88xx_PIXEL_ARRAY_SIZE); i++){ // remove -8 ?
        byte io8=dpw * (i % 8); //x pos
        byte im8=dpw * (i / 8); //y pos

        int col1=-1; //use one pixel to the left...
        if ((i % 8)==0) {col1=1;} // unless we are in the first col, then use pixel to the right

        int pxTmp=myPxTmp(px[i]);
        arcada.display->fillRect(190-im8,io8, dpw18, dpw18, camColors[pxTmp]);
        pxTmp=myPxTmp((px[i]+px[i+col1])/2);
        arcada.display->fillRect(190-im8 + dpw18,io8 , dpw18, dpw18, camColors[pxTmp]);
        pxTmp=myPxTmp((px[i]+px[i+8])/2);
        arcada.display->fillRect(190-im8 ,io8+ dpw18, dpw18, dpw18, camColors[pxTmp]);
        pxTmp=myPxTmp((px[i]+px[i+col1]+px[i+8])/3);
        arcada.display->fillRect(190-im8 + dpw18,io8 + dpw18, dpw18, dpw18, camColors[pxTmp]);
      } 


  } 
  else {
    data_buffer.clear();
    sensornum = 0;
    return;
  }
  if ((sensornum > 0)&&(sensornum < 10)) {
    arcada.blitFrameBuffer(0, 0, false, true);
  }
  //Serial.printf("Drew in %d ms\n", millis()-timestamp);
}

/**********************************************************************************/


void plotBuffer(GFXcanvas16 *_canvas, const char *title, 
                CircularBuffer<float, PLOT_W> &buffer1, 
                CircularBuffer<float, PLOT_W> &buffer2, 
                CircularBuffer<float, PLOT_W> &buffer3) {
  _canvas->fillScreen(BACKGROUND_COLOR);
  _canvas->drawLine(PLOT_LEFTBUFFER-1, PLOT_TOPBUFFER, 
                    PLOT_LEFTBUFFER-1, PLOT_H+PLOT_TOPBUFFER, BORDER_COLOR);
  _canvas->drawLine(PLOT_LEFTBUFFER-1, PLOT_TOPBUFFER+PLOT_H+1, 
                    ARCADA_TFT_WIDTH, PLOT_TOPBUFFER+PLOT_H+1, BORDER_COLOR);
  _canvas->setTextSize(3);
  _canvas->setTextColor(TITLE_COLOR);
  uint16_t title_len = strlen(title) * 12;
  _canvas->setCursor((_canvas->width()-title_len)/2, 0);
  _canvas->print(title);
  
  float minY = 0;
  float maxY = 0;

  if (buffer1.size() > 0) {
    maxY = minY = buffer1[0];
  }
  for (int i=0; i< buffer1.size(); i++) {
    minY = min(minY, buffer1[i]);
    maxY = max(maxY, buffer1[i]);
  }
  for (int i=0; i< buffer2.size(); i++) {
    minY = min(minY, buffer2[i]);
    maxY = max(maxY, buffer2[i]);
  }
  for (int i=0; i< buffer3.size(); i++) {
    minY = min(minY, buffer3[i]);
    maxY = max(maxY, buffer3[i]);
  }
  //Serial.printf("Data range: %f ~ %f\n", minY, maxY);

  float MIN_DELTA = 10.0;
  if (maxY - minY < MIN_DELTA) {
     float mid = (maxY + minY) / 2;
     maxY = mid + MIN_DELTA / 2;
     minY = mid - MIN_DELTA / 2;
  } else {
    float extra = (maxY - minY) / 10;
    maxY += extra;
    minY -= extra;
  }
  //Serial.printf("Y range: %f ~ %f\n", minY, maxY);

  printTicks(_canvas, 5, minY, maxY);

  int16_t last_y = 0, last_x = 0;
  for (int i=0; i<buffer1.size(); i++) {
    int16_t y = mapf(buffer1[i], minY, maxY, PLOT_TOPBUFFER+PLOT_H, PLOT_TOPBUFFER);
    int16_t x = PLOT_LEFTBUFFER+i;
    if (i == 0) {
      last_y = y;
      last_x = x;
    }
    _canvas->drawLine(last_x, last_y, x, y, PLOT_COLOR_1);
    last_x = x;
    last_y = y;
  }

  last_y = 0, last_x = 0;
  for (int i=0; i<buffer2.size(); i++) {
    int16_t y = mapf(buffer2[i], minY, maxY, PLOT_TOPBUFFER+PLOT_H, PLOT_TOPBUFFER);
    int16_t x = PLOT_LEFTBUFFER+i;
    if (i == 0) {
      last_y = y;
      last_x = x;
    }
    _canvas->drawLine(last_x, last_y, x, y, PLOT_COLOR_2);
    last_x = x;
    last_y = y;
  }

  last_y = 0, last_x = 0;
  for (int i=0; i<buffer3.size(); i++) {
    int16_t y = mapf(buffer3[i], minY, maxY, PLOT_TOPBUFFER+PLOT_H, PLOT_TOPBUFFER);
    int16_t x = PLOT_LEFTBUFFER+i;
    if (i == 0) {
      last_y = y;
      last_x = x;
    }
    _canvas->drawLine(last_x, last_y, x, y, PLOT_COLOR_3);
    last_x = x;
    last_y = y;
  }
}


void printTicks(GFXcanvas16 *_canvas, uint8_t ticks, float minY, float maxY) {
  _canvas->setTextSize(2);
  _canvas->setTextColor(TICKTEXT_COLOR);
  // Draw ticks
  for (int t=0; t<ticks; t++) {
    float v = mapf(t, 0, ticks-1, minY, maxY);
    uint16_t y = mapf(t, 0, ticks-1, ARCADA_TFT_HEIGHT - PLOT_BOTTOMBUFFER - 4, PLOT_TOPBUFFER);
    printLabel(_canvas, 0, y, v);
    uint16_t line_y = mapf(t, 0, ticks-1, ARCADA_TFT_HEIGHT - PLOT_BOTTOMBUFFER, PLOT_TOPBUFFER);
    _canvas->drawLine(PLOT_LEFTBUFFER, line_y, ARCADA_TFT_WIDTH, line_y, TICKLINE_COLOR);
  }
}

void printLabel(GFXcanvas16 *_canvas, uint16_t x, uint16_t y, float val) {
  char label[20];
  if (abs(val) < 1) {
    snprintf(label, 19, "%0.2f", val);
  } else if (abs(val) < 10) {
    snprintf(label, 19, "%0.1f", val);
  } else {
    snprintf(label, 19, "%d", (int)val);
  }
  
  _canvas->setCursor(PLOT_LEFTBUFFER-strlen(label)*6-5, y);
  _canvas->print(label);
}

/*****************************************************************/

int16_t minwave, maxwave;
short sampleBuffer[256];// buffer to read samples into, each sample is 16-bits
volatile int samplesRead;// number of samples read

int32_t getPDMwave(int32_t samples) {
  minwave = 30000;
  maxwave = -30000;
  
  while (samples > 0) {
    if (!samplesRead) {
      yield();
      continue;
    }
    for (int i = 0; i < samplesRead; i++) {
      minwave = min(sampleBuffer[i], minwave);
      maxwave = max(sampleBuffer[i], maxwave);
      //Serial.println(sampleBuffer[i]);
      samples--;
    }
    // clear the read count
    samplesRead = 0;
  }
  return maxwave-minwave;  
}


void onPDMdata() {
  // query the number of bytes available
  int bytesAvailable = PDM.available();

  // read into the sample buffer
  PDM.read(sampleBuffer, bytesAvailable);

  // 16-bit, 2 bytes per sample
  samplesRead = bytesAvailable / 2;
}

static float mapf(float x, float in_min, float in_max,
                  float out_min, float out_max) {
  return (x - in_min) * (out_max - out_min) / (in_max - in_min) + out_min;
}

/******************************* Drawing functions */

void updateScreenCallback(void) {  }

void screenClearCallback(void) {  }

void drawPixelCallback(int16_t x, int16_t y, uint8_t red, uint8_t green, uint8_t blue) {
    arcada.display->drawPixel(x, y, arcada.display->color565(red, green, blue));
}

void drawLineCallback(int16_t x, int16_t y, uint8_t *buf, int16_t w, uint16_t *palette, int16_t skip) {
    uint16_t maxline = arcada.display->width();
    bool splitdisplay = false;
    
    uint8_t pixel;
    //uint32_t t = millis();
    //x += gif_offset_x;
    //y += gif_offset_y;
    if (y >= arcada.display->height() || x >= maxline ) {
      return;
    }
    
    if (x + w > maxline) {
      w = maxline - x;
    }
    if (w <= 0) return;


    uint16_t buf565[2][w];
    bool first = true; // First write op on this line?
    uint8_t bufidx = 0;
    uint16_t *ptr;

    for (int i = 0; i < w; ) {
        int n = 0, startColumn = i;
        ptr = &buf565[bufidx][0];
        // Handle opaque span of pixels (stop at end of line or first transparent pixel)
        if (skip == -1) {// no transparent pixels
          while(i < w) {
            ptr[n++] = palette[buf[i++]];
          }
        }
        else {
          while((i < w) && ((pixel = buf[i++]) != skip)) {
            ptr[n++] = palette[pixel];
          }
        }
        if (n) {
            arcada.display->dmaWait(); // Wait for prior DMA transfer to complete
            if (first) {
              arcada.display->endWrite();   // End transaction from prior callback
              arcada.display->startWrite(); // Start new display transaction
              first = false;
            }
            arcada.display->setAddrWindow(x + startColumn, y, min(maxline, n), 1);
            arcada.display->writePixels(ptr, min(maxline, n), false, true);
            bufidx = 1 - bufidx;
        }
    }
//    arcada.display->dmaWait(); // Wait for last DMA transfer to complete
}


bool fileSeekCallback(unsigned long position) {
  return file.seek(position);
}

unsigned long filePositionCallback(void) { 
  return file.position(); 
}

int fileReadCallback(void) {
    return file.read(); 
}

int fileReadBlockCallback(void * buffer, int numberOfBytes) {
    return file.read((uint8_t*)buffer, numberOfBytes); //.kbv
}


byte myPxTmp(float a) {
  if (a>100){a=0;}// temps below 0 return as 5xx, perhaps a library error
  //moved from 46 element array to 33 element array
  //int temp=(45*(a-MINTEMP[tempScale % 6]))/(MAXTEMP[tempScale % 6]-MINTEMP[tempScale % 6]);
  //if (temp>45){temp=45;}
  int temp=(32*(a-MINTEMP[tempScale % 6]))/(MAXTEMP[tempScale % 6]-MINTEMP[tempScale % 6]);
  if (temp>32){temp=32;}
  if (temp<0){temp=0;}
  return temp;  
}
