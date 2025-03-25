> 제출했던 파일들에서 쓸데 없는 주석 조금 정리한 버전

> 배운 것을 베이스로 하여 코드를 짬. 해당 베이스적인 부분에 달린 주석은 다른 사람의 주석임.(예전에 올렸던 코드와 동일한 기능을 다른 사람이 다른 방법으로 풀이했던 부분도 있음. 빠르게 원하는 기능을 구현하기 위해 이전에 구현했던 기능이자 안쓰는 부분을 따로 정리하진 않음)
> 설명을 위한 코드라기보다 목표 성능을 제한 시간 내에 빠르게 완료하고자 한 코드임
  >> 주석 설명을 많이 하진 않음.
  >> 급하게 코드를 수정하거나 디버깅할 때(다른 방법으로 전환) 주석도 함께 수정하지 않아서 코드 라인과 주석의 내용이 일치하지 않는 것들 있음. 이전에 쓴 방법에서 필요했던 부분이라 가장 마지막으로 사용한 방법에서는 빼도 되는 변수 이런 것도 걍 하나하나 보면서 지우고 다시 모든 동작을 확인하지 않고 걍 냅뒀음(어차피 있어도 정상작동 될 부분은 빠르게 다른 방법 시도해볼 때 냅뒀었음). 시간 제한이 있기에 코드 업그레이드보다 목표 만족에 초점을 둠

> AUTO-CAR올렸다고 자율주행차 쪽에서 일하고 있는 건 아니고 진로는 약간 고민중

