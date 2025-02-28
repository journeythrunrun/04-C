### Ch Queue

- (1) 선형 큐
  + 공백 상태 :  front == rear
	+ front(제거 시 증가), rear(삽입 시 증가)
	+ 데이터는 보통 [front+1, rear]에 있음

  + 단점 : 나중에 다시 앞 공간에 데이터 넣고 싶을 수 있는데 어려움
  
  + ```C
    #define _CRT_SECURE_NO_WARNINGS
    #include <stdio.h>
    #include <stdlib.h>
    #define MAX_QUEUE_SIZE 5
    
    // 배열 구현
    typedef int element;
    typedef struct {
    	int front;
    	int rear;
    	element data[MAX_QUEUE_SIZE];
    } QueueType;

    void error(char* message) { // (0) 에러 출력 함수
    	fprintf(stderr, "%s\n", message);
    	exit(1);
    }

    void init_queue(QueueType* q) { // (1) 초기화
    	q->rear = -1; // rear, front = -1
    	q->front = -1;
    }

    int is_full(QueueType* q) { // (2-1) 꽉 찼는지 검사 : rear
    	if (q->rear == MAX_QUEUE_SIZE - 1) {
    		return 1;
    	}
    	else
    		return 0;
    }

    int is_empty(QueueType* q) { // (2-2) 비었는지 검사 : front == rear
    	if (q->front == q->rear) // 이 검사식 return해도 됨
    		return 1;
    	else
    		return 0;
    }

    void enqueue(QueueType* q, int item) { // (3-1) 삽입 _ 꽉찼는지 검사, 인덱스 유사 rear, 값
    	if (is_full(q)) {
    		error("큐가 포화상태입니다.\n");
    	}
    	q->data[++(q->rear)] = item; // rear의 다음 거에 새 거 넣기 // 배열:[++]_다음 거 & rear도 증가
    }
    
    int dequeue(QueueType *q) { // (3-2) 제거 & 반환 _ 비었는지 검사, 인덱스 유사 front, 삭제하는 값 반환
    	if (is_empty(q)) {
    		error("큐가 공백상태입니다.\n");
    	}
    	int item = q->data[++(q->front)]; // "데이터 존재하는, front의 다음 거[++]"의 데이터 return // front 뒤 부터를 데이터로 치기에 증가 후의 front값 자체는 삭제되는 값 위치임
    	return item;
    }

    void queue_print(QueueType* q) { // (4) 큐 print
    	for (int i = 0; i < MAX_QUEUE_SIZE; i++) { 
    		if (i <= q->front || i > q->rear) { // 데이터 빈 곳 : front 이하 || rear 초과 
    			printf("   | "); 
    		}
    		else { // 데이터
    			printf("%d | ", q->data[i]);
    		}

    	}
    	printf("\n");
    }

    int main(void) {
    	int item = 0;
    	QueueType q1;
    	init_queue(&q1); // (1) 초기화
    
    	enqueue(&q1, 10); queue_print(&q1); // (3-1) 삽입 with (2-1) // (4) print
    	enqueue(&q1, 20); queue_print(&q1);
    	enqueue(&q1, 30); queue_print(&q1);
    
    	item = dequeue(&q1); queue_print(&q1); // (3-2) 삭제 with (2-2)
    	item = dequeue(&q1); queue_print(&q1);
    	item = dequeue(&q1); queue_print(&q1);
    
    	return 0;
    }
    ```

