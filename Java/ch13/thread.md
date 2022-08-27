# thread

## 1. process & thread
- `프로세스` : 실행 중인 프로그램, 자원(resources)과 쓰레드로 구성 (자원 : 메모리, CPU ...)
- `쓰레드` : 프로세스 내에서 실제 작업을 수행
- 모든 프로세스는 최소한 하나의 쓰레드를 가진다.
- `싱글 쓰레드` : 자원 + 쓰레드
- `멀티 쓰레드` : 자원 + 쓰레드 + 쓰레드 + 쓰레드 ...
- 하나의 새로운 프로세스를 생성하는 것보다 하나의 새로운 쓰레드를 생성하는 것이 더 적은 비용이 든다.

## 2. 멀티쓰레드의 장단점
- 대부분의 프로그램은 멀티쓰레드로 작성되어 있다.
- 장점
    + 시스템 자원을 `효율적`으로 사용할 수 있다.
    + 사용자에 대한 `응답성`이 `향상`된다.
    + 작업이 분리되어 코드가 간결해진다.
- 단점
    + 동기화에 주의해야한다.
    + `교착상태`(dead-lock)가 발생하지 않도록 주의해야한다.
    + 각 쓰레드가 효율적으로 고르게 실행될 수 있도록 해야한다. (`기아 문제 발생`)
> 기아 : 여러 쓰레드가 있다보니 어떤 쓰레드는 자신이 실행할 기회를 얻지 못하고 작업이 진행이 안되는 현상

## 3. 쓰레드의 구현과 실행
- `Thread`클래스를 `상속` (비추천)
```java
class MyThread extends Thread {
    // Thread클래스의 run메서드 오버라이딩
    public void run() {
        /* 쓰레드가 작업할 내용 */
    }
}
```
- `Runnable`인터페이스를 `구현` (`추천`)
```java
class MyThread2 implements Runnable {
    // Runnable인터페이스의 추상메서드 run()을 구현
    public void run() {
        /* 쓰레드가 작업할 내용 */
    }
}
```
> Java는 단일 상속만을 허용하기 때문에 Thread를 상속할 경우 다른 클래스를 상속하지 못하기 때문에 유동적인 `Runnable`을 구현하는 것이 더 좋다.

- 쓰레드 실행
```java
// Thread를 상속한 경우
MyThread t1 = new MyThread();
t1.start();

// Runnable을 구현한 경우
Runnable r = new MyThread2();
Thread t2 = new Thread(r);
// Thread t2 = new Thread(new MyThread2()); // 한줄로 만들기
t2.start();
```

### 3.1 쓰레드의 실행 - start()
- 쓰레드를 생성한 후에 `start()`를 호출해야 쓰레드가 작업을 시작한다.
- start()를 호출하면 실행가능한 상태가 되는 것일 뿐 t1과 t2중 어느 것이 먼저 실행될지는 `OS 스케줄러`가 결정한다.
```java
ThreadEx t1 = new ThreadEx();
ThreadEx t2 = new ThreadEx();
// 쓰레드 실행
t1.start();
t2.start();
```

## 4. start() vs run()
- 작성한건 run()이지만 start()를 호출하는 이유를 start()를 호출하면 일어나는 과정을 보며 이해해보자.
1. main메서드에서 쓰레드의 start()를 호출한다.
2. start()는 새로운 쓰레드를 생성하고, 쓰레드가 작업하는데 사용될 호출스택을 생성한다.
3. 새로 생성된 호출스택에 run()이 호출되고 start()는 종료되면서 각각의 쓰레드가 독립적인 자신만의 호출 스택을 가지고 작업을 수행한다.
4. 이제는 호출스택이 2개이므로 스케줄러가 정한 순서에 의해서 번갈아 가면서 실행한다.

### 4.1 main 쓰레드
- main메서드의 코드를 수행하는 쓰레드
- 쓰레드는 `'사용자 쓰레드'`와 `'데몬 쓰레드'` 두 종류가 있다.
- 데몬 쓰레드 : 사용자 쓰레드가하는 작업을 보조하는 쓰레드
- 실행 중인 사용자 쓰레드가 하나도 없을 때 프로그램은 종료된다. (멀티쓰레드에서 main쓰레드가 종료되도 쓰레드가 남아있으면 종료 x)

