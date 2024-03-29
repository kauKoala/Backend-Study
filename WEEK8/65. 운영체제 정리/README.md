### 1. 용어 정리

- 운영체제: 커널과 인터페이스로 이루어져 있는 시스템 소프트웨어
- 커널: 프로세스 / 메모리 / 저장장치 관리와 같은 운영체제 기능을 구현한 프로그램
- 인터페이스: 사용자와 응용 프로그램이 요청한 명령을 커널에 전달하거나, 커널 실행 결과를 사용자와 응용 프로그램에 반환하는 프로그램
- 시스템 콜: 커널이 제공하는 시스템 자원을 사용하기 위한 함수
    - 사용 이유: 컴퓨터 자원을 보호하기 위해 사용자와 응용프로그램이 직접 접근하는 것을 차단해서 대신 인터페이스를 이용해 접근함
- 인터럽트: CPU가 다른 작업을 수행하던 도중 급하게 다른 일을 처리하고자할 때 사용하는 기능
    - 이벤트 드리븐: 버튼이 눌리면 프로세스에게 알려주는 방식

### 2. 프로세스와 스레드

- 프로세스: 실행중에 있는 프로그램
    - 코드가 메모리에 올라와서 작업이 진행되는 것
    - code, data, stack, heap 영역이 존재함
    - 운영체제는 PCB를 통해 프로세스 정보를 파악하여 프로세스를 관리함
- 스레드:  프로세스 내부의 실행 단위
    - stack만 고유한 영역을 가짐
    - stack이외의 영역은 공유되므로 문맥 교환 비용은 적지만, 다른 스레드에 이상이 생기면 같이 문제가 발생할 수 있음
- 문맥 교환 (context switching)
    - CPU에서 실행됐던 프로세스가 나가고, 새로운 프로세스를 들여오는 과정
    - 문맥 교환을 할 때, CPU에서 처리했던 프로세스를 PCB에 저장 → 새로운 프로세스의 코드와 데이터를 메모리에 올림 → 새로운 프로세스를 준비 큐에 삽입 → CPU 스케줄러가 CPU에게 프로세스가 해야할 일을 전달함

프로세스 영역 구조 (자바와 같음)

- code: 실제 코드가 있는 곳
- data: 정의한 변수(=상수)와 데이터가 저장된 곳
- stack: 함수의 호출, 지역 변수, 매개 변수가 저장된 곳
- heap: 객체, 동적으로 할당된 변수가 있는 곳

### 3. CPU 스케줄링

- 선점형 스케줄러: 프로세스가 CPU를 사용하고 있는데, 우선순위가 더 높은 프로세스가 현재 CPU를 사용중인 프로세스를 중단하고 사용하는 것
- 비선점형 스케줄러: 프로세스가 CPU를 사용하고 있으면 다른 프로세스가 CPU를 사용하지 못하는 스케줄러

- FCFS (First Come First Served, 비선점형)
    - 큐에 도착한 순서대로 CPU를 할당함
    - 단점: 처리 시간이 긴 프로세스가 있으면 다른 프로세스들이 하염없이 기다리므로 성능 저하 발생
- SJF (Shortest Job First, 비선점형)
    - 처리 시간이 적은 프로세스부터 CPU를 할당함
    - 단점: 처리 시간이 매우 긴 프로세스가 있으면 영영 처리를 못할 수도 있음
- 라운드 로빈 (선점형)
    - 특정 시간동안 작업을 하다가 완료하지 못하면 준비 큐 맨 뒤로 감
- 우선순위 스케줄링 (선점형, 비선점형)
    - 프로세스의 중요도에 따라 CPU를 할당함
- 멀티 레벨 큐 스케줄링
    - 우선순위에 따라 준비 큐를 여러개 사용하는 방법
    - 각 큐는 라운드 로빈으로 실행

### 4. 가상메모리

- 프로세스가 물리 메모리의 크기와 상관없이 메모리 공간을 제공해주는 기술
- 하드디스크를 메모리로 취급하여 프로세스를 사용할 수 있게 함
- 운영체제는 한 메모리 공간에 프로세스가 있는 것으로 생각함
    - 물리메모리 + 가상메모리로 사용하기 때문에 이 둘을 매핑하기 위해 페이지 테이블을 사용함
    - 페이지: 가상 메모리를 일정 크기로 나누어 사용하는 방법
    - 프레임: 물리 메모리를 일정 크기로 나누어 사용하는 방법 (페이지 크기와 같음)
- 이로 인해 한 프로세스를 물리메모리의 남은 자투리 영역에 저장할 수 있는 장점도 생김
