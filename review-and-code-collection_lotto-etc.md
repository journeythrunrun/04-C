> Refresh and review C language basics

> 책
  >> 서현우. *혼자 공부하는 C언어*. 개정판, 한빛미디어, 2023.


P.S. 다른 문서나 파일에 내용이 있어서 수정 예정
## 텍스트

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