- (2) 원형 큐
  + 공백상태와 포화상태를 구별하기 위해 하나의 요소 공간은 항상 비워둔다. 
  + 포화상태 : front % M == (rear+1 )% M // front ~~ rear + 1 : 딱 저 위치 한 곳만 빈 거라 포화 상태임
  
	#~ 인덱스 0?
  + ```C
    #define _CRT_SECURE_NO_WARNINGS
    #include <stdio.h>
    #include <stdlib.h>
    #define MAX_QUEUE_SIZE 5
    
    // 배열 구현
    typedef int element; //// 명시적으로 구분하기 위해 사용

    typedef struct {
    	int front;
    	int rear;
    	element data[MAX_QUEUE_SIZE];
    } QueueType;
    
    void error(char* message) {
    	fprintf(stderr, "%s\n", message);
    	exit(1);
    }

    void init_queue(QueueType* q) { // (1)
    	q->rear = q->front = 0 ;
    }
        
    int is_full(QueueType* q) { // (2-1)
    	return ((q->rear + 1) % MAX_QUEUE_SIZE == q->front); // front/rear에서 +1 안 한거는, 어차피 이미 나머지화 돼있는 값이 저장돼있는 거라 굳이 % 안해도 됨
    }
    
    int is_empty(QueueType* q) { // (2-2)
    	return (q->rear == q->front);
    }
    
    void enqueue(QueueType* q, int item) { // (3-1) 삽입 - 검사, 인덱스 유사 rear, 값
    	if (is_full(q)) { 
    		error("큐가 포화상태입니다.\n");
    	}
    	q->rear = (q->rear + 1) % MAX_QUEUE_SIZE; // 원형큐라 다음 = rear +1 ) % 최대-큐-사이즈 [회전하며 1 증가] /// +1은 rear 자체를 변화시키는 게 아니기에 다시 rear에 넣어줘야함
    	q->data[q->rear] = item; // 값
    }
    
    element dequeue(QueueType *q) { // (3-2) 삭제 - (2-2), 인덱스 유사 front, 삭제하는 값 반환
    	if (is_empty(q)) {
    		error("큐가 공백상태입니다.\n");
    	}
    	q->front = (q->front + 1) % MAX_QUEUE_SIZE;
    	return q->data[(q->front)];
    }
    
    element peek(QueueType* q) { // (5) 비었는지 검사(2-2),삭제 없이 첫 요소값 반환(front의 바로 다음 거)

    	if (is_empty(q)) {
    		error("큐가 공백상태입니다.\n");
    	}
    	return q->data[(q->front+1)%MAX_QUEUE_SIZE]; //// dequeue와 다르게 front에 값 변화가 없음
    } 

    void queue_print(QueueType* q) {
    	printf("QUEUE(front=%d, rear=%d) = ", q->front, q->rear);

    	if (!is_empty(q)) { //// q에 데이터 있으면
    		int i = q->front;  /// i -> front 다음 것 부터 회전하며 인쇄
    		do {
    			i = (i + 1) % MAX_QUEUE_SIZE; // 원형으로 회전하면서 다음 인덱스
    			printf("%d | ", q->data[i]); 
    			if (i == q->rear) break; //// rear까지 다 print했으면 탈출
    		} while (i != q->front);// i == front면 탈출. 설령 꽉 찼어도 rear의 다음인 front 까지 올 일 없이 앞 if 문에서 탈출할
    	}
    	printf("\n");
    	
    }

    int main(void) {
    	QueueType queue;
    	int item;
    
    	init_queue(&queue);
    	while (!is_full(&queue)) { // not 꽉참 이면 [ 꽉 찰 때까지 ]
    		printf("정수를 입력하세요 : ");
    		scanf("%d", &item);
    		enqueue(&queue, item); // 삽입
    		queue_print(&queue);
    	}
    	printf("큐는 포화상태 입니다.\n\n");
    
    	printf("--데이터 꺼냄 단계--\n");
    	while (!is_empty(&queue)) { //// empty가 될 때까지 돌림
    		item = dequeue(&queue);
    		printf("꺼내온 정수 : %d\n", item);
    		queue_print(&queue);
    	}
    	printf("큐가 공백 상태 입니다.");
    
    	return 0;
    }
    ```

  > Ex. 랜덤 생성해서 큐에 넣고 빼기
    ```C
    #define _CRT_SECURE_NO_WARNINGS
    #include <stdio.h>
    #include <stdlib.h>
    #define MAX_QUEUE_SIZE 5
    
    typedef int element;
    
    typedef struct {
    	int front;
    	int rear;
    	element data[MAX_QUEUE_SIZE];
    } QueueType;
    
    void error(char* message) {
    	fprintf(stderr, "%s\n", message);
    	exit(1);
    }

    void init_queue(QueueType* q) {
    	q->rear = q->front = 0 ;
    }
    
    void queue_print(QueueType* q) {
    	printf("QUEUE(front=%d, rear=%d) = ", q->front, q->rear);
    	if (!is_empty(q)) {
    		int i = q->front;
    		do {
    			i = (i + 1) % MAX_QUEUE_SIZE;
    			printf("%d | ", q->data[i]);
    			if (i == q->rear) break;
    		} while (i != q->front);
    	}
    	printf("\n");
    }
    
    int is_full(QueueType* q) {
    	return ((q->rear + 1) % MAX_QUEUE_SIZE == q->front);
    }
    
    int is_empty(QueueType* q) {
    	return (q->rear == q->front);
    }
    
    void enqueue(QueueType* q, int item) {
    	if (is_full(q)) {
    		error("큐가 포화상태입니다.\n");
    	}
    	q->rear = (q->rear + 1) % MAX_QUEUE_SIZE;
    	q->data[q->rear] = item;
    }
    
    element dequeue(QueueType *q) {
    	if (is_empty(q)) {
    		error("큐가 공백상태입니다.\n");
    	}
    	q->front = (q->front + 1) % MAX_QUEUE_SIZE;
    	return q->data[(q->front)];
    }
    
    element peek(QueueType* q) {
    	if (is_empty(q)) {
    		error("큐가 공백상태입니다.\n");
    	}
    	return q->data[(q->front+1)%MAX_QUEUE_SIZE];
    }  
    
    int main(void) {
    	// 랜덤이라 그런지 실행해서 안 될 때도 있음
    	QueueType* queue;
    	int element;
    
    	init_queue(&queue);
    	srand(time(NULL)); // 랜덤 함수 초기화(시간으로 seed 설정)
    
    	for (int i = 0; i < 100; i++) {
    		if (rand() % 5 == 0) {
    			enqueue(&queue, rand() % 100);
    		}
    		queue_print(&queue);
    		if (rand() % 10 == 0) {
    			int data = dequeue(&queue);
    		}
    		queue_print(&queue);
    	}
    	return 0;
    }
    ```  

