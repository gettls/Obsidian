# 기존 Java Thread

Java에서는 java.util.concurrent.ExecutorService를 사용해서 JVM내에서 Thread를 관리한다. 실제 스레드가 실행되는 부분을 살펴보자

```java
java.util.concurrent.ThreadPoolExecutor.java 

public void execute(Runnable command) { 
	... 
	int c = ctl.get(); -> 현재 RUNNUNG 상태인 스레드 수를 가져오고 
	if (workerCountOf(c) < corePoolSize) { 
		if (addWorker(command, true))  -> 풀 수보다 작으면 워커에 추가한다. 
			return; 
		c = ctl.get(); 
	} 
	... 
} 

private boolean addWorker(Runnable firstTask, boolean core) { 
	.. ThreadPoolExecutor가 실행해도 된다고 판단하면 
	if (workerAdded) { 
		container.start(t); -> 실행한다. 
		workerStarted = true; 
	} 
}
```

```java
java.lang.Thread.java 

private native void start0(); -> 실제 실행은 결국 JNI를 통한다. 

public void start() { 
	synchronized (this) { // zero status corresponds to state "NEW". 
	if (holder.threadStatus != 0) 
		throw new IllegalThreadStateException(); 
	start0(); 
	} 
}
```

위의 내용을 정리하면 다음과 같다.

- Java Thread의 스케줄링은 ExecutorService가 담당한다.
- 실질적인 Thread 실행은 JNI를 통해 실행된다.

JNI의 내용을 JDK21 기준으로 확인하자.

```cpp
static JNINativeMethod methods[] = { 
	{"start0", "()V", (void *)&JVM_StartThread}, 
	{"setPriority0", "(I)V", (void *)&JVM_SetThreadPriority},
```
- `{"start0", "()V", (void *)&JVM_StartThread}` 
	- `"start0"` : native method 의 함수명을 의미
	- `"()V"` : (parameters type)return type으로 JNI signature를 의미 -> V는 void
	- `"(void *)$JVM_StartThread"` : 함수의 포인터

```cpp
JVM_ENTRY(void, JVM_StartThread(JNIEnv* env, jobject jthread)) 
	... 
	native_thread = new JavaThread(&thread_entry, sz); # 스레드를 생성함
	... 
JVM_END
```
- `JVM_ENTRY, JVM_END` : JVM native 메서드 선언을 위한 매크로 블락

이렇게 OS에 맞게 커널 스레드가 생성되면 해당 스레드는 실행되게 된다.

![[스크린샷 2024-05-31 오후 11.17.52.png]]

---
#  Virtual Thread

![[스크린샷 2024-05-31 오후 11.18.43.png]]

Virtual Thread의 기본 개념은 VT(N) : PT(1) : KTL(1) 이다. 즉, Heap에 수많은 Virtual Thread를 할당시켜놓고 Platform Thread에 VT를 mount/unmount 하면서 컨텍스트 스위칭을 수행한다. 따라서 컨텍스트 스위칭 비용이 작아질 수 밖에 없다.

- 경량화 스레드
	- Virtual Thread는 기존 Thread 모델보다 사이즈가 훨씬 작다.![[스크린샷 2024-05-31 오후 11.26.47.png]]
	- 사이즈가 적기 때문에 Context Switching시 생성/관리/삭제 비용이 적다.
- 스레드 생성 방식 
	- Virtual Thread는 JVM에 의해 생성되기 때문에 커널 모드에서의 호출이 적다.

## 동작 원리
![[스크린샷 2024-06-01 오후 2.46.42.png]]

- Virtual Thread는 Carrier Thread를 가지고 있는데, 이는 실제 수행의 주체인 Platform Thread를 의미한다. Carrier Thread는 Virtual Thread의 작업, Runnable을  runContinuation을 workQueue로 관리한다.
- workQueue에 들어간 runContinuation들은 Scheduler인 ForkjoinPool에 의해 work stealing 방식으로 처리된다.
- 처리되던 runContinuation들은 I/O, Sleep으로 인한 interrupt 나 작업 완료시 queue에서 pop되어 힙 메모리로 park 된다.

## Pinning

![[스크린샷 2024-06-01 오후 2.58.02.png]]

- Virtual Thread의 가장 큰 장점은 Platform Thread에 고정되지 않고 JVM내에서 스케줄링하기 때문에 Context Switching 비용이 적다는 점이다. 하지만, Platform Thread에 고정되어 이러한 장점을 활용하지 못하는 경우가 있다.
- 바로, Virtual Thread에서 synchronized block을 사용하거나 JNI를 통해 native method를 사용하는 경우인데, 이 경우 park될 수 없는 상태가 되버리게 된다. 
- spring의 경우 로직 내에 많은 synchronized block이 있기 때문에 효율이 좋지 않다. 따라서, snychronized block을 사용하는 경우 이를 `ReentrantLock`로 변경하는 걸 고려해야 한다.

#### Thread Context Switching Cost

- **레지스터 저장 및 복원**
	- 실행중이던 스레드의 레지스터 값을 저장하고, 새로운 스레드의 레지스터 값을 복원
- **메모리 캐시 무효화**
	- 새로운 스레드가 다른 메모리 공간을 사용할 수 있기 때문에 캐시가 무효화됨
- **TLB(Translation Lookaside Buffer) 무효화**
	- 새 스레드가 다른 주소 공간을 사용할 수 있기 때문에 TLB를 무효화
- **스케줄링 오버헤드**
	- OS 스케줄러가 어느 스레드를 실행할 지 결정하는 과정에서의 오버헤드

# References
- https://d2.naver.com/helloworld/1203723
- https://www.latera.kr/blog/2019-02-09-java-string-intern/
- https://inpa.tistory.com/entry/JAVA-%E2%98%95-JVM-%EB%82%B4%EB%B6%80-%EA%B5%AC%EC%A1%B0-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%98%81%EC%97%AD-%EC%8B%AC%ED%99%94%ED%8E%B8#%ED%9E%99_%EC%98%81%EC%97%AD_heap_area
- https://techblog.woowahan.com/15398/