## 5. 싱글 쓰레드 & 멀티 쓰레드
- 싱글 쓰레드보다 멀티 쓰레드가 `컨텍스트 스위칭`때문에 시간이 더 오래 걸린다.
> context switching : 프로세스 or 쓰레드 간의 작업 전환

### 5.1 쓰레드의 I/O 블락킹
- 입출력시 작업 중단
- `싱글 쓰레드의 경우`   
: 사용자로부터 입력을 기다리는 구간에 아무 일도 일어나지않는다.
- `멀티 쓰레드의 경우`   
: 사용자로부터 입력을 기다리는 구간에 다른 쓰레드가 수행된다. (효율적, 시간 절약)

## 6. 쓰레드의 우선순위
- 작업의 중요도에 따라 쓰레드의 우선순위를 다르게 하여 특정 쓰레드가 더 많은 작업시간을 갖게 할 수 있다.
- 우선순위는 JVM기준 1 ~ 10 단계까지 있다.
- 쓰레드의 `기본 우선순위`는 `5`이다.
- 우선순위를 지정하는 것은 OS 스케줄러에게 희망사항을 요구하는 것일 뿐 OS 스케줄러가 듣지 않을 수 도있다.   
> OS스케줄러의 역할은 해당 OS에서 돌아가는 모든 프로세스와 쓰레드에 공평하게 돌아가야하기 때문에 </br> 
OS에 돌아가고 있는 수많은 프로세스와 쓰레드가 있는데 우선순위로 지정한 쓰레드에게만 특혜를 줄 수 없다.

```java
// 쓰레드의 우선순위를 지정한 값으로 변경
void setPriority(int newPriority)
// 쓰레드의 우선순위를 반환
int getPriority()
```

## 7. 쓰레드 그룹
- 서로 관련된 쓰레드를 그룹으로 묶어서 다루기 위한 것
- 모든 쓰레드는 반드시 하나의 쓰레드 그룹에 포함되어 있어야 한다.
- 쓰레드 그룹을 지정하지 않고 생성한 쓰레드는 `main쓰레드 그룹`에 속한다.
- 자신을 생성한 쓰레드(부모 쓰레드)의 그룹과 우선순위를 상속받는다.
```java
// 쓰레드 그룹을 지정하는 생성자를 사용하지 않는 쓰레드
Thread(ThreadGroup group, String name)
Thread(THreadGroup group, Runnable target)
Thread(THreadGroup group, Runnable target, String name)
Thread(THreadGroup group, Runnable target, String name, long stackSize)

// Thread의 쓰레드 그룹과 관련된 메서드
// 쓰레드 자신이 속한 쓰레드 그룹을 반환 (Thread 클래스에 있는 메서드)
ThreadGroup getThreadGroup()    
// 예외 호출 메서드 (ThreadGroup 클래스에 있는 메서드)
void uncaughtException(Thread t, Throwable e)   // 처리되지 않은 예외에 의해 쓰레드 그룹의 쓰레드가 실행이 종료되었을 때 JVM에 의해 이 메서드가 자동적으로 호출된다.
```

### 7.1 쓰레드 그룹의 생성자와 메서드
- 생성자

| 생성자 | 설명 |
|---|---|
| ThreadGroup(String name) | 지정된 이름의 새로운 쓰레드 그룹을 생성 |
| ThreadGroup(ThreadGroup parent, String name) | 지정된 쓰레드 그룹에 포함되는 새로운 쓰레드 그룹을 생성 |

- 메서드

