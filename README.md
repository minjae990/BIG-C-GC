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

이 과정에서 객체가 `Garbage`인지 아닌지를 판단하기 위해 `reachability`개념을 사용한다. 보통 하나의 객체는 다른 객체를 참조하고 그 객체는 또 다른 객체를 참조하게 되는데 여기서 **최초에 참조한 객체를 `Root Set`이라고 한다.**

![gc_process](/assets/images/gc_process.png)

위의 그림과 같이 각각의 객체가 서로 참조하면서 참조 사슬이 형성되는데 `Root Set`에서 참조 사슬에 의해  `Unreachable`한 객체들은 **Garbage Collection**의 대상이 된다.

<br/>

