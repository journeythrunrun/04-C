## Ch 통신

- (1) 
  ```C
  // - 작동 테스트 법
  // 코드 실행 -> SerialPortMon -> ( Connect_TCP Client_지역 IP 0.0.0.0 (Any)  ),  127.0.0.1 , 포트 8080
  // => 코드 및 cmd(Server)와 SerialPortMon(Client)
  
  // - ip
  // 127.0.0.1(무조건 자기자신), cmd에 나온 포트번호 쓰기
  // 192.168.x.x(남이 할 때)
  // 10.10.16.x. < -- x안되는듯 --> 192.168.7.x
  
  #include <stdio.h>
  #include <stdlib.h>
  #include <WinSock2.h>
  
  #pragma comment(lib,"ws2_32.lib")
  #define PORT 8080
  #define BUFFER_SIZE 1024
  
  int main(void) {
      WSADATA wsa;
      WSAStartup(MAKEWORD(2, 2), &wsa);
  
      SOCKET server_fd, new_socket;
      struct sockaddr_in address;
      int addrlen = sizeof(address);
      char buffer[BUFFER_SIZE] = { 0 };
      char* message = "Hello from server";
  
      //소켓 설정 // 이더넷에서 소켓 통신이 대세임
      //server_fd = socket(AF_INET, SOCK_STREAM, 0);
      //address.sin_family = AF_INET;
      //address.sin_addr.s_addr = INADDR_ANY;
      //address.sin_port = htons(PORT);
  
      // - cmd : ipconfig
      // - ping www.naver.com / 내컴 주소 ping 
      // - TCP :  1대1통신 ㅣ 중요한 통신(은행) 
      // - UDP : (동영상) | 받았는지 확인 없이 보낼 때.
      // - serial port mon //https://m.blog.naver.com/sbspace/221754708164 / https://www.com-port-monitoring.com/
      //   + TCP 서버
  
      // 소켓 연결 
      bind(server_fd, (struct sockaddr*)&address, sizeof(address));
      listen(server_fd, 3);
  
      printf("서버 대기 중 .. 포트 %d\n", PORT);
  
      // 접속 대기
      new_socket = accept(server_fd, (struct sockaddr*)&address, &addrlen);
  
      // 데이터 수신
      while (1) {
          recv(new_socket, buffer, BUFFER_SIZE, 0);
          printf("클라이언트 : %s\n", buffer);
          if (strcmp(buffer, "stop") == 0) break;
      }
      // 접속 종료
      closesocket(new_socket);
      closesocket(server_fd);
      return 0;
  }
  ```

