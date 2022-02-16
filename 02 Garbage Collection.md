# GC 란?

GC를 알아보기 전 stop-the-world 라는 용어를 살펴보자, GC을 실행하기 위해 JVM이 애플리케이션 실행을 멈추는 것이다. stop-the-world가 발생하면 GC를 실행하는 쓰레드를
제외한 나머지 쓰레드는 모두 작업을 멈춘다. 어떤 GC 알고리즘을 사용하더라도 발생 되며 GC작업이 완료된 이후에 다시 중단했던 작업을 시작한다. 대게 GC 튜닝은 이 stop-the-world
시간을 줄이는 것이다.

Java 프로그램 코드에서 메모리를 명시적으로 지정하여 해제하지 않는다. 그렇기에 이 작업을 GC가 더이상 필요없는 객체를 찾아 지우는 작업을 한다. 가비지 컬렉션은 두 가지 가설하에 
만들어 진 알고리즘이다.

- 대부분 객체는 금방 접근 불가능 상태가 된다.

- 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.

이 가설의 장점을 최대한 살리기 위해 HotSpotVM에서는 크게 2개의 물리적 공간으로 나누었다.

- Young 영역 : 새롭게 생성한 객체의 대부분이 여기에 위치한다. 대부분의 객체가 금방 접근 불가능 상태가 되기 때문에 매우 많은 객체가 Young영역에 생성되었다가 사라진다.
              이 영역에서 객체가 사라질 때 Minor Gc가 발생한다고 말한다.
              
- Old 영역 : 접근 불가능 상태로 되지 않아 young영역에서 살아남는 객체가 여기로 복사된다. GC가 적게 발생하고 이 영역에서 객체가 사라질 때 Major GC(혹은 Full GC)가 발생
            한다고 말한다.
            
            
흐름을 살펴보자 

