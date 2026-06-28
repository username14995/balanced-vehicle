/**
* @par Copyright (C): 2018-2028, Shenzhen Yahboom Tech
* @file         // main.c
* @author       // lly
* @version      // V1.0
* @date         // 240628
* @brief        // ������� Program entry
* @details      
* @par History  // �޸���ʷ��¼�б���ÿ���޸ļ�¼Ӧ�����޸����ڡ��޸��߼�
*               // �޸����ݼ���  Modification history list, each modification record should include the modification date, modifier and a brief description of the modification content
*/ 

#include "AllHeader.h"
#include "intsever.h"
//ע��:������������ʱ��Ҫ�ж��Ƿ���������ѹ
//Attention: When operating the buzzer, check if it is at normal voltage

uint8_t GET_Angle_Way=2;                             //��ȡ�Ƕȵ��㷨��1����Ԫ��  2��������  3�������˲�  //Algorithm for obtaining angles, 1: Quaternion 2: Kalman 3: Complementary filtering
float Angle_Balance,Gyro_Balance,Gyro_Turn;     		//ƽ����� ƽ�������� ת�������� //Balance tilt angle balance gyroscope steering gyroscope
int Motor_Left,Motor_Right;                 	  		//���PWM���� //Motor PWM variable
int Temperature;                                		//�¶ȱ��� 		//Temperature variable
float Acceleration_Z;                           		//Z����ٶȼ�  //Z-axis accelerometer
int Voltage,Mid_Angle;                          		//��ص�ѹ������صı�������е��ֵ Battery voltage sampling related variables, mechanical median
float Move_X,Move_Z; //Move_X:ǰ���ٶ�  Move_Z��ת���ٶ�  //Move_X: Forward speed Move_Z: Steering speed
u8 Stop_Flag = 1; //0:��ʼ 1:ֹͣ  //0: Start 1: Stop


char showbuf[20]={'\0'};

extern u8 newLineReceived;
extern u8 bulettohflag;

int main(void)
{	
	Mid_Angle = 1; //����С������ȡ //Obtain based on the car
	
	
	bsp_init();
	
	MPU6050_EXTI_Init();					//���жϷ������ŵ���� //This interrupt service function is placed last
	
	OLED_Draw_Line("put down key start!", 1, true, true); 

	while(!Key1_State(1));

	USART3_Send_U8('B');

	Stop_Flag = 0; //��ʼ���� //Start controlling

	
	OLED_Draw_Line("start control!", 1, true, true); 
	


	while(1)
	{
		
		if (newLineReceived) //����ң�ط��� Bluetooth remote control service
		{
			ProtocolCpyData();
			Protocol();
		}
		if(bulettohflag == 1) //�˷����ϱ������ݣ�app�����bug The data reported by this method may cause a bug in the app
		{
			bulettohflag = 0;
			SendAutoUp();//�����Զ��ϱ����� Bluetooth automatically reports data 
		}
		
		
		sprintf(showbuf,"angle = %.2f  ",Angle_Balance);
		OLED_Draw_Line(showbuf, 3, false, true); 
	
	}
}

