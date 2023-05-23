# 가비지 컬렉션(Garbage Collection)

## 개념

프로그램을 실행하다보면 'Garbage' 즉, 쓰레기가 발생하게 되는데 이는 쉽게 말하자면 정리되지 않은 메모리, 유효하지 않은 주소다.

```java
int[] arr = new int[3];

arr[0] = 1;
arr[1] = 2;
arr[2] = 3;

arr = new String[3];

arr[0] = "Java";
arr[1] = "JavaScript";
arr[2] = "Node.js";
```

위의 코드를 보면 `int`타입의 배열 `arr`를 사용하다가 `String`타입의 배열을 만들고 `arr`가 이를 가리키도록 하고 있다.

여기서 처음 생성했던 `int`타입의 배열은 **사용할 수 없는 메모리**가 되고 이를 **프로그래밍 언어로는 `Dangling Object`, 자바에서는 `Garbage`라고 한다.**

<br/>

## GC(Garbage Collector)의 구조

`Garbage Collector`가 담당하는 메모리 영역은 [JVM](https://github.com/Im-D/Dev-Docs/blob/master/Java/JVM(Java%20Virtual%20Machine).md)에서 `Heap`영역을 다룬다.

`Young`, `Old`, `Permanent` 세 영역으로 나뉘게 되며 이를 세분화 하면 총 4개의 영역으로 나뉘게 된다.

- **Young** : `Eden`, `Survivor`
- **Old** : `Old`
- **Permanent** : `Permanent(이하 Perm)`

<br/>


## GC(Garbage Collector)의 필요성

가비지 컬렉터는 알아서 메모리 관리를 해준다는 장점이 있지만, 반대로 수동으로 메모리 해제를 할 수 없기 때문에 발생하는 메모리 누수 문제

가 있다. Closure는 Mark And Sweep 알고리즘을 이용해 실행이 끝난 함수의 Lexical Environment를 메모리에서 해제하지 않고 유지한다. 즉,

Closure의 필요성을 마친 뒤에도 ‘필요없음’을 가비지 컬렉터에게 알리지 않는다면 메모리 누수가 발생하게 되는 것이다!

<br/>


## GC(Garbage Collector)의 작동 원리

GC의 역할의 공통적인 원리는 `Heap`내의 객체 중 `Garbage`를 찾아 이를 처리하고 메모리를 회수한다는 것이다.

이 과정에서 객체가 `Garbage`인지 아닌지를 판단하기 위해 `reachability`개념을 사용한다. 보통 하나의 객체는 다른 객체를 참조하고 그 객체는 

또 다른 객체를 참조하게 되는데 여기서 **최초에 참조한 객체를 `Root Set`이라고 한다.**

![gc_process](https://github.com/minjae990/BIG-C-GC/assets/119609098/1d6cd8b9-c980-4cc0-8e46-88613124bdf4)

위의 그림과 같이 각각의 객체가 서로 참조하면서 참조 사슬이 형성되는데 `Root Set`에서 참조 사슬에 의해  `Unreachable`한 객체들은 **Garbage Collection**의 대상이 된다.

<br/>

## GC(Garbage Collector)의 올바른 동작 코드

JVM이 메모리를 자동으로 관리해주는 것은 개발자의 입장에서 상당한 메리트이다. 하지만 문제는

GC를 수행하기 위해 Stop The World(STW)에 의해 애플리케이션이 중지되는 것에 있다. Heap 의 사이즈가 커지면서 애플리케이션의 지연 현상이 두드러지게 되었고,

이를 막기 위해 다양한 GC 알고리즘을 지원하고 있다.

### 1. Serial GC

Serial GC의 Young 영역은 앞서 설명한 알고리즘 대로 수행된다. 하지만 Old 영역에서의 Mark Sweep Compact 알고리즘이 사용되는데, 기존의 Mark Sweep에

Compact라는 작업이 추가되었다. Compact는 Heap 영역을 정리하기 위한 단계로 유요한 객체들이 연속되게 쌓이도록 힙의 가장 앞 부분부터

채워서 객체가 존재하는 부분과 객체가 존재하지 않는 부분으로 나누는 것이다.
```
java --XX: +UseSerialGc -jar Application.java
```
Serial GC는 서버의 CPU 코어가 1개일 때 사용하기 위해 개발되었으며, 모든 가비지 컬렉션 일을 처리하기 위해 1개의 쓰레드만을 이용한다. 

그렇기 때문에 CPU의 코어가 여러 개인 운영 서버에서 Serial GC를 사용하는 것은 반드시 피해야 한다.

<br/>

### 2. Parallel GC

Parallel GC는 Throughput GC로도 알려져 있으며, 기본적인 처리 과정은 Serial GC와 동일하다. 하지만 Parallel GC는 여러 개의 쓰레드를 

통해 Parallel하게 GC를 수행함으로써 GC의 오버헤드를 상당히 줄여준다. Parallel GC는 멀티 프로세서 또는 멀티 쓰레드 머신에서 중간 

규모부터 대규모의 데이터를 처리하는 애플리케이션을 위해 고안되었으며, 옵션을 통해 애플리케이션의 최대 지연 시간 또는 

GC를 수행할 쓰레드의 갯수 등을 설정해줄 수 있다.

```
java -XX: +UseParallelGC -jar Application.java

// 사용할 쓰레드의 갯수
-XX:ParrallelGCThreads=<N>

// 최대 지연 시간
-XX:MaxGCPauseMills=<N>
```
Parallel GC가 GC의 오버헤드를 상당히 줄여주었고, Java8까지 기본 가비지 컬렉터(Default Garbage Collector)로 사용되었다. 

그럼에도 불구하고 Application이 멈추는 것은 피할 수 없었고, 이러한 부분을 개선하기 위해 다른 알고리즘이 더 등장하게 되었다.

<br/>

### 3. G1(Garbage First) GC

G1(Garbage First) GC는 장기적으로 많은 문제를 일으킬 수 있는 CMS GC를 대체하기 위해 개발되었고, Java7부터 지원되기 시작하였다.

기존의 GC 알고리즘에서는 Heap 영역을 물리적으로 Young 영역(Eden 영역과 2개의 Survivor 영역)과 Old 영역으로 나누어 사용하였다. 

G1 GC는 Eden 영역에 할당하고, Survivor로 카피하는 등의 과정을 사용하지만 물리적으로 메모리 공간을 나누지 않는다. 

대신 Region(지역)이라는 개념을 새로 도입하여 Heap을 균등하게 여러 개의 지역으로 나누고, 각 지역을 역할과 함께 논리적으로 

구분하여(Eden 지역인지, Survivor 지역인지, Old 지역인지) 객체를 할당한다.

![2222](https://github.com/minjae990/BIG-C-GC/assets/119609098/158b4a14-0719-4336-ba71-b63e90699f34)

G1 GC에서는 Eden, Survivor, Old 역할에 더해 Humongous와 Availabe/Unused라는 2가지 역할을 추가하였다. 

Humonguous는 Region 크기의 50%를 초과하는 객체를 저장하는 Region을 의미하며, Availabe/Unused는 사용되지 않은 Region을 의미한다. 

G1 GC의 핵심은 Heap을 동일한 크기의 Region으로 나누고, 가비지가 많은 Region에 대해 우선적으로 GC를 수행하는 것이다.

그리고 G1 GC도 다른 가비지 컬렉션과 마찬가지로 2가지 GC(Minor GC, Major GC)로 나누어 수행되는데, 각각에 대해 살펴보도록 하자.

<br/>

### 3-1. Minor GC

한 지역에 객체를 할당하다가 해당 지역이 꽉 차면 다른 지역에 객체를 할당하고, Minor GC가 실행된다. G1 GC는 각 지역을 추적하고 있기 때문에,

가비지가 가장 많은(Garbage First) 지역을 찾아서 Mark and Sweep를 수행한다.

Eden 지역에서 GC가 수행되면 살아남은 객체를 식별(Mark)하고, 메모리를 회수(Sweep)한다. 그리고 살아남은 객체를 다른 지역으로 이동시키게 된다.

복제되는 지역이 Available/Unused 지역이면 해당 지역은 이제 Survivor 영역이 되고, Eden 영역은 Available/Unused 지역이 된다.

<br/>

### 3-2. Major GC(Full GC)

시스템이 계속 운영되다가 객체가 너무 많아 빠르게 메모리를 회수 할 수 없을 때 Major GC(Full GC)가 실행된다. 

그리고 여기서 G1 GC와 다른 GC의 차이점이 두각을 보인다.

기존의 다른 GC 알고리즘은 모든 Heap의 영역에서 GC가 수행되었으며, 그에 따라 처리 시간이 상당히 오래 걸렸다. 

하지만 G1 GC는 어느 영역에 가비지가 많은지를 알고 있기 때문에 GC를 수행할 지역을 조합하여 해당 지역에 대해서만 GC를 수행한다. 

그리고 이러한 작업은 Concurrent하게 수행되기 때문에 애플리케이션의 지연도 최소화할 수 있는 것이다.

물론 G1 GC는 다른 GC 방식에 비해 잦게 호출될 것이다. 하지만 작은 규모의 메모리 정리 작업이고 Concurrent하게 수행되기 때문이 지연이 크지 않으며, 

가비지가 많은 지역에 대해 정리를 하므로 훨씬 효율적이다.

```
java -XX: +UseG1GC -jar Application.java
```
이러한 구조의 G1 GC는 당연히 앞의 어떠한 GC 방식보다 처리 속도가 빠르며 큰 메모리 공간에서 멀티 프로레스 기반으로 운영되는 애플리케이션을

위해 고안되었다. 또한 G1 GC는 다른 GC 방식의 처리속도를 능가하기 때문에 Java9부터 기본 가비지 컬렉터(Default Garbage Collector)로 

사용되게 되었다.

<br/>

## GC의 메모리 누수(leak)

자바에서의 메모리 누수(memory leak)이란 더 이상 허용되지 않는 객체들이 GC에 의해 회수되지 않고 계속 누적이 되는 현상을 말한다.

Old영역에 계속 누적된 객체로 인해 Major GC가 빈번하게 발생하게 되면서, 프로그램 응답속도가 늦어지면서

성능 저하를 불러온다. 이는 결국 OutOfMemory Error로 프로그램이 종료되게 된다.

가비지 컬렉션을 통해 소멸 대상이 되는 객체가 되기 위해서는 어떠한 reference 변수에서 가르키지 않아야 한다.

다 쓴 객체에 대한 참조를 해제하지 않으면 가비지 컬렉터의 대상이 되지 않아 계속 메모리가 할당 되는 메모리 누구 현상이 발생 된다.

아래는 GC를 사용했을 때 발생하는 메모리 누수에 대한 몇 가지 예시이다.

### 1. Integer, Long 같은 래퍼 클래스(Wrapper)를 이용하여, 무의미한 객체를 생성하는 경우

![1 0](https://github.com/minjae990/BIG-C-GC/assets/119609098/1d5004ae-d5e7-479c-b3f1-fbf3516607cd)

long 대신 Long을 사용함으로써 , 오토 박싱으로 인해 sum=sum+l;에서 매 반복마다 새 개체를 생성하므로 1000개의 불필요한 객체가 생성된다.

<br/>

### 2. 맵에 캐쉬 데이터를 선언하고 해제하지 않는 경우

![2 0](https://github.com/minjae990/BIG-C-GC/assets/119609098/dcb734f8-bba1-496b-a4ef-1eeece4386df)

캐시에 직원과 직종을 넣었지만, 캐시를 지우지 않았다. 객체가 더 이상 사용되지 않을 때도 Map에 강력한 참조가 있기 때문에 GC가 되지 않는다.  

캐시의 항목이 더 이상 필요하지 않을 떄는 캐시를 지워주는 것이 바람직하다. 

또한 WeakHashMap으로 캐시를 초기화 할 수 있다. WeakHashMap의 장점은 키가 다른 객체에서 참조되지 않는 경우 해당 항목이 GC가 된다.

하지만, 캐시에 저장된 값을 재사용하려면 해당 키가 다른 객체에 의해 참조되지 않을 수 있으므로 항목이 GC되고 해당 값이 사라질 수 있기 때문에 주의하여야 한다.

<br/>

### 3. 스트림 객체를 사용하고 닫지 않는 경우

![3 0](https://github.com/minjae990/BIG-C-GC/assets/119609098/62f030c9-38c5-44f0-bf24-27db0d7537ce)

try 블록에서 연결 리소스를 닫으므로 예외가 발생하는 경우 연결이 닫히지 않는다. 

따라서이 연결이 풀로 다시 돌아 오지 않기 때문에 메모리 누수가 발생한다. 또한 닫아지지 않아서 데드락이 발생할 가능 성이 크다.

항상 finally 블록에 닫는 내용을 넣거나, TryWhitResource를 사용해야 한다.

<br/>

### 4. 자료구조를 생성하여 사용하면서, 구현 오류로 인해 메모리를 해제하지 않는 경우

![5 0](https://github.com/minjae990/BIG-C-GC/assets/119609098/39b40f26-7f05-42b4-a09a-550fafec4c98)

Stack이 1000으로 커지면서 내부 배열인 stackArray도 값이 채워지게 된다. 그 후 Stack를 pop하면 Stack의 참조된 공간이 비어있지만, 

stackArray에는 pop된 모든 참조가 포함되게 된다. 자바에선 이를 쓸모없는 참조라 불린다. 혹은 구식 참조(역 참조 할 수 없는 참조)라고 불린다. 

배열에 해당 요소가 포함되어 있으면 GC가 될 수 없지만, pop된 후 에는 불필요하다.

이 문제를 해결하려면 pop이 실행될 때, null 값을 설정하여 해당 객체가 GC가 되도록 해야한다. 

![6 0](https://github.com/minjae990/BIG-C-GC/assets/119609098/09ea7254-6c57-4e86-a9a1-3395cb488936)

---

#### Reference

- [가비지 컬렉션 leak](https://junghyungil.tistory.com/133)
- [가비지 컬렉션 GC](https://itkjspo56.tistory.com/285)
- [Java Garbage Collection(GC)](https://github.com/im-d-team/Dev-Docs/blob/master/Java/Java%20Garbage%20Collection(GC).md)

