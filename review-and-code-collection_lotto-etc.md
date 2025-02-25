> Refresh and review C language basics

> 책
  >> 서현우. *혼자 공부하는 C언어*. 개정판, 한빛미디어, 2023.

> P.S. 다른 문서나 파일에 내용이 있어서 수정 예정. 깃허브에 업데이트는 상황 상 챕터 순서대로는 안할듯.

# 목차



## 텍스트
### ch 17 사용자 정의 자료형
#### 구조체
> 다양한 자료형을 저장할 때 유용함

- 구조체 선언
  + ``` 
    // M3. typedef 형 재정의를 통해  예약어 없애기=typedef ㅇ_"struct student"_[구조체 이름] ㅁ_Student;
    // M2. 기본 "struct student" { }  ((person1 = {1,'a'})) ; 
    // M1. 최대압축 "typedef" "struct" {} "studdent2_t" ;

    "struct student" person1 ; [예약어 구조체이름 변수명]/ student2_t person2 ;

    person1.num=2 ;
    ```
  + 함수 밖에서 구조체를 선언하면 선언되는 변수가 전역 변수이므로 디폴트 0으로 초기화됨.

  + 다른 구조체를 멤버로 사용 가능
    + 멤버 선언 : struct student s; 
    + 사용 : person1.s.age=
  + 포인터 멤버 : Ex. 동적할당 예정

- 구조체의 주소 = &person1. 이름 아님.

- 구조체 초기화, 선언 && 초기화
  + struct student person1 = {1, 'a'}, person2 ={2, 'b'};
    > 중괄호, 순서에 맞게 나열


- 자료형이 같은 구조체 변수는 대입 연산 가능
  > person1=person2

- 구조체 & 함수
  + 입출력이 구조체일 때 "struct student" exchange (struct student a );

- sizeof ( struct student ) // 예약어 구조체 이름

- 멤버 간 데이터 형식의 크기가 다를 때, 가장 큰 크기가 메모리를 할당하는 기준 단위가 됨.
  + 멤버의 순서에 따라 구조체 크기가 달라질 수 있음. 
    + 1 byte, 2(이전 변수에 1 패딩), 1, 4(이전 변수에 3패딩), 8(이전 변수에 4패딩), 1(남은 7바이트 패딩됨)

  > 멤버 사이 패딩 바이트를 넣음
  > 패딩 바이트 안 넣는 법 : #pragma pack(1); ( 보통 include 다음에 넣음 )
    > 데이터를 읽고 쓰는 시간은 증가함. 메모리는 최소화. 


- 함수 넣는 거 인수 받는 거 매개변수인듯

- 구조체 포인터
  + "struct student" * ps = & person1 
    > 변수 형식 이름만 다르고 똑같음
  
  + M1. ps -> name 
  + M2. (사용 시 괄호) (*ps).name
    +  .이 *보다 우선 순위 높음

- 구조체 변수 자체가 하나의 큰 변수임. -> 구조체 배열 가능.
  + 초기화
    + struct student people[100] = { {1,2},{3,4} }

  + (para+i)->name / para[i].name / ( *(para+i)).name


- 자기 참조 구조체
  + 멤버 변수가 자신의 구조체을 가리킴 
  + 연결 리스트 구현[링크드 리스트]
    + 첫 번째 변수의 위치만 알면 다른 건 따라가서 모두 사용할 수 있음.

  
  + ```C // 아래 예시는 각 객체가 이름을 가지고 있는데, 어차피 연결리스트로 연결하면 접근할 수 있어서인지, 예제 2는 tail->next에 동적공간 할당만해나감 
    //...
    struct student * next ;
    //...

    struct student person1 ={1,2}, person2 = {3,4}, person3={5,6};
    struct student * head = & person1, *current; 헤드 포인터 초기화
    // head는 person1 가리킴

    person1.next=&person2; // person1의 포인터인 멤버가 person2를 가리킴
    person2.next=&person3; 

    // head->name 출력
    // head->next->name 출력
    
    current = head;
    
    while (curret!=NULL){ // 원형 연결 리스트 아니기에 마지막 놈은 next가 0으로 초기화돼있음.
      //current->name 출력
      current = current-> next; // next가 다음 person가르키게 해뒀었음.
    }
    ```

    + head로 3번째 꺼 사용하려면 : head->next->next->name

    + 길면 계속 next 번거로우므로 움직이는 current 사용. 이 연결 리스트는 이전 꺼로 돌아가는 거 없으므로 첫 주소 잃어버리면 안돼서 head고정.

