// Lcd pinout settings
sbit LCD_RS at RD0_bit;
sbit LCD_EN at RD1_bit;
sbit LCD_D7 at RD4_bit;
sbit LCD_D6 at RD5_bit;
sbit LCD_D5 at RD6_bit;
sbit LCD_D4 at RD7_bit;
// Pin direction
sbit LCD_RS_Direction at TRISD0_bit;
sbit LCD_EN_Direction at TRISD1_bit;
sbit LCD_D7_Direction at TRISD4_bit;
sbit LCD_D6_Direction at TRISD5_bit;
sbit LCD_D5_Direction at TRISD6_bit;
sbit LCD_D4_Direction at TRISD7_bit;
unsigned char petsnumber = 0;
unsigned char answr2 = 0;
unsigned char hh = 0;  //hours
unsigned char mm = 0; //min
unsigned char cursor_pos = 0;
unsigned char buffer[5];
unsigned char i,j,petscan;
unsigned char Distance;
unsigned char petnumber[3];
unsigned char foodlevel[3];
unsigned char pettimeH[3];
unsigned char pettimeM[3];
unsigned char T1overflow;
unsigned char T1counts;
unsigned char T1time;
unsigned char seconds1 = 0;
unsigned char minutes1 = 0;
unsigned char hours1 = 0;
unsigned char seconds2 = 0;
unsigned char minutes2 = 0;
unsigned char hours2 = 0;
unsigned char timersave;
 unsigned char sec=0;
void __delay_ms(unsigned char milliseconds) ; //delay for millisecound
void set_servo_position(unsigned char degrees);
 unsigned char minutes = 0;
unsigned char hours = 0;
   unsigned char timerCount = 0;
void read_sonar(void);
 void pwm_init(void);
  void Delay_us(unsigned int microseconds);   //delay for microsecound
  void init_sonar(void);

void interrupt(void) {/// for count the hours and min will enter every  32ms
    if (INTCON&0x04) // Timer 0 interrupt flag
    {
        asm BCF INTCON,3; // Clear Timer 0 interrupt flag
        sec++;
         if(sec==31){//reach one sec count
        timerCount++;
          sec=0;
        if (timerCount >= 61) // Increment every approximately 1 minute
        {
            timerCount = 0;
            minutes++; // Increment minutes
               minutes1++;
                minutes2++ ;
            if (minutes >= 60)
            {
                minutes = 0; // Reset minutes
                hours1++; 
                hours2++;// Increment hours

            }
        }
    }
    }
    }

void set_servo_position(unsigned char degrees) {  //to change the
    int pulse_width = (degrees + 90) * 8 + 500; // Calculate pulse width (500 to 2400)
    CCPR1L = pulse_width >> 2; // Set CCPR1L register
    CCP1CON = (CCP1CON & 0xCF) | ((pulse_width & 0x03) << 4); // Set CCP1CON register
    Delay_ms(50*4); // Delay for the servo to reach the desired position
}

void read_sonar(void){

    T1overflow=0;
    TMR1H=0;
    TMR1L=0;

    PORTB=0x04;//Trigger the ultrasonic sensor (RB2 connected to trigger)
    Delay_us(10);//keep trigger for 10uS
    PORTB=0x00;//Remove trigger
    while(!(PORTB&0x02));
    T1CON=0x19;//TMR1 ON,  Fosc/4 (inc 1uS) with 1:2 prescaler (TMR1 overflow after 0xFFFF counts ==65536)==> 65.536ms
    while(PORTB&0x02);
    T1CON=0x18;//TMR1 OFF,  Fosc/4 (inc 1uS) with 1:1 prescaler (TMR1 overflow after 0xFFFF counts ==65536)==> 65.536ms
    T1counts=((TMR1H<<8)|TMR1L)+(T1overflow*65536);
       if(TMR1L>100) PORTC=0xFF;
       T1time=T1counts;//in microseconds
       Distance=((T1time*34)/(1000))/2; //in cm, shift left twice to divide by 2
       //range=high level time(usec)*velocity(340m/sec)/2 >> range=(time*0.034cm/usec)/2
       //time is in usec and distance is in cm so 340m/sec >> 0.034cm/usec
       //divide by 2 since the travelled distance is twice that of the range from the object leaving the sensor then returning when hitting an object)
}
void pwm_init() {
   TRISC=TRISC&0xf9;
   CCP1CON=0X0C;
     T2CON=T2CON | 0X07;
    PR2 = 249; // Set period register for 50Hz frequency
}


