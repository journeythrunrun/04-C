> 시간 나면 [2503에 임시로 써놓은 md]업데이트 예정

> 기능을 추가 및 변경하며 사용한 것으로, 사용하지 않은 변수도 있음

### 1.1 버튼 4개로 LED제어 with bit 연산
- LED, Button, FSM
  + main.c
    ```C
    // 문제
    // delay 50 ms 화 ( 버튼을 위해 )
    // 1. Button 0 : led_all_on <-> _off
    // 2. Button 1 : Shift_left_ledon <->
    // 3. button 2 : shift_left_keeP_ledon
    // 4.		   : flower_on <->
    
    // 7segment 결선 : PORTB 4567(D1~D4)   PORTC 0~7(8개 다)
    
    #define F_CPU 16000000UL // Unsigned long 16MHz
    #include <avr/io.h> // PORTA PORTB 등의 IO관련 레지스터 등이 있다.
    #include <util/delay.h> // _delay_ms _delay_us
    
    #include "button.h" // button.c에서 include해놨지만, main 함수 내에서 관련된 게 쓰이면 여기서도 미리 include해야함
    extern int led_main(void); // 외부 파일의 함수
    
    extern void led_all_on(void);
    extern void led_all_off(void);
    extern void shift_left_ledon(void);
    extern void shift_right_ledon(void);
    extern void shift_left_keep_ledon(void);
    extern void shift_right_keep_ledon(void);
    extern void flower_on(void);
    extern void flower_off(void);
    
    extern void init_button(void);
    extern int get_button(int button_num, int button_pin);
    
    // none OS 또는 loop monitor 방식
    int main(void) // VSC((일반 상황))와 다르게 반환형이 int여야함
    {	
    	// DDR
    	// 레지스터이기에 형식 선언 안 해도 됨. 양수 8bit
    	// 상위 nibble (4bits), 하위 nibble
    	
    	//led_main();
    	//// 아스키코드 0(0x30), space(0x20), A(0x41), a(0x61)
    	//DDRA=0b11111111; //DDR(Data Direction Register) : 방향설정// 1(출력) 0(입력) //0b, 0x, //// 출력 모드
    	//DDRD=0x00;
    	
    	//while (1)
    	//{ 
    		//PORTA=0b11111111; //0xff; //// All on
    		//_delay_ms(1000);
    		//PORTA=0b00000000; // All off
    		//_delay_ms(1000);
    	//}
    	
    	init_button(); // DDRD 입력 설정
    	DDRA=0b11111111;
    	// FSM [Finite State machine] 
    	int button0_state=0;
    	while(1)
    	{
    		//// 1 button 처리(toggle)
    		//// button0을 1번 누르면 led_all_on, 또 누르면 _off
    		if ( get_button(BUTTON0, BUTTON0PIN))
    		{
    			////버튼 한 가지만 사용할 시 : button0_state =!button0_state; 
    			
    			// 1위치만 반전 
    			button0_state ^=0b00000001; // (2-not 등과 무한 응용가능) ^ - with 0(상대값유지),1(반전)
    			//// 번갈아 나와야하기에 다른 스위치를 idle상태로 바꾸진 않았음. 
    			// 다른 스위치의 마지막 상태(다른 bit위치의 값은)는 유지
    
    			// (3-not 등과 무한 응용가능) | - with 0(상대값유지),1(1화) ////물론 NAND NOR 등이 더 간단한 건 있지만 코드짜기 쉬운 것도 어느정도는 반영
    			if((button0_state| 0b11111110 ) ==0xff )//인덱스0위치가 1인지(다른 인덱스위치 무슨값이든 괜찮) // <<7bit가 0x80인지 확인하는 방법도 있음. 
    			//// 0인덱스 아닌 곳들은 어차피 대응하게 지정한 값이랑 비교해줄거라 다른 0화 기호써도 상관없었긴함
    			////버튼 한 가지만 사용할 시 : button0_state==1) 
    				led_all_on();
    			else led_all_off();
    		} 
    		
    		else if ( get_button(BUTTON1, BUTTON1PIN))
    		{
    			button0_state ^=0b00000010;
    			if( (button0_state| 0b11111101 ) ==0xff  )
    				shift_left_ledon();
    			else shift_right_ledon();
    		}
    		
    		else if ( get_button(BUTTON2, BUTTON2PIN))
    		{
    			button0_state ^=0b00000100;
    			if( (button0_state| 0b11111011 ) ==0xff  ) 
    				shift_left_keep_ledon();
    			else shift_right_keep_ledon();
    		}
    
    		else if ( get_button(BUTTON3, BUTTON3PIN))
    		{
    			button0_state ^=0b00001000;
    			if( (button0_state| 0b11110111 ) ==0xff  )
    				flower_on();
    			else flower_off();
    		}
    	}
    return 0;
    }
    ```

  + button.c
    ```C
    #include "button.h"
    void init_button(void);
    int get_button(int button_num, int button_pin);
    
    void init_button(void){
    	// - 버튼 초기화 입력
    	//   + M1. DDRD &= 0x87 
    	//     + (1-not 등과 무한 응용가능) & - with 0(0화),1(상대값유지) : DDRD값이 뭐였든 타겟비트 0화 //// .3.4.5.6을 input으로 설정한다.
    	// 1000 0111 
    	// (1) 가독성이 떨어진다 (2) port 변경 시 프로그램 수정이 어렵다
    
    	//   + M2. DDRD & DDRD : 확장성 좋음 & <<kbit shift | 좋음.
    	// & (10000111)
    	BUTTON_DDR &= ~(1<<BUTTON0PIN | 1<< BUTTON1PIN | 1<<BUTTON2PIN | 1<<BUTTON3PIN); //2**BUTTON0PIN | -> 쓰려는 핀만 1. 
        // <- #define BUTTON_DDR DDRD
    	// DDRD는 레지스터이기에 따로 값을 지정해준 적 없어도 값 있음
    }
     
    // BUTTON0 : SW번호
    // BUTTON0PIN : button pin
    // 리턴값 : 1(버튼을 눌렀다 떼면), 0(idle)
    // 버튼을 눌렀을 때 값이 흔들려서 의도와 다르게 여러 입력으로 받지 않고 한 번으로 받는 코드(60ms간격 앞뒤). 전에 했던 hw느낌 디바운싱에서는 눌린 값이 지속되는 것도 체크했었는데 그건 누르지 않았을 때의 순간적인 1노이즈가 잘못 포착되는 걸 방지 &좀더 하드웨어라서 차이가 있을 수 있음. GPT는 샘플링을 늘리는 방법도 있다함
    
    int get_button(int button_num, int button_pin)
    {
    	// static으로 이 함수의 해당 변수 값 유지 ////함수 빠져나갔다가 다시 들어와도 유지됨
    	static unsigned char button_status[]=
    	{BUTTON_RELEASE, BUTTON_RELEASE, BUTTON_RELEASE, BUTTON_RELEASE}; 
    		
    	int current_state;  
    	current_state = BUTTON_PIN &(1<<button_pin); // 버튼을 읽어온 것(노이즈 포함됨) & 타겟비트(버튼이기에 일반 숫자를 원핫벡터화)
    	// 타겟버튼 입력값만 원핫 벡터화
    	if (current_state && button_status[button_num]==BUTTON_RELEASE ) // 타겟버튼값이 있었고 && 해당버튼이 이전에는 안눌러졌어음(처음 눌려짐)
    	{
    		_delay_ms(60); // noise가 지나가길 기다린다
    		button_status[button_num]=BUTTON_PRESS; //해당 버튼 1이었다 기록 // noise가 지나간 상태의 high 상태
    		return 0; // 아직은 완전히 눌렸다 떼어진 상태가 아니다. main 문에서 while빠르게 돌며 이어서 검사함
    	}
    	// 60ms가 지나면 대부분의 튐 현상이 사라지게 됩니다.
    	else if(current_state==BUTTON_RELEASE && button_status[button_num]==BUTTON_PRESS) // 이전(약60ms이전)에 눌렸었다가 지금은 뗴어진 경우
    	{
    		_delay_ms(60);
    		button_status[button_num]=BUTTON_RELEASE; // 지금 떼어진 거 기록
    		return 1; // 완전히 1번 눌렸다 떼어진 상태로 인정 
    	}
    	return 0; // 버튼이 open
    }
    ```

  + led.c
    ```C
    #include "led.h"
    #include <math.h>
    
    int led_main(void);
    void led_all_on(void);
    void led_all_off(void);
    void shift_left_ledon(void);
    void shift_right_ledon(void);
    void shift_left_keep_ledon(void);
    void shift_right_keep_ledon(void);
    void flower_on(void);
    void flower_off(void);
    
    int led_main(void)
    { 
    	DDRA = 0xff; // PORTA에  연결된 pin8개를 output mode로 
    	while(1)
    	{
    		led_all_on();
    		_delay_ms(1000);
    		led_all_off();
    		_delay_ms(1000);
    		flower_on();
    		flower_off();
    		
    		shift_left_ledon();
    		shift_right_ledon();	
    		shift_left_keep_ledon();//	
    		shift_right_keep_ledon(); //
    	}
    }
    void led_all_on(void)
    {
    	PORTA=0xff;
    }
    
    void led_all_off(void)
    {
    	PORTA=0x00;
    }
    
    void shift_left_ledon(void)
    {
    	for (int i=0; i<8;i++)
    	{
    		PORTA = 0b00000001 << i;
    		_delay_ms(50);
    	}
    }
    
    void shift_right_ledon(void)
    {
    	PORTA=0;
    	for (int i=0; i<8;i++)
    	{
    		PORTA = 0b10000000 >> i;
    		_delay_ms(50); //
    	}
    }
    
    void shift_left_keep_ledon(void)
    { 
    	PORTA=0;
    	_delay_ms(200);
    	//PORTA = 0b00000001;
    	for (int i=0; i<8;i++)
    	{
    		// M2.
    		//PORTA = (1<<(i+1)) -1; // << 관련 연산에도 괄호 쳐야함. shift보다 사칙연산이 우선순위 높음
    		// 2**1-1 =1, 2**2-1=3,
    		// Mx.
    		//PORTA = (   ( pow(2, (i ) )  ) -1 );
    		// - 에러 상황 
    		//    출력하면 right랑 똑같은 숫자들 맞게 나오는데 왜안되지. 계산이 느린가. PORTA의 대입 방식이 다른가
    		//    pow연산에 시간이 걸리나 그러기엔 right는 잘됨
    		// - 안 된 이유  :  pow가 더블형 반환임.
    		//   부동소수점연산 때 컴보다 MCU는 스펙이 낮아서 오차가 컸을 수 있음
    		//   정수형에 넣으면 '내림'됨. 그래서 1이 작은 결과가 나왔을 수 있음
    		//   VSC로 확인할 땐 int만생각하며 int형 변수에 받고 %d로해서 몰랐을 수 있음.
    		//     실수값을 %d로 출력하면 완전 이상한값 나오는데, int형변수에 넣었다가하면 변수에 넣을때 내림으로 돼서 출력됨
    		// --> 한방향으로((곱하기는 1버려지고 나누기는 ㄱㅊ함)) 1이 발생하는 거보니 맞나봄
    		
    		// M1-1.
    		PORTA += 1<<i ;
    		////PORTA +=pow(2,i); //누적합이어도 2**i여야함// PORT에 누적*2는 shift일 때만이고 keep일 때는 안됨//2*3같은 단순 곱도 안됨// 아니면 이전 4같은 i곱부분만 따로 저장해서 2곱하던 몫만큼 2를 곱하던 변수하나 더 추가해서 만들수야 있겠지만 복잡해지니 패쓰
    		//// 0+2**0, 1+2**1=3, 3+2**2=7
    		
    		// M1-2. 교수님 풀이 : 더 비트 연산자
    		//PORTA |= 0b00000001 << i; // 내가 처음에 생각한 거랑 값수식은 같은데 bit연산을 먼저 떠올리지 못했네 //// OR. for문 이전 값이 0이라 여기에서의 1위치만 지속적으로 1로 만들 수 있음
    		
    		_delay_ms(50);
    	}
    }
    
    
    void shift_right_keep_ledon(void)
    { 
    	PORTA=0xff; // 1111 1111
    	_delay_ms(50);
    	for (int i=7; i>0;i--)
    	{
    		PORTA -= (1<<i);//pow(2,i); // pow도 됨 // i를 0부터로 하면 0b10000000 >> i; 이런 방식도 있음
    		_delay_ms(50);
    	}
    }
    
    
    //// (1) (200ms->50ms) 4,3키기 --(양옆 한칸씩 더 키기)-> 5~1키기 -> 6~1키기 -> 7~0키기
    //// (2) (200ms->50ms) (1) 의 역단계 (LED all on,...) -> all off
    //void flower_on(void)
    //{ 
    	//PORTA=0b00011000; // 수식으로 어거지로는 가능한데 그냥 손이 빠를듯
    	//_delay_ms(50);
    	//PORTA=0b00111100;
    	//_delay_ms(50);
    	//PORTA=0b01111110;
    	//_delay_ms(50);
    	//PORTA=0xff;
    	//_delay_ms(50);
    //}
    //
    //
    //
    //void flower_off(void)
    //{ 
    	//PORTA=0xff;
    	//_delay_ms(50);
    	//PORTA=0b01111110;
    	//_delay_ms(50);
    	//PORTA=0b00111100;
    	//_delay_ms(50);
    	//PORTA=0b00011000; 
    	//_delay_ms(50);
    	//PORTA=0;
    	//_delay_ms(50);
    //
    //}
    
    // M2. 교수님 코드
    void flower_on(void)
    {
    	PORTA=0;
    	for (int i=0; i<4; i++){
    		PORTA = 0x10<<i | 0x08 >> i; // 중간을 기준으로 양쪽이 규칙성이 있으면, 4bit나눠서 가진값으로 각각 shift하고 |로 더함//// 0001 0000, 0000 1000 
    		_delay_ms(50);
    	}
    }
    
    // M2. 교수님 코드
    void flower_off(void)
    {   
    	unsigned char h=0xf0;  // 레지스터가 꽉 찼으면 stack memory에 잡힘
    	unsigned char l=0x0f;
    	
    	PORTA=0;
    	for(int i=0; i<4; i++)
    	{ 
    		PORTA=(h>>i) & 0xf0 | (l<<i) & 0x0f; // 가운데쪽으로 shift시킬 때 0으로 남겨야하는 부분은 0과의 &처리 // 11110000-> &11110000 | 00001111<- &00001111 
    		_delay_ms(50);	
    	}
    	PORTA=0;
    	_delay_ms(50);
    
    }
    ```

  + button.h
    ```C
    #ifndef BUTTON_H_
    #define BUTTON_H_
    #define F_CPU 16000000UL 
    #include <avr/io.h> 
    #include <util/delay.h>
    
    #define LED_DDR DDRA // 코드에서 레지스터이름으로 사용 안하고 다르게 LED_DDR처럼 define하는 이유 : LED PORT가 변경됐을 때 모든 DDRA 바꾸러 다니지 않고, LED_DDR로 코딩한 거에서 #define만 DDRB로 바꿔주면 compiler가 변경 
    #define LED_PORT PORTA // LED -((전압)출력장치라) PORT
    #define BUTTON_DDR DDRD
    #define BUTTON_PIN PIND // BUTTON - (입력_핀으로 값을 읽음)PIN  // PIND는 PORTD를 읽는 레지스터
    
    #define BUTTON0PIN 3
    #define BUTTON1PIN 4
    #define BUTTON2PIN 5
    #define BUTTON3PIN 6
    
    #define BUTTON0 0 // PORTD.3의 가상의 index(SW번호)
    #define BUTTON1 1 // PORTD.4의 가상의 index(SW번호)
    #define BUTTON2 2 // PORTD.5의 가상의 index(SW번호)
    #define BUTTON3 3 // PORTD.6의 가상의 index(SW번호)
    #define BUTTON_NUMBER 4 // btn 개수
    
    #define BUTTON_PRESS 1 // 버튼을 누르면 high(active-high)
    #define BUTTON_RELEASE 0
    #endif
    ```

  + led.h
    ```C
    #ifndef LED_H_
    #define LED_H_
    
    #define F_CPU 16000000UL 
    #include <avr/io.h>
    #include <util/delay.h> 
    
    #endif 
    ```

