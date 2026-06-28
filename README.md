#ifndef __LCD1602_H_
#define __LCD1602_H_

void WriteDataLCD(unsigned char WDLCD);
void WriteCommandLCD(unsigned char WCLCD, unsigned char BuysC);
unsigned char ReadDataLCD(void);
unsigned char ReadStatusLCD(void);
void LCDInit(void);
void LCDClear(void);
void DisplayOneChar(unsigned char X, unsigned char Y, unsigned char DData);
void DisplayListChar(unsigned char X, unsigned char Y, unsigned char *DData);
void Delay5Ms(void);
void Delay400Ms(void);

#endif