- 나중에 코드 다시 필요하게 되면 떠올리기용으로 작성하는 코드에 대한 메모
  + 디버깅과 추가 기능을 위해 블루투스, USART를 이용하여 코드 중간의 데이터 / MCU에 있는 데이터 / 타이밍을 핸드폰 어플에 출력하며 진행함. 
  + 주의할 점은, 어플에서는 출력되는 속도의 한계가 있으므로 너무 많이 출력하면 타이밍이 동기화돼있지 않고 밀림. 따라서 상황에 따라(MCU와 복합적인 모듈 사용은 타이밍이 중요하기에) 순서를 보고 싶으면 시간 간격이 매우 짧아서 어플에서의 출력 시각은 밀리는 데이터도 출력 하되, 특정 동작을 할 때 해당변수의 값을 정확한 타이밍으로 알고 싶을 땐 msec_count등의 변수와 % 연산을 이용하여 1초 마다 등 핸드폰 어플에서의 출력이 밀리지 않는 속도로 출력하게 해야함.

  + 자동주행모드로도 레일을 정상 완주. 
  + 다른 성능이 필요하게 될 수도 있으므로 시간이 촉박해도 기왕이면 계속 바꿔가며 작동시킬 코드에 대해선 그 부분의 코드만이라도 결과가 어느정도 나왔는지와 함께 [버전관리]를 하는 것도 좋음( [시간이 아주 급해도] [카카오톡 나에게 보내기] 방법을 썼으면 [시간 손실 없는 수준으로도] 버전관리 가능했을 것임 )
    > 배경 : 나중에 완주였던 성능 목표에 속도가 추가 되어서 완주한 느린 방법을 조금 바꿔보니 같이 영향 받는 요소가 많았음. 
      차라리 초기 간단한 방법의 코드가 속도가 빠르기도 했어서 과거의 방법에서 업그레이드 하는 게  시간이 덜 걸릴 상황이 있었음(/레일 다른 부분이나 새레일까지 감안하니 과거의 것이 더 잘가는 상황). 
      그러나 빠른 시간 안에 답을 찾기 위해 여러 방법을 변경해가며 시도했었다보니 과거의 특정 방법으로 돌아가기 어려웠음.(그때는 성능 목표가 완주라길래 변경 후 코드가 레일 완주를 더 잘해서 이전 것이 덮어쓰여져도 업그레이드일 뿐이니 완주 실패한 이전 코드는 정확히 몰라도 상관없을 줄 알았음. 
      나중에 빠르게 도달하는 것도 중요해졌기에 이전 특정 방법이 더 유용하게 됐었음. 

  + main코드 두 버전의 차이 :  소리를 더 커스터마이징하였는데, 장치를 분해했기에 변경된 소리를 확인해보진 않았음. 따라서 소리 추가 커스터마이징 이전인 버전이자 원하는 기능 및 각종 주행이 정상적으로 되는 걸 많이 확인 완료한 버전을 따로 올림

- main.c : Version 1
  + 주석은 version2의 main.c에 더 많음. Version 1에서는 주석 좀 더 많이 지움
  ```C
  #define  F_CPU 16000000UL  // 16MHZ
  
  #include "button.h"
  
  #include <avr/interrupt.h>  // sei()등 
  #include <stdio.h>  // printf scanf fgets puts gets 등이 들어 있다. 
  #include "extern.h"
  #include "def.h"
  
  volatile int msec_count=0;
  volatile int ultrasonic_check_timer=0;
  
  extern void init_speaker_pwm(void);
  extern void BeepShort(int  );
  extern int get_button(int, int );
  extern int ultrasonic_get_distance(int);
  
  extern void I2C_LCD_Test(void);
  
  
  void manul_mode(void);
  void distance_check(void);
  void auto_mode(int, int, int);             // 자율주행
  
  // for printf 
  FILE OUTPUT = FDEV_SETUP_STREAM(UART0_transmit, NULL, _FDEV_SETUP_WRITE);
  
  int func_index=MANUAL_MODE;
  volatile uint32_t ultrasonic_trigger_timer=0;
  char buffer[128];
  
  
  extern int distance1, distance2, distance3;
  void (*pfunc[]) () =
  {
  	manul_mode,           // 수동 mode 
  	distance_check,       // 초음파 거리 측정
  	auto_mode_check,      // button check
  	auto_mode             // 자율주행 	
  };
  
  int led_toggle=0; 
  int mode_state=0;
  int count=0;
  
  ISR(TIMER0_OVF_vect)
  {
  	TCNT0=6;
  	msec_count++;   // 1ms마다 1씩 증가
      ultrasonic_trigger_timer++;
      
  	ultrasonic_check_timer++;
  }
  
  void USART_Transmit(char data) {
      while (!(UCSR0A & (1<<UDRE0)));
      UDR0 = data;
  }
  
  //void USART_SendString(char *str) {
      //while (*str != 0) {
          //USART_Transmit(*str++);
      //}
  //}
  void USART1_SendString(const char *str) {
      while (*str) {
          // UCSR1A의 UDRE1 비트가 1이 될 때까지 대기 (UART1 전송 버퍼 빔 확인)
          while (!(UCSR1A & (1<<UDRE1)));
          UDR1 = *str++;  // 문자를 UART1 데이터 레지스터에 전송
      }
  }
  
  int main(void)
  {
      
  	init_timer0();
  	init_uart0();
  	init_uart1(); // 수정
  	init_button();
  	init_L298N();
  	init_timer1();
      init_speaker_pwm();    
  	init_ultrasonic(); //뭔가 모터 돌아가는 소리가 더 작아짐. 더 느려지는건가싶기도
      
  
      if ( msec_count%100==0)
          I2C_LCD_Test(); //onece 버전으로 바꿈. 딜레이도안줬는뎅
  
  	stdout = &OUTPUT;   // printf가 동작 될 수 있도록 stdout에 OUTPUT 화일 포인터 assign 
  	sei();				// 전역(대문)으로 interrupt 허용
  	
      int mode=0;
      
      DDRA|= (1<<4) | (1<<5) |(1<<6) |(1<<7);
      
  	while(1)
  	{
          
                  if ( get_button(BUTTON0, BUTTON0PIN)){ // 보면 바로 함수 핵심 알고리즘 떠오를 정돈가?
  
                      mode_state = (mode_state+1)%4; // != != =!
                      
                                      bt_data=0;// 다른 모드 갔다 왔을 때 이전 수동모드의 작동이 안되도록
  count=0;
                                      stop(); // 한번만 stop이 아니라 모드 바뀔시 아예 stop
                                      OCR3A=0;   // 소리 0
                  }
                  
             if(mode_state==0)
             PORTA= (1<<4) ;
             else if(mode_state==1)
             PORTA= (1<<5) ;
             else if(mode_state==2)
             PORTA= (1<<6) ;
             else
             PORTA= (1<<7) ;
          //distance1 = 
          ultrasonic_get_distance(0); // L
          //distance2 = 
          ultrasonic_get_distance(1); // M
          //distance3 = 
          ultrasonic_get_distance(2); // R
          if ( msec_count%1000==0)//distance1||distance2||distance3)//
          {
              sprintf(buffer, "-Distance1: %d cm, Distance2: %d cm, Distance3: %d cm\n", distance1, distance2, distance3);
          USART1_SendString(buffer); 
          }
          
  		// pfunc[func_index]();
          
          if (mode_state==0) 
  	    {
              mode=0;
              manul_mode();   
          }
          
          else if(mode_state==1) // ambulance
          { 
                  	//if ((msec_count%20 == 0 )&& (count<100) )
                  	////for(j=0; j < 100; j++)
                  	//{
                      	//OCR3A += 10;
                          //count+=1;
                      	////_delay_ms(20);  // msec_count
                  	//}
                      //
                  	//if ((msec_count%20 == 0 )&& (100<=count) &&(count<200) ) // 순차적진행을 100 < count로 해결
                  	//{
                      	//OCR3A -= 10;
                      	////_delay_ms(20);
                          //count= (count+1)%200;// 199-> 0
                  	//}
                      
                      ////아래의 것을 변환
                      //
                      ////OCR3A = 900;
                          	////
                      ////for(j=0; j < 100; j++)
                      ////{
                      ////OCR3A += 10;
                      ////_delay_ms(20);
                      ////}
                      ////for(j=0; j < 100; j++)
                      ////{
                      ////OCR3A -= 10;
                      ////_delay_ms(20);
                      ////}
                      
              //Siren(10);
              auto_mode(distance1, distance2, distance3);
          }
  
          else if(mode_state==2)// fire_car
          {
              // RRR();
              
              //if ((msec_count%120 == 0 ))
              ////for(j=0; j < 100; j++)
              //{
                  //OCR3A = 1136;
              //}
              //if (msec_count%100 == 0 )
              //{
              //OCR3A = 0;
              //}
              //// 
              ////for(i=0; i<20; i++)
              ////{
                  ////OCR3A = 1136;
                  ////_delay_ms(100);
                  ////OCR3A = 0;
                  ////_delay_ms(20);
              ////}
  
              auto_mode(distance1, distance2, distance3);
          }        
          
          else if(mode_state==3) // kindergarten car
          
          { 
              //if (( (msec_count+200) %400 == 0 )) 
              //{
                  //OCR3A = 500;
              //}
              //if (msec_count%400 == 0 )
              //{
                  //OCR3A = 0;
              //}
              //// 아래의 것을 변환
                      ////OCR3A=500; //SPEAKER_TIMER_OCRA = 500;  // 0.00025sec (250us) : 0.0000005 * 500
                      //
                      ////
                      ////_delay_ms(200);
                      ////OCR3A=0;//SPEAKER_TIMER_OCRA = 0;
                      ////_delay_ms(200);
              auto_mode(distance1, distance2, distance3);
          }
  	}
  }
  
  void init_timer0(void)
  {
  // 16MHZ /64 분주(down) 분주: divider/prescale
  // ------ 분주비 계산 ----
  // (1) 16000000Hz/64 ==> 250,000HZ
  // (2) T(주기) 1clock의 소요시간 : 1/f = 1/250,000 ==> 0.0000004sec(4us) : 0.004ms	
  // (3) 8bit timer OV(OVflow) : 0.004ms x 256 = 0.001024sec --> 1.024ms 
  // 1ms마다 정확하게 INT를 띄우고 싶은면 0.004ms x 250개를 count = 0.001sec ==>1ms
  	TCNT0=6;   // TCNT : 0~256 1ms 마다 TIMER0_OVF_vect로 진입 한다. 
  	           // TCNT0 = 6으로 설정을 한 이유: 6-->256 : 250개의 펄스를 count하기 때문에 정확히 1ms가 된다.
  // (4) 분주비 설정 64분주 (250,000HZ --> 250KHz) P296 표13-1
  	TCCR0 |= 1 << CS02 | 0 << CS01 | 0 << CS00;  // TCCR0 |= 0xf4 보다는 죄측의 code 권장 
  // (5) Timer0 overflow INT를 허용(enable)
  	TIMSK |= 1 << TOIE0;  // TIMSK |= 0x01; 
  
  }
  
  void manul_mode(void)           // 수동 mode
  {
  	switch (bt_data)
  	{
  		case 'F':
  
  		case 'f':
  			forward(700);   // 4us x 500 = 0.002sec (2ms)
              //Beep(5);//노래 - PE3
              //if (( (msec_count+200) %400 == 0 )) // +200됐기에 얘가 더 빨리 400의 나머지가 0이됨. 맨 초기 이상적인 오차까지 방지하려면 msec가 0인 경우0이되지않게 기본적으로 +1해주는 방법도 있긴한데 난여기서 굳이
              //{
                  //OCR3A = 500;
              //}
              //if (msec_count%400 == 0 )
              //{
                  //OCR3A = 0;
              //}            
  			break;
  		case 'B':
  		case 'b':
  			backward(700);    // 4us x 500 = 0.002sec (2ms)
              BeepShort(5);
  			break;
  		case 'L':
  		case 'l':
              OCR3A=0;
  			turn_left(700);
  			break;
  		case 'R':
  		case 'r':
          OCR3A=0; 
  			turn_right(700);
  			break;
  		case 'S':
  		case 's':
          OCR3A=0; 
  			stop();
  			break;
  		default:
  			break;
  	}
  	func_index=DISTANCE_CHECK;
  }
  
  void distance_check(void)       // 초음파 거리 측정
  {
  	func_index=MODE_CHECK;
  }
  
  int SAFE_DISTANCE = 15;
  int CRITICAL_DISTANCE = 10;  // 더욱 긴급한 회피가 필요한 거리
  
  void auto_mode(int distance1, int distance2, int distance3) {
      
      // 긴급 회피 로직: 모든 센서가 매우 가까운 거리를 감지할 경우
      if (distance1 <= CRITICAL_DISTANCE && distance2 <= CRITICAL_DISTANCE && distance3 <= CRITICAL_DISTANCE) {
          backward(500);  // 일단 급히 후진
          _delay_ms(1000); // 잠깐 대기
          if (distance1 > distance3) {
              turn_left(700);  // 좌측 공간이 더 많으면 좌회전
              } else {
              turn_right(700); // 우측 공간이 더 많으면 우회전
          }
      }
      // 중앙 센서에 장애물이 감지되는 경우
      else if (distance2 <= SAFE_DISTANCE) {
          backward(300);  // 먼저 약간 후진
          _delay_ms(500); // 공간 확보를 위해 잠시 대기
          if (distance1 > distance3) {
              turn_left(700);  // 좌측이 더 많은 공간이 있으면 좌회전
              } else {
              turn_right(700); // 우측이 더 많은 공간이 있으면 우회전
          }
      }
      // 좌측 센서만 장애물 감지
      else if (distance1 <= SAFE_DISTANCE) {
          turn_right(700);  // 우회전
      }
      // 우측 센서만 장애물 감지
      else if (distance3 <= SAFE_DISTANCE) {
          turn_left(700);   // 좌회전
      }
      // 모든 센서에서 안전 거리 이상 감지될 경우
      else {
          forward(500);
      }
      func_index = MANUAL_MODE;
  }
  ```
  
- main.c : version 2
  
  ```C
  #define  F_CPU 16000000UL  // 16MHZ
  
  #include "button.h"
  
  #include <avr/interrupt.h>  // sei()등 
  #include <stdio.h>  // printf scanf fgets puts gets 등이 들어 있다. 
  #include "extern.h"
  #include "def.h"
  
  volatile int msec_count=0;
  volatile int ultrasonic_check_timer=0;
  
  extern void init_speaker_pwm(void);
  extern void BeepShort(int  );
  extern int get_button(int, int );
  extern int ultrasonic_get_distance(int);
  
  extern void I2C_LCD_Test(void);
  
  
  void manul_mode(void);
  void distance_check(void);
  void auto_mode(int, int, int);             // 자율주행
  
  // for printf 
  FILE OUTPUT = FDEV_SETUP_STREAM(UART0_transmit, NULL, _FDEV_SETUP_WRITE);
  
  int func_index=MANUAL_MODE;
  volatile uint32_t ultrasonic_trigger_timer=0;
  char buffer[128];
  
  
  extern int distance1, distance2, distance3;
  void (*pfunc[]) () =
  {
  	manul_mode,           // 수동 mode 
  	distance_check,       // 초음파 거리 측정
  	auto_mode_check,      // button check
  	auto_mode             // 자율주행 	
  };
  // TIMER0_OVF_vect : P278 표12-3
  // ISR(Interrupt Service Routine(함수)
  // --> H/W가 SW한테 event(상황변화)가 일어 났다고 알려 주는 공간 
  // 250개의 pulse를 count(1ms)하면 이곳으로 자동적으로 들어 온다. 
  // ISR루틴(함수)은 가능한 짧게 작성 한다.
  int led_toggle=0; 
  int mode_state=0;
  int count=0;
  //ISR(TIMER0_OVF_vect)
  //{
      //TCNT0 = 6;
      //
      //++msec_count;
      //
      //if(msec_button_delay > 0) {
          //--msec_button_delay;
      //}
      //
      //if(turn_delay_count > 0) {
          //--turn_delay_count;
      //}
      //
      //if(dir_delay_count > 0) {
          //--dir_delay_count;
      //}
      //
      //if(back_delay_count >0) {
          //--back_delay_count;
      //}
      //
      //if(play_count > 0) {
          //--play_count;
      //}
  //
      //if(ultrasonic_check_timer > 0) {
          //--ultrasonic_check_timer;
      //}
  //
      //++g_fps;
  //}
  
  ISR(TIMER0_OVF_vect)
  {
  	// 6~256 : 250(1ms) 그래서 TCNT0를 6으로 설정하는것이다. 
  	TCNT0=6;
  	msec_count++;   // 1ms마다 1씩 증가
      ultrasonic_trigger_timer++;
      
  	ultrasonic_check_timer++;
  }
  
  void USART_Transmit(char data) {
      while (!(UCSR0A & (1<<UDRE0)));
      UDR0 = data;
  }
  
  //void USART_SendString(char *str) {
      //while (*str != 0) {
          //USART_Transmit(*str++);
      //}
  //}
  void USART1_SendString(const char *str) {
      while (*str) {
          // UCSR1A의 UDRE1 비트가 1이 될 때까지 대기 (UART1 전송 버퍼 빔 확인)
          while (!(UCSR1A & (1<<UDRE1)));
          UDR1 = *str++;  // 문자를 UART1 데이터 레지스터에 전송
      }
  }
  
  int main(void)
  {
      
  	init_timer0();
  	init_uart0();
  	init_uart1(); // 핸드폰 블루투스 어플과 통신
  	init_button();
  	init_L298N();
  	init_timer1();
      init_speaker_pwm();    
  	init_ultrasonic(); //뭔가 모터 돌아가는 소리가 조금 더 작아짐. 조금 더 느려지는건가 싶기도 ( 굳이 이 부분 의문 해결을 위해 시간을 더 쓰진 않음 )
      
      
      
      if ( msec_count%100==0)
          I2C_LCD_Test(); //once 버전으로 바꿈.
          
  	//int n;
  	//while (1)
  	////{
      	//dist = ultrasonic_get_distance();
      	//n = ultrasonic_led_num(dist);
      	//fnd_display_num(&fnd, dist);
      	//led_on(n);
  	//}
      
      
  	stdout = &OUTPUT;   // printf가 동작 될 수 있도록 stdout에 OUTPUT 화일 포인터 assign 
  	sei();				// 전역(대문)으로 interrupt 허용
  	
      int mode=0;
      
      DDRA|= (1<<4) | (1<<5) |(1<<6) |(1<<7); // state 구별용 LED
      
  	while(1)
  	{
          
             if ( get_button(BUTTON0, BUTTON0PIN)){ // 보면 바로 함수 핵심 알고리즘 떠오를 정도로 할지는 진로 따라 다를듯
                  mode_state = (mode_state+1)%4; // != != =!
                  bt_data=0;// 다른 모드 갔다 왔을 때 이전 수동모드의 작동이 안되도록
                  count=0;
                  stop(); // 한번만 stop이 아니라 모드 바뀔시 아예 stop
                  OCR3A=0;   // 소리 0
                  }
             
             // state에 따른 LED 구별. 멀리서 빛 보기 & fnd 써봐서 쓸 줄 알지만 빠르게 구현하는 방법 = LED라 선택함     
             if(mode_state==0)
             PORTA= (1<<4) ;
             else if(mode_state==1)
             PORTA= (1<<5) ;
             else if(mode_state==2)
             PORTA= (1<<6) ;
             else
             PORTA= (1<<7) ;
          //distance1 = // 걍 빠른 풀이를 위해 전역변수로 함.
          ultrasonic_get_distance(0); // L 초음파
          //distance2 = 
          ultrasonic_get_distance(1); // M 중앙
          //distance3 = 
          ultrasonic_get_distance(2); // R 초음파
          
          if ( msec_count%1000==0)//distance1||distance2||distance3)//
          {
              sprintf(buffer, "-Distance1: %d cm, Distance2: %d cm, Distance3: %d cm\n", distance1, distance2, distance3);
              USART1_SendString(buffer); // 핸드폰으로 원격 제어할 때든 멀리서 핸드폰으로도 필요한 정보(MCU_car가 측정한 데이터)를 알 수 있도록 핸드폰 어플에 데이터 뜨게 함
          }
          //if (distance1<10)
          //
              //BeepShort(5);
          // 여기서 모드가 바뀔 때마다
          //or 인터럽트
          
  		// pfunc[func_index]();
          
          if (mode_state==0) 
  	    {
              mode=0;
              manul_mode();   
          }
          
          else if(mode_state==1) // ambulance
          { 
                  	//if ((msec_count%20 == 0 )&& (count<100) )
                  	////for(j=0; j < 100; j++)
                  	//{
                      	//OCR3A += 10;
                          //count+=1;
                      	////_delay_ms(20);  // msec_count
                  	//}
                      //
                  	//if ((msec_count%20 == 0 )&& (100<=count) &&(count<200) ) // for에 있던 순차적진행을 '100 < count'로 해결
                  	//{
                      	//OCR3A -= 10;
                      	////_delay_ms(20);
                          //count= (count+1)%200;// 199-> 0
                  	//}
                      
                      ////아래의 것을 delay없게 변환. ( 그 딜레이에 갇혀서 다른 모듈의 기능이 작동하지 않는 걸 방지)
                      //
                      ////OCR3A = 900;
                          	////
                      ////for(j=0; j < 100; j++)
                      ////{
                      ////OCR3A += 10;
                      ////_delay_ms(20);
                      ////}
                      ////for(j=0; j < 100; j++)
                      ////{
                      ////OCR3A -= 10;
                      ////_delay_ms(20);
                      ////}
                      
              //Siren(10);
              auto_mode(distance1, distance2, distance3);
          }
  
          else if(mode_state==2)// fire_car
          {
              // RRR();
              // 어차피 (주기적) 소리라  OCR3A = 1136;  OCR3A = 0;든 시작 위치 바뀌는 건 ㄱㅊ
              //if ((msec_count%120 == 0 ))
              ////for(j=0; j < 100; j++)
              //{
                  //OCR3A = 1136;
              //}
              //if (msec_count%100 == 0 )
              //{
              //OCR3A = 0;
              //}
              //// 
              ////for(i=0; i<20; i++)
              ////{
                  ////OCR3A = 1136;
                  ////_delay_ms(100);
                  ////OCR3A = 0;
                  ////_delay_ms(20);
              ////}
  
              auto_mode(distance1, distance2, distance3);
          }        
          
          else if(mode_state==3) // 유치원차
          
          { // 소리는 자유이기에 긴 비행기 노래 대신 그냥 비프 // 비행기할까 했었음 [미레도레미미미] 레레레 미미미  [미레도레미미미]  레레 미레도
              //if (( (msec_count+200) %400 == 0 )) // +200됐기에 얘가 더 빨리 400의 나머지가 0이됨. 맨 초기 이상적인 오차까지 방지하려면 msec가 0인 경우0이되지않게 기본적으로 +1해주는 방법도 있긴한데 난여기서 굳이
              //{
                  //OCR3A = 500;
              //}
              //if (msec_count%400 == 0 )
              //{
                  //OCR3A = 0;
              //}
              //// 아래의 것을 구현
                      ////OCR3A=500; //SPEAKER_TIMER_OCRA = 500;  // 0.00025sec (250us) : 0.0000005 * 500
                      //
                      ////
                      ////_delay_ms(200);
                      ////OCR3A=0;//SPEAKER_TIMER_OCRA = 0;
                      ////_delay_ms(200);
              auto_mode(distance1, distance2, distance3);
          }
  	}
  }
  
  void init_timer0(void)
  {
  // 16MHZ /64 분주(down) 분주: divider/prescale
  // ------ 분주비 계산 ----
  // (1) 16000000Hz/64 ==> 250,000HZ
  // (2) T(주기) 1clock의 소요시간 : 1/f = 1/250,000 ==> 0.0000004sec(4us) : 0.004ms	
  // (3) 8bit timer OV(OVflow) : 0.004ms x 256 = 0.001024sec --> 1.024ms 
  // 1ms마다 정확하게 INT를 띄우고 싶은면 0.004ms x 250개를 count = 0.001sec ==>1ms
  	TCNT0=6;   // TCNT : 0~256 1ms 마다 TIMER0_OVF_vect로 진입 한다. 
  	           // TCNT0 = 6으로 설정을 한 이유: 6-->256 : 250개의 펄스를 count하기 때문에 정확히 1ms가 된다.
  // (4) 분주비 설정 64분주 (250,000HZ --> 250KHz) P296 표13-1
  	TCCR0 |= 1 << CS02 | 0 << CS01 | 0 << CS00;  // TCCR0 |= 0xf4 보다는 죄측의 code 권장 
  // (5) Timer0 overflow INT를 허용(enable)
  	TIMSK |= 1 << TOIE0;  // TIMSK |= 0x01; 
  
  }
  
  void manul_mode(void)           // 수동 mode
  {
  	switch (bt_data)
  	{
  		case 'F':
  
  		case 'f':
  			forward(700);   // 4us x 500 = 0.002sec (2ms)
              //Beep(5);//노래 - PE3
              //if (( (msec_count+200) %400 == 0 )) // +200됐기에 얘가 더 빨리 400의 나머지가 0이됨. 맨 초기 이상적인 오차까지 방지하려면 msec가 0인 경우0이되지않게 기본적으로 +1해주는 방법도 있긴한데 난여기서 굳이
              //{
                  //OCR3A = 500;
              //}
              //if (msec_count%400 == 0 )
              //{
                  //OCR3A = 0;
              //}            
  			break;
  		case 'B':
  		case 'b':
  			backward(700);    // 4us x 500 = 0.002sec (2ms)
              BeepShort(5);//센서 및 비프음 빵빵
  			break;
  		case 'L':
  		case 'l':
              OCR3A=0;//소리 끄기.
  			turn_left(700);
  			break;
  		case 'R':
  		case 'r':
              OCR3A=0; 
  			turn_right(700);
  			break;
  		case 'S':
  		case 's':
              OCR3A=0; 
  			stop();
  			break;
  		default:
  			break;
  	}
  	func_index=DISTANCE_CHECK;
  }
  
  void distance_check(void)       // 초음파 거리 측정
  {
  	func_index=MODE_CHECK;
  }
  
  
  //int SAFE_DISTANCE = 15;
  //int CRITICAL_DISTANCE = 10;  // 더욱 긴급한 회피가 필요한 거리
  //
  //void auto_mode(int distance1, int distance2, int distance3) {
      //
      //// 긴급 회피 로직: 모든 센서가 매우 가까운 거리를 감지할 경우
      //if (distance1 <= CRITICAL_DISTANCE && distance2 <= CRITICAL_DISTANCE && distance3 <= CRITICAL_DISTANCE) {
          //backward(500);  // 일단 급히 후진
          ////_delay_ms(1000); // 잠깐 대기
          //if (distance1 > distance3) {
              //turn_left(1000);  // 좌측 공간이 더 많으면 좌회전
              //} else {
              //turn_right(1000); // 우측 공간이 더 많으면 우회전
          //}
      //}
      //// 중앙 센서에 장애물이 감지되는 경우
      //else if (distance2 <= SAFE_DISTANCE) {
          //backward(300);  // 먼저 약간 후진
          //// _delay_ms(500); // 공간 확보를 위해 잠시 대기
          //if (distance1 > distance3) {
              //turn_left(1000);  // 좌측이 더 많은 공간이 있으면 좌회전
              //} else {
              //turn_right(1000); // 우측이 더 많은 공간이 있으면 우회전
          //}
      //}
      //// 좌측 센서만 장애물 감지
      //else if (distance1 <= SAFE_DISTANCE) {
          //turn_right(1000);  // 우회전
      //}
      //// 우측 센서만 장애물 감지
      //else if (distance3 <= SAFE_DISTANCE) {
          //turn_left(1000);   // 좌회전
      //}
      //// 모든 센서에서 안전 거리 이상 감지될 경우
      //else {
          //forward(500);
      //}
      //func_index = MANUAL_MODE; // auto mode가 맞는데 자동모드 작성 때 바로 위 수동 모드 코드 수정하던 걸 복붙한 걸로 시작하다보니 이게 이어짐. 작동에 영향 안 끼치는 요소라 이 부분은 신경 안썼긴 한데 모든 기능 정상작동 되고 많이 확인한 코드의 원본을 남겨놓고자 AUTO로 수정하지 않고 그대로 MANUAL로 냅둠
  //}
  
  // 레일 완주 완료됐던 버전. 근데 장애물 피하는 게 너무 느림 
  int SAFE_DISTANCE = 15;
  int CRITICAL_DISTANCE = 10;  // 더욱 긴급한 회피가 필요한 거리
  
  void auto_mode(int distance1, int distance2, int distance3) {
      
      // 긴급 회피 로직: 모든 센서가 매우 가까운 거리를 감지할 경우
      if (distance1 <= CRITICAL_DISTANCE && distance2 <= CRITICAL_DISTANCE && distance3 <= CRITICAL_DISTANCE) {
          backward(500);  // 일단 급히 후진
          _delay_ms(1000); // 잠깐 대기
          if (distance1 > distance3) {
              turn_left(700);  // 좌측 공간이 더 많으면 좌회전
              } else {
              turn_right(700); // 우측 공간이 더 많으면 우회전
          }
      }
      // 중앙 센서에 장애물이 감지되는 경우
      else if (distance2 <= SAFE_DISTANCE) {
          backward(300);  // 먼저 약간 후진
          _delay_ms(500); // 공간 확보를 위해 잠시 대기
          if (distance1 > distance3) {
              turn_left(700);  // 좌측이 더 많은 공간이 있으면 좌회전
              } else {
              turn_right(700); // 우측이 더 많은 공간이 있으면 우회전
          }
      }
      // 좌측 센서만 장애물 감지
      else if (distance1 <= SAFE_DISTANCE) {
          turn_right(700);  // 우회전
      }
      // 우측 센서만 장애물 감지
      else if (distance3 <= SAFE_DISTANCE) {
          turn_left(700);   // 좌회전
      }
      // 모든 센서에서 안전 거리 이상 감지될 경우
      else {
          forward(500);
      }
      func_index = MANUAL_MODE;
  }
  
  //void auto_mode(int distance1, int distance2, int distance3)             // 자율주행
  //{
      //// 굳이 자연스러운 커브까진 패스하고 배운거로 최대한 간단하고 빠르게 구현
      //// 초음파센서별 각도차이 적으니 미리 조금씩 회전돼있는게 낫긴함.
      //
      //// 미리 좌회전까진 너무감
      // m0 (1) (2) (3) (4) 값조정
      ////m1 벽과 특정거리 유지 or m2 한쪽 벽 따라가기 (미로 찾기 마냥)
      //if ( (distance2 > 30) && (distance1>30) && (distance3>30) ) // 직진을 얼만큼이나 할건지//40, 60도 박음 // (1)
      //{
          //forward(500);
          //
      //}
      //// 왼쪽이나 오른쪽이 너무 가까우면 반대방향으로
  //
  //
      //// 그렇게 가깝지도 멀지도 않은데 오른쪽이나 왼쪽이 10보다 더 크다.
      //else if( distance3 -distance1 >= 10)// d_r이 d_l보다 몇센치 이상길면 오른쪽으로 // (2)//지금왼쪽 초음파 고장났으니 R -> 다음 날 초음파 센서 변경하여 해결. 
      //{
          //turn_right(500);
      //}
      //else if ( distance1  >= 10 + distance3) /(3)
      //{
          //turn_right(500);
      //}
      //
      ////else if (distance1<30) // distance3도  // 
      ////{
          ////turn_right(500);
      ////}
      ////
      ////else if (distance3<30)
      ////{
          ////turn_left(500);
      ////}
      ////
      //
      //else if ((distance2 < 10) || (distance1<10) || (distance3<10)) // 뭐라도 하나가 너무 가까우면 back?
      //{
          //backward(300);// 뒤에 초음파 없어서 조금만 쓰는 게 낫다
          //
      //}
      //
     //
      //else  //(4)
      //{
          //backward(300);
      //}
           //
  	//func_index=MANUAL_MODE;
  //} 

  ```
- ultrasonic.c
  ```C
  // 그때그때 필요한 것에 따른 방법을 시도해보며 주석(생각난 거 메모)을 달았던 것들을, 최종버전의 방법과 비교하며 팩트체크하고 수정하기 번거로워서 그냥 올리는 건 주석 지워서 올림
  
  // current sensor가 초음파 센서 3개라 idle0부터 ~ 3까지의 숫자를 썼었음. 나중에 방법 바꿔서 디버깅해보며 ~6까지도 씀
  
  #include "ultrasonic.h"
  
  void init_ultrasonic();
  void trigger_ultrasonic();
  void distance_ultrasonic();
  int ultrasonic_get_distance(int);
  void ultrasonic_trigger(int);
  
  extern void USART1_SendString(const char *str) ;
  
  extern volatile int ultrasonic_check_timer;
  extern volatile int ultrasonic_trigger_timer;
  
  volatile int ultrasonic_dis=0;
  volatile int ultrasonic_distance=0;
  
  volatile int distance1, distance2, distance3;
  volatile uint8_t current_sensor = 0;
  
  volatile char scm[50];
  int dist = 0;
  int dist_temp = 0;
  extern char buffer[128];
  extern int msec_count;
  
  
  // current sensor가 초음파 센서 3개라 idle0부터 ~ 3까지의 숫자를 썼었음. 나중에 방법 바꿔서 디버깅해보며 ~6까지도 씀. 뒤늦게 변수이름 바꾸기보단 Auto Car 잘 운행했던 코드 원본 유지
  
  ISR(INT4_vect) // , 5, 6
  {
      //if (TCNT2>0)
      //{
      ////sprintf(buffer, "[*]  %d ------- \n", TCNT2);
      ////USART1_SendString(buffer); // 원격 제어를 위해 핸드폰에 뜨게 함    static uint8_t edge_detected = 0;  // 상승 에지 감지 플래그
      //}
      
      static uint8_t edge_detected4 = 0;  // 상승 에지 감지 플래그
  
      // 폰출력 밀림
      //sprintf(buffer, "this %d %d %d \n", edge_detected,  PINE, current_sensor);
      //USART1_SendString(buffer); // 원격 제어를 위해 핸드폰에 뜨게 함    static uint8_t edge_detected = 0;  // 상승 에지 감지 플래그
      
      if( edge_detected4 == 0 &&  (PINE &(1<<4) ) && current_sensor==0 )
      {
          
          TCNT2=0;
          current_sensor=1; /
          edge_detected4 = 1;  
  
      }
      else if(edge_detected4 && !(PINE & (1<<4)) && current_sensor==1) 
      
      {
          ultrasonic_distance = 1000000.0 * TCNT2 * 1024 / F_CPU;
          distance1 = (ultrasonic_distance /58);  //50 - 4);
  
          edge_detected4 = 0;
          //sprintf(buffer, "[2] this %d %d %d distance:%d\n TCNT2%d\n", edge_detected4,  PINE, current_sensor,distance1, TCNT2);
          //USART1_SendString(buffer); // 원격 제어를 위해 핸드폰에 뜨게 함    static uint8_t edge_detected = 0;  // 상승 에지 감지 플래그
          
          current_sensor=2;
      }
  }
  
  ISR(INT5_vect) // , 5, 6
  {
      
      static uint8_t edge_detected5 = 0;  // 상승 에지 감지 플래그
      
      if( edge_detected5 == 0 &&  PINE &(1<<5) && current_sensor==2 ) 
      {
          TCNT2=0;
          current_sensor=3; 
          edge_detected5 = 1;  
          //sprintf(buffer, "[3] this %d %d %d \n", edge_detected5,  PINE, current_sensor);
          //USART1_SendString(buffer); // 원격 제어를 위해 핸드폰에 뜨게 함    static uint8_t edge_detected = 0;  // 상승 에지 감지 플래그
  
      }
      else if(edge_detected5 && !(PINE & (1<<5)) && current_sensor==3) 
      
      {
          ultrasonic_distance = 1000000.0 * TCNT2 * 1024 / F_CPU;
          distance2 = (ultrasonic_distance /58);  //50 - 4);
          edge_detected5 = 0;
          //sprintf(buffer, "[4] this %d %d %d distance:%d\n TCNT2%d\n", edge_detected5,  PINE, current_sensor,distance2, TCNT2);
          //USART1_SendString(buffer);// 원격 제어를 위해 핸드폰에 뜨게 함    static uint8_t edge_detected = 0;  // 상승 에지 감지 플래그
          current_sensor=4;
          
          //ultrasonic_trigger(2); 이런 방법이 코드로는 더 간단할 거 같아서 하려다가 맘. 복잡한 타이밍/인터럽트 안쌓이도록 인터럽트에서는 최대한 여기 내에서만 연산하게함
      }
  }
  
  
  ISR(INT6_vect) // , 5, 6
  {
      
      static uint8_t edge_detected6 = 0;  // 상승 에지 감지 플래그
      
      if( edge_detected6 == 0 &&  PINE &(1<<6) && current_sensor==4 ) 
      {
          TCNT2=0;
          current_sensor=5;
          edge_detected6 = 1; 
          //sprintf(buffer, "[5] this %d %d %d distance:%d\n TCNT2%d\n", edge_detected6,  PINE, current_sensor,distance2, TCNT2);
          //USART1_SendString(buffer); // 원격 제어를 위해 핸드폰에 뜨게 함    static uint8_t edge_detected = 0;  // 상승 에지 감지 플래그
  
      }
      else if(edge_detected6 && !(PINE & (1<<6)) && current_sensor==5) 
      
      {
          ultrasonic_distance = 1000000.0 * TCNT2 * 1024 / F_CPU;
          distance3 = (ultrasonic_distance /58);  
          edge_detected6 = 0;
          //sprintf(buffer, "[6] this %d %d %d distance:%d\n TCNT2%d\n", edge_detected6,  PINE, current_sensor,distance3, TCNT2);
          //USART1_SendString(buffer); // 원격 제어를 위해 핸드폰에 뜨게 함    static uint8_t edge_detected = 0;  // 상승 에지 감지 플래그
          current_sensor=0;
  
      }
  }
  
  void init_ultrasonic(void)
  {
      for (int i =0; i<3; i++)
      {
          DDRA|=1<<i; // TRIG 초음파 세개 //TRIG_DDR |= 1 << TRIG;
          DDRE &= ~(1 << (i+4) ); // ECHO // ECHO_DDR &= ~(1 << TRIG);   // input mode ECHO_DDR &= 0b1111"0"111;
      }
      
      
  
      EICRB |= (1<<ISC40) | (1<<ISC50) | (1<<ISC60);  
      EICRB &= ~((1<<ISC41)|(1<<ISC51)|(1<<ISC61));   
      
      TCCR2 |= (1<<CS22) | (1<<CS20); //TCCR1B = 1 << CS12 | 1 << CS10; 
      
      EIMSK |= (1<<INT4) | (1<<INT5) | (1<<INT6);
  
  }
  
  void ultrasonic_trigger(int PinNum)
  {
      PORTA |= 1 << PinNum; 
      _delay_us(15);
      PORTA &= ~(1 << PinNum);
  }
  
  
  
  
  int ultrasonic_get_distance(int PinNum)
  {
      static int did_sensor=-1;
      
      switch(current_sensor) 
      {
          case 0 :
          did_sensor =current_sensor;
          ultrasonic_trigger(0); 
          break;
          
          case 2 :
          did_sensor =current_sensor;
  
          ultrasonic_trigger(1);
          
          break;
  
          case 4 :
          did_sensor =current_sensor; 
          ultrasonic_trigger(2);
          
          break;
          default:
          break;
      }
      if      (PinNum == 0) return distance1; 
      else if (PinNum == 1) return  distance2;
      else if (PinNum == 2) return  distance3;
  }
  ```

- ultrasonic.h
  ```C
  /*
   * ultrasonic.h
   *
   * Created: 2025-03-12 오후 2:49:05
   *  Author: microsoft
   */ 
  
  
  #ifndef ULTRASONIC_H_
  #define ULTRASONIC_H_
  #define  F_CPU 16000000UL  // 16MHZ
  #include <avr/io.h>
  #include <util/delay.h>  // _delay_ms _delay_us
  #include <avr/interrupt.h>  // sei()등
  
  #define TRIG 4
  #define TRIG_PORT PORTG 
  #define TRIG_DDR  DDRG
  
  #define ECHO 4
  #define ECHO_PIN  PINE   // External INT 4
  #define ECHO_DDR  DDRE 
  #endif /* ULTRASONIC_H_ */  
  ```
  
- button.c
  ```C
   /*
   * button.c
   *
   * Created: 2025-03-05 오후 2:30:27
   *  Author: microsoft
   */ 
  #include "button.h"
  #include "led.h"
  #include "def.h"
  
  #include "extern.h" 
  
  
  void init_button(void);
  int get_button(int button_num, int button_pin);
  void auto_mode_check(void);      // button check;
                                                  
  void init_button(void)
  {
  	BUTTON_DDR &= ~( 1 << BUTTON0PIN );
  }
  // 예) 
  // BUTTON0 : S/W 번호  --> button_num
  // BUTTON0PIN : button_pin  
  // 리턴값 : 1 :  버튼을 눌렀다 떼면 1을 return 
  //          0 :  ide 
  
  int get_button(int button_num, int button_pin)
  {
  	static unsigned char button_status[BUTTON_NUMER] =
  	{BUTTON_RELEASE};	
  	// 	지역 변수에 static을 쓰면 전역 변수처럼 함수를 빠져 나갔다 다시 들어 와도 값을 유지 한다.  
  	int currtn_state;
  	
  	currtn_state = BUTTON_PIN & (1 << button_pin);   // 버튼을 읽는다. 
  	if (currtn_state && button_status[button_num] == BUTTON_RELEASE)  // 버튼이 처음 눌려진 noise high 
  	{
  		_delay_ms(60);   // noise가 지나가기를 기다린다. 
  		button_status[button_num] = BUTTON_PRESS;   // noise가 지나간 상태의 High 상태 
  		return 0;   // 아직은 완전히 눌렸다 떼어진 상태가 아니다. 
  	}
  	else if (currtn_state==BUTTON_RELEASE && button_status[button_num] == BUTTON_PRESS)
  	{
  		_delay_ms(60);
  		button_status[button_num] = BUTTON_RELEASE;   // 다음 버튼 체크를 위해서 초기화
  		return 1;   // 완전히 1번 눌렸다 떼어진 상태로 인정		
  	}
  	
  	return 0;   // 버튼이 open상태 
  }
  
  void auto_mode_check(void)      // button check
  {
  	static uint8_t button_state=0;
  	
  	if (get_button(BUTTON0, BUTTON0PIN))
  		button_state = !button_state;
  		
  	if (button_state)
  		AUTO_RUN_LED_PORT |= 1 << AUTO_RUN_LED_PIN;  // LED ON
  	else 
  	{
  		AUTO_RUN_LED_PORT &= ~(1 << AUTO_RUN_LED_PIN);  // LED OFF
  		stop();
  	}
  		
  	func_index=AUTO_MODE;
  }
  ```

- button.h
  ```C
    /*
   * button.h
   *
   * Created: 2025-03-05 오후 2:30:04
   *  Author: microsoft
   */ 
  
  
  #ifndef BUTTON_H_
  #define BUTTON_H_
  #define  F_CPU 16000000UL  // 16MHZ
  #include <avr/io.h>  // PORTA PORTB등의 I/O 관련 register등이 있다. 
  #include <util/delay.h>  // _delay_ms _delay_us
  
  #define LED_DDR DDRA   // 이렇게 하는 이유: LED PORT가 변경 되면
                         // #define만 바꿔주면 compiler가 변경 
  #define LED_PORT PORTA 
  #define BUTTON_DDR DDRC 
  #define BUTTON_PIN PINC    // PINC는 PORTC를 읽는 register 5v:1  0v:0 
  #define BUTTON0PIN  7    // PORTC.7
  
  #define BUTTON0   0   // PORTD.3의 가상의 index(S/W 번호)
  
  #define BUTTON_NUMER 1   // button갯수 
  #define BUTTON_PRESS  1   // 버튼을 누르면 high(active-high)
  #define BUTTON_RELEASE 0  // 버튼을 떼면 low
  #endif /* BUTTON_H_ */
  ```

- dht11.c
  + 내 파일에서 하나하나 복붙하고 주석 지워서 옮기는 것도 시간이 걸리므로 추후 옮김
  
#### Reference
허경용. (2016). ATmega128로 배우는 마이크로컨트롤러 프로그래밍. 제이펍.