#### 공용체
- 모든 멤버가 하나의 저장 공간을 같이 사용함
  + 그래서 다른 멤버 값 바꿨는데 다른 멤버 값이 바뀌기도 함.
    > 항상 각 멤버의 값을 확인해야함


- 선언
  + 예약어가 union인 거 빼고는 구조체와 같음

- 초기화
  + 중괄호를 사용해서 첫 멤버만 초기화. 다른 멤버를 초기화할 땐 접근 연산자로 멤버 직접 지정.
    ```C
    "union student" person1 = {315}; 
    union student person2= { .grade = 97.7 }
    ```


- 공용체 변수의 크기
  + 멤버 중 크기가 가장 큰 멤버의 크기.를 다른 멤버들도 공유하며 씀.


- 열거형
  + 변수에 저장할 수 있는 정수 값을 기호로 정의해서 나열함 
  + "enum season" {SPRING=5, SUMMER, FALL=10, WINTER};// 디폴트 0,1,2,3 // 저 케이스엔 그 다음 숫자인 6, 11
    + 해당 열거형에 없는 값도 가질 수 있음 (++) 
  > 숫자 대신 이름 써서 읽기 쉬운 코드화
  + enum season first24=SPRING;
  
  
  -  head(포인터임) = tail = (Train *) malloc(sizeof(Train)); // 각각 생성 아니고, 같은 거 가르키는 거임. 포인터잖슴.





## 코드
### Ch 10
- 로또 번호 생성 프로그램(도전 실전 예제, 로또 번호 생성 프로그램)
  ```C
  /* Ch 10_도전 실전 예제*/
  /* P.S. 코딩테스트를 위한 풀이라기보다 C언어 복습공부의 연장선인 빨리 풀기임.(문제도 코테용아님) 따라서 문제에서 말하는 조건과 실행결과를 만족하는 풀이이며, 문제에 언급되지 않은 예외케이스를 시간들여 전부 다루진 않음.*/
  #define _CRT_SECURE_NO_WARNINGS
  #include <stdio.h>
  //#include <string.h>
  void input_nums(int* lotto_nums) {
  	int numbers[46] = { 0 };
  	// - 조건에 갖춰진 골격에 따라 문제를 푼다면, 
  	//   + m1) 딕셔너리 유사 풀이는 매개변수를 더 건네야하는데 못함 -> main 말고 입력 함수 내에서로 옮김  
  	// - 맞다 0 초기화는 한 방 가능
  	//for (int i = 1; i < 46; i++) {
  	//	numbers[i] = 0;//없음
  	//}
  
  	for (int i = 0; i < 6; i++) {
  		printf("input (one): ");
  		scanf("%d", &lotto_nums[i]); 
  
  		// m1
  		while (numbers[lotto_nums[i]] == 1) {// 이미 있음
  			printf("already exist\n");// 엔터 습관~
  			printf("input (one): ");
  			scanf("%d", &lotto_nums[i]);
  		}
  		numbers[lotto_nums[i]] = 1;
  
  		// m2) 변수로 중복있는지 체크	
  		//int exist = 1;
  		//while (exist==1) { // - 해설의 방법은 이 while을 안씀. 중복이 있을 시 i--를 해서 가장 바깥의 for이 끝나지 않게함((어차피 6개의 서로 다른 입력 받는 상황)). 그러면서 scanf 여러 곳에서 쓰지 않고 압축함.
  		//	exist = 0;
  		// 
  		//	for (int j = 0; j < i; j++) { //이전 인덱스 까지 보면 되니까 j<i
  		//		if (lotto_nums[i] == lotto_nums[j]) { 
  		//			exist = 1;
  		//			printf("already exist");
  		//			printf("input (one): ");
  		//			scanf("%d", &lotto_nums[i]); // - 인덱스 있기에 배열이든 포인터든 & 붙여야함. //이름이 주소인 거 맞는데, 인덱스붙인 건 이미 값임. 그래서 
  
  		//			break; 
  		//		}
  		//	}
  		//}
  	}
  }
  
  void print_nums(int* lottos_nums) {
  	printf("lotto numbers :");
  	for (int i = 0; i < 6; i++) {
  		printf(" %d", lottos_nums[i]);
  	}
  }
  
  int main(void) {
  	int lotto_nums[6];
  
  	input_nums(lotto_nums);
  	print_nums(lotto_nums);  
  	return 0;
  }

  ```