- 헤더 파일에 들어간 항목
  + #define으로 중복방지, 새변수 정의, 사용할 함수의 선언 (정의는 main이 아닌 c파일에 작성)
	

- Deque : double-ended queue [덱]
  + front에서도 넣고 rear에서도 뺄 수 있는 코드. 이전 함수에선 둘 다 +1 +1 이었다면 반대인 건 다 -1 -1
  + ```C  
    #define _CRT_SECURE_NO_WARNINGS
    #include <stdio.h>
    #include <stdlib.h>
    #define MAX_QUEUE_SIZE 5
    
    typedef int element;
    
    typedef struct {
    	int front;
    	int rear;
    	element data[MAX_QUEUE_SIZE];
    } DequeType;
    
    void error(char* message) {
    	fprintf(stderr, "%s\n", message);
    	exit(1);
    }

    void init_deque(DequeType* q) {
    	q->rear = q->front = 0 ;
    }
    
    void deque_print(DequeType* q) {
    	printf("DEQUE(front=%d, rear=%d) = ", q->front, q->rear);
    	if (!is_empty(q)) {
    		int i = q->front;
    		do {
    			i = (i + 1) % MAX_QUEUE_SIZE;
    			printf("%d | ", q->data[i]);
    			if (i == q->rear) break;
    		} while (i != q->front);
    	}
    	printf("\n");
    }
    
    int is_full(DequeType* q) {
    	return ((q->rear + 1) % MAX_QUEUE_SIZE == q->front);
    }
    
    int is_empty(DequeType* q) {
    	return (q->rear == q->front);
    }
    
    void enqueue(DequeType* q, int item) { // (3-1)
    	if (is_full(q)) {//
    		error("큐가 포화상태입니다.\n");
    	}
    	q->rear = (q->rear + 1) % MAX_QUEUE_SIZE;
    	q->data[q->rear] = item;
    }
    
    element delete_front(DequeType*q) { //dequeue // (3-2)
    	if (is_empty(q)) {
    		error("큐가 공백상태입니다.\n");
    	}
    	q->front = (q->front + 1) % MAX_QUEUE_SIZE;
    	return q->data[(q->front)];
    }
    
    element get_front(DequeType* q) { // (5) , peek
    	if (is_empty(q)) {
    		error("큐가 공백상태입니다.\n");
    	}
    	return q->data[(q->front+1)%MAX_QUEUE_SIZE];
    
    }  
    
    void add_front(DequeType* q, element val) { // (6-1) add_front : 포화 검사(2-1), front 위치에 값 저장, front는 -1(+)화 //// front 뒤부터는 이미 데이터가 있으므로
    	if (is_full(q)) {
    		error("큐가 포화상태입니다.");
      }
    	q->data[q->front] = val;
    	q->front = (q->front - 1 + MAX_QUEUE_SIZE) % MAX_QUEUE_SIZE; // + 아니고 -1이라 최대-큐-사이즈를 더한 것에서 나머지를 구해줌.
    }
    
    element delete_rear(DequeType* q) { // (6-2) : (2-2), temp = rear & rear = -1(+)화, 삭제한 데이터 temp 위치를 통해 반환
    	if (is_empty(q)) {
    		error("큐가 공백상태입니다.\n");
    	}
    	int prev = q->rear;
    	q->rear = (q->rear - 1 + MAX_QUEUE_SIZE) % MAX_QUEUE_SIZE;
    	return q->data[prev];
    }
    
    element get_rear(DequeType * q) { // (5)
    	if (is_empty(q)) {
    		error("큐가 공백상태입니다.\n");
    	}
    	return q->data[q->rear];
    }

    int main(void) {
    	DequeType queue;
    	init_deque(&queue);
    	for (int i = 0; i < 3; i++) {
    		add_front(&queue, i);
    		deque_print(&queue);
    	}
    	for (int i = 0; i < 3; i++) {
    		delete_rear(&queue);
    		deque_print(&queue);
    	}
    	return 0;
    }
    ```