### 1.2 7segment 스톱워치, 시계/반시계방향 회전, FSM  (&관련 사소한 실행 코드 & 이전 코드 어느정도 누적 & 주석 일부 추가)
- 4자리 7segment, 분할 컴파일
  + main.c
    ```C
    // - 분&초 시계[스톱워치]  FSM (버튼0으로 state 회전)
    //   + state 0 : 초 단위에 뒤에서 두번째 dp를 1초에 1회씩 on/off || 1번버튼 누르면 카운팅리셋.
    //   + state 1 : 초 시계 display ( 앞 두 segment가 회전하도록.  반시계, 시계)  || 1번버튼 누르면 회전시계((&카운팅)) 리셋.
    
    // 7segment 결선 : PORTB 4567(D1~D4)   PORTC 0~7(8개 다)
    #define F_CPU 16000000UL // Unsigned long 16MHz
    #include <avr/io.h> // PORTA PORTB 등의 IO관련 레지스터 등이 있다.
    #include <util/delay.h> // _delay_ms _delay_us
    
    #include "button.h" // button.c에서 include해놨지만, main 함수 내에서 관련된 게 쓰이면 여기서도 미리 include해야함
    extern int led_main(void); // 외부 파일의 함수. 자동으로 extern이긴 해도 코드 파악이 좀 더 빠를 수 있지(or 경고 없애기)
    extern void led_all_on(void);
    extern void led_all_off(void);
    extern void shift_left_ledon(void);
    extern void shift_right_ledon(void);
    extern void shift_left_keep_ledon(void);
    extern void shift_right_keep_ledon(void);
    extern void flower_on(void);
    extern void flower_off(void);
    
    extern void init_button(void); 
    extern int get_button(int button_num, int button_pin);
    
    // fnd.h를 include 안했는데 이것저것 잘되네 : 보통 헤더파일에서 저 함수 선언을 대신 해주곤 했던 거고, 주된 작동을 거의 fnd.c에서 해서 여기서는 딱히 그 파일이 쓰일 일 없었음. c파일 끼리 각각 컴파일 되고 링크돼서 거기에 맞게 있으면 여기에서 선언만 해줘도 사용할 수 있음
    extern void init_fnd(void); 
    extern void fnd_display(void); 
    //extern int fnd_main(void);
    
    void led_state (int);
    
    #if 0 // 접기 가능.// 정수 OR 정수로 치환되는 매크로 상수 식
    #else
    #endif
    
    // none OS 또는 loop monitor 방식
    int main(void) // VSC((일반 상황))와 다르게 반환형이 int여야함
    {	
    	// - DDR [Data Direction Register] : 방향설정
        // DDRA=0b11111111;  1(출력) 0(입력), 양수 8bit
    	// 레지스터이기에 형식 선언 안 해도 됨. 
    	// 상위 nibble (4bits), 하위 nibble
        	
    	//// led_main();
    	//// 아스키코드 0(0x30), space(0x20), A(0x41), a(0x61)
    
        // 초기화 (LED, 버튼) //// 안 쓰이는 곳도 있지만 여러 곳에서 쓰여서 아래 #if 안으로 각각 넣는거 패쓰
    	DDRA=0b11111111; //-> PORTA=0b11111111; [0xff;] : All led on
    	int button0_state=0; 
    
        init_button(); // DDRD 입력 설정
        //// 여기부터 fnd_main(); 을 main 함수로 가져옴
        init_fnd();
        
        //// init_button() ; 
        //int * pstate; 나중에 하고 싶으면 포인터로 바꿔서 구현해보기.함수내 이중포인터도 (여거에서 포인터가 자주 쓰이는 게 아니니까 굳이 이 케이스로 포인터연습할 필요는 없는데, 그래도 MCU에서 테스트 해보는 느낌 해보고 싶으면 하기) 
        int state=0;
        //pstate= &state;
    
        uint32_t ms_count=0;  //unsigned integer
        uint32_t sec_count=0;
    
        while(1)
        {
            if ( get_button(BUTTON0, BUTTON0PIN)){ // 보면 바로 함수 핵심 알고리즘 떠오를 정돈가?
                ms_count=0;
                sec_count=0;
                state =! state; // != != =!
            }
            
            if (get_button(BUTTON1, BUTTON1PIN)){ // 다른 구현에선 else if로 했지만 여기선 완전 독립적으로 해줌. 어차피 while로 처음으로 돌아갈 거라 뒤 조건 통과하게 될 때도 시간 등 큰 차이 안나겠지만 그래도. <-> else if 
                ms_count=0;
                sec_count=0;
            }
            
            fnd_display(state);
            _delay_ms(1); // 1ms 마다 다음 자리수 digit 출력. 이거 관련은 .md 텍스트에 적었었음. 4자리수의 7segment이지만 1ms 마다 옆에 있는 걸로 옮겨서 출력해주면서 잔상효과로, 적은 핀의 수로 많은 segment 사용. ((참 효율적이라 마음의 박수))
            
            ms_count++; //// 굳이 여기 연산에서 분, 초, 1의자리, 10의자리 맞춰서 연산해놓기보다, 나중에 해당 값 필요할 때 이 total값으로 바로 연산함.
            if (ms_count >=1000)
            {
                ms_count=0;
                sec_count++;
            }
            // 1s delay 누적 + 출력 및 while문 내 나머지 코드  인데도 동작이 거의 초로 맞게 되는 거 보면, MCU여도 꽤나 빠르나봄
        }
        return 0;
    }	
    
    #if 0 // (아래 문제에서) 함수 포인터 배열법 (스위치보다 좋음)  
    // 다른 .c에 있는 함수들 다 extern해야하긴해도 잘 사용하는 방법인가봄
    	void (*fp[])(void)= // 함수포인터 '배열' 이라서 main 안에서 하는 것도 자연스러움 //// []: 초기값 있어서 자동으로 배열의 길이 지정해줌 
    	{   // 우선 순위 : []가 *보다 우세함.  근데 그냥 *fp[] 배열이다 암기.
    		led_all_off,
    		led_all_on,
    		shift_left_ledon,
    		shift_right_ledon,
    		shift_left_keep_ledon,
    		shift_right_keep_ledon,
    		flower_on,
    		flower_off
    	}; // 함수 이름이 들어감
    	
    	while(1)
    	{
    		if (get_button(BUTTON0, BUTTON0PIN)){ //// state 값 설정
    			button0_state = (button0_state+1)%8; 
    		}
    		fp[button0_state] (); // => 해당 요소에 있는 함수를 ();_실행
    	}
    	return 0;
    }
    #endif
    
    #if 0// 아래의 문제 + (1) 해당 모드에서 반복실행, (2) 버튼 한개로 FSM 
    	while(1)
    	{
            //// FSM [Finite State machine] : 순차적.
    		//// 리셋은 MCU의 버튼
    		
    		//// 인터럽트 없이 코드 짜보기
    		// - 문제 상황 : 버튼입력을 딜레이 중에 받고있으면 안됨.
    		//   + M1. led 단계 사이의 delay 낮추기, 쓸데없는 delay 없애기 ////교수님도 led작동의 delay 시간을 낮추셨네
    		//   + M2. 디바운싱 함수 특성 상 꾹 누르고 있다가 떼면 더 잘됨. (사용한 디바운싱이 1존재-딜레이->0존재 카운팅이라, 1의 값이 확실히 전달되도록 주면 되기에.(led작동 중이었어도 쫌 길게 누르고 있으면 작동끝났을 때도 1 들어가게됨))
    		
    		////_delay_ms(200); // 하면 버튼 입력 못받고 있는 기간이 더 길어지고 더 느려짐. 
            
            //// FSM state 값 설정
    		if ( get_button(BUTTON0, BUTTON0PIN)){
    			button0_state = (button0_state+1)%8; // +1, 간격 내에서 순환을 위한 % //// 0이면 1, ..., 7면 0 
    			//led_state(button0_state);// if 안에서 X: 버튼이 눌릴 때만 led기능 수행하게돼버림. 전에 버튼 입력이 잘 됐던 4개의 버튼을 이용한 코드는 해당 버튼이 눌리면 그 if문 안에서 했지만, 그건 '반복이 아니라서' delay와 함께 led작동이 돌아가고 있을 때 누를 일이 더 적었음.
    		}
            
    		// M1.
    		led_state(button0_state) ;
            // else 문 안에서 할 시 : else 추가하면 if가 True일 때도 함수가 실행되던걸 못 실행하게 되는데, 어차피 while문으로 돌아서 이 else로 오게되긴 함. 근데 이론상으론 바로 되는 게 더 부합함
        	
    		// M2. 교수님은 아래와 같은switch로 하심. 단순 함수로 빼는 것보다 더 간단함.
    		//switch(button0_state){
    			//case 0: 
    				//led_all_off();
    				//break;
    			//case 1: 
    				//led_all_on();
    				//break;
    			//case 2:
    				//shift_left_ledon();
    				//break;
    			//case 3:
    				//shift_right_ledon();
    				//break;
    			//case 4:
    				//shift_left_keep_ledon();
    				//break;
    			//case 5:
    				//shift_right_keep_ledon();
    				//break;
    			//case 6:
    				//flower_on();
    				//break;
    			//case 7:
    				//flower_off();
    				//break;
    		//}
    	} 
        return 0;
    }
    
    void led_state (int button0_state){
        if ( button0_state==0 )
        {
            led_all_off();
        }
        
        else if ( button0_state==1 )
        {
            led_all_on();
        }
        
        else if ( button0_state==2 )
        {
            shift_left_ledon();
        }
        
        else if ( button0_state==3 )
        {
            shift_right_ledon();
        }
        
        else if ( button0_state==4 )
        {
            shift_left_keep_ledon();
        }
        
        else if ( button0_state==5 )
        {
            shift_right_keep_ledon();
        }
    
        else if ( button0_state==6 )
        {
            flower_on();
        }
    
        else if ( button0_state==7 )
        {
            flower_off();
        }
        
    }    
    #endif
    
    #if 0 // org // old
    // 문제
    // delay 50 ms 화 ( 버튼을 위해 )
    // 1. Button 0 : led_all_on <-> _off
    // 2. Button 1 : Shift_left_ledon <->
    // 3. button 2 : shift_left_keeP_ledon
    // 4.		   : flower_on <->
    	while(1)
    	{
    		//// 1 button 처리(toggle)
    		//// button0을 1번 누르면 led_all_on, 또 누르면 _off
    		if ( get_button(BUTTON0, BUTTON0PIN))
    		{
    			////버튼 한 가지만 사용할 시 : button0_state =!button0_state; 
    			
    			// 1위치만 반전 
    			button0_state ^=0b00000001; // (2-not 등과 무한 응용가능) ^ - with 0(상대값유지),1(반전)
    			//// 번갈아 나와야하기에 다른 스위치를 idle상태로 바꾸진 않았음. 
    			// 다른 스위치의 마지막 상태(다른 bit위치의 값은)는 유지
    
    			// (3-not 등과 무한 응용가능) | - with 0(상대값유지),1(1화) ////물론 NAND NOR 등이 더 간단한 건 있지만 코드짜기 쉬운 것도 어느정도는 반영
    			if((button0_state| 0b11111110 ) ==0xff )//인덱스0위치가 1인지(다른 인덱스위치 무슨값이든 괜찮) // <<7bit가 0x80인지 확인하는 방법도 있음. 
    			//// 0인덱스 아닌 곳들은 어차피 대응하게 지정한 값이랑 비교해줄거라 다른 0화 기호써도 상관없었긴함
    			////버튼 한 가지만 사용할 시 : button0_state==1) 
    				led_all_on();
    			else led_all_off();
    			
    		} 
    		
    		else if ( get_button(BUTTON1, BUTTON1PIN))
    		{
    			button0_state ^=0b00000010;
    			if( (button0_state| 0b11111101 ) ==0xff  )
    				shift_left_ledon();
    			else shift_right_ledon();
    		}
    		
    		else if ( get_button(BUTTON2, BUTTON2PIN))
    		{
    			button0_state ^=0b00000100;
    			if( (button0_state| 0b11111011 ) ==0xff  ) 
    				shift_left_keep_ledon();
    			else shift_right_keep_ledon();
    		}
    
    		else if ( get_button(BUTTON3, BUTTON3PIN))
    		{
    			button0_state ^=0b00001000;
    			if( (button0_state| 0b11110111 ) ==0xff  )
    				flower_on();
    			else flower_off();
    		}
    	}
        return 0;
    }
    #endif // endif
    ```

  + fnd.c
    ```C
    #include "fnd.h"
    // 함수 선언은 헤더파일로 빼거나 안 해도 되지만, 여기서 fnd_main으로 자체적 코드 하는 경우도 있으므로 선언 냅뒀음
    //// 여기서 fnd_main구현할 때는 그 함수를 중간에 놓고 앞에 선언도 하지만, 현재 구현 중인 건 main으로 함수 빼줘놨긴함(현재 구현하려하는 게 버튼도 쓰는 건데 button.c,h도 따로 있길래 걍 main으로 옮겨줌)
    void init_fnd(void);
    void fnd_display(int  state);
    
    //extern void init_button(void); // main에서 해주고 옴. 함수 선언은 여러 번 해줘도 되지만, 개인적으로 보기 편하게 주석처리했음
    extern int get_button(int button_num, int button_pin);
    
    //// 아래 함수 main으로 옮겨줬음
    //int fnd_main(void){
    //}
    
    void init_fnd(void)
    {
        FND_DATA_DDR = 0xff;	// 출력 모드로 설정
        
        // DDR_: 레지스터이기에 따로 값을 지정해준 적 없어도 값 있음
        FND_DIGIT_DDR |= 1<<FND_DIGIT_D1 | 1<<FND_DIGIT_D2| 1<<FND_DIGIT_D3 | 1<< FND_DIGIT_D4; // |= 0xf0;  [0b11110000]보다 general한 방법임 //// |= shift : 해당 비트 출력(1)화
        
        //// fnd를 all off
        #if 1 // common anode
        FND_DATA_PORT = ~0x00;
        #else // common cathode
        FND_DATA_PORT = 0x00;
        #endif
    }
    
    void fnd_display(int state){
        // 모든 부분을 anode cathode로 나눠서 구현하지 않고 그냥 내 거에 맞춰서 anode용 코드만 쓰기도 했음
        
        // - 회전 세그먼트값 (dp,g~a -> 0xXX[0bXXXXXXXX])
        //// 반시계, 시계방향
        
        // 한 회전방향 : 다끄기+8 단계 = 9 단계.
        // anode이긴한데 편하게 cathode형으로 출력값 만들어주고 마지막에 뒤집기
        // 반시계 : 다끄기 0x00-> 키기2a-> +3a -> +3f -> +3e -> +3d -> +2d -> + 2c-> +2b ->
        // 시계   : 다끄기 0x00-> 키기 저거 역순으로 더하기
        // case 2 / 3 따로구현 : a랑 b 따로변수 저장 - 자신의 차례 아닐때는 +0더함
        // 두 방향의 회전 : % 18
        // 누적합을 데이터 포트값에다 더하며 하기엔 옆에서도 같이쓰는 data port니까 따로 변수로 저장해서 전달. 기본 스톱워치에서도 FND_DATA_PORT = fnd_font[ 그렇게 했었음
        
        // v3 : 시계방향도 포함
        int operand_index2[] = {~0,  ~1,~1,~1         ,~1         ,~1   ,~0x09,~0x0D,~0x0F
        , ~0,  ~2,~6,~14 ,~14,  ~14,~14,~14,~15 }; 
        int operand_index3[] = {~0,  ~0,~1,~0b00100001,~0b00110001,~0b00111001,~0b00111001   ,~0b00111001   ,~0b00111001
        , ~0,  ~0,~0,~0, ~8,  ~0b00011000,~0b00111000,~0b00111001,~0b00111001};
        
        // v2 : 숫자로 누적해두기 //// off -> 반시계 누적
        //int operand_index2[9] = {~0,~1,~1,~1         ,~1         ,~1   ,~0x09,~0x0D,~0x0F };
        //int operand_index3[9] = {~0,~0,~1,~0b00100001,~0b00110001,~0b00111001,~0b00111001   ,~0b00111001   ,~0b00111001  };
        
        // v1 : (해당 인덱스 순서부터) 누적해서 더해질 값
        //int operand_index2[9] = {0,  1,0,0         ,0         ,0   ,0x08,0x04,0x02 }; // off -> 반시계
        //int operand_index3[9] = {0,  0,1,0b00100000,0b00010000,0x08,0   ,0   ,0  };
    
    
        #if 1 // common anode
        // - 0~9의 segment값. (dp,g~a -> 0xXX[0bXXXXXXXX])
        // 0x inverting : 각자리수의 인덱스 숫자 더해서 15가 나오는 상대의 값[ㅁ=15-ㅇ]. 0(0) a(10) f(15)
        uint8_t fnd_font[] = {0xc0, 0xf9, 0xa4, 0xb0,  0x99, 0x92, 0x82 , 0xd8, 0x80, 0x98 };// 0x7f} ; dp(온점)만 켜진값, 카운팅하려면 그걸 1초로 읽으며 순차적으로 이어가면 안돼서 걍 여기선 빼고 따로 점 깜빡이게 함
        // dp 1초 on/off를 위 변수에다가 반영해서 더해주면, 일의/십의자리수 등등이 꼬임( 깜빡이길 원하는 위치의 자리수(case)에서만 그 dp값을 더한 변수의 배열을 이용하는 방법도 있긴 하겠다만 난 다르게 구현함. 저렇게 하면 일의 자리는 쉬워도 다른 자리수는 추가작업해줘야함)
        
        #else // common cathode
        uint8_t fnd_font[] = {~ 0xc0, ~ 0xf9, ~ 0xa4, ~ 0xb0, ~  0x99, ~0x92, ~0x82 ,~ 0xd8, ~0x80, ~0x98 };
        #endif
    
        static int digit_select=0; // static으로 값유지 //// while에서 이 함수 계속 호출되는데 ms
        switch(digit_select)
        {
            case 0: // 가장 낮은 위치의 segment
            #if 1 // common anode
            // 자리수라 : | with  0(상대값 유지) , 1(화)
            FND_DIGIT_PORT =1<<FND_DIGIT_D4;  // 0x80;// 연결한 port번호 4~7 중 7이랑 연결했었음. // X_ |= 1<< FND_DIGIT_D4; 이건 입출력 설정 때 자기 bit만 강제화할 때 쓰던 거고, 여기선 나머지bit가 꼭 0이어야함. 오히려 |없이 저 shift값을 넣어주는 게
            #else // common cathode
            //           & with  0(화) , 1(상대값 유지)
            FND_DIGIT_PORT =~(1<<FND_DIGIT_D4)//0x80;
            #endif
            FND_DATA_PORT = fnd_font[  sec_count%10  ] ; // - 이 한 세그먼트 출력 모습만 생각해보면 : (2) 값 '회전[%]' '10가지[10]' // 일의 자리 초(0~9)
            //// 가장 낮은 위치의 segment_dp 깜빡이게 하려면
            // '초'를 2로 나눈 나머지에 따라 : 원래값 <-> 원래값 & 0b01111111해서 원래값에 dp bit만 추가로 항상 on. (anode는 값 ~ 항상 생각)
            // (sec_count%2 ==0)? fnd_font[  sec_count%10  ] : (fnd_font[  sec_count%10  ] ) & 0b01111111 ;
            break;
            case 1:
            #if 1 // common anode
            FND_DIGIT_PORT =0x40;
            #else // common cathode
            FND_DIGIT_PORT = ~ 0x40;
            #endif
    
            #if 1 // 1초마다 깜빡임
            FND_DATA_PORT =
            (sec_count%2 ==0)? fnd_font[((sec_count)/10 )%6] : (fnd_font[((sec_count)/10 )%6]) & 0b01111111 ;
            #else
            FND_DATA_PORT = fnd_font[((sec_count)/10 )%6];  // (1) /10(십의자리수), (2) %6(회전)(6가지)//(십의자리수0~5 다음은 0) //// 십의자리 초
            #endif
            break;
            
            //#if state : 정수값인 매크로명이거나 정수여야했음     //// 회전 하는 state인지
            case 2:
            FND_DIGIT_PORT =0x20;
            
            if (state)//// 회전하는 state인지
                FND_DATA_PORT =operand_index2[sec_count%18];  //sec_count%18. 초를 기준으로 18단계 회전.
            else
                FND_DATA_PORT = fnd_font[(sec_count/60 %10)]; // 분-일의자리 // (1) /60 : 초변수기준 60이 1임 (2) %10. : 값의 회전 10단계
            break;
            
            case 3:
            FND_DIGIT_PORT =0x10;//|= 1<< FND_DIGIT_D1;
            
            if (state)
                FND_DATA_PORT =operand_index3[sec_count%18];
            else
                FND_DATA_PORT = fnd_font[sec_count/600%6];//(1)_타겟의 1값이 변수가 몇개 모여야하는지_몫 /60(분화),/10(십의자리), (2)_   //// 분-십의 자리
            
            break;
        }
        digit_select++; // 다음 위치의 segment로 이동
        digit_select %=4; // 끝일 경우 처음으로
    }
    ```
    
  + fnd.h
    ```C
    #ifndef FND_H_
    #define FND_H_
    
    #define F_CPU 16000000UL
    #include <avr/io.h>
    #include <util/delay.h>
    
    // 사용할 각 레지스터, 포트의 고유 이름 지정
    #define FND_DATA_PORT PORTC // (2) 포트 지정 ( <- a~g, dp)
    #define FND_DATA_DDR DDRC   // (1) 레지스터 입출력 설정
    
    // FND_DIGIT : segment 4자리 수 중에 선택 비트
    #define FND_DIGIT_PORT PORTB // (2) 자리수 선택도 출력임. 버튼(MCU밖)에서 입력을 주는 거랑, SW(MCU안)에서 입력을 주는 건[출력] 다름. //// ~~ 7segment에서 자리수 스스로 선택해서 MCU에 주는 거 아님
    #define FND_DIGIT_DDR DDRB   // (1) 
    #define FND_DIGIT_D1 4     // 저 숫자를 PORT번호로 하면 DDR입출력 값 설정해줄 때 shift할 수 있어서 편함
    #define FND_DIGIT_D2 5     // 0b1111xxxx이런 식의 값을 대입해줄 것임
    #define FND_DIGIT_D3 6
    #define FND_DIGIT_D4 7
    
    #endif /* FND_H_ */
    ```
    
  + button.c : 1.1과 기능 동일
    ```C
    #include "button.h"
    void init_button(void);
    int get_button(int button_num, int button_pin);
    
    void init_button(void){
    	// - 버튼 초기화 입력
    	//   + M1. DDRD &= 0x87 
    	//     + (1-not 등과 무한 응용가능) & - with 0(0화),1(상대값유지) : DDRD값이 뭐였든 타겟비트 0화 //// .3.4.5.6을 input으로 설정한다.
    	// 1000 0111 
    	// (1) 가독성이 떨어진다 (2) port 변경 시 프로그램 수정이 어렵다
    
    	//   + M2. DDRD & DDRD : 확장성 좋음 & <<kbit shift | 좋음.
    	// & (10000111)
    	BUTTON_DDR &= ~(1<<BUTTON0PIN | 1<< BUTTON1PIN | 1<<BUTTON2PIN | 1<<BUTTON3PIN); //2**BUTTON0PIN | -> 쓰려는 핀만 1. 
        // <- #define BUTTON_DDR DDRD
    	// DDRD는 레지스터이기에 따로 값을 지정해준 적 없어도 값 있음
    }
     
    // BUTTON0 : SW번호
    // BUTTON0PIN : button pin
    // 리턴값 : 1(버튼을 눌렀다 떼면), 0(idle)
    // 버튼을 눌렀을 때 값이 흔들려서 의도와 다르게 여러 입력으로 받지 않고 한 번으로 받는 코드(60ms간격 앞뒤). 전에 했던 hw느낌 디바운싱에서는 눌린 값이 지속되는 것도 체크했었는데 그건 누르지 않았을 때의 순간적인 1노이즈가 잘못 포착되는 걸 방지 &좀더 하드웨어라서 차이가 있을 수 있음. GPT는 샘플링을 늘리는 방법도 있다함
    
    int get_button(int button_num, int button_pin)
    {
    	// static으로 이 함수의 해당 변수 값 유지 ////함수 빠져나갔다가 다시 들어와도 유지됨
    	static unsigned char button_status[]=
    	{BUTTON_RELEASE, BUTTON_RELEASE, BUTTON_RELEASE, BUTTON_RELEASE}; 
    		
    	int current_state;  
    	current_state = BUTTON_PIN &(1<<button_pin); // 버튼을 읽어온 것(노이즈 포함됨) & 타겟비트(버튼이기에 일반 숫자를 원핫벡터화)
    	// 타겟버튼 입력값만 원핫 벡터화
    	if (current_state && button_status[button_num]==BUTTON_RELEASE ) // 타겟버튼값이 있었고 && 해당버튼이 이전에는 안눌러졌어음(처음 눌려짐)
    	{
    		_delay_ms(60); // noise가 지나가길 기다린다
    		button_status[button_num]=BUTTON_PRESS; //해당 버튼 1이었다 기록 // noise가 지나간 상태의 high 상태
    		return 0; // 아직은 완전히 눌렸다 떼어진 상태가 아니다. main 문에서 while빠르게 돌며 이어서 검사함
    	}
    	// 60ms가 지나면 대부분의 튐 현상이 사라지게 됩니다.
    	else if(current_state==BUTTON_RELEASE && button_status[button_num]==BUTTON_PRESS) // 이전(약60ms이전)에 눌렸었다가 지금은 뗴어진 경우
    	{
    		_delay_ms(60);
    		button_status[button_num]=BUTTON_RELEASE; // 지금 떼어진 거 기록
    		return 1; // 완전히 1번 눌렸다 떼어진 상태로 인정 
    	}
    	return 0; // 버튼이 open
    }
    ```

  + button.h : 1.1과 기능 동일
    ```C
    #ifndef BUTTON_H_
    #define BUTTON_H_
    #define F_CPU 16000000UL 
    #include <avr/io.h> 
    #include <util/delay.h>
    
    #define LED_DDR DDRA // 코드에서 레지스터이름으로 사용 안하고 다르게 LED_DDR처럼 define하는 이유 : LED PORT가 변경됐을 때 모든 DDRA 바꾸러 다니지 않고, LED_DDR로 코딩한 거에서 #define만 DDRB로 바꿔주면 compiler가 변경 
    #define LED_PORT PORTA // LED -((전압)'출력'장치라) 'PORT'A
    #define BUTTON_DDR DDRD
    #define BUTTON_PIN PIND // BUTTON - ('입력'_핀으로 값을 읽음)'PIN'D  // PIND는 PORTD를 읽는 레지스터
    
    #define BUTTON0PIN 3  // DDR값 입출력 설정해줄 때 shift하며 사용하곤 함
    #define BUTTON1PIN 4
    #define BUTTON2PIN 5
    #define BUTTON3PIN 6
    
    #define BUTTON0 0 // PORTD.3의 가상의 index(SW번호) // SW적인 알고리즘 짤 때 주로 사용
    #define BUTTON1 1 // PORTD.4의 가상의 index(SW번호)
    #define BUTTON2 2 // PORTD.5의 가상의 index(SW번호)
    #define BUTTON3 3 // PORTD.6의 가상의 index(SW번호)
    #define BUTTON_NUMBER 4 // btn 개수
    
    #define BUTTON_PRESS 1 // 버튼을 누르면 high(active-high)
    #define BUTTON_RELEASE 0
    #endif 
    ```

  + led.c, led.h : 사용 안했음
 
#### Reference
허경용. (2016). ATmega128로 배우는 마이크로컨트롤러 프로그래밍. 제이펍.
    