| 메서드 | 설명 |
|---|---|
| int activeCount() | 쓰레드 그룹에 포함된 `활성상태`에 있는 쓰레드의 `수`를 `반환` |
| int activeGroupCount() | 쓰레드 그룹에 포함된 `활성상태`에 있는 쓰레드 그룹의 `수`를 `반환` |
| void checkAccess() | 현재 실행중인 쓰레드가 쓰레드 그룹을 변경할 `권한이 있는지 체크` |
| void destroy() | 쓰레드 그룹과 하위 쓰레드 그룹까지 모두 `삭제`. 단, 비어있어야 삭제 가능 |
| int enumerate(Thread[] list) </br> int enumerate(Thread[] list, boolean recurse) </br> int enumerate(ThreadGroup[] list) </br> int enumerate(ThreadGroup[] list, boolean recurse)| 쓰레드 그룹에 속한 쓰레드 or 하위 쓰레드 그룹의 목록을 지정된 배열에 담고 그 `개수`를 `반환` </br> 두 번째 매개변수인 recurse의 값을 true로 하면 그룹에 속한 하위 쓰레드 그룹에 </br> 쓰레드 or 쓰레드 그룹까지 배열에 담는다. |
| int getMaxPriority() | 쓰레드 그룹의 `최대 우선순위`를 `반환` |
| String getName() | 쓰레드 그룹의 `이름`을 `반환` |
| ThreadGroup getParent() | 쓰레드 그룹의 `상위` 쓰레드 그룹을 `반환` |
| void interrupt() | 쓰레드 그룹에 속한 모든 쓰레드를`interrupt` |
| boolean isDaemon() | 쓰레드 그룹이 데몬 쓰레드 그룹인지 `확인` |
| boolean isDestroyed() | 쓰레드 그룹이 삭제되었는지 `확인` |
| void list() | 쓰레드 그룹에 속한 쓰레드와 하위 쓰레드 그룹에 대한 `정보`를 `출력` |
| boolean parentOf(ThreadGroup g) | 지정된 쓰레드 그룹의 상위 쓰레드 그룹인지 `확인` |
| void setDaemon(boolean daemon) | 쓰레드 그룹을 데몬 쓰레드 그룹으로 `설정/해제` |
| void setMaxPriority(int pri) | 쓰레드 그룹의 `최대 우선순위`를 `설정` |

## 8. 데몬 쓰레드
- 일반 쓰레드의 작업을 돕는 보조적인 역할을 수행
- 일반 쓰레드가 모두 종료되면 자동으로 종료
- 가비지 컬렉터, 자동저장, 자동갱신 등에 사용
- 무한루프와 조건문을 이용해서 실행 후 대기하다가 특정조건이 만족되면 작업을 수행하고 다시 대기하도록 작성
```java
public void run() {
    while(true) {   // 일반 쓰레드가 모두 종료되면 자동으로 종료되기 때문에 무한루프로 만들어도 상관 x
        try {
            Thread.sleep(3 * 1000); // 3초 마다
        } catch(InterruptedException e) {}

        if(autoSave) {
            autoSave();
        }
    }
}
```
```java
boolean isDaemon() // 쓰레드가 데몬 쓰레드인지 확인 (데몬 쓰레드이면 true반환)
void setDaemon(boolean on) // 쓰레드를 데몬 쓰레드로 or 사용자 쓰레드로 변경 (매개변수 on을 true로 지정하면 데몬 쓰레드가 된다.)
```
> `SetDaemon(boolean on)`은 반드시 `start()를 호출하기 전`에 `실행`되어야 한다. 그렇지 않으면 IllegalThreadStateException 발생

## 9. 쓰레드의 실행제어
- 쓰레드의 실행을 제어할 수 있는 메서드가 제공(이 들을 활용해서 보다 효율적인 프로그램 작성)

- 쓰레드의 상태 

| 상태 | 설명 |
|---|---|
| NEW | 쓰레드가 생성되고 아직 start()가 호출되지 않은 상태 |
| RUNNABLE | 실행 중 or 실행 가능한 상태 |
| BLOCKED | 동기화 블럭에 의해서 일시정지된 상태(lock이 풀릴 때까지 기다리는 상태) |
| WAITING, TIMED_WAITING | 쓰레드의 작업이 종료되지는 않았지만 실행가능하지 않은 일시정지상태, TIMED_WAITING은 일시정지시간이 지정된 경우를 의미 |
| TERMINATED | 쓰레드의 작업이 종료된 상태 |

- 쓰레드의 수행과정 
1. `쓰레드를 생성`하고 `start()를 호출`하면 바로 실행되는 것이 아니라 `실행대기열에 저장`되어 자신의 차례가 될 때까지 기다려야 한다. </br> 실행대기열은 큐와 같은 구조로 먼저 실행대기열에 들어온 쓰레드가 먼저 실행된다.
2. 실행대기상태에 있다가 자신의 차례가 되면 `실행상태`가 된다.
3. 주어진 실행시간이 다되거나 yield()를 만나면 다시 `실행대기상태`가 되고 다음 차례의 쓰레드가 실행상태가 된다.
4. 실행 중에 `suspend()`, `sleep()`, `wait()`, `join()`, `I/O block`에 의해 `일시정지상태`가 될 수 있다.
5. 지정된 일시정지시간이 다되거나(`time-out`), `notify()`, `resume()`, `interrupt()`가 호출되면 일시정지상태를 벗어나 </br> 다시 `실행대기열에 저장`되어 자신의 차례를 기다리게 된다.
6. 실행을 모두 마치거나 stop()이 호출되면 쓰레드는 `소멸`한다.
> `주의` : 순서대로 쓰레드가 수행되는 것은 아니다.

