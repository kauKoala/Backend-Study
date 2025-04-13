## 1. Spring Batch 4 → 5 변경점

스프링 부트 3.x로 넘어가면서 Spring Batch도 자동으로 5.x 버전을 사용하게 됐는데 이때 코드 변경점이 생김

1. JobBuilderFactory, StepBuilderFactory → JobBuilder, StepBuilder로 변경

[기존]

```java
@Configuration
@RequiredArgsConstructor
public class TrMigrationJobConfig {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    
    ...
    
    @Bean
    public Job trMigrationJob(Step trMigrationStep) {
        return jobBuilderFactory.get("trMigrationJob")
                .incrementer(new RunIdIncrementer())
                .start(trMigrationStep)
                .build();
    }
    
    ...
    
}
```

[변경]

```java
@Configuration
@RequiredArgsConstructor
public class TrMigrationJobConfig {
    
    ...
    
    @Bean
    public Job trMigrationJob(Step trMigrationStep) {
        return JobBuilder("trMigrationJob", jobRepository)
                .incrementer(new RunIdIncrementer())
                .start(trMigrationStep)
                .build();
    }
    
    ...
    
}
```

1. JobParameter는 무조건 Long, Double, String, Date만 가능 → Generic으로 다양한 값을 넣을 수 있도록 변경

[기존]

![image](https://github.com/user-attachments/assets/5335a638-0b9d-4535-af5d-b7903f1af771)

![image](https://github.com/user-attachments/assets/9d555e78-2f47-408b-a45c-0ccee93d4ee3)

[변경]

![image](https://github.com/user-attachments/assets/de132285-0c57-49fb-bd43-0cc250570c33)

이외에도 여러가지가 있는데, 주로 사용하는 내용만 적음

TMI) 이거 찾아보면서 팀이 스프링 배치 4.x.x 버전과 5.x.x 버전 둘 다 사용하는걸 알게 댐

## 2. 스프링 배치 코드 작성 예시

### [1] Tasklet 으로 처리하기 (외부 API)

![image](https://github.com/user-attachments/assets/ca160eb2-115c-4d64-ac79-04bc1a2c17f9)

```kotlin
class DeleteMemberJobConfig (
	private val jobRepository: JobRepository,
	private val batchTransactionManager: PlatformTransactionManager,
	private val memberDataSource: DataSource
	private val memberTransactionManager: PlatformTransactionManager,
	private val chunkSize: Int
) {

	@Bean
	fun deleteMemberJob(): Job {
		return JobBuilder("deleteMemberJob", jobRepository)
			.start(deleteMemberStep())
			.listener(LoggingJobExecuteListner())
			.build()
	}
	
	@Bean
	@JobScope
	fun deleteMemberStep(): Step {
		return StepBuilder("deleteMemberStep", jobRepository)
			.tasklet(
				DeleteMemberTasklet(
					memberDataSource,
					memberTransactionManager,
					chunkSize
				),
				batchTransactionManager)
			.build()
	}

}

```

```kotlin
class DeleteMemberTasklet(
	private val dataSource: DataSource
	private val transactionManager: PlatformTransactionManager,
	private val chunkSize: Int
) : Tasklet {

	...
	
	private var processing: Boolean = true
	private var page: Int = 1
	
	override fun execute(contribution: StepContribution, chunkContext: ChunkContext): RepeatStatus {
		if (processing) {
			val deleteMembers = memberTeamOuterApi.getDeleteMembers(today, chunkSize, page)
				
			if (deleteMember.size() < chunkSize) {
				processing = false
			}
		}
		
		val deleteMemberIds = deleteMembers
			.map(OuterMemberTeam::id)
			.toList()
		
		val params = MapSqlParameterSource()
			.addValue("deleteMemberIds", deleteMemberIds)
			.addValue("deletedAt", LocalDateTime.now())
		
		val sql = """
			UPDATE member
			SET deleted_at = :deletedAt
			WHERE memberPlatformId IN (:deleteMemberIds) AND deleted_at is NULL
		""".trimIndent()
		
		jdbcTemplate.update(sql)
		
		return RepeatStatus.continueIf(processing)
		// if (processing) {
		//   return RepeatStatus.CONTINUABLE
		// }
		// return RepeatStatus.FINISHED
	}

}
```

### [3] Chunk로 처리하기 (내부 데이터 처리)

회원 등급을 Normal → VIP로 변경

```kotlin
class UpgradeMemberJobConfig (
	private val jobRepository: JobRepository,
	private val batchTransactionManager: PlatformTransactionManager,
	private val memberDataSource: DataSource
	private val memberTransactionManager: PlatformTransactionManager,
	private val chunkSize: Int
) {

	@Bean
	fun upgradeMemberJob(): Job {
		return JobBuilder("upgradeMemberJob", jobRepository)
			.start(UpgradeMemberStep())
			.listener(LoggingJobExecuteListner())
			.build()
	}
	
	@Bean
	@JobScope
	fun UpgradeMemberStep(): Step {
		return StepBuilder("upgradeMemberStep", jobRepository)
			.chunk<Member, Long>(chunkSize, batchTransactionManager)
			.reader(UpgradeMemberReader(memberDataSource, chunkSize)
			.processor { it.id }
			.writer(UpgradeMemberWriter(memberDataSource, memberTransactionManager)
			.build()
	}
}
```

```kotlin
class UpgradeMemberReader(
	private val dataSource: DataSource,
	private val chunkSize: Int
) : ItemReader<Member> {
	
	...
	
	private var memberIterator: Iterator<Member> = emptyList<Member>().iterator()

	override fun read(): Member? {
		if (!memberIterator.hasNext()) {
			memberIterator = readMembers()
		}
		
		return memberIterator.takeIf { it.hasNext() }?.next()
	}
	
	private fun readMembers(): Iterator<Member> {
		val sql """
			SELECT id, status
			FROM member
			WHERE id > $offsetId AND today_spend_money > 100 AND status = "normal"
			ORDER BY id
			LIMIT $chunkSize
		""".trimIndent()
		
		val members = jdbcTemplate.query(sql) { rs, _ ->
			Member(
				rs.getLong("id"),
				rs.getString("status")
			)
		)
		
		return members.iterator()
	}
}
```

```kotlin
class UpgradeMemberWriter(
	private val dataSource: DataSource,
	private val transactionManager: TransactionManager
) : ItemReader<Member> {
	
	...
	
	private val transactionTemplate = TransactionTemplate(transactionManager)
	
	override fun writer(targetIds: Chunk<out Long>) {
		transactionTemplate.execute {
			val sql = "UPDATE member SET status = "VIP" WHERE id in (:targetIds)"
			jdbcTemplate.update(sql, MapSqlParameterSource("targetIds", targetIds))
		}
	}
}
```

## 이후의 영상 추천

![image](https://github.com/user-attachments/assets/7a6df966-d24b-45bd-9ff7-4f6153284418)

![image](https://github.com/user-attachments/assets/4436b072-4368-40a4-8062-b5cc6142e480)
