---
layout: post
title:  "[SpringBoot] Override clock"
<!-- date:   2019-03-21 23:55:00 -->
categories: jekyll update
---
### Temporary

```kotlin
// File: Service
@Service
class ServiceBlabla(
    val exampleRepository: ExampleRepository,
    val clock: Clock
) {

    fun test() {
        val currentDate = LocalDateTime.now(clock)
        val currentTime = Timestamp.valueOf(currentDate)

        val entity = ExampleEntity(1, currentTime,
            currentDate.format(DateTimeFormatter.ofPattern("yyyyMMdd")))

        exampleRepository.save(entity)
    }
}

```

```kotlin
// File: Test
@ExtendWith(SpringExtension::class)
@SpringBootTest
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class ServiceBlablaTests {

    @MockBean
    lateinit var repository: ExampleRepository

    @MockBean
    lateinit var clock: Clock

    @Autowired
    lateinit var serviceBlabla: ServiceBlabla

    fun clock() = Clock.fixed(Instant.parse("2019-03-03T00:00:00.00Z"), ZoneId.of("Asia/Tokyo"))

    @BeforeAll
    fun init() {
        given(clock.instant()).willReturn(clock().instant())
        given(clock.zone).willReturn(clock().zone)
    }

    @Test
    fun test1() {
        val time = Timestamp.valueOf(LocalDateTime.now(clock))
        val exampleEntity = ExampleEntity(
            1,
            time,
            "20190322"
        )

        given(repository.save(exampleEntity)).willReturn(exampleEntity)

        serviceBlabla.test()

        verify(repository).save(exampleEntity)
    }

    @Test
    fun test2() {
    }
}

```