- 쓰레드의 스케줄링과 관련되 메서드

| 메서드 | 설명 |
|---|---|
| static void sleep(long millis) </br> static void sleep(long millis, int nanos) | 지정된 시간(천분의 일초 단위)동안 쓰레드를 잠들게시킨다. </br> 지정한 시간이 지나고 나면, 자동적으로 다시 실행대기상태가 된다. |
| void join() </br> void join(long millis) </br> void join(long millis, int nanos) | 지정된 시간동안 쓰레드가 실행되도록 한다. </br> 지정된 시간이 지나거나 작업이 종료되면 join()을 호출한 쓰레드로 다시 돌아와 실행을 계속한다. |
| void interupt() | sleep()이나 join()에의해 일시정지상태인 쓰레드를 깨워서 실행대기 상태로 만든다. </br> 해당 쓰레드에서는 interrupted Exception이 발생함으로써 일시정지 상태를 벗어나게 된다. |
| void stop() | 쓰레드를 즉시 종료시킨다. |
| void suspend() | 쓰레드를 일시정지 시킨다. resume()을 호출하면 다시 실행대기상태가 된다. |
| void resume() | suspend()에 의해 일시정지 상태에 있는 쓰레드를 실행대기상태로 만든다. |
| static void yield() | 실행 중에 자신에게 주어진 실행시간을 다른 쓰레드에게 양보(yield)하고 자신은 실행대기상태가 된다 |
> static이 붙은 메서드들은 다른 쓰레드에게 적용될 수 없고 자기 자신에게만 호출이 가능하다.   
> resume(), stop(), suspend()는 쓰레드를 교착상태로 만들기 쉽게 때문에 deprecated되었다.

### 9.1 sleep()
- 현재 쓰레드를 지정된 시간동안 멈추게 한다. (잠자기)
```java
static void sleep(long millis) 
static void sleep(long millis, int nanos)
```
- 반드시 예외처리를 해야한다. (InterruptedException이 발생하면 깨어남)
```java
try {
    Thread.sleep(1,50000);  // 0.0015초 동안 sleep 
} catch (InterruptedException e) {}

// time up(시간 종료), interrupted(깨우기)를 이용해서 깨울 경우 throw new InterruptedException()를 발생시키고 catch문에서 해당 예외를 처리
// 단지, 깨우기 위해서 예외를 활용하는 것이기 때문에 {}안에 비어있어도 된다.
```
```java
// 일반적으로는 메서드를 이용해서 호출하는 식을 선호한다.
void delay(long millis) {
    try {
        Thread.sleep(millis);
    } catch(InterruptedException e) {}
}

delay(15);
```
- 특정 쓰레드를 지정해서 멈추게 하는 것은 불가능하다. (자기 자신만 멈추게 가능)
```java
try {
    Thread.sleep(2000); // th1.sleep(2000);이라고 쓰지않는다.(오해의 소지가 있음)
} catch(InterruptedException e) {}
```

### 9.2 interrupt()
- 대기상태(WAITING)인 쓰레드를 실행대기 상태(RUNNABLE)로 만든다.
```java
void interrupt()                // 쓰레드의 interrupted상태를 false -> true로 변경
boolean isInterrupted()         // 쓰레드의 interruped상태를 반환
static boolean interrupted()    // 현재 쓰레드의 interrupted상태를 알려주고, false로 초기화
```

### 9.3 suspend(), resume(), stop()
- resume(), stop(), suspend()는 쓰레드를 교착상태로 만들기 쉽게 때문에 deprecated되었다.
- Thread에 정의된 메서드 대신 직접 정의해서 사용할 수 있다.
```java
class ThreadEx implements Runnable {
    boolean suspended = false;
    boolean stopped = false;

    public void run() {
        while(!stopped) {
            if(!suspended) {
                /* 쓰레드가 수행할 코드 작성 */
            }
        }
    }
    public void suspend() { suspended = true; }
    public void resume() { suspended = false; }
    public void stop() { stopped = true;}
}
```
> 잘 동작하지 않는다면 `volatile` boolean suspended = false;를 사용하자   
> volatile : 해당 변수는 자주 바뀌는 값이므로 캐쉬메모리에서 복사본을 사용하지말고 RAM에서 원본에서 값을 가져와 사용해라.

