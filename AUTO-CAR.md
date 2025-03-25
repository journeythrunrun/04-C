> 제출했던 파일들에서 쓸데 없는 주석 조금 정리한 버전

> 배운 것을 베이스로 하여 코드를 짬. 해당 베이스적인 부분에 달린 주석은 다른 사람의 주석임.(예전에 올렸던 코드와 동일한 기능을 다른 사람이 다른 방법으로 풀이했던 부분도 있음. 빠르게 원하는 기능을 구현하기 위해 이전에 구현했던 기능이자 안쓰는 부분을 따로 정리하진 않음)
> 설명을 위한 코드라기보다 목표 성능을 제한 시간 내에 빠르게 완료하고자 한 코드임
  >> 주석 설명을 많이 하진 않음.
  >> 급하게 코드를 수정하거나 디버깅할 때(다른 방법으로 전환) 주석도 함께 수정하지 않아서 코드 라인과 주석의 내용이 일치하지 않는 것들 있음. 이전에 쓴 방법에서 필요했던 부분이라 가장 마지막으로 사용한 방법에서는 빼도 되는 변수 이런 것도 걍 하나하나 보면서 지우고 다시 모든 동작을 확인하지 않고 걍 냅뒀음(어차피 있어도 정상작동 될 부분은 빠르게 다른 방법 시도해볼 때 냅뒀었음). 시간 제한이 있기에 코드 업그레이드보다 목표 만족에 초점을 둠

> AUTO-CAR올렸다고 자율주행차 쪽에서 일하고 있는 건 아니고 진로는 약간 고민중 정규직 취업 전이라 

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
  ```C
  /*
   * dht11.c
   *
   * Created: 2025-03-18 오전 10:52:44
   *  Author: microsoft
   */ 
  
  #include "dht11.h"
  
  void init_dht11(void);
  void dht11_main(void);
  
  void init_dht11(void)
  {
  	DHT_DDR |= 1 << DHT_PIN_NUM;
  	DHT_PORT |= 1 << DHT_PIN_NUM;
  }
  
  void dht11_main(void)
  {
  	uint8_t data[5] = {0,};   // 5bytes(humi(2)+temp(2)+checksum(1)
  		
  	init_dht11();
  	
  	while(1)
  	{
  		memset(data,0,sizeof(data));
  		dht11_state=OK;
  		//======= step1 : start siginal ============
  		init_dht11();
  		_delay_ms(100);
  		
  		DHT_PORT &= ~(1 << DHT_PIN_NUM);  // DATA LOW 
  		_delay_ms(20);
  		
  		DHT_PORT |= 1 << DHT_PIN_NUM;  // DATA HIGH 
  		DHT_DDR &= ~(1 << DHT_PIN_NUM);  // input mode
  		_delay_us(1);
  		
  		// response check 
  		us_count=0;
  		while ( (DHT_PIN & 1 << DHT_PIN_NUM) >> DHT_PIN_NUM )   // 아직도 HIGH 인지 check
  		{
  			_delay_us(2);
  			us_count += 2;
  			if (us_count > 50)   // 50us만큼 기다렸는데도 HIGH이면 ERROR
  			{
  				dht11_state=TIMEOUT;
  				break;
  			}
  		}
  		//======= step2 : respose signal check ============
  		if (dht11_state == OK)   // 정상 상태면 DATA pin이 LOW이다. 
  		{
  			// response check
  			us_count=0;
  			while ( !((DHT_PIN & 1 << DHT_PIN_NUM) >> DHT_PIN_NUM) )   // LOW 일동안 반복
  			{
  				_delay_us(2);
  				us_count += 2;
  				if (us_count > 100)   // spec에는 80us인데 여유를 둬서 100us만큼 기다렸는데도 HIGH이면 ERROR
  				{
  					dht11_state=TIMEOUT;
  					break;
  				}
  			}			
  		}
  		// response HIGH check 
  		if (dht11_state == OK)   // 정상 상태면 DATA pin이 LOW이다.
  		{
  			// response check
  			us_count=0;
  			while ( (DHT_PIN & 1 << DHT_PIN_NUM) >> DHT_PIN_NUM )   // HIGH 일동안 반복
  			{
  				_delay_us(2);
  				us_count += 2;
  				if (us_count > 100)   // spec에는 80us인데 여유를 둬서 100us만큼 기다렸는데도 HIGH이면 ERROR
  				{
  					dht11_state=TIMEOUT;
  					break;
  				}
  			}
  		}
  		// state=OK이면 DATA LINE(PORTG.0)은 LOW상태이다. 
  		// start signal request & response는 정상적으로 끝났다. 
  		
  		
  		//========= step3: data bit 수신 from DHT11 ==============
  		// 이제부터 40개의 pulse를 count한다. (습H,습L,온H,온L,checksum) 8 8 8 8 8bit를 묶는 작업
  		// '0' --> 50us동안 LOW, 26~28us동안 HIGH
  		// '1' --> 50us동안 LOW, 70us동안 HIGH
  		if (dht11_state == OK)
  		{
  			uint8_t pulse[8] = {0,};  // 1개의 pulse를 담는 그릇(변수) 
  			
  			for (int i=0; i < 5; i++)   // 5 bytes 이니까 
  			{
  				for (int j=7; j >= 0; j--)  // byte당 8bit이므로 
  				{
  					// LOW 확인 50us check
  					// response check
  					us_count=0;
  					while ( !((DHT_PIN & 1 << DHT_PIN_NUM) >> DHT_PIN_NUM) )   // LOW 일동안 반복
  					{
  						_delay_us(2);
  						us_count += 2;
  						if (us_count > 70)   // spec에는 50us인데 여유를 둬서 70us만큼 기다렸는데도 HIGH이면 ERROR
  						{
  							dht11_state=TIMEOUT;
  							i=5;
  							j=-1;  // for문 전체 탈출
  							break;
  						}
  					}
  					
  					// 이시점에 정상이면 state=OK이고 전압은 HIGH(PG.0)	
  					if (dht11_state == OK)
  					{
  						us_count=0;
  						while ( (DHT_PIN & 1 << DHT_PIN_NUM) >> DHT_PIN_NUM )   // HIGH 일동안 반복
  						{
  							_delay_us(2);
  							us_count += 2;
  							if (us_count > 90)   // '0': HIGH길이: 26~28us '1': HIGH길이: 70us인데 이것보다 길게 90us동안 유지
  							{
  								dht11_state=TIMEOUT;
  								i=5;
  								j=-1;  // for문 전체 탈출
  								break;
  							}
  						}	
  						if (dht11_state == OK)
  						{
  							if (us_count < 40) // '0'
  								pulse[j] = 0;
  							else if (us_count > 40) // '1'
  								pulse[j] = 1;
  						}					
  					} 
  				}
  				if (dht11_state == OK)  // pulse를 8개를 정상 처리 했으면
  				{
  					data[i] = pulse[0] << 0 |  pulse[1] << 1 |  pulse[2] << 2 | pulse[3] << 3 |
  					pulse[4] << 4 | pulse[5] << 5 | pulse[6] << 6 | pulse[7] << 7;
  				}
  			}
  			// check-sum check
  			if (dht11_state == OK)
  			{
  				if (data[4] != data[0]+data[1]+data[2]+data[3])
  					dht11_state=VALUE_ERROR;
  			}
  			_delay_us(50);   // SPEC에는 50us인데 여유를 둬서 60us
  			// HIGH가 유지 되면 정상 전압: HIGH
  			us_count=0;
  			while ( (DHT_PIN & 1 << DHT_PIN_NUM) >> DHT_PIN_NUM )   // HIGH 일동안 반복
  			{
  				_delay_us(2);
  				us_count += 2;
  				if (us_count > 90)   // '0': HIGH길이: 26~28us '1': HIGH길이: 70us인데 이것보다 길게 90us동안 유지
  				{
  					break;
  				}
  			}
  			if (us_count < 90)  // 무엇가 data가 계속 발생 
  			{
  				dht11_state = TRANS_ERROR;
  			}
  		}
          // 값 출력----------
  		if (dht11_state == OK)
  		{
  			printf("temp: %d.%d\n", data[2],data[3]);
  			printf("humi: %d.%d\n", data[0],data[1]);
  		}	
  		if (dht11_state != OK)
  		{
  			printf("error code : %d\n", dht11_state);
  		}	
  		_delay_ms(1500);   // DHT11 안정화 시간
  	}
  }  
  ```

