> 시간 나면 [2503에 임시로 써놓은 md]업데이트 예정

> 기능을 추가 및 변경하며 사용한 것으로, 사용하지 않은 변수도 있음

- 버튼 4개로 LED제어 with bit 연산
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
