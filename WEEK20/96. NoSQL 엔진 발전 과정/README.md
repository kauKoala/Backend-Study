## 0. 공부하게 된 이유

하둡 수업에서 SSTable이 나왔는데, 시험 공부하다보니까 SSTable이 NoSQL과 밀접한 관련이 있다는 것을 알게 되었음

이번 내용은 해시 테이블, B-Tree / B+Tree를 알고 있으면 이해하기 좋음. 왜냐하면 읽기 성능이 좋은 기술은 B+Tree이고, 쓰기 성능이 좋은 것은 이번 수업에 다룰 SSTable과 LSM Tree임

우리가 배운 CS 지식들을 베이스로 활용하는 개념이라 시간만 들인다면 충분히 이해할 수 있음

---

<br/>
<br/>
<br/>


## 1. 잠깐 보고 넘어가는 NoSQL VS RDB

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/35cf5fc6-d93e-4427-9853-03d2829cd23a)

링크: https://github.com/backtony/Backend_Interview_for_Beginner

깃허브에 있는 CS 레포중에 하나를 가져옴

글을 읽으면서 생각해볼만한 점

- 왜 수평적 확장이 가능할까?
- 왜 대용량 처리가 가능할까?

위에 잘못된 내용도 있음

- 데이터를 자주 읽는 경우 → RDB 사용하는게 성능이 좋음 (B Tree는 범위 검색애 유리)
- 데이터를 자주 쓰는 경우 → NoSQL 사용하는게 성능이 좋음

---

<br/>
<br/>
<br/>

## 2. 시작: 로그 데이터 저장

로그 데이터는 아래 내용으로 계속 설명할 예정

```cpp
| key  | value |
| 권민재 | 1     |
| 우찬희 | 2     |
| 이상재 | 3     |
| 정다빈 | 4     |
| 이상재 | 5     |
....
```

일반적인 로그 데이터는 보통 적재만 하지, 데이터 삭제/수정을 가하지는 않음

= Append Only File (AOF)

AOF가 가장 원시적인 NoSQL의 기본적인 방법으로 명령어는 처음에 set, get만 있다고 가정함

- set: 데이터를 추가한다
- get: 데이터를 읽는다.

[set 동작 방식]

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/bcd75ceb-bf21-46bd-bfa4-d9f4cafe7500)

[get 동작 방식]

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/e7641b13-b07f-40de-8cac-6b7b97da09f3)

여기서 AOF 기반 데이터베이스 엔진을 만든다면 이런 특징을 가지고 있음

- 쓰기 연산: 파일 맨 마지막에 추가만 하면 되므로 빠르게 동작한다.
- 읽기 연산: 맨 아래에서부터 데이터를 찾아보기 때문에 시간이 오래 걸린다.

그래서 NoSQL은 AOF 기반 파일로 동작하기 때문에 쓰기 성능이 RDB보다 높음

---

<br/>
<br/>
<br/>

## 3. 발전 과정

### [1] File Segment

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/cbea4a78-23ef-4129-bb54-2c30968a670b)

하나의 파일에 데이터만 계속 추가하게 된다면 어떻게 될까?

지금 상태는 디스크 Scale Up을 통해야만 데이터를 더 적재할 수 있음

Scale Out을 적용하기 위해 파일을 분리하는 방법을 사용함

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/194d857f-5e1c-42c5-b727-53a0b04642ed)

이제 여러 디스크에 나눠서 데이터 저장을 할 수 있음

<br/>

### [2] Compaction

파일 용량을 줄이고 싶음 → AOF 파일 특징을 이용함

파일마다 중복된 내용이 있는데 NoSQL은 최신순으로 보기 때문에 파일별 중복된 내용은 제거 가능

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/2c013fd7-2d62-405d-9e30-955c5c295d3d)

<aside>
💡 각 파일의 중복된 내용만 삭제하는 이유 (모든 이전 데이터를 삭제하지 않는 이유)

만약 이전 모든 데이터를 삭제한다면 아래 과정이 필요해서 시간이 더 오래 걸림
1. 현재 파일이 최신 파일인지 확인한다
2. 파일 안에 모든 데이터를 확인해보며 삭제해도 문제 없는 데이터인지 확인한다 (데이터마다 get 호출)

</aside>

<br/>

### [3] Sorted String Table (SSTable), LSM Tree

위에 예시는 하나의 파일에 데이터를 5개씩 넣었지만, 실제 파일에는 엄청 많은 데이터가 들어가 있을 예정임

어떻게하면 파일마다 탐색 속도를 올릴 수 있을까??

- 코딩테스트: 완전 탐색 → (데이터가 정렬되어 있다) → 이진탐색으로 O(N) → O(logN) 사용 가능

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/6a1cb056-c057-433f-aa45-10a484233f04)

최신 순서로 파일을 읽어야한다는 내용은 변함 없지만, 그래도 각 파일마다 데이터를 읽는 것은 O(logN) 탐색이 가능해짐

이때 정렬된 값을 넣는 방식은 레드블랙트리, AVL 트리와 같은 균형 이진 트리를 사용함

여기서 그러면 하나를 더 개선 가능함

<aside>
💡 파일이 너무 많아지는데, 디스크에 저장된 파일들은 하나로 합칠 수 있는거 아닌가요?
왜냐하면 파일 접근하는데도 리소스가 들잖아요

</aside>

<br/>

<br/>