### 9.4 join()
- 지정된 시간동안 특정 쓰레드가 작업하는 것을 기다린다.
```java
void join()                         // 작업이 모두 끝날 때까지
void join(long millis)              // 천분의 일초 동안
void join(long millis, int nanos)   // 천분의 일초 + 나노초 동안
```
- 예외처리를 해야한다.
```java
class ThreadEx {
    static long startTime = 0;

    public static void main(String[] args) {
        ThreadEx1 th1 = new ThreadEx1();
        ThreadEx2 th2 = new ThreadEx2();
        th1.start();
        th2.start();
        startTime = System.currentTimeMillis();

        try {
            th1.join();
            th2.join();
        } catch(InterruptedException e) {}

        System.out.print("소요시간:" + (System.currentTimeMillis() - ThreadEx.startTIme));
    }
}
```

### 9.5 yield()
- 남은 시간을 다음 쓰레드에게 양보하고, 자신(현재 쓰레드)은 실행대기한다.
- yield()와 interrupt()를 적절히 사용하면, 응답성과 효율을 높일 수 있다.
- yield()는 단지 OS 스케줄러에게 통보를 하는 것일 뿐 OS 스케줄러가 양보하지 않을 수도 있다.
```java
class ThreadEx implements Runnable {
    boolean suspended = false;
    boolean stopped = false;

    public void run() {
        while(!stopped) {
            if(!suspended) {    // stopped가 false이고 suspended가 true일 경우 무한루프이기 때문에 계속 실행이된다. (busy - waiting)
                /* 
                    쓰레드가 수행할 코드 작성 
                */
                try {
                    Thread.sleep(1);    // 바뀐다음 자고있을 수 있으므로 예외처리를 이용해 깨운다.
                } catch(InterruptedException e) {}
            } else {            // 무한 루프로인해 busy-waiting을 방지하기위해 양보한다.
                Thread.yield();
            }
        }
    }
    public void suspend() {
        suspended = true;
        th.interrupt();
    }
    public void stop() { 
        stopped = true;
        th.interrupt();
    }
    public void resume() { suspended = false; }
}
```

## 10. 쓰레드의 동기화(synchronization)
- 한 쓰레드가 진행중인 작업을 다른 쓰레드가 간섭하지 못하게 막는 것
- 동기화하려면 간섭받지 않아야 하는 문장들을 `'임계 영역'`으로 설정
- 임계영역은 `락`(lock)을 얻은 단 하나의 쓰레드만 출입가능(객체 1개 당 락 1개)
- 동기화해야하는 객체를 읽고 쓰는 메서드에 모두 동기화해야한다.
- 배경 : 멀티 쓰레드 프로세스에서는 다른 쓰레드의 작업에 영향을 미칠 수 있으며 진행중인 작업이 다른 쓰레드에게 간섭받지 않게 하려면 '동기화'가 필요하다.

### 10.1 synchronized를 이용한 동기화
- synchronized로 임계영역을 설정하는 방법 2가지
```java
// <1> 메서드 전체를 임계 영역으로 지정
public synchronized void calSum() {
    ...
} 
// <2> 특정한 영역을 임계 영역으로 지정
synchronized(객체의 참조변수) {
    ...
}
```

### 10.2 wait() & notify()
- 동기화 효율을 높이기 위해 사용
- Object클래스에 정의되어있으며, 동기화 블럭 내에서만 사용할 수 있다.
- `wait()` : 객체의 lock을 풀고 쓰레드를 해당 객체의 waiting pool에 넣는다.
- `notify()` : waiting pool에서 대기중인 쓰레드 중 아무거나 하나를 깨운다.
- `notifyAll()` : waiting pool에서 대기중인 모든 쓰레드를 깨운다.
```java
// Object에 정의되있는 형태
void wait();
void wait(long timeout)
void wait(long timeout, int nanos)
void notify()
void notifyAll()
```
```java
class Account {
    int balance = 1000;

    public synchronized void withdraw(int money) {
        while(balance < money) {
            try {
                wait();
            } catch(InterruptedException e) {}
        }
        balance -= money;
    }
    public synchronized void depesit(int money) {
        balance += money;
        notify();
    }
}
```

# Reference
> 자바의 정석 - 남궁성