- dht11.h
  ```C
  /*
   * dht11.h
   *
   * Created: 2025-03-18 오전 10:52:28
   *  Author: microsoft
   */ 
  
  
  #ifndef DHT11_H_
  #define DHT11_H_
  #define  F_CPU 16000000UL  // 16MHZ
  #include <avr/io.h>  // PORTA PORTB등의 I/O 관련 register등이 있다.
  #include <util/delay.h>  // _delay_ms _delay_us
  
  #define DHT_DDR  DDRG
  #define DHT_PORT PORTG
  #define DHT_PIN  PING
  #define DHT_PIN_NUM 0
  
  uint8_t  us_count=0;
  
  enum state_define {OK, TIMEOUT, VALUE_ERROR, TRANS_ERROR};
  enum state_define dht11_state=OK;
  
  #endif /* DHT11_H_ */
  ```
  C내 파일에서 하나하나 복붙하고 주석 지워서 옮기는 것도 시간이 걸리므로 추후 옮김
 
- extern.h
  ```C
  /*
   * extern.h
   *
   * Created: 2025-03-20 오전 11:29:13
   *  Author: microsoft
   */ 
  
  
  #ifndef EXTERN_H_
  #define EXTERN_H_
  
  extern void init_timer1(void);
  extern void init_L298N(void);
  extern void stop(void);
  extern void forward(int speed);
  extern void backward(int speed);
  extern void turn_left(int speed);
  extern void turn_right(int speed);
  
  extern void init_button(void);
  extern int get_button(int button_num, int button_pin);
  
  extern void init_uart0(void);
  extern void UART0_transmit(uint8_t data);
  
  extern void distance_ultrasonic();
  extern void init_ultrasonic();
  
  extern void init_led(void);
  extern void auto_mode_check(void);      // button check;
  extern void init_uart1(void);
  
  
  extern volatile uint8_t rx_buff[80];   // uart0로 부터 들어온 문자를 저장 하는 버퍼(변수)
  extern volatile uint8_t rx_msg_received;
  extern volatile uint8_t bt_data;
  extern int func_index;
  
  #endif /* EXTERN_H_ */
  ```
- fnd.c
  ```C
  /*
   * fnd.c
   *
   * Created: 2025-03-06 오후 12:23:41
   *  Author: microsoft
   */ 
  #include "fnd.h"
  void init_fnd(void);
  int fnd_main(void);
  void fnd_display(void);
  
  uint32_t ms_count=0;  // ms를 재는 count변수 unsigned int --> uint32_t
  uint32_t sec_count=0;  // 초를 재는 count변수 unsigned int --> uint32_t
  
  int fnd_main(void)
  {
  	init_fnd();
  	
  	while (1)
  	{
  		fnd_display();
  		_delay_ms(1);
  		ms_count++;
  		if (ms_count >= 1000)   // 1000ms --> 1sec
  		{
  			ms_count=0;
  			sec_count++;
  		}
  	}
  	
  	return 0;
  }
  
  void fnd_display(void)
  {
  #if 0  // common 애노우드
                           // 0   1    2    3     4    5   6     7   8    9    .
  	uint8_t fnd_font[] = {0xc0,0xf9,0xa4,0xb0,0x99,0x92,0x82,0xd8,0x80,0x90,0x7f};
  #else  // common 캐소우드 
  						 // 0   1    2    3     4    5   6     7   8    9    .
      uint8_t fnd_font[] = {~0xc0,~0xf9,~0xa4,~0xb0,~0x99,~0x92,~0x82,~0xd8,~0x80,~0x90,~0x7f};
  #endif 
  
  	static int digit_select=0;  // static를 쓰면 전역 변수 처럼 함수가 빠져 나갔다가 다시 들어 오더라도 값을 유지
  	
  	switch(digit_select)
  	{
  		case 0:
  #if 0  // common 애노우드 
  			FND_DIGIT_PORT = 0x80;   // 10000000 FND_DIGIT_PORT = 0x80
  #else   // common 캐소우드 
  			FND_DIGIT_PORT = ~0x80;   // 011111111 FND_DIGIT_PORT = ~0x80
  #endif 
  			FND_DATA_PORT = fnd_font[sec_count % 10];   // 0~9초 
  			break;
  		case 1:
  #if 0  // common 애노우드
  			FND_DIGIT_PORT = 0x40;   // 10000000  
  #else   // common 캐소우드
  			FND_DIGIT_PORT = ~0x40;  
  #endif 
  			FND_DATA_PORT = fnd_font[sec_count / 10 % 6];   // 10단위초
  			break;
  		case 2:
  #if 0  // common 애노우드
  			FND_DIGIT_PORT =  0x20;      
  #else   // common 캐소우
  			FND_DIGIT_PORT =  ~0x20; // 011111111
  #endif
  			FND_DATA_PORT = fnd_font[sec_count / 60 % 10];   // 1단위 분 
  			break;
  		case 3:
  #if 0  // common 애노우드
  			FND_DIGIT_PORT = 0x10;   // 10000000
  #else   // common 캐소우드
  			FND_DIGIT_PORT = ~0x10;   // 011111111
  #endif
  			FND_DATA_PORT = fnd_font[sec_count / 600 % 6];   // 10단위 분 
  			break;
  	}
  	digit_select++;
  	digit_select %= 4;   // 다음 표시할 자리수 선택
  }
  
  void init_fnd(void)
  {
  	FND_DATA_DDR = 0xff;  // 출력 모드로 설정
  	// FND_DIGIT_DDR |= 0xf0;   // 자릿수 선택 7654 
  	FND_DIGIT_DDR |= 1 << FND_DIGIT_D1 | 1 << FND_DIGIT_D2 | 1 << FND_DIGIT_D3 
  					 | 1 << FND_DIGIT_D4; 
  	// fnd를 all off 
  #if 0  // common 애노우드 
  	FND_DATA_PORT = ~0x00;   // fnd를 all off  0xff;
  #else  // common 캐소우드
  	FND_DATA_PORT = 0x00;   // fnd를 all off   
  #endif 
  }
  ```
- fnd.h
  ```C
  /*
   * fnd.h
   *
   * Created: 2025-03-06 오후 12:23:18
   *  Author: microsoft
   */ 
  
  
  #ifndef FND_H_
  #define FND_H_
  #define  F_CPU 16000000UL  // 16MHZ
  #include <avr/io.h>
  #include <util/delay.h>  // _delay_ms _delay_us
  
  #define FND_DATA_PORT  PORTC
  #define FND_DATA_DDR   DDRC
  
  #define FND_DIGIT_PORT PORTB
  #define FND_DIGIT_DDR  DDRB
  #define FND_DIGIT_D1  4
  #define FND_DIGIT_D2  5
  #define FND_DIGIT_D3  6
  #define FND_DIGIT_D4  7
  
  #endif /* FND_H_ */
  ```

- I2C.c
  ```C
  /*
   * I2C.c
   *
   * Created: 2020-01-07 오후 7:58:00
   *  Author: kccistc
   */ 
  #include <avr/io.h>
  #include "I2C.h"
  
  void I2C_init(unsigned int baud){
    TWBR = baud;
  }
  
  void I2C_start(void)
  {
    TWCR = (1<<TWINT) | (1<<TWSTA) | (1<<TWEN);
    while (!(TWCR & (1<<TWINT)));  // 시작 완료 대기
  }
  
  void I2C_transmit(uint8_t data)
  {
    TWDR = data;
    TWCR = (1<<TWINT) | (1<<TWEN);
    while (!(TWCR & (1<<TWINT)));
  }
  
  void I2C_write_byte(uint8_t address, uint8_t data)
  {
    I2C_start();
    I2C_transmit(address);
    I2C_transmit(data);
    I2C_stop();
  }
  
  void I2C_stop(void)
  {
    TWCR = (1<<TWINT)|(1<<TWEN)| (1<<TWSTO);
  }
  
  uint8_t I2C_receive_ACK(void)
  {
    TWCR = (1<<TWINT) | (1<<TWEN) |(1<<TWEA);
    
    while( !(TWCR & (1<<TWINT)));             // 수신 완료 대기
    
    return TWDR;
  }
  
  uint8_t I2C_receive_NACK(void)
  {
    TWCR = (1<<TWINT) | (1<<TWEN);
    
    while( !(TWCR & (1<<TWINT)));             // 수신 완료 대기
    
    return TWDR;
  }
  ```