- (2-1) -> (2-3)
  ```C
    // - (2-1) -> (2-3) 
  // 코드 실행 -> SerialPortMon -> ( Connect_TCP Client_지역 IP 0.0.0.0 (Any)  ),  127.0.0.1 , 포트 8080
  // => cmd 서버와 portMon 이 서로 입력하고 볼 수 있음
  // disconect하면 클라이언트 : 가 계속 출력되는 건 뒷 코드에선 해결
  
  #include <stdio.h>
  #include <stdlib.h>
  #include <WinSock2.h>
  #include <windows.h>
  
  #pragma comment(lib,"ws2_32.lib")
  #define PORT 8080
  #define BUFFER_SIZE 1024
  
  SOCKET new_socket;
  
  typedef enum {
      NONE = 0,
      RUN,
      STOP,
      EXIT
  }STATE_APP;
  
  STATE_APP stateApp;
  
  // 송신 스레드 - (2-1)
  DWORD WINAPI ThreadFunction(LPVOID param) {
      int id = *(int*)param;
      char txBuffer[80];
      for (int i = 0; i < 5; i++) {
          printf("Thread %d 실행 중...%d \n", id, i);
          Sleep(100);
      }
      return 0;
  }
  
  // 송신 스레드 - (2-2) -> (2-3)
  DWORD WINAPI thread_tx(LPVOID param) {
      char txBuffer[80];
      while (1) {
          fgets(txBuffer, sizeof(txBuffer), stdin);
          if (strncmp(txBuffer, "disconnect", 10) == 0) break;
          send(new_socket, txBuffer, strlen(txBuffer), 0);
      }
      /* (2-2)
      for (int i = 0; i < 5; i++) {
          printf("Thread %d 실행 중... \n", i);
          Sleep(1000);
      }*/
      stateApp = EXIT;
      return 0;
  }
  
  // 수신 스레드 - (2-2) -> (2-3)
  DWORD WINAPI thread_rx(LPVOID param) {
      char buffer[BUFFER_SIZE] = { 0 };
      /*for (int i = 0; i < 5; i++) {
          printf("Thread %d 실행 중... \n", id);
          Sleep(800);
      }*/
  
      // 데이터 수신
      while (1) {
          memset(buffer, 0, sizeof(buffer));
          recv(new_socket, buffer, BUFFER_SIZE, 0);
          if (stateApp == EXIT)break;
          printf("클라이언트 : %s\n", buffer);
          if (strcmp(buffer, "stop") == 0) break;
      }
  
      return 0;
  }
  
  
  // - (2-1) -> (2-3) 
  // 양쪽 통신 가능 
  int main(void) {
      WSADATA wsa;
      WSAStartup(MAKEWORD(2, 2), &wsa);
  
      SOCKET server_fd;// , new_socket; 전역변수로 뺌
      struct sockaddr_in address;
      int addrlen = sizeof(address);
      // char buffer[BUFFER_SIZE] = { 0 };
      char* message = "Hello from server";
  
      // 소켓 설정
      server_fd = socket(AF_INET, SOCK_STREAM, 0);
      address.sin_family = AF_INET;
      address.sin_addr.s_addr = INADDR_ANY;
      address.sin_port = htons(PORT);
  
      // 소켓 연결
      bind(server_fd, (struct sockaddr*)&address, sizeof(address));
      listen(server_fd, 3);
  
      printf("서버 대기 중 .. 포트 %d\n", PORT);
  
      // 접속 대기(waiting..)
      new_socket = accept(server_fd, (struct sockaddr*)&address, &addrlen);
  
      // 새로운 스레드 생성
      HANDLE thread_transmit, thread_receive;
      thread_transmit = CreateThread(NULL, 0, thread_tx, NULL, 0, NULL);
      if (thread_transmit == NULL) printf("transmit 스레드 생성 실패 \n");
      thread_receive = CreateThread(NULL, 0, thread_rx, NULL, 0, NULL);
      if (thread_receive == NULL) printf("receive 스레드 생성 실패 \n"); // ' 안됨
      //HANDLE threads[2];
      //int threadIDs[2] = { 1,2 };
      //for (int i = 0; i < 2; i++) {
      //	threads[i] = CreateThread(NULL, 0, ThreadFunction, &threadIDs[i], 0, NULL);
      //	if (threads[i] == NULL) {
      //		printf("스레드 생성 실패\n");
      //		return 1;
      //	}
      //}
  
      while (stateApp != EXIT);
  
      // 접속 종료
      closesocket(new_socket);
      closesocket(server_fd);
  
      // 스레드 종료 대기
      // WaitForMultipleObjects(2, threads, TRUE, INFINITE); // 대소문자
      // WaitForMultipleObjects(1, thread_transmit, TRUE, INFINITE); // 대소문자
      // WaitForMultipleObjects(1, thread_receive, TRUE, INFINITE); // 대소문자
  
      // 스레드 핸들 닫기 
      //for (int i = 0; i < 2; i++)
      //CloseHandle(threads[i]);
      CloseHandle(thread_transmit);
      CloseHandle(thread_receive);
  }
  
  
  
  // 혹시 몰라서 교수님 원본. disconnect 10 이런 후 수정된 건 미반영
  /*
  #include <stdio.h>
  #include <stdlib.h>
  #include <WinSock2.h>
  #include <windows.h>
  
  #pragma comment(lib, "ws2_32.lib")
  
  #define PORT 8080
  #define BUFFER_SIZE 1024
  
  DWORD WINAPI thread_tx(LPVOID param);
  DWORD WINAPI thread_rx(LPVOID param);
  
  SOCKET new_socket;
  typedef enum {
      NONE = 0,
      RUN,
      STOP,
      EXIT
  } STATE_APP;
  
  STATE_APP stateApp;
  
  int main(void) {
      WSADATA wsa;
      WSAStartup(MAKEWORD(2, 2), &wsa);
  
      SOCKET server_fd;
      struct sockaddr_in address;
      int addrlen = sizeof(address);
      char* message = "Hello from server";
  
      // 소켓 설정
      server_fd = socket(AF_INET, SOCK_STREAM, 0);
      address.sin_family = AF_INET;
      address.sin_addr.s_addr = INADDR_ANY;
      address.sin_port = htons(PORT);
  
      // 소켓 연결
      bind(server_fd, (struct sockaddr*)&address, sizeof(address));
      listen(server_fd, 3);
  
      printf("서버 대기 중... 포트 : %d\n", PORT);
  
      // 접속 대기(waiting...)
      new_socket = accept(server_fd, (struct sockaddr*)&address, &addrlen);
  
      // 새로운 스레드 생성
      HANDLE thread_transmit, thread_receive;
      thread_transmit = CreateThread(NULL, 0, thread_tx, NULL, 0, NULL);
      if (thread_transmit == NULL) printf("transmit 스레드 생성 실패\n");
      thread_receive = CreateThread(NULL, 0, thread_rx, NULL, 0, NULL);
      if (thread_receive == NULL) printf("receive 스레드 생성 실패\n");
  
      while (stateApp != EXIT);
  
      // 접속 종료
      closesocket(new_socket);
      closesocket(server_fd);
  
      // 스레드 종료 대기
      //WaitForMultipleObjects(1, thread_transmit, TRUE, INFINITE);
      //WaitForMultipleObjects(1, thread_receive, TRUE, INFINITE);
      // 스레드 핸들 닫기
      CloseHandle(thread_transmit);
      CloseHandle(thread_receive);
  }
  
  // 송신 스레드
  DWORD WINAPI thread_tx(LPVOID param) {
      char txBuffer[80];
      while (1) {
          fgets(txBuffer, sizeof(txBuffer), stdin);
          if (strcmp(txBuffer, "disconnect") == 0) break;
          send(new_socket, txBuffer, strlen(txBuffer), 0);
      }
      stateApp = EXIT;
      return 0;
  }
  
  // 수신 스레드
  DWORD WINAPI thread_rx(LPVOID param) {
      char buffer[BUFFER_SIZE] = { 0 };
  
      // 데이터 수신
      while (1) {
          memset(buffer, 0, sizeof(buffer));
          recv(new_socket, buffer, BUFFER_SIZE, 0);
          printf("클라이언트 : %s\n", buffer);
          if (strcmp(buffer, "stop") == 0) break;
      }
  
      return 0;
  }
  */
  ```

