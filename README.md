#include "reg51.h"
#include "lcd1602.h"

sbit LCD_RS = P1^0;
sbit LCD_RW = P1^1;
sbit LCD_E  = P1^2;

#define LCD_Data P0
#define Busy    0x80

void WriteDataLCD(unsigned char WDLCD)
{
    ReadStatusLCD();
    LCD_Data = WDLCD;
    LCD_RS = 1;
    LCD_RW = 0;
    LCD_E = 0;
    LCD_E = 0;
    LCD_E = 1;
}

void WriteCommandLCD(unsigned char WCLCD, unsigned char BuysC)
{
    if (BuysC) ReadStatusLCD();
    LCD_Data = WCLCD;
    LCD_RS = 0;
    LCD_RW = 0;
    LCD_E = 0;
    LCD_E = 0;
    LCD_E = 1;
}

unsigned char ReadDataLCD(void)
{
    LCD_RS = 1;
    LCD_RW = 1;
    LCD_E = 0;
    LCD_E = 0;
    LCD_E = 1;
    return LCD_Data;
}

unsigned char ReadStatusLCD(void)
{
    LCD_Data = 0xFF;
    LCD_RS = 0;
    LCD_RW = 1;
    LCD_E = 0;
    LCD_E = 0;
    LCD_E = 1;
    while (LCD_Data & Busy);
    return LCD_Data;
}

void LCDInit(void)
{
    LCD_Data = 0;
    WriteCommandLCD(0x38, 0);
    Delay5Ms();
    WriteCommandLCD(0x38, 0);
    Delay5Ms();
    WriteCommandLCD(0x38, 0);
    Delay5Ms();

    WriteCommandLCD(0x38, 1);
    WriteCommandLCD(0x08, 1);
    WriteCommandLCD(0x01, 1);
    WriteCommandLCD(0x06, 1);
    WriteCommandLCD(0x0C, 1);
}

void LCDClear(void)
{
    WriteCommandLCD(0x01, 1);
}

void DisplayOneChar(unsigned char X, unsigned char Y, unsigned char DData)
{
    Y &= 0x1;
    X &= 0xF;
    if (Y) X |= 0x40;
    X |= 0x80;
    WriteCommandLCD(X, 0);
    WriteDataLCD(DData);
}

void DisplayListChar(unsigned char X, unsigned char Y, unsigned char *DData)
{
    unsigned char ListLength;
    ListLength = 0;
    Y &= 0x1;
    X &= 0xF;
    while (DData[ListLength] >= 0x20)
    {
        if (X <= 0xF)
        {
            DisplayOneChar(X, Y, DData[ListLength]);
            ListLength++;
            X++;
        }
    }
}

void Delay5Ms(void)
{
    unsigned int TempCyc = 5552;
    while (TempCyc--);
}

void Delay400Ms(void)
{
    unsigned char TempCycA = 5;
    unsigned int TempCycB;
    while (TempCycA--)
    {
        TempCycB = 7269;
        while (TempCycB--);
    }
}