### Ch 12
- 12-2 확인 문제 3번 (입력한 단어의 길이가 5자를 넘는 경우 6자리부터 *출력. 최대길이 15자)
  ```C
  // m2) 교수님 코드
  // result[80]={0}
  // int i = 0; 미리 있으면 for (; 가능
  // result라는 변수를 따로 만들어서 입력의 모든 길이를 돌며[내코드의 for을 for 밖 조건문 없이] result에 변화돼야하는 결과로 저장함.
  // 내 코드가 코드로선 복잡하고 a < 5인 케이스 반영하면 효율일 수도 있음
  // m1)
  #define _CRT_SECURE_NO_WARNINGS
  #include <stdio.h>
  #include<string.h> 
  
  int main( void){
    char a[16];
    printf("단어 입력 :");
    scanf("%s", a); // c, 엔터 저장돼도 안쓰임
  
    if (strlen(a) > 5) {
      printf("\n입력한 단어 : %s, 생략한 단어 : ");
      for (int i = 0; i < strlen(a); i++) {
        if (i<5)
          printf("%c", a[i]);
        else 
          printf("*");
      }
    }
    else 
      printf("\n입력한 단어 : %s, 생략한 단어 : %s", a, a);// [0:5]
  
    return 0;
  }
  ```

- ch 12 단어 정렬 프로그램[도전 실전 예제]
  ```C
  // m2) 교수님 코드
  // 1,2비교  1,3비교,   2,3비교  // 두 번의 비교로 첫 번 째에 가장 작은 거 찾음. 나머지인 2,3비교로 두 번째 작은 거 찾음.
  // ->9,5,x   ->9,5, 8/3    ->
  
  // m1) 코드론 조금 더 복잡 & 효율은 나을 수도 (물론 if문 내부 등이 복잡하진 않을 땐 큰 차이 없을듯. 간단하게 하는 swap이 그렇게 무거울 거 같지도 않고)
  #define _CRT_SECURE_NO_WARNINGS
  #include <stdio.h>
  #include<string.h> 
  
  void strswap(char* first, char* second) { //*
    char temp[80];
    strcpy(temp, first);
    strcpy(first, second);
    strcpy(second, temp);
  }
  int main(void) {
    char a[80], b[80], c[80];
    scanf("%s %s %s", a, b, c);
    if (strcmp(a, b) == 1) { // - abc순 배열은 작은 게 앞으로임!
      strswap(a, b); // 9, 5,  x
    }
    //printf("%s %s %s", a, b, c);
  
    if (strcmp(b, c) == 1) {// 9,5, 7/10 
      //strswap(a, b);
      int value = strcmp(a, c);
      if (value == -1) {
        strswap(b, c);// - 코드 중복 없도록 미리 앞으로 안 뺸 이유: 케이스에 의해 최종 바꿔줘야하는 것에 따라 더 효율적이게 될 수도 있다고 생각했음. 케이스에 의해 최종 바꿔줘야하는 것들을 보니 효율은 차이 없음
      }
      else if (value = 1) {
        strswap(b, c);
        strswap(a, b);
      }
    }
    printf("%s %s %s", a, b, c);
  
    return 0;
  }
  ```