- (3) 이번엔 코드 및 cmd(클라이언트), SerialPortMon(서버)_서버로 먼저 열기(127.0.0.1)
  ```C
  // (주요) 수정 부분 : //!!
  // pirnt는 이전대로 해서 클라이언트가 거꾸로 나오긴함
  #include <stdio.h>
  #include <stdlib.h>
  
  #include <WinSock2.h>
  #include <ws2tcpip.h> //!! VS2019에선 inet_pton() 추가
  #include <windows.h>
  
  
  #pragma comment(lib,"ws2_32.lib")
  
  #define IS_CLIENT //!!
  #define PORT 8080
  #define BUFFER_SIZE 1024
  #define SERVER_IP "127.0.0.1" //!!
  
  SOCKET new_socket, sock;//!! server_fd 다 sock으로 바꿈
  typedef enum {
      NONE = 0,
      RUN,
      STOP,
      EXIT
  }STATE_APP;
  
  STATE_APP stateApp;
  
  // 송신 스레드 
  DWORD WINAPI thread_tx(LPVOID param) {
      char txBuffer[80];
      while (1) {
          fgets(txBuffer, sizeof(txBuffer), stdin);
          if (strncmp(txBuffer, "disconnect", 10) == 0) break;
  #ifdef IS_CLIENT
          send(sock, txBuffer, strlen(txBuffer), 0);
  #else
          send(new_socket, txBuffer, strlen(txBuffer), 0);
  #endif
      }
      /*m2
      for (int i = 0; i < 5; i++) {
          printf("Thread %d 실행 중... \n", i);
          Sleep(1000);
      }*/
      stateApp = EXIT;
      return 0;
  }
  
  // 수신 스레드 
  DWORD WINAPI thread_rx(LPVOID param) {
      char buffer[BUFFER_SIZE] = { 0 };
      /*for (int i = 0; i < 5; i++) {
          printf("Thread %d 실행 중... \n", id);
          Sleep(800);
      }*/
  
      // 데이터 수신
      while (1) {
          memset(buffer, 0, sizeof(buffer));
  #ifdef IS_CLIENT
          recv(sock, buffer, BUFFER_SIZE, 0);
  #else
          recv(new_socket, buffer, BUFFER_SIZE, 0);
  #endif
          if (stateApp == EXIT)break;
          printf("클라이언트 : %s\n", buffer);
          if (strcmp(buffer, "stop") == 0) break;
      }
  
      return 0;
  }
  
  // 양쪽 통신 가능
  int main(void) {
      WSADATA wsa;
      WSAStartup(MAKEWORD(2, 2), &wsa);
  
      //SOCKET sock;//!!
      struct sockaddr_in address;
      int addrlen = sizeof(address);
  
      // char buffer[BUFFER_SIZE] = { 0 };
      //// char* message = "Hello from server";
  
      // 소켓 설정 
      sock = socket(AF_INET, SOCK_STREAM, 0);//!! socket -> inet_pton => 다시 socket? 
      address.sin_family = AF_INET;
      address.sin_port = htons(PORT);
      //!!
  #ifdef IS_CLIENT
      //!!!!  address.sin_addr.s_addr = inet_addr(SERVER_IP);
      if (inet_pton(AF_INET, SERVER_IP, &address.sin_addr) != 1) {
          printf("inet_pton 변환 실패! 잘못된 IP 주소: %s\n", SERVER_IP);
          return 1;
      }
  #else
      address.sin_addr.s_addr = INADDR_ANY;
  #endif
  
  
      // 소켓 연결
  #ifdef IS_CLIENT 
      connect(sock, (struct sockaddr*)&address, sizeof(address)); // conect
      // 내가 상대방에게 접속하는 거기에 connect면 끝남. listen accptt없음
  #else
      bind(sock, (struct sockaddr*)&address, sizeof(address));
      listen(sock, 3);
      printf("서버 대기 중 .. 포트 %d\n", PORT);
  
      // 접속 대기
      new_socket = accept(sock, (struct sockaddr*)&address, &addrlen);
  #endif
  
      // 새로운 스레드 생성
      HANDLE thread_transmit, thread_receive;
      thread_transmit = CreateThread(NULL, 0, thread_tx, NULL, 0, NULL);
      if (thread_transmit == NULL) printf("transmit 스레드 생성 실패 \n");
      thread_receive = CreateThread(NULL, 0, thread_rx, NULL, 0, NULL);
      if (thread_receive == NULL) printf("receive 스레드 생성 실패 \n"); // ' 안됨
      /*HANDLE threads[2];
      int threadIDs[2] = { 1,2 };
      for (int i = 0; i < 2; i++) {
          threads[i] = CreateThread(NULL, 0, ThreadFunction, &threadIDs[i], 0, NULL);
          if (threads[i] == NULL) {
              printf("스레드 생성 실패\n");
              return 1;
          }
      }
      */
  
      while (stateApp != EXIT);
  
      // 접속 종료
      closesocket(new_socket);
      closesocket(sock); //!!
  
      // 스레드 종료 대기
      //WaitForMultipleObjects(2, threads, TRUE, INFINITE); // 대소문자
      //WaitForMultipleObjects(1, thread_transmit, TRUE, INFINITE); // 대소문자
      //WaitForMultipleObjects(1, thread_receive, TRUE, INFINITE); // 대소문자
  
      // 스레드 핸들 닫기 
      //for (int i = 0; i < 2; i++)
      //	CloseHandle(threads[i]);
      CloseHandle(thread_transmit);
      CloseHandle(thread_receive);
  
  
      return 0;
  }
  
  //// 소스 (내 VS19위해서 아래 한줄 추가) - 추가 수정 있을 수도?
  //#include <ws2tcpip.h>
  //#include <WinSock2.h>
  //#include <stdio.h>
  //#include <stdlib.h>
  //#include <windows.h>
  //
  //#pragma comment(lib, "ws2_32.lib")
  //
  //#define IS_CLIENT
  //#define SERVER_IP "127.0.0.1"
  //#define PORT 8080
  //#define BUFFER_SIZE 1024
  //
  //DWORD WINAPI thread_tx(LPVOID param);
  //DWORD WINAPI thread_rx(LPVOID param);
  //
  //SOCKET sock, new_socket;
  //typedef enum {
  //    NONE = 0,
  //    RUN,
  //    STOP,
  //    EXIT
  //} STATE_APP;
  //
  //STATE_APP stateApp;
  //
  //int main(void) {
  //    WSADATA wsa;
  //    WSAStartup(MAKEWORD(2, 2), &wsa);
  //
  //    struct sockaddr_in address;
  //    int addrlen = sizeof(address);
  //
  //    // 소켓 설정
  //    sock = socket(AF_INET, SOCK_STREAM, 0);
  //    address.sin_family = AF_INET;
  //    address.sin_port = htons(PORT);
  //#ifdef IS_CLIENT
  //    //address.sin_addr.s_addr = inet_addr(SERVER_IP);
  //    if (inet_pton(AF_INET, SERVER_IP, &address.sin_addr) != 1) {
  //        printf("inet_pton 변환 실패! 잘못된 IP 주소: %s\n", SERVER_IP);
  //        return 1;
  //    }
  //#else
  //    address.sin_addr.s_addr = INADDR_ANY;
  //#endif
  //
  //    // 소켓 연결
  //#ifdef IS_CLIENT
  //    connect(sock, (struct sockaddr*)&address, sizeof(address));
  //#else
  //    bind(sock, (struct sockaddr*)&address, sizeof(address));
  //    listen(sock, 3);
  //
  //    printf("서버 대기 중... 포트 : %d\n", PORT);
  //
  //    // 접속 대기(waiting...)
  //    new_socket = accept(sock, (struct sockaddr*)&address, &addrlen);
  //#endif
  //
  //    // 새로운 스레드 생성
  //    HANDLE thread_transmit, thread_receive;
  //    thread_transmit = CreateThread(NULL, 0, thread_tx, NULL, 0, NULL);
  //    if (thread_transmit == NULL) printf("transmit 스레드 생성 실패\n");
  //    thread_receive = CreateThread(NULL, 0, thread_rx, NULL, 0, NULL);
  //    if (thread_receive == NULL) printf("receive 스레드 생성 실패\n");
  //
  //    while (stateApp != EXIT);
  //
  //    // 접속 종료
  //    closesocket(new_socket);
  //    closesocket(sock);
  //
  //    // 스레드 종료 대기
  //    //WaitForMultipleObjects(1, thread_transmit, TRUE, INFINITE);
  //    //WaitForMultipleObjects(1, thread_receive, TRUE, INFINITE);
  //    // 스레드 핸들 닫기
  //    CloseHandle(thread_transmit);
  //    CloseHandle(thread_receive);
  //}
  //
  //// 송신 스레드
  //DWORD WINAPI thread_tx(LPVOID param) {
  //    char txBuffer[80];
  //    while (1) {
  //        fgets(txBuffer, sizeof(txBuffer), stdin);
  //        if (strncmp(txBuffer, "disconnect", 10) == 0)
  //            break;
  //#ifdef IS_CLIENT
  //        send(sock, txBuffer, strlen(txBuffer), 0);
  //#else
  //        send(new_socket, txBuffer, strlen(txBuffer), 0);
  //#endif
  //    }
  //    stateApp = EXIT;
  //    return 0;
  //}
  //
  //// 수신 스레드
  //DWORD WINAPI thread_rx(LPVOID param) {
  //    char buffer[BUFFER_SIZE] = { 0 };
  //
  //    // 데이터 수신
  //    while (1) {
  //        memset(buffer, 0, sizeof(buffer));
  //#ifdef IS_CLIENT
  //        recv(sock, buffer, BUFFER_SIZE, 0);
  //#else
  //        recv(new_socket, buffer, BUFFER_SIZE, 0);
  //#endif
  //        if (stateApp == EXIT) break;
  //        printf("클라이언트 : %s\n", buffer);
  //        if (strcmp(buffer, "stop") == 0) break;
  //    }
  //
  //    return 0;
  //}
  ```
