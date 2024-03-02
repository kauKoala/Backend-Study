### 1. 해시 테이블

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/c415c49b-f116-42b5-8500-8c25bc2199a9)

key, value로 데이터를 저장하는 자료구조로 해시함수를 사용하여 변환한 값을 인덱스로 삼아 해시키를 만든다.

특징

- 일반적으로 데이터 삽입, 수정, 삭제의 연산이 O(1)이 걸린다.
- 해시값이 계속 충돌하게 될 경우 최악의 경우 O(N)이 걸린다.

### 2. 해시 충돌 해결법

해시함수를 통해 해시값을 만드는 경우, 같은 해시값이 존재할 수 있다.

해시 충돌을 해결하는 방법은 대표적으로 2가지가 존재한다.

[1] Separate Chaining (분리 연결법)

- 동일한 인덱스를 가지고 있으면 연결리스트를 사용하여 저장
- 단점의 경우, 데이터가 많아지면 데이터 검색하는데 시간이 오래 걸림

[2] Open Addressing (개방 주소법)

- Linear Probing: 원래 인덱스로부터 특정 칸만큼씩 이동함
- Double Hashing Probing: 해시된 값을 한 번 더 해싱함

### 3. 프로그래밍 언어에서 사용하는 해시 충돌 해결법

Load Factor = n / k (저장된 데이터 개수 / 해시 테이블 총 칸)

[1] 자바

HashMap의 경우, 처음에는 Separate Chaining을 통해 링크드리스트를 통해 해시 충돌을 해결함

충돌이 많이 일어나게 되면 링크드리스트에서 Red Tree Black으로 변경하여 검색/삭제/수정의 시간이 O(logN)이 걸리도록 보장함

Load Factor의 값이 3/4가 될 때 2배로 늘림

[2] 파이썬

Dictionary의 경우, Open Addressing에서 Linear Probing을 사용함

3.11 버전 기준으로 (5*index + 1) % len을 사용하여 충돌이 최소한으로 될 수 있도록 함

Load Factor의 값이 2/3이 될 때 2배로 늘림
