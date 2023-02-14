## 기본 개념
### Mark, Sweep, Compact
- Mark: 가비지 컬렉션의 대상을 마킹하는 작업, 즉 더 이상 참조되지 않은 객체인가
- Sweep: TBD
- Compact: TBD

### Young Generation (Eden, S0, S1), Old Generation
#### Young Generation
TBD
#### Old Generation
TBD

### Minor GC, Full GC
#### Minor GC
- Young Generation의 GC
#### Full GC
- Young + Old Generation의 GC (가비지 컬렉션의 꽃)

## 가비지 컬렉션 로직을 보기 위한 관점(필수 기준)
### Throughput
- 메모리 할당 및 가비지 컬렉션 대비 유용한 응용 프로그램 작업에 소요된 총 시간의 백분율
- 예를 들어 처리량이 95%인 경우 응용 프로그램 코드가 95% 실행되고 가비지 컬렉션이 5% 실행되고 있음을 의미
- 고부하 비즈니스 애플리케이션에 대해 더 높은 처리량을 원함
### Latency (Stop-The-World)
- 가비지 수집 일시 중지의 영향을 받는 응용 프로그램 응답성
#### Stop The World
TBD
### Footprint
- 페이지 및 캐시 라인으로 측정된 프로세스의 작업 세트

## GC의 종류
### Serial
- 단일 스레드에서 모든 작업을 수행
- 여러 스레드 간에 통신 오버헤드가 없기 때문에 효율성을 높힐 수 있음
- 단일 프로세서 시스템에 가장 적합
- 활성 옵션: `-XX:+UseSerialGC`
### Parallel
- 단일 스레드를 사용하는 Serial GC와는 달리 여러 개의 쓰레드를 사용
- 대기 시간보다 처리량을 중요한 경우 최선의 선택이 될 수 있기 때문에 Throughput GC라고도 부른다
- 애플리케이션 요구 사항이 최고의 처리량을 달성하는 것이고 1초 이상의 일시 중지가 허용되는 경우 적합할 수 있음
- 활성 옵션 : `-XX:+UseParallelGC`
### G1
TBD
### ZGC
TBD

## Default GC by Java Version
### Java 8
- Parallel
### Java 11
- G1
### Java 17
- ZGC

## Reference
- 인프런 백기선 이펙티브 자바 1편
- [How to choose the best Java garbage collector](https://developers.redhat.com/articles/2021/11/02/how-choose-best-java-garbage-collector#)
- [Parallel Collector](https://docs.oracle.com/en/java/javase/11/gctuning/parallel-collector1.html)