#include "reg51.h"
#include "lcd1602.h"

volatile unsigned char cmd = 0;

void main(void)
{
    Delay400Ms();
    LCDInit();

    TMOD = 0x20;
    TH1 = 0xF3;
    TL1 = 0xF3;
    TR1 = 1;
    SCON = 0x50;
    ES = 1;
    EA = 1;

    while (1)
    {
        if (cmd == 'B')
        {
            cmd = 0;
            LCDClear();
            DisplayListChar(0, 0, "Blue_Tooth");
        }
    }
}

void UART_ISR(void) interrupt 4
{
    if (RI)
    {
        RI = 0;
        cmd = SBUF;
    }
}