void  __delay_ms(unsigned int milliseconds) {

        for (j = 0; j < 238; j++) {
            Delay_us(1000);

    }
}
void Delay_us(unsigned int microseconds) {

    while (microseconds--) {
        for (j = 0; j < 12; j++) {
            asm nop;
        }
    }
}
void init_sonar(void){
   T1overflow=0;
    T1counts=0;
    T1time=0;
    Distance=0;
    TMR1H=0;
    TMR1L=0;
    TRISB=0x02; //RB2 for trigger, RB1 for echo
    PORTB=0x00;
    INTCON=INTCON|0xC0;//GIE and PIE
    PIE1=PIE1|0x01;// Enable TMR1 Overflow interrupt

    T1CON=0x18;//TMR1 OFF,  Fosc/4 (inc 1uS) with 1:2 prescaler (TMR1 overflow after 0xFFFF counts ==65536)==> 65.536ms
}

void main() {

ADCON1 =0x06;//
 TRISA = 0xFF;
 PORTA=0x00;
 ADCON1 =0x06;//
 OPTION_REG = 0xC7; // Set T0CS and T08BIT bits for 8-bit mode, and PSA and PS2-PS0 bits for 8 prescaler
   TMR0 = 0; // Initialize Timer 0


// Initialize LCD module
Lcd_Init();
Lcd_Cmd(_LCD_CLEAR);
Lcd_Cmd(_LCD_CURSOR_OFF);
Lcd_Out(1, 1, "NO.pets?");
Delay_ms(1000);
 while (!(PORTA & 0x04)) { // for number of pets
        if (PORTA & 0x01) {
            petsnumber++;
            // Perform bounds checking for answr1 if necessary
            Delay_ms(200);
        }
        if (PORTA & 0x02) {
            petsnumber--;
            // Perform bounds checking for answr1 if necessary
            Delay_ms(200);
        }
        Lcd_Out(2, 8, "   ");

        ByteToStr(petsnumber, buffer);
        Lcd_Out(2, 8, buffer);
    }
    while (PORTA & 0x04) {}
      Lcd_Cmd(_LCD_CLEAR);
    Lcd_Cmd(_LCD_CURSOR_OFF);
    for(i=1;i<=petsnumber;i++){
    Lcd_Out(1, 1, "Pick a level");
    Lcd_Out(2, 1, "Answer:");
    answr2=0;
    Delay_ms(1000);
    while (!(PORTA & 0x04)) {  //check if enter is pressed
    // for level time     loop
        if (PORTA & 0x01) {     //inc button
            answr2++;
            // Perform bounds checking for answr2 if necessary
            Delay_ms(200);
        }
        if (PORTA & 0x02) {      //dec button
            answr2--;
            // Perform bounds checking for answr2 if necessary
            Delay_ms(200);
        }
        Lcd_Out(2, 8, "   ");
        ByteToStr(answr2, buffer);
        Lcd_Out(2, 8, buffer);

    }
    Lcd_Cmd(_LCD_CLEAR);
    while (PORTA & 0x04) {}
    answr2=foodlevel[i];
        if(answr2==1){ //the cm for food level
          foodlevel[i]=18;

          }
          if(answr2==2){
          foodlevel[i]=16;
           }
          if(answr2==3){
          foodlevel[i]=14;

          }
    Lcd_Cmd(_LCD_CLEAR);
    Lcd_Cmd(_LCD_CURSOR_OFF);
    Lcd_Out(1, 1, "Meals timing");
    Lcd_Out(2, 1, "Hours:");
       hh=0;
    while (!(PORTA & 0x04)) { // for hours
        if (PORTA & 0x01) {
            hh = (hh + 1) % 24;
            // Perform bounds checking for hh if necessary
            Delay_ms(200);
        }
        if (PORTA & 0x02) {
            hh = (hh + 23) % 24;
            // Perform bounds checking for hh if necessary
            Delay_ms(200);
        }
        Lcd_Chr(2, 10, '0' + (hh / 10));
        Lcd_Chr(2, 11, '0' + (hh % 10));
    }

    while (PORTA & 0x04) {}
      pettimeH[i]=hh;
    Lcd_Cmd(_LCD_CLEAR);
    Delay_ms(200);
    Lcd_Cmd(_LCD_CURSOR_OFF);
    Lcd_Out(1, 1, "Timer");
    Lcd_Out(2, 1, "Min:");
      mm=0;
    while (!(PORTA & 0x04)) { // for minutes
        if (PORTA & 0x01) {
            mm = (mm + 1) % 60;
            // Perform bounds checking for mm if necessary
            Delay_ms(200);
        }
        if (PORTA & 0x02) {
            mm = (mm + 59) % 60;
            // Perform bounds checking for mm if necessary
            Delay_ms(200);
        }
        Lcd_Chr(2, 10, '0' + (mm / 10));
        Lcd_Chr(2, 11, '0' + (mm % 10));

    }

    while (PORTA & 0x04) {} ;
     pettimeM[i]=mm;
    Lcd_Cmd(_LCD_CLEAR);
    Lcd_Cmd(_LCD_CURSOR_OFF);
    Lcd_Out(1, 1, "Scan ");
    while(!(PORTB&0x10));//rb4
    Lcd_Cmd(_LCD_CLEAR);
    Delay_ms(200);
    Lcd_Cmd(_LCD_CURSOR_OFF);
    Lcd_Out(1, 1, "Scan done");
    Delay_ms(5000);


    }
    pwm_init();
    init_sonar();
    

     set_servo_position(90); // closed
      TMR0 = 0; // Initialize Timer 0
     // INTCON = 0xA0; // Enable Timer 0 interrupt and global interrupts
 while(1){
          if(PORTB&0x20){//rb5        //chech if pet came
           INTCON = 0x20; //disable the inturrept
          timersave= TMR0;
            petscan=1;
            
            
          }
          if(PORTB&0x40){ //rb6
          INTCON = 0x20;
          timersave= TMR0;
            petscan=2;
          }

        if(petscan==1){     //do the configration for the selected pet
            if(pettimeH[1]>=hours1 && pettimeM[1]>=minutes1){

               Lcd_Out(1, 1, "door opens pet1");
               Delay_ms(5000);
                set_servo_position(0);
                 while(!(Distance >= foodlevel[1])){
                     read_sonar();
                 }
                 set_servo_position(90);
                 Lcd_Cmd(_LCD_CLEAR);
                 Lcd_Cmd(_LCD_CURSOR_OFF);
                 Lcd_Out(1, 1, "door clos pet1");
                 Delay_ms(5000);
                  hours1=0;
                 minutes1=0;
                 seconds1=0;
                 petscan=0;
                  Distance=0;
                  TMR0= timersave;
                   INTCON = 0xA0;     //enable the overflow inturrept
        }
        }
        if(petscan==2){       //do the configration for the selected pet
             if(pettimeH[2]>=hours2 && pettimeM[2]>=minutes2){
                 Lcd_Out(1, 1, "door opens pet2");
                 Delay_ms(5000);
                set_servo_position(0);
                 while(!(Distance >= foodlevel[2])){
                     read_sonar();
                 }
                 set_servo_position(90);
                 Lcd_Cmd(_LCD_CLEAR);
                 Lcd_Cmd(_LCD_CURSOR_OFF);
                 Lcd_Out(1, 1, "door clos pet2");
                 Delay_ms(5000);
                 hours2=0;
                 minutes2=0;
                 seconds2=0;
                petscan=0;
                Distance=0;
                TMR0= timersave;
                 INTCON = 0xA0;

        }

        }

}
                  }