![GC 흐름도](https://d2.naver.com/content/images/2015/06/helloworld-1329-1.png)

위 그림의 Permanent Generation 영역(현재는 Meta Space로 변경)은 Method Area라고도 한다. 객체나 억류된 문자열 정보를 저장하는 곳이며, Old 영역에서 살아남은 객체가 영원히 남아 있는 곳은 절대 아니다.
이 영역에서도 GC가 발생할 수 있는데 Major GC의 횟수에 포함된다.

현재는 `perm` 영역이 `metaSpace`로 변경 되었다. 그렇다면 어떤 부분이 바뀌었을 까?

| 구분 | Perm | Meta Space |
|----------|--------------------------------------------|------------------------------------------|
|저장 정보  | 클래스 meta, 메소드 meta, static 변수, 상수 | 클래스 meta, 메소드 meta                  |
|관리 포인트| Heap 영역 튜닝 + Perm 영역 별도             | Native 영역 동적 조정                     |
|GC        | Full GC                                   | Full GC                                   |
|메모리 측면| -XX: PermSize / -XX: MaxPermSize          | 	-XX: MetaSpaceSize, -XX: MaxMetaspaceSize|


## Young 영역의 구성

Young 영역은 3개의 영역으로 나뉜다.

- Eden 영역
- Survivor 영역(2개)

Survival 영역이 2개이기 때문에 총 3개의 영역으로 나뉘는 것이다. 각 영역의 처리 절차를 순서에 따라서 기술하면 다음과 같다. 

- 새로 생성한 객체는 대부분 Eden영역에 위치
- Eden 영역에서 GC가 한 번 발생한 후 살아남은 객체는 Survival 영역 중 하나로 이동한다.
- Suvival 영역으로 객체가 계속 쌓인다.
- 하나의 영역이 가득 차면, 그 중 살아남은 객체를 다른 Survival 영역으로 이동한다. 그리고 가득 찬 영역은 아무 데이터가 없는 상태로 된다.
- 이 과정을 반복하다가 계속 살아남은 객체는 Old로 이동한다.


> Young 영역에서 Old 영역으로 옮겨지는 기준
> 
> 오래 되었다고 하는 기준은 Young 영역에서 Minor GC가 발생하는 동안 얼마나 오래 살아남았는지로 판단한다. 각 객체는 Minor GC에서 살아남은 횟수를 기록하는 
> age bit가 있고, Minor GC가 발생할 때마다 age bit 값은 1씩 증가하게 되며, age bit이 설정 값을 초과하게 되는 경우 이동한다. 또한 설정 값이 초과하기 전이라도
> Survival 영역의 메모리가 부족할 경우에는 미리 Old 영역으로 옮겨질 수 있다.


이 절차를 확인해 보면 알겠지만 Survival 영역 중 하나는 반드시 비어 있는 상태로 남아 있어야 한다. 모두 데이터가 존재하거나 모두 0의 사용량이라면 정상적인 상황은 아닌 것이다.

![GC 전과 후 비교](https://d2.naver.com/content/images/2015/06/helloworld-1329-3.png)

참고로, HotSpotVM에서는 보다 빠른 메모리 할당을 위해서 두 가지 기술을 사용한다.

1. bump-the-pointer - 새로운 객체를 생성할 때 마지막에 추가된 객체만 점검하므로 매우 빠르게 메모리 할당이 이루어짐. 
2. TLABs - 멀티 스레드 환경에서 Thread-safe할때 lock이 발생하여 성능이 떨어지게 되는 것을 해결하기 위해 나온 기술
           각각의 스레드가 각각의 못에 해당하는 Eden 영역의 작은 덩어리를 가질 수 있도록 하는것.
           
           
## Old 영역의 구성

JDK 7을 기준으로 5가지 먼저 살펴보자

- Serial GC (운영 서버에서 절대 사용 금지)
- Parallel GC (JDK 8버전 기준 기본 GC)
- Parallel Old GC
- Concurrent Mark & Sweep GC
- G1GC (JDK 11 버전 기준 기본 GC)

Serial GC는 CPU 코어가 하나만 있을 때 사용하기 위한 방식이다. 사용 시 애플리케이션 성능이 많이 저하된다.

### Serial GC

Old 영역에서는 mark-sweep-compact라는 알고리즘을 사용한다. 첫 단계는 Old 영역에 살아 있는 객체를 식별(Mark)하는 것이고, 다음에는 힙의 앞 부분부터 확인하여 살아 있는 것만
남긴다(sweep). 마지막 단계에선 각 객체들이 연속되게 쌓이도록 힙의 가장 앞 부분부터 채워서 객체가 존재하는 부분과 없는 부분으로 나눈다(compaction).

### Parallel GC(병렬 GC)

Serial GC와 기본적인 알고리즘은 같지만 Parallel GC는 처리하는 쓰레드가 여러개이다. 그렇기 때문에 보다 빠르게 객체를 처리할 수 있고, 메모리가 충분하고 코어의 갯수가 많을 때
유리하다. 

다음은 Serial과 Parallel의 스레드를 비교한 그림이다.

![비교도](https://d2.naver.com/content/images/2015/06/helloworld-1329-4.png)

### Parallel Old GC

이 방식은 Mark-Summary-Compaction 단계를 거치며, 앞서 수행된 영역에 대해서 별도로 살아있는 객체를 식별한다. 더 복잡한 단계를 거친다.

### CMS GC

CMS GC는 좀 더 복잡한 방식을 갖고 있다.

먼저 그림으로 살펴보자

![Serial vs CMS](https://d2.naver.com/content/images/2015/06/helloworld-1329-5.png)

1. Initial Mark - 클래스로더에서 가장 가까운 객체 중 살아있는 객체만 찾는 것으로 끝낸다. 따라서 멈추는 시간은 매우 짧다.
2. Concurrent Mark - 방금 살아있다고 확인한 객체에서 참조하고 있는 객체들을 따라가면서 확인한다.  다른 스레드가 실행되고 있는 상황에서 진행한다.
3. Remark - Concurrent Mark 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다. 
4. Concurrent Sweep - 쓰레기를 정리하는 작업을 실행한다.  다른 스레드가 실행되고 있는 상황에서 진행한다.

이러한 단계로 진행 되어 stop-the-world 시간이 매우 짧다. 모든 애플리케이션의 응답 속도가 매우 중요할 때 CMS GC를 사용한다.

하지만 단점이 존재한다.

- 다른 GC방식보다 메모리와 CPU를 더 많이 사용함
- Compactation 단계가 기본적으로 제공되지 않음

### G1GC

G1GC는 바둑판의 각 영역에 객체를 할당하고 GC를 실행한다. 그러다가 해당 영역이 꽉차면 다른 영역에서 객체를 할당하고 GC를 실행한다. 위에서 설명한 Young의 세 가지 영역에서 데이터가
Old로 이동하는 단계가 사라진 방식이다. CMS GC를 대체하기 위해 만들어 졌다.

![G1GC](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FelKOzd%2FbtqMmqzdfAz%2F1sJEe9uXkg0YEuQXDOqTr0%2Fimg.png)

가장 큰 장점은 성능이다. 지금 까지 등장한 GC 방식보다 빠르다. JDK 11부턴 기본 가비지 컬렉터로 사용되고 있으며 목표는 대기 시간과 처리량간의 균형을 유지하는 것이다.
G1GC는 높은 처리량을 달성하려고 노력합니다.  

새로 추가된 영역을 살펴보자.

- Humonogous : 지역 크기의 50%를 초과하는 큰 객체를 저장하기 위한 공간
- Available/Unused : 아직 사용되지 않은 지역

G1GC에도 마찬가지로 Minor GC가 존재하며, 과정에서 살아남은 객체들은 Survival로 옮기고, Eden에 대한 영역을 사용 가능한 지역으로 돌리는 형태로 과정이 일어난다.
반면에 Full GC와 유사한 Concurrent Cycle이라는 과정이 존재한다.

![G1GC 과정](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbWj6yv%2FbtqMqBft2cj%2FT69AhDZYGwLomksxpLb82k%2Fimg.png)

 initial Mark : Old 지역에 객체들이 참조하는 Survival 지역을 찾는다.(STW)
 
 Root Region Scan : 위에서 찾은 Servival 객체들에 대한 스캔 작업 실시
 
 Concurrent Mark : 전체 Heap의 scan을 실시하고, GC 대상 객체가 발견되지 않은 Region은 이 후 단계를 제외한다.
 
 Remark : 애플리케이션을 멈추고(STW) 최종적으로 GC 대상에서 제외할 객체 식별
 
 Clean up : 애플리케이션을 멈추고(STW) 살아 있는 객체가 가장 적은 지역에 대한 미사용 객체를 제거
 
 Copy : GC대상의 Region 이었지만, 완전히 비워지지 않은 지역의 살아남은 객체들을 새로운 지역에 복사하여 Compaction을 수행