- ch 17-2 성적 처리 프로그램[도전 실전 예제]
  ```C
  #define _CRT_SECURE_NO_WARNINGS
  #include <stdio.h>
  #include <stdlib.h> 
  
  typedef struct {
  	int number;
  	char name[20];
  	int grades[3];
  	int total;
  	double avg;
  	char alph;
  } Student;
  
  int main (void){
    Student people[5]; // - (a) 앗 하고 수정했으면 코드 다 훑기!!!!!
  	int max_ids[5] = { 0,1,2,3,4 }; // - 아직 각각에 대한 구조 객체를 만들지도 않았어도, 이 변수에 넣을 초기값이 어차피 객체의 번째 인덱스라서 초기값으로 할 거 미리 알 수 있음
  	for (int i = 0; i < 5; i++) {
  		printf("학번 : ");
  		scanf("%d", & (people[i].number ) );
  		printf("이름 : "); // - scanf 입력할 때 막판에 엔터 하나 cmd에 입력시키게됨. 그니까 굳이 \n 안 넣어도 됨.
  		scanf("%s", &( people[i].name));
  		printf("국어, 영어, 수학 점수 : ");
  		scanf("%d%d%d", &(people[i].grades[0]), &(people[i].grades[1]),&( people[i].grades[2]));
  
  		people[i].total = people[i].grades[0] + people[i].grades[1] + people[i].grades[2];
  		people[i].avg = people[i].total / (double)3;
  		people[i].alph = (people[i].avg >= 90.0) ? 'A' : ((people[i].avg >= 80.0) ? 'B' : ((people[i].avg >= 70.0) ? 'C' : 'F'));
  		
  		// i vs i-1 인데 이걸 연달아도 가능하게 한칸 씩 땡겨 나갈꺼므로((~~연결리스트 포인터마냥)) j vs j-1
  		for (int j = i; j-1 >= 0; j --) {// 이전 것들과 비교하면서 가장 앞자리(max)로 갈 수 있는지. [0<~i-1]까지와 비교해보면 됨
  			if (people[  max_ids[j-1]  ].total < people[ i].total) { // i대신 max_ids[j]도 됨. j 는 안됨 . 다음 앞에 것도 또 거슬러 올라가나 검사하는 상황에서, j는 이미 감소돼있음. 새로운 놈이 최대값 순서 배열에서 어디에 위치할지 비교하는 건데 새로운 놈 말고 다른 놈으로 바뀌게 돼버림.  max_ids[ j] 가 되는 이유는, 거슬러 올라온 상황에서 max_ids[j]의 j도 이미 감소했지만 저 요소값이 새로운 놈이고 이미 swap돼어온 것이기 때문임. / 아예 i는 새로운 놈 고정임. 대신 아래의 인덱스 swap에서는 i 주의해라 
  				//swap j, j-1. 여기부턴 max_ids의 값을 지정해주는 거라, max_ids[j]대신 i를 쓸 일 없음.
  				int temp = max_ids[j - 1];
  				max_ids[j-1] = max_ids[j]; // for 돌면서 여러 단계 거슬러 올라갈 수도 있음.// max_ids가 가지고 있는 인덱스 값을 바꾸는 거니까 우변 j절대 안되지! swap의미도 알텐데 왜 그랬대(if문의 주석과 비슷한 맥락으로 그런듯). 여길 그렇게 했을 때도 그정도나 정답에 근접할 수있다니. 약간만 값틀려도 핵심 알고리즘 틀린 부분 있을 수 있는 거 감안해라.(물론 이어서 거슬러 올라가는게 이 케이스에선 흔하지 않아서 그랬을듯)
  				max_ids[j] = temp;
  			}
        // 효율을 위해 첫 for 내부에서 돌리는 중. & 새로운 놈이 더 작으면 더 이상 비교할 필요 없음. 
  			else { // 비교해서 더 작아도 처음이면 저장은 해야지. -> 어차피 비교 대상 개수는 i,j로 범위 잡았으니(=범위 벗어나는 부분 비교는 자동으로 안할테니) 초기값을 각 인덱스로해줄 수 있음.
  				break;
  			}
  		}
  	}
  	// 21034 가 떠야하는데  
  	// 1204-1뜨네 // 기호 -아님 -1 정수임. %d임.
  	// 12034 // 맨 앞 까지 검사를 못하나. 
  	// 11034
  	// printf("%d%d%d%d%d-", max_ids[0],max_ids[1],max_ids[2],max_ids[3],max_ids[4]);
  
  	printf("# 정렬 전 데이터...\n"); // # 앞에 역슬래시 하나 안 하나 같네
  	for (int i = 0; i < 5; i++) {
  		printf("%6d %6s%6d%6d%6d", people[i].number, people[i].name, people[i].grades[0], people[i].grades[1], people[i].grades[2]);
  		printf("%6d%6.1lf%6c\n", people[i].total, people[i].avg, people[i].alph);
  	}
    
    // total max 순서로 출력하기
  	// 완전한 덩어리 swap을 기대했을 수도 있긴 하겠는데 난 인덱스를 기억하겠다.(인덱스를 swap) ////m_x 예전에 만든 3개 비교하는 걸 연속해서? 
  	// sort 알고리즘. 
  	// 멍 때리면 안댕!(이사짐 다 치우곤 더 괜찮을듯. 공부와 기타 등등을 위해 적당히씩 푸는 중 ㅎㅋ 틈새시간에 하기..ㅎㅋ) 
  	// m1. max찾, 그 다음 max찾. 
  	// - strcmp는 어차피 숫자 바로 비교할 수 있어서 의미없
  
  	// M2. 9, 7 ,3 <<--올라갈 수 있는 만큼 거슬러 올라가기->,x,x,x
  	// 효율을 위해 저장하면서 최대값 업데이트 -> 위 for문 안에 작성.
  
  	printf("# 정렬 후 데이터...\n"); 
  	for (int i = 0; i < 5; i++) {
  		printf("%6d %6s%6d%6d%6d", people[max_ids[i]].number, people[max_ids[i]].name, people[max_ids[i]].grades[0], people[max_ids[i]].grades[1], people[max_ids[i]].grades[2]);
  		printf("%6d%6.1lf%6c\n", people[max_ids[i]].total, people[max_ids[i]].avg, people[max_ids[i]].alph);
  	}
    return 0;
  }
  ```
