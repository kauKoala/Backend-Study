## 1. 엔터프라이즈에서 사용하는 Batch란?

---

비즈니스 운영에 필수적인 작업을 벌크 프로세스로 개발하는 것

- 월말 정산 처리
- 보험료 조정
- 데이터 통합

→ 특정 시간 주기로 처리해도 문제 없는 작업

## 2. Spring Batch

---

회사에서는 벌크 프로세스를 개발한다면 재사용 가능한 배치 아키텍처에 대한 표준이 없어서 사내에서 일회용으로 개발하는 경우가 많았음. 그래서 이걸 해결하기 위해 대용량 데이터 처리에 필수적인 기능을 재사용할 수 있는 형태로 제공하고, 파티셔닝/최적화와 같은 고성능 배치도 쉽게 구현할 수 있게 만들었음. 그래서 개발자는 스프링 배치를 사용함으로써 비즈니스 로직에 집중하고, 기반 구조는 프레임워크가 관리하게 함

![image](https://github.com/user-attachments/assets/139fb96a-5597-40fd-a767-c2b3fb94b393)

### Job

- 전체 배치 프로세스를 캡슐화한 엔티티
    - ex) 사용자들 보험료를 조회 > 사용자들 보험료를 업데이트 > 사용자들 보험료를 반영
    위와 같은 작업을 하나의 배치라면 이게 하나의 Job이고, 하나의 트랜잭션이라고 볼 수 있음
    - 여러 Step을 합친 컨테이너 개념임
    Step들을 결합 가능하고, 재시작을 Job에서 구성 가능함

### Step

- Job의 단계를 캡슐화한 객체
- 여기서 우리가 운영에 필요한 배치 작업을 설정하고 수행함

### 예시 상황 설명

애완동물 간식을 구매한 이력이 있는 구매자들에게 애완동물 간식 쿠폰을 전송하고, 알람을 보내야하는 상황

1. Step을 두 개로 나눌 수 있을 것 같음
1) 애완동물 간식을 구매한 이력이 있는 구매자들을 조회 후, 그 사람들에게 애완동물 간식 쿠폰을 전송
2) 애완동물 간식 쿠폰을 받은 사람들을 조회 후, 그 사람들에게 알람을 전송
2. Job은 2개의 Step을 일괄적으로 수행할 수 있게함

이때 실제 실행은 JobLauncher가 하지만, 지금까지 내용에서는 Job이 MainApplication.java와 같은 역할을 한다고도 생각하면 좋음

## Job 상세 설명

---

![image](https://github.com/user-attachments/assets/4fb60d60-1b83-43a6-a30b-18a1e3ac1815)

### JobInstance

- 논리적인 실행의 단위
- 만약 12시간마다 한 번씩 실행되는 Job이 있을 때, 12시간마다 JobInstance가 생성댐
- 이때 Job 실행이 실패하면 새로운 JobInstance가 생성되는게 아닌, 실패한 JobInstance를 재실행함
- JobInstance = Job + JobParameters로 구분할 수 있음
(단순 JobInstance로만 하면 다른 JobInstance들과 구분할 수 없어서 사용함)

### JobParameters

- Job을 실행할 때 JobInstance를 참조할 수 있게하거나
Job을 실행할 때 특정값을 넣을 수 있는 용도로 사용함 (= 메서드 파라미터와 비슷함)
- String, Double, Long, Date 형식을 지원해서 하루에 한 번 실행하는 Job은 Date를 사용해서 JobInstance를 식별할 수도 있음

### JobExecution

- Job이 한 번 실행되면 생성되는 객체 (실행 기록)
- ex) Job이 실패하고 재시도하면 JobInstance는 하나지만, JobExecution은 2개가 생성
- 보통 JobExecutionListener와 같이 사용해서 연계함 (성공/실패와 관련된 정보들이 있음)

### JobExecutionListener

- Job 실행 전/후로 로깅이 가능한 객체
- beforeJob, afterJob 메서드를 지원함 (= interceptor랑 비슷함)

![image](https://github.com/user-attachments/assets/f1f8b535-a199-4b05-9f03-733bc89f4548)

## Step 상세 설명

---

![image](https://github.com/user-attachments/assets/efea28c7-0634-4fbd-a57c-14510ce023a6)

### StepExecution

- JobExecution처럼 하나의 Step 실행 시도를 의미하는 객체
    - 주의점: 이전 단계 Step이 실행되지 않았으면 그 다음 단계 Step은 실행되지 않았으므로 Execution이 없음

### StepContribution

- StepExecution가 가지고 있는 정보를 업데이트 할 수 있는 객체
- 예를 들어 100명 사용자의 월말 정산 처리를 할 때, 10개씩 데이터를 불러와서 처리하면 몇명까지 업데이트 했는지 기록이 가능함

![image](https://github.com/user-attachments/assets/f90a3264-49c7-45e3-860a-6e8a59459433)

### StepExecutionListener

- JobExecutionListener처럼 Step 실행 전/후로 기록이 가능한 객체

## 그 외 내용의 설명

### JobLauncher

```java
@FunctionalInterface
public interface JobLauncher {

	JobExecution run(Job job, JobParameters jobParameters) throws JobExecutionAlreadyRunningException,
			JobRestartException, JobInstanceAlreadyCompleteException, JobParametersInvalidException;

}
```

JobRepository에서 유효한 JobExecution을 조회하고, Job을 실행하는 기능을 구현함

### JobRepository

그동안 위에 언급된 모든 저장(persistence) 메커니즘을 담당함

- JobInstance, JobExecution, StepExecution 에 대한 구현체에 CRUD 기능을 제공
- Job을 시행할 때, JobRepository에서 JobExecution을 조회하고, 실행중에는 StepExecution, JobExecution 구현체를 JobRepository에 저장함
- persistence = StepExecution, JobExecution, JobInstance 이런건 Entity로 되어 있음

### ExecutionContext

- 프레임워크에서 유지/관리하는 Map<String, Object>를 가지고 있는 객체
- JobExecution, StepExecution 객체에 속하는 상태를 저장함
- ExecutionContext는 Execution이 재시작을 용이하게 해주는 용도로 쓰임 (= 예기치 못한 에러가 발생했을 때, 에러가 났던 데이터부터 다시 처리할 수 있게 해줌)

## 처리 방식

### 1. 청크 기반 처리

### 2. 작업 기반 처리

모두 [이 링크](https://harvest-redcurrant-ad7.notion.site/74-Spring-Batch-1-dbaba6e2cd5946ea9aa7af50fd7b3fd9?pvs=4)가 잘 써져있음