- I2C.h
  ```C
  /*
   * I2C.h
   *
   * Created: 2020-01-07 오후 7:57:06
   *  Author: kccistc
   */ 
  
  
  #ifndef I2C_H_
  #define I2C_H_
  void I2C_init(unsigned int baud);
  void I2C_start(void);
  void I2C_transmit(uint8_t data);
  void I2C_write_byte(uint8_t address, uint8_t data);
  void I2C_stop(void);
  uint8_t I2C_receive_ACK(void);
  uint8_t I2C_receive_NACK(void);
  
  #endif /* I2C_H_ */
  ```

- I2C_LCD.c
  ```C
  /*
   * I2C_LCD.c
   *
   * Created: 2020-01-07 오후 7:59:31
   *  Author: kccistc
   */ 
  #define F_CPU	16000000UL
  #include <avr/io.h>
  #include <util/delay.h>
  #include "I2C.h"
  #include "I2C_LCD.h"
  
  void I2C_LCD_Test();
  
  #define SLA_W (0x27<<1) //I2C LCD주소는 0x27 인데, <<1로 하는 이유는 wirite 모드를 유지하기 위함.
  
  void I2C_LCD_Test()
  {
  	I2C_LCD_init();
  
  #if 0
  	while(1)
  	{
  		 I2C_write_byte(SLA_W, '4');   // 자기전화번호 끝자리 2
  		 I2C_write_byte(SLA_W, '3');
  		 _delay_ms(100);
  	}
  #else  // org 
  	uint8_t toggle=0;
  	char sbuf[20];
  	int i=0;
  	
  	//while(1)
  	//{
  		//toggle = !toggle;
  		//i++;
  		//i %= 100;
  		//sprintf(sbuf, "%3d", i);
  		//I2C_LCD_clear();
  		//if (toggle)
  		//{
  			//I2C_LCD_write_string_XY(0,0,"Hello !!!"); //개행문자 쓰지마.
  		//}
  		//else
  		//{
  			//I2C_LCD_write_string_XY(1,0,"SIKWON "); //개행문자 쓰지마.
  		//}
  		////I2C_LCD_write_string_XY(0,10,sbuf); 100ms 단위로 카운팅증가
  		//
  		//_delay_ms(100); 
  	//}
      
  	//while(1)
  	//{	
  		I2C_LCD_write_string_XY(0,0,"Have a good day ~"); //개행문자 쓰지마.
  		I2C_LCD_write_string_XY(1,0,"`()/");
  		//_delay_ms(100); //프로토콜에 의해 실행되므로, 데이터를 다 받을때까지 기다려야한다.
  	//}
  #endif 
  }
  // 1byte를 write
  void I2C_LCD_write_data(uint8_t data)
  {
  	char data_u, data_l;
  	uint8_t data_t[4] = {0,};
  		
  	data_u = (data&0xf0);      // 상위 4bit 데이터
  	data_l = ((data<<4)&0xf0); // 하위 4bit 데이터
  	data_t[0] = data_u|0x0D;   //en=1, rs=1           |D7|D6|D5|D4|X|E|RW|RS|
  	data_t[1] = data_u|0x09;   //en=0, rs=1
  	data_t[2] = data_l|0x0D;   //en=1, rs=1
  	data_t[3] = data_l|0x09;   //en=0, rs=1
  
  	for(char i=0;i<4;i++){
  		I2C_write_byte(SLA_W, data_t[i]);
  	}
  }
  
  void I2C_LCD_write_command(uint8_t command)
  {
  	char data_u, data_l;
  	uint8_t data_t[4];
  	data_u = (command&0xf0);      // command의 상위 4bit 저장
  	data_l = ((command<<4)&0xf0); // command의 하위 4bit 저장
  	data_t[0] = data_u|0x0C;  //en=1, rs=0           |D7|D6|D5|D4|X|E|RW|RS|
  	data_t[1] = data_u|0x08;  //en=0, rs=0
  	data_t[2] = data_l|0x0C;  //en=1, rs=0
  	data_t[3] = data_l|0x08;  //en=0, rs=0
  	
  	for(char i=0;i<4;i++){
  		I2C_write_byte(SLA_W, data_t[i]);
  	}
  }
  
  // 화면 clear
  // 화면에 있는 내용만 지운다. 
  void I2C_LCD_clear(void)
  {
  	I2C_LCD_write_command(COMMAND_CLEAR_DISPLAY);
  	_delay_ms(2);
  }
  
  // LCD를 초기화
  void I2C_LCD_init(void)
  {
  	I2C_init(10000);
  	_delay_ms(50);
  	//Initialization of HD44780-based LCD (4-bit HW)
  	I2C_LCD_write_command(0x33);
  	I2C_LCD_write_command(0x32);
  	I2C_LCD_write_command(0x28);   //Function Set 4-bit mode
  	I2C_LCD_write_command(0x0c);   //Display On/Off Control
  	I2C_LCD_write_command(0x06);   //Entry mode set
  	I2C_LCD_write_command(0x01);   //Clear Display
  	//Minimum delay to wait before driving LCD module
  	_delay_ms(10);
  }
  // 현재의 xy좌표에 printf처럼 스트링 값을 출력 
  void I2C_LCD_write_string(char *string)
  {
  	uint8_t i;
  	for(i=0; string[i]; i++) //"hello !!\0" 마지막 널문자에서 조건 거짓이 되어 빠져나온다.
  		I2C_LCD_write_data(string[i]);
  }
  
  // 커서를 x,y좌표로 이동
  void I2C_LCD_goto_XY(uint8_t row, uint8_t col)
  {
  	col %= 16;
  	row %= 2;
  	
  	uint8_t address = (0x40 * row) + col;
  	uint8_t command = 0x80 + address;
  	
  	I2C_LCD_write_command(command);
  }
  
  // x,y좌표로 이동을 하고 string값을 출력 한다. 
  void I2C_LCD_write_string_XY(uint8_t row, uint8_t col, char *string)
  {
  	I2C_LCD_goto_XY(row, col);
  	I2C_LCD_write_string(string);
  }
  ```


- I2C_LCD.h
  ```C
  /*
   * I2C_LCD.h
   *
   * Created: 2020-01-07 오후 8:00:34
   *  Author: kccistc
   */ 
  
  
  #ifndef I2C_LCD_H_
  #define I2C_LCD_H_
  #define COMMAND_CLEAR_DISPLAY	0X01
  
  #define COMMAND_DISPLAY_ON_OFF_BIT	2
  #define COMMAND_CURSOR_ON_OFF_BIT	1
  #define COMMAND_BLINK_ON_OFF_BIT	0
  
  #define START 0x08
  #define SLA_W (0x27<<1)    // I2C LCD 주소 0x27 , <<1 이유는 write모드 유지
  #define SLA_R (0x27<<1 | 0x01)       // I2C LCD 주소 0x27 , Read모드 유지
  
  void I2C_LCD_init(void);
  void I2C_LCD_write_data(uint8_t data);
  void I2C_LCD_write_command(uint8_t command);
  void I2C_LCD_clear(void);
  void I2C_LCD_write_string(char *string);
  void I2C_LCD_goto_XY(uint8_t row, uint8_t col);
  void I2C_LCD_write_string_XY(uint8_t row, uint8_t col, char *string);
  
  
  #endif /* I2C_LCD_H_ */
  ```

- led.c
  ```C
  /*
   * led.c
   *
   * Created: 2025-03-05 오전 10:21:53
   *  Author: microsoft
   */ 
  
  #include "led.h"
  
  void init_led(void);
  
  void init_led(void)
  {
  	AUTO_RUN_LED_PORT_DDR |= 1 << AUTO_RUN_LED_PIN;
  }
  
  ```

- led.h
  ```C
  /*
   * led.h
   *
   * Created: 2025-03-05 오전 10:21:21
   *  Author: microsoft
   */ 
  
  #ifndef LED_H_
  #define LED_H_
  
  #include <avr/io.h>  // PORTA PORTB등의 I/O 관련 register등이 있다.
  
  #define AUTO_RUN_LED_PORT PORTG
  #define AUTO_RUN_LED_PORT_DDR DDRG
  #define AUTO_RUN_LED_PIN 3
  #endif /* LED_H_ */  
  ```

- pwm.c
  ```C
  /*
   * pwm.c
   *
   * Created: 2025-03-13 오후 12:49:06
   *  Author: microsoft
   */ 
  
  #include "pwm.h"
  #include "extern.h"
  #include "button.h"
  
  void init_timer1(void);
  void init_L298N(void);
  void stop(void);
  void forward(int speed);
  void backward(int speed);
  void turn_left(int speed);
  void turn_right(int speed);
  
  /*
     16bit timer 1번 활용
     PWM 출력 신호(2EA)
     ============
     PE5 : OC1A  왼쪽바퀴 
     PE6 : OC1B  오른쪽 바퀴 
     BTN0 : auto/manual
      
  	방향설정
  	========
  	1. LEFT MOTOR
  	  PF0 : IN1 (motor driver)
  	  PF1 : IN2
  	2. RIGHT MOTOR
  	  PF2 : IN3 (motor driver)
  	  PF3 : IN4 
  	  
  	  IN1/IN3   IN2/IN4
  	  ======    =======
  	    0         1    : 역회전
  		1         0    : 정회전
  		1         1    :  stop
  */
  void init_timer1(void)
  {
  	// 분주비 : 64 16000000 / 64 ---> 2500000Hz(250Khz)
  	// T=1/f = 1/2500000Hz ==> 0.000004sec(4us)
  	// 2500000Hz 에서 256개의 펄스를 count하면 소요시간 1.02ms
  	//                127개                           0.5ms
  	// P318 표 14-1
  	TCCR1B |= 1 << CS11 | 1 << CS10;   // 분주비 : 64
  	
  	// timer1 모드14번 고속 PWM (P327 표14-5)
  	TCCR1A |= 1 << WGM11;   // TOP을 ICR1에 지정을 할 수 있다. 
  	TCCR1B |= 1 << WGM13 | 1 << WGM12; 
  	
  	// 반전 모드 TOP : ICR1비교일치값(PWM)지정 : OCR1A, ORC1B 표15-7
  	// 비반전모드: 비교일치 발생시 OCn의 출력핀이 LOW로 바뀌고 BOTTOM에 HIGH바뀐다.
  	// P350 표15-7
  	TCCR1A |= 1 << COM1A1 | 1 << COM1B1;
  
  	ICR1 = 0x3ff;   // 1023  ==> 4ms TOP: PWM값 
  }
  
  
  /****
  	방향설정
  	========
  	1. LEFT MOTOR
  	PF0 : IN1 (motor driver)
  	PF1 : IN2
  	2. RIGHT MOTOR
  	PF2 : IN3 (motor driver)
  	PF3 : IN4
  	
  	IN1/IN3   IN2/IN4
  	======    =======
  	0         1    : 역회전
  	1         0    : 정회전
  	1         1    :  stop
  */
  void init_L298N(void)
  {
  	MOTOR_PWM_DDR |= 1 << MOTOR_LEFT_PORT_DDR | 1 << MOTOR_RIGHT_PORT_DDR;  // 출력 모드로 설정
  	MOTOR_DRIVER_DIRECTION_DDR |= 1 << 0 | 1 << 1 | 1 << 2 | 1 << 3;   // IN1 IN2 IN3 IN4 출력 
  	
  	MOTOR_DRIVER_DIRECTION_PORT &= ~(1 << 0 | 1 << 1 | 1 << 2 | 1 << 3);
  	MOTOR_DRIVER_DIRECTION_PORT |= 1 << 2 | 1 << 0;   // 자동차를 전진 모드로 설정
  }
  
  void stop(void)
  {
  	MOTOR_DRIVER_DIRECTION_PORT &= ~(1 << 0 | 1 << 1 | 1 << 2 | 1 << 3);
  	MOTOR_DRIVER_DIRECTION_PORT |= 1 << 0 | 1 << 1 | 1 << 2 | 1 << 3;   // 자동차를 stop 모드로 설정
  }
  
  void forward(int speed)
  {
  	MOTOR_DRIVER_DIRECTION_PORT &= ~(1 << 0 | 1 << 1 | 1 << 2 | 1 << 3);
  	MOTOR_DRIVER_DIRECTION_PORT |= 1 << 0 | 1 << 2;   // 전진 모드 
  	
  	OCR1A=OCR1B=speed;   // PB5(OCR1A): 왼쪽, PB6(OCR1B): 오른쪽
  }
  
  void backward(int speed)
  {
  	MOTOR_DRIVER_DIRECTION_PORT &= ~(1 << 0 | 1 << 1 | 1 << 2 | 1 << 3);
  	MOTOR_DRIVER_DIRECTION_PORT |= 1 << 3 | 1 << 1;   // 후진 모드 1010 
  	
  	OCR1A=OCR1B=speed;   // PB5(OCR1A): 왼쪽, PB6(OCR1B): 오른쪽
  }
  
  
  void turn_left(int speed)
  {
  	MOTOR_DRIVER_DIRECTION_PORT &= ~(1 << 0 | 1 << 1 | 1 << 2 | 1 << 3);
  	MOTOR_DRIVER_DIRECTION_PORT |= 1 << 0 | 1 << 2;   // 전진 모드 
  	
  	OCR1A=0;       // PB5(OCR1A): 왼쪽,  
  	OCR1B=speed;   //  PB6(OCR1B): 오른쪽
  }
  
  void turn_right(int speed)
  {
  	MOTOR_DRIVER_DIRECTION_PORT &= ~(1 << 0 | 1 << 1 | 1 << 2 | 1 << 3);
  	MOTOR_DRIVER_DIRECTION_PORT |= 1 << 0 | 1 << 2;   // 전진 모드
  	
  	OCR1A=speed;       // PB5(OCR1A): 왼쪽, 
  	OCR1B=0;           //  PB6(OCR1B): 오른쪽
  }
  
  ```
- pwm.h
  ```C
  /*
   * pwm.h
   *
   * Created: 2025-03-13 오후 12:48:49
   *  Author: microsoft
   */ 
  
  
  #ifndef PWM_H_
  #define PWM_H_
  #define  F_CPU 16000000UL  // 16MHZ
  #include <avr/io.h>
  #include <util/delay.h>  // _delay_ms _delay_us
  #include <avr/interrupt.h>  // sei()등
  #include <stdio.h>  // printf scanf fgets puts gets 등이 들어 있다.
  
  #define MOTOR_PWM_DDR  DDRB
  #define MOTOR_LEFT_PORT_DDR  5   // OC1A
  #define MOTOR_RIGHT_PORT_DDR 6   // OC1B
  
  #define MOTOR_DRIVER_DIRECTION_PORT  PORTF
  #define MOTOR_DRIVER_DIRECTION_DDR   DDRF
  
  #include "button.h"
  
  
  #endif /* PWM_H_ */
  ```
  
- Speaker.c
  ```C
  
  #define F_CPU 16000000L
  #include <avr/io.h>
  #include <util/delay.h>
  #include <avr/interrupt.h>
  
  #define DO_01   1911
  #define DO_01_H 1817
  #define RE_01   1703
  #define RE_01_H 1607
  #define MI_01   1517
  #define FA_01   1432
  #define FA_01_H 1352
  #define SO_01   1276
  #define SO_01_H 1199
  #define LA_01   1136
  #define LA_01_H 1073
  #define TI_01   1012
  #define DO_02   956
  #define DO_02_H 909
  #define RE_02   851
  #define RE_02_H 804
  #define MI_02   758
  #define FA_02   716
  #define FA_02_H 676
  #define SO_02   638
  #define SO_02_H 602
  #define LA_02   568
  #define LA_02_H 536
  #define TI_02   506
  #define DO_03   478
  #define DO_03_H 450
  #define RE_03	425
  #define RE_03_H 401
  #define MI_03	378
  
  #define F_CLK       16000000L //클럭
  #define F_SCALER	8 //프리스케일러
  #define BEAT_1_32	42
  #define BEAT_1_16	86
  #define BEAT_1_8	170
  #define BEAT_1_4	341
  #define BEAT_1_2	682
  #define BEAT_1		1364
  
  /*#define DO_01   (F_CLK/(2*16.3516*F_SCALER)-1)
  #define DO_01s  (F_CLK/(2*17.3239*F_SCALER)-1)
  #define RE_01   (F_CLK/(2*18.3541*F_SCALER)-1)
  #define RE_01s  (F_CLK/(2*19.4454*F_SCALER)-1)
  #define MI_01   (F_CLK/(2*20.6017*F_SCALER)-1)
  #define FA_01   (F_CLK/(2*21.8268*F_SCALER)-1)
  #define FA_01s  (F_CLK/(2*23.1247*F_SCALER)-1)
  #define SO_01   (F_CLK/(2*24.4997*F_SCALER)-1)
  #define SO_01s  (F_CLK/(2*25.9565*F_SCALER)-1)
  #define LA_01   (F_CLK/(2*27.5*F_SCALER)-1)
  #define LA_01s  (F_CLK/(2*29.1352*F_SCALER)-1)
  #define SI_01   (F_CLK/(2*30.8677*F_SCALER)-1)
  #define DO_02   (F_CLK/(2*32.7032*F_SCALER)-1)
  #define DO_02s  (F_CLK/(2*34.6478*F_SCALER)-1)
  #define RE_02   (F_CLK/(2*36.7081*F_SCALER)-1)
  #define RE_02s  (F_CLK/(2*38.8909*F_SCALER)-1)
  #define MI_02   (F_CLK/(2*41.2034*F_SCALER)-1)
  #define FA_02   (F_CLK/(2*43.6535*F_SCALER)-1)
  #define FA_02s  (F_CLK/(2*46.2493*F_SCALER)-1)
  #define SO_02   (F_CLK/(2*48.9994*F_SCALER)-1)
  #define SO_02s  (F_CLK/(2*51.913*F_SCALER)-1)
  #define LA_02   (F_CLK/(2*55*F_SCALER)-1)
  #define LA_02s  (F_CLK/(2*58.2705*F_SCALER)-1)
  #define SI_02   (F_CLK/(2*61.7354*F_SCALER)-1)
  #define DO_03   (F_CLK/(2*65.4064*F_SCALER)-1)
  #define DO_03s  (F_CLK/(2*69.2957*F_SCALER)-1)
  #define RE_03   (F_CLK/(2*73.4162*F_SCALER)-1)
  #define RE_03s  (F_CLK/(2*77.7817*F_SCALER)-1)
  #define MI_03   (F_CLK/(2*82.4069*F_SCALER)-1)
  #define FA_03   (F_CLK/(2*87.3071*F_SCALER)-1)
  #define FA_03s  (F_CLK/(2*92.4986*F_SCALER)-1)
  #define SO_03   (F_CLK/(2*97.9989*F_SCALER)-1)
  #define SO_03s  (F_CLK/(2*103.8262*F_SCALER)-1)
  #define LA_03   (F_CLK/(2*110*F_SCALER)-1)
  #define LA_03s  (F_CLK/(2*116.5409*F_SCALER)-1)
  #define SI_03   (F_CLK/(2*123.4708*F_SCALER)-1)
  #define DO_04   (F_CLK/(2*130.8128*F_SCALER)-1)
  #define DO_04s  (F_CLK/(2*138.5913*F_SCALER)-1)
  #define RE_04   (F_CLK/(2*146.8324*F_SCALER)-1)
  #define RE_04s  (F_CLK/(2*155.5635*F_SCALER)-1)
  #define MI_04   (F_CLK/(2*164.8138*F_SCALER)-1)
  #define FA_04   (F_CLK/(2*174.6141*F_SCALER)-1)
  #define FA_04s  (F_CLK/(2*184.9972*F_SCALER)-1)
  #define SO_04   (F_CLK/(2*195.9977*F_SCALER)-1)
  #define SO_04s  (F_CLK/(2*207.6523*F_SCALER)-1)
  #define LA_04   (F_CLK/(2*220*F_SCALER)-1)
  #define LA_04s  (F_CLK/(2*233.0819*F_SCALER)-1)
  #define SI_04   (F_CLK/(2*246.9417*F_SCALER)-1)
  #define DO_05   (F_CLK/(2*261.6256*F_SCALER)-1)
  #define DO_05s  (F_CLK/(2*277.1826*F_SCALER)-1)
  #define RE_05   (F_CLK/(2*293.6648*F_SCALER)-1)
  #define RE_05s  (F_CLK/(2*311.127*F_SCALER)-1)
  #define MI_05   (F_CLK/(2*329.6276*F_SCALER)-1)
  #define FA_05   (F_CLK/(2*349.2282*F_SCALER)-1)
  #define FA_05s  (F_CLK/(2*369.9944*F_SCALER)-1)
  #define SO_05   (F_CLK/(2*391.9954*F_SCALER)-1)
  #define SO_05s  (F_CLK/(2*415.3047*F_SCALER)-1)
  #define LA_05   (F_CLK/(2*440*F_SCALER)-1)
  #define LA_05s  (F_CLK/(2*466.1638*F_SCALER)-1)
  #define SI_05  (F_CLK/(2*493.8833*F_SCALER)-1)
  #define DO_06   (F_CLK/(2*523.2511*F_SCALER)-1)
  #define DO_06s  (F_CLK/(2*554.3653*F_SCALER)-1)
  #define RE_06   (F_CLK/(2*587.3295*F_SCALER)-1)
  #define RE_06s  (F_CLK/(2*622.254*F_SCALER)-1)
  #define MI_06   (F_CLK/(2*659.2551*F_SCALER)-1)
  #define FA_06   (F_CLK/(2*698.4565*F_SCALER)-1)
  #define FA_06s  (F_CLK/(2*739.9888*F_SCALER)-1)
  #define SO_06   (F_CLK/(2*783.9909*F_SCALER)-1)
  #define SO_06s  (F_CLK/(2*830.6094*F_SCALER)-1)
  #define LA_06   (F_CLK/(2*880*F_SCALER)-1)
  #define LA_06s  (F_CLK/(2*932.3275*F_SCALER)-1)
  #define SI_06   (F_CLK/(2*987.7666*F_SCALER)-1)
  #define DO_07   (F_CLK/(2*1046.502*F_SCALER)-1)
  #define DO_07s  (F_CLK/(2*1108.731*F_SCALER)-1)
  #define RE_07   (F_CLK/(2*1174.659*F_SCALER)-1)
  #define RE_07s  (F_CLK/(2*1244.508*F_SCALER)-1)
  #define MI_07   (F_CLK/(2*1318.51*F_SCALER)-1)
  #define FA_07   (F_CLK/(2*1396.913*F_SCALER)-1)
  #define FA_07s  (F_CLK/(2*1479.978*F_SCALER)-1)
  #define SO_07   (F_CLK/(2*1567.982*F_SCALER)-1)
  #define SO_07s  (F_CLK/(2*1661.219*F_SCALER)-1)
  #define LA_07   (F_CLK/(2*1760*F_SCALER)-1)
  #define LA_07s  (F_CLK/(2*1864.655*F_SCALER)-1)
  #define SI_07   (F_CLK/(2*1975.533*F_SCALER)-1)*/
  
  /* */
  // ddr port
  #define TIMER1_DDR         DDRE
  #define TIMER1_PORT         PORTB
  
  #define TIMER1_PIN3         3
  #define TIMER1_PIN4         4
  #define TIMER1_PIN5         5
  
  #define TIMER3_PIN3         3
  
  #define TIMER1_OCRA         OCR1A
  #define TIMER1_OCRB         OCR1B
  #define TIMER1_OCRC         OCR1C
  
  #define TIMER3_OCRA         OCR3A
  
  #define TIMER1_TCCRA      TCCR1A
  #define TIMER1_TCCRB      TCCR1B
  #define TIMER1_TCCRC      TCCR1C
  #define SPEAKER_TIMER_DDR      TIMER1_DDR
  #define SPEAKER_TIMER_PORT      TIMER1_PORT
  
  #define SPEAKER_TIMER_PIN      TIMER3_PIN3
  
  #define SPEAKER_TIMER_COUNT_PIN      TIMER1_PIN4
  
  // 비교 출력 레지스터
  #define SPEAKER_TIMER_OCRA      TIMER1_OCRA
  #define SPEAKER_TIMER_OCRB      TIMER1_OCRB
  #define SPEAKER_TIMER_OCRC      TIMER1_OCRC
  
  #define SPEAKER_TIMER_TCCRA      TIMER1_TCCRA
  #define SPEAKER_TIMER_TCCRB      TIMER1_TCCRB
  #define SPEAKER_TIMER_TCCRC      TIMER1_TCCRC
  
  
  
  int School_Bell_Tune[] = {SO_01, SO_01, LA_01, LA_01, SO_01, SO_01, MI_01, 
  					 	  SO_01, SO_01, MI_01, MI_01, RE_01, 
  					 	  SO_01, SO_01, LA_01, LA_01, SO_01, SO_01, MI_01, 
  					 	  SO_01, MI_01, RE_01, MI_01, DO_01,'/0'};
  const int School_Bell_Beats[] = {BEAT_1_4, BEAT_1_4, BEAT_1_4, BEAT_1_4, BEAT_1_4, BEAT_1_4, BEAT_1_2,
  						   BEAT_1_4, BEAT_1_4, BEAT_1_4, BEAT_1_4, BEAT_1,
  						   BEAT_1_4, BEAT_1_4, BEAT_1_4, BEAT_1_4, BEAT_1_4, BEAT_1_4, BEAT_1_2, 
  						   BEAT_1_4, BEAT_1_4, BEAT_1_4, BEAT_1_4, BEAT_1};
  						   
  // const int i = 0;
  // i =20; 안되 ..... 읽는 작업만 가능하고 write는 안되  
  // int j;
  // j = i;
  
  
  /*int He_Pirate[] = {RE_04, RE_04, RE_04, RE_04,
  	RE_04, RE_04, RE_04, LA_03, DO_04,
  	RE_04, RE_04, RE_04, MI_04,
  	FA_04, FA_04, FA_04, SO_04,
  	MI_04, MI_04, RE_04, DO_04,
  	DO_04, RE_04, 0, LA_03, DO_04,
  	RE_04, RE_04, RE_04, MI_04,
  	FA_04, FA_04, FA_04, SO_04,
  	MI_04, MI_04, RE_04, DO_04,
  	RE_04, 0, 0, LA_03, DO_04,
  	RE_04, RE_04, RE_04, FA_04,
  	SO_04, SO_04, SO_04, LA_04,
  	LA_04s, LA_04s, LA_04, SO_04,
  	
  	LA_04, RE_04, 0, RE_04, MI_04,
  	FA_04, FA_04, SO_04,
  	LA_04, RE_04, 0, RE_04, FA_04,
  	MI_04, MI_04, FA_04, RE_04,
  	MI_04, 0, 0, LA_04, DO_05,
  	RE_05, RE_05, RE_05, MI_05,
  	FA_05, FA_05, FA_05, SO_05,
  	MI_05, MI_05, RE_05, DO_05,
  	DO_05, RE_05, 0, LA_04, DO_05,
  	RE_05, RE_05, RE_05, MI_05,
  	FA_05, FA_05, FA_05, SO_05,
  	MI_05, MI_05, RE_05, DO_05,
  	RE_05, 0, 0, LA_04, DO_05,
  	RE_05, RE_05, RE_05, FA_05,
  	SO_05, SO_05, SO_05, LA_05,
  	LA_05s, LA_05s, LA_05, SO_05,
  	LA_05, RE_05, 0, RE_05, MI_05,
  	FA_05, FA_05, SO_05,
  	LA_05, RE_05, 0, RE_05, FA_05,
  	MI_05, MI_05, RE_05, DO_05,
  	RE_05, RE_05, MI_05,
  	
  	FA_05, FA_05, FA_05, SO_05,
  	LA_05, FA_05, 0, 0, FA_05, RE_05,
  	LA_04, 0, 0, 0,
  	LA_04s, 0, 0, SO_05, RE_05,
  	LA_04s, 0, 0, 0,
  	MI_05, MI_05, RE_05,
  	FA_04, 0, FA_04, SO_04,
  	LA_04, LA_04, LA_04,
  	LA_04s, LA_04,0 , 0,
  	SO_04, SO_04, SO_04,
  	SO_04, LA_04, 0, 0,
  	LA_04, LA_04, LA_04,
  	LA_04s, LA_04, 0, 0,
  	SO_04, FA_04, MI_04,
  	RE_04, 0, 0, RE_04, MI_04,
  	FA_04, SO_04, LA_04,
  	SO_04, FA_04, MI_04,
  	FA_04, SO_04, LA_04,
  	SO_04, 0, 0, FA_04, SO_04,
  	LA_04, 0, 0, SO_04, FA_04,
  	MI_04, FA_04, MI_04,
  	RE_04, 0, 0, MI_04, DO_04,
  	RE_04, 0, 0, RE_05, MI_05,
  	
  	FA_05, 0, 0, MI_05, FA_05,
  	SO_05, FA_05, SO_05,
  	LA_05, SO_05, FA_05,
  	RE_05, 0, 0, RE_05, MI_05,
  	FA_05, SO_05, LA_05,
  	LA_05s, RE_05, SO_05,
  	FA_05, 0, 0, SO_05, MI_05,
  	RE_05, 0, 0, MI_05, DO_05s,
  	LA_05, 0, 0, LA_05s, 0, 0,
  	LA_05, LA_05, LA_05,
  	LA_05, SO_05, 0, 0,
  	SO_05, 0, 0,
  	FA_05, 0, 0,
  	FA_05, SO_05, MI_05,
  	RE_05, RE_05, MI_05, FA_05,
  	LA_05, RE_05, MI_05, FA_05,
  	LA_05s, RE_05, MI_05, FA_05,
  	LA_05, LA_05, DO_06,
  	LA_05, SO_05, 0, 0,
  	SO_05, 0, 0,
  	FA_05, 0, 0,
  	FA_05, SO_05, MI_05,
  	RE_05, 0, 0,
  RE_04, '/0'};
  int He_Pirate1_Beat[] = {BEAT_1_4, BEAT_1_8, BEAT_1_4, BEAT_1_8,
  	BEAT_1_4, BEAT_1_8, BEAT_1_16, BEAT_1_16, BEAT_1_16,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	
  	BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	
  	BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4,
  	BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_4, BEAT_1_2,
  	BEAT_1_2, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	
  	BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_2, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_2, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_2, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_4,
  	BEAT_1_2, BEAT_1_8, BEAT_1_4,
  BEAT_1};*/
  
  const int Elise_Tune[] = {MI_02, RE_02_H, MI_02, RE_02_H, MI_02, TI_01, RE_02, DO_02, LA_01, 0,
  	DO_01, MI_01, LA_01, TI_01, 0, MI_01, SO_01_H, TI_01, DO_02, 0,
  	MI_01, MI_02, RE_02_H, MI_02, RE_02_H, MI_02, TI_01, RE_02, DO_02, LA_01, 0,
  	DO_01, MI_01, LA_01, TI_01, 0, MI_01, DO_02, TI_01, LA_01,
  	MI_02, RE_02_H, MI_02, RE_02_H, MI_02, TI_01, RE_02, DO_02, LA_01, 0,
  	DO_01, MI_01, LA_01, TI_01, 0, MI_01, SO_01_H, TI_01, DO_02, 0,
  	MI_01, MI_02, RE_02_H, MI_02, RE_02_H, MI_02, TI_01, RE_02, DO_02, LA_01, 0,
  	DO_01, MI_01, LA_01, TI_01, 0, MI_01, DO_02, TI_01, LA_01, 0,
  	TI_01, DO_02, RE_02, MI_02, 0, SO_01, FA_02, MI_02, RE_02, 0, FA_01, MI_02, RE_02, DO_02, 0,
  	MI_01, RE_02, DO_02, TI_01, 0, MI_01, MI_02, MI_03,
  	RE_02_H, MI_02, RE_02_H, MI_02, RE_02, MI_02, RE_01_H, MI_02, TI_01, RE_01, DO_02, LA_01, 0,
  	DO_01, MI_01, LA_01, TI_01, 0, MI_01, SO_01_H, TI_01, DO_02, 0,
  	MI_01, MI_02, RE_02_H, MI_02, RE_02_H, MI_02, TI_01, RE_02, DO_02, LA_01, 0, DO_01, MI_01, LA_01, TI_01, 0,
  RE_01, DO_02, TI_01, LA_01,'/0'};
  
  const int Elise_Beats[] = 
     { BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_2,
  	BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4,
  	BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_2, BEAT_1_8,
  	BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8,
  	BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  	BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8,
  BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8};
  
  int GrandFa_Tune[] = {SO_01, DO_02, TI_01, DO_02, RE_02, DO_02, RE_02, MI_02, FA_02, MI_02, LA_01, RE_02, RE_02, DO_02, DO_02, DO_02, TI_01, LA_01, TI_01, DO_02, 0,
  	SO_01, DO_02, TI_01, DO_02, RE_02, DO_02, RE_02, MI_02, FA_02, MI_02, LA_01, RE_02, RE_02, DO_02, DO_02, DO_02, TI_01, LA_01, TI_01, DO_02, 0,
  	DO_02, MI_02, SO_02, MI_02, RE_02, DO_02, TI_01, DO_02, RE_02, DO_02, TI_01, LA_01, SO_01, DO_02, MI_02, SO_02, MI_02, RE_02, DO_02, TI_01, DO_02, RE_02,
  SO_01, DO_02, RE_02, MI_02, FA_02, MI_02, LA_01, RE_02, RE_02, DO_02, DO_02, DO_02, TI_01, LA_01, TI_01, DO_02, '/0'};
  
  const int GrandFa_Beats[] = {BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_2, BEAT_1_4,
  	BEAT_1_4, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_2, BEAT_1_4,
  	BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4,
  BEAT_1_4, BEAT_1_2, BEAT_1_2, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_4, BEAT_1_8, BEAT_1_8, BEAT_1_2};
  
  void perform(int);
  
  
  void perform(int status) // 8가지 -> 다른 곳에서도 extern
  {
      //switch(status)
      //{
          //case 1 : // 도 #~
              ////if(msec_count % ((1906*4*2)/1000 ) )
              ////OCR3A = 1136;  
              ////
              ////if((msec_count+(1906*4)/1000 )  %( (1906*4*2)/1000 ) )// 상수 추가
              ////OCR3A = 0;            
  //
      //}
      
  }
  void Music_Player(int *tone, int *Beats)
  {
  	while(*tone != '/0')
  	{
  		OCR3A = *tone;
  		delay_ms(*Beats);
  		tone++;
  		Beats++;
  		OCR3A = 0;
  		_delay_ms(10);		
  	}
  	return;
  }  
  // Timer3 위상교정 PWM
  //void init_speaker(void)
  //{
  	//DDRE |= 0x08;   // PWM CHANNEL  OC3A(PE3) 출력 모드로 설정 한다. 
  	//TCCR3A = (1<<COM3A0); // COM3A0 : 비교일치시 PE3 출력 반전 (P328 표14-6 참고)
  	//TCCR3B = (1<<WGM32) | (1<<CS31);   // WGM32 : CTC 4(P327 표14-5) CS31: 8분주(P318)
  	//// CTC mode : 비교일치가 되면 카운터는 reset되면서 PWM 파형 출력 핀의 출력이 반전 됨. 
  	//// 정상모드와 CTC모드의 차이점은 비교일치 발생시 TCNT1의 레지스터값을 0으로 설정 하는지 여부 이다. 
  	//// 정상모드를 사용시 TCNT1이 자동으로 0으로 설정 되지 않아 인터럽트 루틴에서 TCNT1을 0으로 설정 해 주었다. 
  	//// 위상교정 PWM mode4  CTC 분주비 8  16000000/8 ==> 2,000,000HZ(2000KHZ) : 
  	//// up-dounting: 비교일치시 LOW, down-counting시 HIGH출력
  	//// 1/2000000 ==> 0.0000005sec (0.5us)
  	//// P599 TOP 값 계산 참고
  	//// PWM주파수 = OSC(16M) / ( 2(up.down)x N(분주율)x(1+TOP) ) 
  	//// TOP =  (fOSC(16M) /  2(up.down)x N(분주율)x 원하는주파수 )) -1 
  	////-----------------------------------------------------------
  	//// - BOTTOM :  카운터가 0x00/0x0000 일때를 가리킨다.
      //// - MAX : 카운터가 0xFF/0xFFFF 일 때를 가리킨다.
      //// - TOP?:  카운터가 가질 수 있는 최대값을 가리킨다. 오버플로우 인터럽트의 경우 TOP은 0xFF/0xFFFF
      ////          이지만 비교일치 인터럽트의 경우 사용자가 설정한 값이 된다. 
      //
  	//TCCR3C = 0;  // P328 그림 14-11 참고 
  	//OCR3A = 0;   // 비교 일치값을 OCR3A에 넣는다. 
  	//
  	//return;
  //}
  void init_speaker_pwm(void);
  void init_speaker_pwm() {
      
      // (0) 출력설정
      DDRE|=1<<3;//PE3(타이머3) /// SPEAKER_TIMER_DDR |= 1 << SPEAKER_TIMER_PIN; 도 변경해놨긴한데 디버깅가독성을 위해  다완성후에야 저렇게하자 ////SPEAKER_TIMER_DDR |= 1 << SPEAKER_TIMER_PIN;  // pb5 출력 OC1a // DDRB
      
      
      // (1) 타이머 설정
      // 비교일치 발생 시 출력 반전 (Toggle on Compare Match)
      //  (P328 표14-6 참고)
      TCCR3A = (1 << COM3A0); ////타이머1->3으로 변경 //// SPEAKER_TIMER_TCCRA = (1 << COM1A0);  //TIMER1_TCCRA   TCCR1A    <<6
      // TCCR
      
      
      // 아래 둘 정확히 알아야해 TDOO
      // TODO
      // 여기 잘 이해 못함.. 사운드를
      
      
      // CTC 모드 설정 (WGM12) (P327 표14-5)
      TCCR3B|=1<<3;// 저 위치의 포트 모터드라이브에 연결돼있긴한데 설정용이라 괜찮은건가//SPEAKER_TIMER_TCCRB |= (1 << WGM12); // TIMER1_TCCRB      TCCR1B // 3
      
      // 분주비 8 설정 (CS11)
      TCCR3B|=1<<1;////SPEAKER_TIMER_TCCRB |= (1 << CS11); //                          // 1
      
      
      // CTC mode : 비교일치가 되면 카운터는 reset되면서 PWM 파형 출력 핀의 출력이 반전 됨.
      // 정상모드와 CTC모드의 차이점은 비교일치 발생시 TCNT1의 레지스터값을 0으로 설정 하는지 여부 이다.
      // 정상모드를 사용시 TCNT1이 자동으로 0으로 설정 되지 않아 인터럽트 루틴에서 TCNT1을 0으로 설정 해 주었다.
      // 위상교정 PWM mode4  CTC 분주비 8  16000000/8 ==> 2,000,000HZ(2000KHZ) :
      // up-dounting: 비교일치시 LOW, down-counting시 HIGH출력
      // 1/2000000 ==> 0.0000005sec (0.5us)
      // P599 TOP 값 계산 참고
      // PWM주파수 = OSC(16M) / ( 2(up.down)x N(분주율)x(1+TOP) )
      // TOP =  (fOSC(16M) /  2(up.down)x N(분주율)x 원하는주파수 )) -1
      //-----------------------------------------------------------
      // - BOTTOM :  카운터가 0x00/0x0000 일때를 가리킨다.
      // - MAX : 카운터가 0xFF/0xFFFF 일 때를 가리킨다.
      // - TOP?:  카운터가 가질 수 있는 최대값을 가리킨다. 오버플로우 인터럽트의 경우 TOP은 0xFF/0xFFFF
      //          이지만 비교일치 인터럽트의 경우 사용자가 설정한 값이 된다.
      
      TCCR3C=0; //SPEAKER_TIMER_TCCRC = 0;  // P328 그림 14-11 참고 //             __C      TCCR1C
      
      
      OCR3A=0; ////SPEAKER_TIMER_OCRA = 0;   // 비교 일치값을 OCR3A에 넣음 // TIMER1_OCRA         OCR1A
  }
  
  
  void Beep(int repeat)
  {
      int  i;
      
      // for delay로 해버리면 여기에 갇힘
      for(i=0; i < repeat; i++)
      {
          OCR3A=500; //SPEAKER_TIMER_OCRA = 500;  // 0.00025sec (250us) : 0.0000005 * 500
          //OCR1A= 500으로 하면 왼쪽 바퀴만 움직임ㅣ
          
          _delay_ms(200);
          OCR3A=0;//SPEAKER_TIMER_OCRA = 0;
          _delay_ms(200);
      }
  }
  
  void BeepShort(int repeat)
  {
      int  i;
      
      // for delay로 해버리면 여기에 갇힘
      //for(i=0; i < repeat; i++)
      //{
          OCR3A=500; //SPEAKER_TIMER_OCRA = 500;  // 0.00025sec (250us) : 0.0000005 * 500
          //OCR1A= 500으로 하면 왼쪽 바퀴만 움직임ㅣ
          
          _delay_ms(100);
          //OCR3A=0;// 끄기도.  아래 값0임. for없앴으니 0을 다른 거 앞에?
          //OCR3A=0;//SPEAKER_TIMER_OCRA = 0;
          //_delay_ms(200);
      //}
  }
  
  void Siren(int repeat)
  {
  	int i, j;
  	
  	OCR3A = 900;
  	
  	for(i=0; i < repeat; i++)
  	{
  		for(j=0; j < 100; j++)
  		{
  			OCR3A += 10;
  			_delay_ms(20);  // msec_count 
  		}
  		for(j=0; j < 100; j++)
  		{
  			OCR3A -= 10;
  			_delay_ms(20);
  		}
  	}
  }
  
  void RRR(void)
  {
  	int i;
  	
  	for(i=0; i<20; i++)
  	{
  		OCR3A = 1136;
  		_delay_ms(100);
  		OCR3A = 0;
  		_delay_ms(20);
  	}
  }
  
  void delay_ms(int ms)
  {
  	while(ms-- != 0)_delay_ms(1);
  }
  ```
- uart0.c
  ```C
  /*
   * uart0.c
   *
   * Created: 2025-03-10 오전 10:32:31
   *  Author: microsoft
   */ 
  #include "uart0.h"
  #include <string.h>   // strcpy strncmp stcmp....
  
  void init_uart0(void);
  void UART0_transmit(uint8_t data);
  
  /*
     PC comportmaster로 부터 1byte기 들어 올때 마다 이속(ISR(USART0_RX_vect)으로 들어 온다. (RX INT)
     예)  led_all_on\n ==> 11번 이곳으로 들어 온다 
          led_all_off\n
  */
  volatile uint8_t rx_msg_received=0;
  ISR(USART0_RX_vect)
  {
  	volatile uint8_t rx_data;
  	volatile static int i=0;
  	
  	rx_data = UDR0;  // uart0의 H/W register(UDR0)로 부터 1byte를 읽어 들인다. 
  	                 // rx_data = UDR0;를 실행하면 UDR0의 내용이 빈다.(empty)
  	if (rx_data == '\n' )
  	{
  		rx_buff[rear++][i] = '\0';
  		rear %= COMMAND_NUMBER;  // rear : 0~9
  		i=0;   // 다음 string을 저장하기 위한 1차원 index값을 0으로 
  		// !!!! rx_buff queue full check 하는 logic 추가 
  	}
  	else
  	{
  		rx_buff[rear][i++] = rx_data;
  		// COMMAND_LENGTH 를 check하는 logic 추가 
  	}
  }
  /*
    1. 전송속도: 9600bps : 총글자수 :9600/10bit ==> 960자 
       ( 1글자를 송.수신 하는데 소요 시간 : 약 1ms)
    2. 비동기 방식(start/stop부호 비트로 데이터를 구분)
    3. RX(수신) : 인터럽트 방식으로 구현 
  */
  
  void init_uart0(void)
  {
      
  	// 1. 9600bps로 설정
  	UBRR0H = 0x00;
  	UBRR0L = 207;  // 9600bps P219 표9-9
  	// 2. 2배속 통신  표9-1
  	UCSR0A |= 1 << U2X0;  // 2배속 통신 
  	UCSR0C |= 0x06;   // 비동기/data8bits/none parity
  	// P215 표9-1
  	// RXEN0 : UART0로 부터 수신이 가능 하도록 
  	// TXEN0 : UART0로 부터 송신이 가능 하도록 
  	// RXCIE0 : UART0로 부터 1byte가 들어오면(stop bit가 들어 오면)) rx interrupt를 발생 시켜라
  	UCSR0B |= 1 << RXEN0 | 1 << TXEN0 | 1 << RXCIE0;
  	
  }
  
  // UART0로 1byte를 전송 하는 함수 (polling방식)
  void UART0_transmit(uint8_t data)
  {
  	// 데이터 전송 중이면 전송이 끝날떄 까지 기다린다. 
  	while ( !(UCSR0A & 1 << UDRE0))
  		;   // no operation
  	UDR0 = data;  // data를 H/W전송 register에 쏜다. 
  }
  
  void pc_command_processing(void)
  {
  	if (front != rear)   // rx_buff에 data가 존재 
  	{
  		printf("%s\n", rx_buff[front]);  // &rx_buff[fron][0]
  		if (strncmp(rx_buff[front], "led_all_on",strlen("led_all_on")) == NULL)
  		{
  			printf("find: led_all_on\n");
  		}
  		front++;
  		front %= COMMAND_NUMBER;
  		// !!!! queue full check하는 logic들어 가야 한다. !!!!!
  	}
  }
  ```
- uart0.h
  ```C
  /*
   * uart0.h
   *
   * Created: 2025-03-10 오전 10:32:07
   *  Author: microsoft
   */ 
  
  
  #ifndef UART0_H_
  #define UART0_H_
  #define  F_CPU 16000000UL  // 16MHZ
  #include <avr/io.h>
  #include <util/delay.h>  // _delay_ms _delay_us
  #include <avr/interrupt.h>  // sei()등
  #include "def.h"
  // led_all_on\n
  // led_all_off\n
  volatile uint8_t rx_buff[COMMAND_NUMBER][COMMAND_LENGTH];   // uart0로 부터 들어온 문자를 저장 하는 버퍼(변수)
  volatile int rear=0;   // input index: USART0_RX_vect에서 집어 넣어 주는 index
  volatile int front=0;  // ouput index 
  
  
  #endif /* UART0_H_ */
  ```
  
- uart1.c
  ```C
  /*
   * uart1.c
   *
   * Created: 2025-03-20 오전 11:40:10
   *  Author: microsoft
   */ 
  /*
   * uart1.c
   *
   * Created: 2025-03-10 오전 10:32:31
   *  Author: microsoft
   */ 
  #include "uart1.h"
  #include "extern.h"
  
  void init_uart1(void);
  /*
     BT로 부터 1byte기 들어 올때 마다 ISR(USART1_RX_vect)으로 들어 온다. (RX INT)
  */
  volatile uint8_t bt_data=0;
  ISR(USART1_RX_vect)
  {
  	
  	bt_data = UDR1;  // BT로 부터 들어온 HW reg(UDR1)을 1byte 읽어 들인다.  
  	                 // bt_data = UDR1;를 실행하면 UDR1의 내용이 빈다.(empty)\
  					 
  	UART0_transmit(bt_data);   // BT로 부터 들어온 문자가 어떤것인지 확인 하기 위해서 comport로 출력
  }
  /*
    1. 전송속도: 9600bps : 총글자수 :9600/10bit ==> 960자 
       ( 1글자를 송.수신 하는데 소요 시간 : 약 1ms)
    2. 비동기 방식(start/stop부호 비트로 데이터를 구분)
    3. RX(수신) : 인터럽트 방식으로 구현 
  */
  
  void init_uart1(void)
  {
  	// 1. 9600bps로 설정
  	UBRR1H = 0x00;
  	UBRR1L = 207;  // 9600bps P219 표9-9
  	// 2. 2배속 통신  표9-1
  	UCSR1A |= 1 << U2X1;  // 2배속 통신 
  	
      UCSR1C = (1<<UCSZ11)|(1<<UCSZ10);  // 8 데이터 비트, 노 패리티, 1 스톱 비트 설정
      //UCSR1C |= 0x06;   // 비동기/data8bits/none parity
  	
      // P215 표9-1
  	// RXEN1 : UART1로 부터 수신이 가능 하도록 
  	// TXEN1 : UART1로 부터 송신이 가능 하도록 
  	// RXCIE1 : UART1로 부터 1byte가 들어오면(stop bit가 들어 오면)) rx interrupt를 발생 시켜라
  	
      UCSR1B = (1<<RXEN1)|(1<<TXEN1)|(1<<RXCIE1);  // 수신 및 송신 가능, 수신 완료 인터럽트 활성화
      //UCSR1B |= 1 << RXEN1 | 1 << TXEN1 | 1 << RXCIE1;
  	
  }

  ```

- uart1.h
  ```C
  /*
   * uart1.h
   *
   * Created: 2025-03-20 오전 11:39:49
   *  Author: microsoft
   */ 
  
  
  #ifndef UART1_H_
  #define UART1_H_
  #define  F_CPU 16000000UL  // 16MHZ
  #include <avr/io.h>
  #include <util/delay.h>  // _delay_ms _delay_us
  #include <avr/interrupt.h>  // sei()등
  
  
  
  #endif /* UART1_H_ */
  ```


#### Reference
허경용. (2016). ATmega128로 배우는 마이크로컨트롤러 프로그래밍. 제이펍.