![스크린샷 2024-06-23 오전 10 36 07](https://github.com/kauKoala/Backend-Study/assets/79046106/5f8c1de2-6758-47bc-a169-5612ccd295cf)

어? 정렬되어 있네? 어? 머지 소트 사용 가능하네? 어 파일 합치기 가능하네?

여기서 궁금한 점이 생김

<aside>
💡 그러면 이제 데이터를 추가하면 NoSQL 근본인 AOF를 사용 불가능한게 아닌가요?
→ 어는정도 맞는 말임. 바로 이어서 설명

</aside>

### [4] 메모리를 사용하자 + failover

디스크에서 정렬한다는건 속도를 어느정도 포기한다는 것과 비슷한 말이니 메모리를 사용한다

메모리는 속도가 굉장히 빠름 → ex. Redis

메모리에서 정렬을 하고, 디스크는 정렬된 데이터를 그대로 쓰기만 하면 AOF 컨셉을 그대로 이어나갈 수 있음

이러면 이제 데이터를 읽는 방법도 달라짐

[AS-IS]

1. 디스크에 최신순으로 저장된 파일부터 읽는다

[TO-BE]

1. 메모리에 저장된 데이터부터 확인한다
2. 디스크에 최신순으로 저장된 파일부터 읽는다

하지만 메모리는 컴퓨터가 다운되면 데이터가 모두 휘발된다는 단점을 가지고 있기 때문에 메모리 데이터를 복구하는 과정이 필요

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/6a502c92-9e9b-47f5-a39f-3631b8337c64)

디스크에 AOF처럼 저장하다가 메모리에 있는 내용이 디스크로 넘어가게 되면 삭제하고 다시 메모리에 올라온 데이터를 적재함

---

<br/>
<br/>
<br/>

## 4. 이후의 성능 최적화

---

여기까지 글을 읽어도 결국 데이터는 최신 파일 순서대로 읽어야한다는 단점이 존재함

### [1] 블룸 필터 (Bloom Filter)

```cpp
[개념]

특정 원소가 어떤 집합 안에 속하는지 여부를 검사하는데 사용하는 방법
```

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/a01825cd-15ab-4c3e-8674-2cac6b5a6164)

방법은 간단하게 해시 함수를 사용함

예를 들어서 ‘ribeye’라는 단어가 들어오면 위 사진은 3개의 해시 함수를 사용해서 값을 추출함

- 첫번째 해시 값: 1
- 두번째 해시 값: 3
- 세번째 해시 값: 4

배열에 arr[1] = 1, arr[3] = 1, arr[4] = 1을 가지고, 각 파일마다 들고 있는 배열 값을 서로 비교함

모두 일치하게 된다면 값이 있다고 판정

<aside>
💡 근데 해시는 해시 충돌 문제가 있으니까 배열값이 우연하게 모두 1일 수 있는거 아닌가요?

</aside>

맞음. 그래서 실제 데이터는 없는데, 있다고 판단하는 False Positive 문제점이 존재함

대신 데이터 여부는 O(logN)으로 빠르게 판별 가능하니까 (파일에 데이터가 있냐/없냐) 이 프로세스만 개선해도 큰 효과를 보일 수 있음

반대로 실제 데이터는 있는데, 없다고 하는 케이스는 존재하지 않음. 왜냐하면 단어를 저장할 때 해싱해서 arr[x] = 1를 저장하게 되는데, 검색 했을 때 arr[x] = 0인 상황이 존재할 수가 없음

<br/>

[2] 파일 병합 방법

![스크린샷 2024-06-23 오전 10 36 07](https://github.com/kauKoala/Backend-Study/assets/79046106/5f8c1de2-6758-47bc-a169-5612ccd295cf)

병합하는 것은 알겠는데, 어떤 조건에 따라 병합을 할까??

이건 실제 NoSQL 기술마다 달라진다.

1. Size Tiered Compaction Strategy (크기 계층 컴팩션)

오래된 SSTable에 계속 병합을 진행한다

ex) HBase, Cassandra

2. Leveled Compaction (레벨 컴팩션)

레벨 별로 파일을 분류하는데, 단계가 높아질수록 오래되고 큰 파일임

오래된 파일을 개별 레벨을 가진다는 특징을 가지고 있음

파일이 어느정도 쌓이면 해당 파일의 레벨을 승격해서 같은 레벨의 오래된 파일에 병합을 진행함

ex) Level DB, Rocks DB

<br/>

### [3] 이후의 내용

![image](https://github.com/kauKoala/Backend-Study/assets/79046106/75809754-8608-4696-a2c1-9bad3eab8f08)

이외에도 실제 기술에 적용된 다양한 방법이 있겠지만, 중요한 부분은 얼추 다 살펴보아서 나머지 내용은 각 기술 공식 문서를 참조

---

<br/>
<br/>
<br/>

## 5. 해당 기술을 사용하는 NoSQL

---

1. Mongo DB (Log Structure)
2. HBase (Hadoop Ecosystem)
3. Apache Cassandra
4. Level DB
5. Rocks DB
6. Google Bigtable

AOF는 다른 곳에도 쓰일까

- Redis 장애가 일어나면 복구하는 방법 → AOF

---

## 6. 출처 및 참고하면 좋은 책

---

- 데이터 중심 애플리케이션 설계

아래 책에 있는 내용을 정리한 블로그 글, 유튜브 영상을 참고함

책 후기들을 살펴보니 호평 일색이라 나중에 읽어보면 좋은 것 같음

블룸 필터

- [ByteByteGo - Bloom Filters](https://youtu.be/V3pzxngeLqw?feature=shared)
- [네이버 D2 기술 블로그 - 확률적 자료구조를 이용한 추정 - 원소 포함 여부 판단과 Bloom Filter](https://d2.naver.com/helloworld/749531)
- 가상 면접 사례로 배우는 대규모 시스템 설계 기초 책에도 나와있음

---
