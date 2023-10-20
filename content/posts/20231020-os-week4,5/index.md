---
title: "20231018 OS Week4,5 - Synchronization"
date: 2023-10-20T18:51:29+09:00
# weight: 1
# tags: []
author: "Byungjun Yoon"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
# categories: []
# description: ""
disableHLJS: true 
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
# cover:
#     image: "<image path/url>" # image path/url
#     alt: "<alt text>" # alt text
#     caption: "<text>" # display caption under cover
#     relative: false # when using page bundles set this to true
#     hidden: true # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
---
# Synchronization
동시성은 멀티 프로세스 또는 멀티 스레드 환경에서 꼭 고려해야하는 문제이다. CPU 스케줄러는 프로세스 1의 작업 명령어를 전부 실행하지 않은 상태에서 프로세스 2로 스위치 할 수도 있기 때문에, 이러한 문제의 원인을 제공해. 운영체제 내에는 프로세스가 가지고 있는 독립적인 리소스들도 있지만, 여러 프로세스가 공유하는 자원이 있을 때, 이 자원에 접근하는 순서나 실행 순서에 따라서 의도하지 않은 동작을 하는 경우가 많아. 

예를 들자면, 
```c
while (true)
	while (counter == BUFFER_SIZE)
		; /* no nothing */
	buffer[in] = next_produced
	in = (in + 1) % BUFFER_SIZE
	counter++;

while (true)
	while (counter == 0)
		; /* no nothing */
	next_consumed = buffer[out];
	out = (out + 1) % BUFFER_SIZE
	counter--;
```

위의 예시에서 `counter`가 5일 때, `counter++; counter--` 가 연속으로 실행되었을 때 그 값이 5인 것을 보장할 수 없어. 
`counter++` 은 기계어로 `reg1 = counter; reg1 = reg1 + 1; counter = reg1` 의 순서로 실행되고, `counter--` 도 비슷해. 하지만 이 인스트럭션들의 순서가 꼬이게 된다면, 그 결과가 5가 아닐 수도 있어. 왜냐하면 `counter=reg1`가 마지막에 수행된다면, reg1 에 저장되어있던 4나 6이 counter 에 들어갈 수도 있기 때문이야. 

## Critical Section
이러한 현상은 우리가 `critical section` 에 여러 프로세스가 동시에 접근할 수 있게 허용하기 때문이야. 이러한 공유 변수에 변경을 가할 수 있는 코드 섹션은 우리는 `critical section` 이라고 해. 이러한 Critical section 을 다루는 문제에 대한 해결은 다음 3가지 조건을 꼭 만족해야해. 
1. Mutual exclusion: 만약에 프로세스 $P_i$ 가 크리티컬 섹션을 실행하고 있다면 다른 어떠한 프로세스도 해당 섹션을 실행하면 안된다
2. Progress: 어떠한 프로세스도 크리티컬 섹션을 수행하고 있지 않고, 해당 색션에 진입하고 싶어하는 프로세스가 있다면, 어떤 프로세스가 진입할지 결정하는 문제는 무기한 연장되어서는 안된다
3. Bounded Waiting: 크리티컬 색션에 들어가기 위해 요청한 시간과 요청을 승인한 시간은 유한하다. -> 프로세스가 무한으로 기다려서는 안된다. 

## Solution
### Peterson's solution
```c
do {
	flag[i] = true;
	turn = j;
	while (flag[j] && turn == j);
	/* critical section */
	flag[i]=false;
} while(true);
```
피터슨의 제안한 방법은 시스템에 현재 임계영역에 접근하려는 프로세스가 두 개인 경우를 가정한다. `turn`은 현재 누가 임계영역에 접근할 차례인지를 나타내는 변수이고, `flag`는 누가 준비를 완료한 상태인지를 나타낸다. 만약에 $P_i$ 가 준비를 완료하고 자기가 들어갈 차례일 경우에만 while 루프를 나와 임계영역에 진입한다. $i=1-j$ 의 관계를 성립하는데, 본인이 들어갈 준비를 마치면 다음 차례는 $j$ 에게 양보하지. 그리고 while 문에서는 $P_j$ 가 작업하는 동안 대기를 하다가, $P_i$ 가 들어갈 수 있는 상태가 되었을 때 임계영역에 진입해.  그렇게 실행을 다하고 나면, 준비 작업을 나중에 다시해야겠지? 그래서 `flag[i]` 을 `false`로 설정하고 다시 위로 돌아가. 

이 해결책에서 위에서 우리가 언급한 조건이 맞나 확인을 해보자. 첫번째 속성은 $i$ 가 실행중일때는 $j$는 진입하면 안되어야해. `flag`는 i, j 모두 참이 될 수도 있지만, turn 값 때문에 동시에 while 문을 통과할 방법이 없어. $P_j$ 가 실행중일 때는, `flag[j]==true && turn == j`을 만족해. 이 조건 때문에 $P_i$ 는 대기를 해야해, 그리고, 이 조건은 $P_j$ 가 임계영역 안에 존재하는 한 그대로 유지 되기 때문에 Mutual exclusion 이 만족해. 
두번째 속성은 루프의 조건은 무조건 한쪽만 가능하다. 만약에 $P_j$ 가 임계영역에 진입할 준비가 안되어있다면,  $P_i$ 가 진입할 수 있고, 그 역도 성립한다. 따라서, 무기한 대기는 하지 않는다. 
마지막으로 $P_i$ 의 루프 안에서는 `turn`의 값이 바뀌지 않으므로 최대 한번의 $P_j$ 의 진입 이후 $P_i$ 가 진입할 수 있다. 


## Hardware solution
위 언급한 소프트웨어적인 해결책 이외에도 HW 적인 방법을 이용한 방법도 있다. concurrency 에서 문제가 생기는 경우는 하나의 기능이 여러 인스트럭션으로 나누어져, 스케쥴링에 따라 기능을 완전히 수행하지 않은 상태에서 다른 기능을 수행할 때 생겨. 따라서, 아래 언급하는 방법들은 하나의 atomic 한 인스트럭션으로 이루워져있어 중간에 인터럽트가 발생하지 않아. 

```c
boolena test_and_set(boolean *target){
	boolean rv = *target
	*target = true;
	return rv;
}

boolean lock = false;
do {
	while(test_and_set(&lock));
	// Critical Section
	lock = false;
} while (ture);
```
`test-and-set` 은 어떠한 boolean 값을 참으로 바꿔주고, 이전 값을 반환한다. 만약 `target` 이 거짓이라면, 루프를 빠져나오며, 임계영역에 접근한다. 그 후 임계영역을 나오며 lock 을 false 로 바꿔 다른 프로세스들이 진입할 수 있도록 해준다. 하지만, 프로세스의 개수가 3개 이상이 될 경우에는 하나의 프로세스가 계속 임계영역을 진입을 못하는 경우가 있기 때문에, bound-waiting 요구사항을 만족하지 못한다. 

따라서, 다음과 같이 대기열을 사용해서 구현을 해 bound-waiting 조건을 만족시켜. 
```c
boolean lock = flase; 
do { // 프로세스 Pi의 진입 영역 
	waiting[i] = true; // 프로세스 i를 대기열에 넣음. 
	key = true; 
	while (waiting[i] && key) // 대기가 풀리거나 lock이 풀릴 때까지 대기함. 
	key = TestAndSet(&lock); // lock 값을 key에 반환. key와 lock은 같은 값으로 보면 됨. 
	waiting[i] = false; // CS에 접근할 수 있으므로 대기를 풀어줌. 이 코드가 없다면 무한 대기를 하게 됨. 
	// Critical Section 
	j = (i + 1) % n; 
	while ((j != i) && !waiting[j]) // 대기 중인 프로세스를 찾음
		j = (j + 1) % n; 
	if (j == i) // 대기 중인 프로세스가 없으면 
		lock = false; // 다른 프로세스의 진입을 허용할 수 있게 lock을 풀어줌. 
	else // 대기 프로세스가 있으면 다음 순서로 임계 영역에 진입하게 함. 
		waiting[j] = false; // Pj가 임계 영역에 진입할 수 있도록 대기를 품. 
} while (true);
```
이렇게, `test_and_set` 을 이용해 구현하는 방법이 있고, `compare_and_swap` 이라는 방법도 있다. 
```c
int compare_and_swap(int *value, int expected, int new_value){
	int temp = *value;
	if (*value == expected)
		*value = new_value;
	return temp;
}

int lock = 0
do{
	while(compare_andswap(&lock, 0, 1) != 0)
	; /* do nothing */
	/* critical section */
	lock = 0;
	
} while (true);
```
처음 lock 으로 진입할 때는, 임계영역에 진입하기 위해 기대되는 `lock`의 값은 0 이다. 만약에 `lock`의 값이 0이라면, 새로운 값 1로 덮어쓰고, 임계 영역으로 진입한다. 이러면 루프에 있는 다른 프로세스는 lock 이 기대한 값과 다르기 때문에 덮어쓰지 못하고 대기를 하게 된다. 처음 임계영역에 진입한 프로세스가 작업을 마치면, 나오면서 `lock`을 다시 0으로 만들어줘, 다른 프로세스들이 진입할 수 있게 해줘. 

하지만, 이런 식의 하드웨어 솔류션은 busy-wait 를 해야하기 때문에, 루프 안에서 대기하는 동안은 프로세스가 다른 작업을 하지 못해. 또한 이런 하드웨어 인스트럭션에 대한 접근은 일반 프로그램 개발자에게는 접근 불가능하기도 해. 그래서 다른 방법들이 존재한다. 

## Mutex Locks (Spin Lock)
가장 간단한 방법중 하나는 Mutex lock  이다. 프로세스는 임계영역에 접근하기 전에 이 락을 확보하고 진입을 하고, 나올때 락을 해제하고 나오는 구조를 가지고 있어. 
```cpp
SpinLock::acquire(){
	while (TestAndSet(&lock) == BUSY); 
}

SpinLock::release(){
	lock = FREE;
	memorybarrier();
}

acquire() {
	while (!available)
		;
	available = false;
}

release() {
	available = true;
}
```
위 두 가지 예시는 사실 같다. 첫번째는 실제 구현이고, 두번째는 추상적인 구현으로 이해하면 될꺼 같다. 추상적이라 하는 이유는 `acquire()` 같은 경우는 하나의 명령어로 atomic 하게 이루워져야 되기 때문이다. 이러한 형태의 락은 busy-wait 를 하게 된다. 루프 안에 계속 갇혀있다가, 임계영역에 진입할 순서가 되면 들어가게 된다. 이런 구현은 Spinlock 이라고 부르는데, CPU 사이클을 낭비하기 때문에, 단점으로 여겨진다. 하지만 이런 구현이 장점인 경우도 있다. 만약에 wait 이 짧을 경우에는 sleep-wake 방식으로 구현을 하게 되면, 대기 할때마다 컨텍스트 스위칭의 비용이 커진다. 따라서 대기 시간이 짧을 것으로 예상 되는 경우에는 스핀락이 장점이야. 

## Semaphore
이런 락은 간단하게 락을 걸고 해제하는 구조였어. 세마포어의 경우는 좀 더 일반화된 방법의 동기화 방법이라고 생각하면 될거 같아. 세마포어는 두 개의 동작이 있어 `wait()` 과 `signal()` `wait()`는 P operation 이라고도 하고, `signal`은  V operation 이라고도 해. 이 두 동작은 모두 atomic 하게 동작해야해. 위에서 계속 언급했듯이 스케쥴링에 의해 이 작업을 수행하는 중 다른 명령어를 수행하면 안되기 때문이야. 따라서 핀토스 등에서는 이러한 아토믹한 동작들을 구현하는 데 있어서 interrupt 를 disable 하고 진행해. 이렇게 할 경우, 타이머 인터럽트 등에 의해서 프로세서를 뺏기는 일이 없기 때문이야. 

```c
wait(S){
	while (S<=0)
		;
	S--;
}

signal(S){
	S++;
}
```
세마포어는 counting semaphore 와 binary semaphore 두종류로 나위어. 카운팅은 제한 없이 값을 올리고 내릴수 있는데 반면에, 바이너리는 0 과 1만 존재해. 바이너리 세마포어는 뮤텍스락이랑 동일하다고 생각하면 돼. 세마포어는 어떠한 리소스가 여러개의 인스턴스를 가지고 있을 때 사용해. 최초의 세마포어는 이 인스턴스 개수로 초기화가 되. 그리고, 이러한  리소스를 사용한 임계영역이 있을 때, 이 영역에 진입할 때마다, wait 를 불러 가용 인스턴스의 개수를 하나씩 줄여. 그리고 임계영역을 나올 때, 다른 프로세스도 이 리소스를 사용할 수 있도록 `signal`을 보내 다른 리소스도 사용할 수 있게 만들어주는 거야. 

이렇게 리소스 관리 이외에서 어떠한 프로세스가 다른 프로세스 다음에 특정 인스트럭션을 수행해야하는 경우에서도 사용할 수 있어. 
```c
// P1
S1
signal(synch);

// P2
wait(synch);
S2
```
만약에 S2 가 S1 다음이 실행되어야하는 로직이야. 그러면, `synch`를 0으로 초기화 한 다음에 시작을 하게 되면 P2는 wait 가 synch 가 양수가 될 때 까지 기다려, S1 이 실행되고 난 후에 시그널을 주면, synch 가 1이 되며 P2가 busy-wait 에서 빠져나와 S2를 실행하는 방법등으로 사용할 수 있어. 이러한 사용법은 스레드의 `thread_exit`과 `thread_join` 등에서 사용할 수 있어. 부모 스레드에서 자식 스레드를 만들었을 때 부모에서 `join`을 콜하면 자식이 `exit` 하는 것을 기다려야해. 따라서, 0으로 초기화된 세마포어를 하나 만들고, 부모는 그 세마포어가 양수가 되는 것을 `wait` 하고, 자식은 `exit` 할 때, `signal`을 주게 되면, 순차적으로 실행하게 만들수 있어. 

### Semaphore 구현
위 뮤텍스 예시에서 busy waiting 의 문제가 있었던 것은 기억할 꺼야. 따라서, 세마포어를 구현하는데 있어서 이 문제는 꼭 다뤄야해. 만약에 임의의 프로세스가 `wait()`를 실행해 세마포어의 값이 0인 것을 확인하면, busy-wait 하지 않고, 자기 자신을 block 해. 이렇게 하여, CPU에게 컨트롤을 넘겨 다른 프로세스를 실행할 수 있도록 해주는 거야. `signal()`을 통해 세마포어 값이 증가하면, 아까 블락해둔 프로세스를 깨워서 다시 실행하해. 

실제 구현을 하게 되면 다음과 같이 될꺼야. 
```c
class Semaphore { 
	int value = initialValue; 
	struct process *list;	
}
Semaphore::P() {
	//Disable interrupts;
	while (value == 0) {
		put on queue of threads waiting for this semaphore;
		go to sleep; // enable before sleep 
	}
	value = value – 1;
	// Enable interrupts;
}

Semaphore:V(){
	// Disable interrupts;
	if any one is in waiting queue {
		take waiting process P_i from queue
		wakeup P_i
	}
	value = value + 1;
	// Enable interrupts;
}

```
이러한 대기 리스트는 프로세스의 PCB 등에 적어놔. 이렇게 해야, 프로세스 내에서 여러 스레드 들이 세마포어를 쓰고 싶을 때, 공유되는 PCB에 접근해 적절한 다음 스레드를 꺼내올 수 있어. 세마포어의 대기, 시그널 등의 동작은 무조건 아토믹하게 이뤄져야해. 싱글 코어 프로세서에서는 간단해, 세마포어 동작 전에 인터럽트는 막고, 끝날때 인터럽트를 끼면, 다른 프로세스의 인스트럭션이 끼어들어오는 일이 없기 때문이야. 하지만, 멀티 프로세서 환경에서는 인터럽트을 끄고 키는 것이 모든 프로세서에 전파되어야하기 때문에, 스핀락등을 사용해 대기와 시그널의 atomicity 를 지키는 방식을 취해. 결과적으로 busy-wait 를 완전히 제거하는 것은 불가능하지만, 대기와 시스널에 대해서만 제한적인 busy-wait 하는 점에서 이점이 있어. 

## Condition Variable
컨디션 베리어블은 임계영역내에서 특정한 조건을 만족하길 기다리는 기능을 구현할 때 쓰는 기능이다. 위에서 세마포어를 이용해 S1, S2의 순서를 정해주었던 예시를 생각해보자. 여기서 P2는 하는 일도 없으면서 CPU 사이클을 소비하고 있다. 이런 경우를 막기 위해서 특정한 조건 동안 무조건 대기를 해야만 한다면, 그냥 프로세스를 재워서 대기를 시킬 수 있다. condition variable 은 세가지 함수를 가진다. 
- `wait(&lock)`: 락을 해제하자마자 아토믹하게 sleep 으로 간다. 그 후 다시 락을 확보한다.
- `signal()`: 대기하고 있는 프로세스를 깨운다
- `broadcast()`: 모든 대기하고 있는 프로세스를 깨운다. 
### Condition Variable 구현
```c
class CV {
	private Queue waiting;
	public void wait(Lock *lock);
	public void signal();
	public void broadcast();
}

void CV::wait(Lock *lock*){
	// check lock is held
	// add this tcb to waiting queue
	// lock 을 풀고, 현재 스레드를 블락하고, 새 스레드를 뽑는다
	// 그 후 다시 락을 잡는다
}

void CV::signal(){
	if (waiting.notEmpty()){
		// waiting 에서 스레드를 팝하여 레디 상태로 바꾼다
	}
}
```

## Deadlock and Starvation
데드락은 두개 이상의 프로세스가 하나의 대기 프로세스로 인해 무기한 웨이팅을 하는 상황을 이야기 한다. 이러한 데드락은 starvation 이나 priority inversion 을 발생시킨다. 

```c
// P0
wait(S);
wait(Q);
// do somthing
signal(S);
signal(Q);


// P1
wait(Q);
wait(S);
// do somthing
signal(Q);
signal(S);
```

위와 같은 상황을 가정해보자. $P_0$ 입장에서는 S를 대기를 등록하고, $P_1$ 은 Q의 대기를 등록한 상태이다. $P_0$ 가 Q에 대한 대기를 등록하려했는데 $P_1$ 이  이미 대기를 등록해버려, $P_1$ 이 시그널 Q를 할때까지 기다려야한다. 하지만, 반대 입장에서도 P0가 시그널 S를 해줘야지 P1이 대기 S를 할 수 있다. 이러한 상황에서 어떠한 프로세스도 앞으로 나아가지 못하는 상황이 존재해. 
### Priority Inversion
우선순위 역전은 락으로 인해서 우선순위에 따른 스케쥴링이 안되는 경우를 말해. L,M, H 의 우선순위를 가지는 프로세스가 있는 상황을 생각해보자, 만약 L 이 현재 락을 가지고 있는 상황에서 H 가 생겨나, 근데, H도 L이 가지고 있는 락이 필요해 H가 L 을 대기하는 상황이 벌어질 때가 있어. 우선순위상으로는 H가 L 보다 먼저 실행되어야하는데, 락으로 인해 반대로 H가 L을 대기하는 상황이 벌어져. 

이 상황에서 M이 추가가 되면 더 복잡해져. 이 M은 락이 필요가 없는 얘야. H가 L의 락을 대기하는 중에 M이 도착하게 되면, 우선순위 상 M이 L의 cpu를 뺏어. 그러면 M -> L -> H 순으로 실행되는 거지. 하지만, M1, M2, ....Mn 의 프로세스가 도착하게 된다면, 운영체제 상에서 가장 높은 우선순위에 있는 프로세스가 오랜기간 실행을 못하게 되는 일이 벌어져. 이러한 문제를 막지 위해서 *priority donation* 등의 프로토콜을 사용해 우선순위를 업데이트하기도 해. 이 부분은 핀토스에서 질리도록 했으니 스킵하도록 하자. 

## Bounded Buffer Problem 
다시 처음에 봤던 bounded buffer problem 으로 돌아가자, 지금까지 설명한 세마포어 등을 사용해서 생산자와 소비자를 다시 쓸 수 있다. 
```c
do {
	/* produce an item in next_produced */
	wait(empty);
	wait(mutex);

	/* add next_produced to the buffer */

	signal(mutex);
	signal(full);
} while (true);


do {
	wait(full);
	wait(mutex);
	/* remove item from buffer to next_consumed */
	signal(mutex);
	signal(empty);
	/* consume item in next_consumed */
}
```

`empty`는 총 버퍼내 개수를 나타낸다. `wait` (P)로 `empty=n` 을 하나씩 줄여가면서, 아이템을 채워놓는다. 또한, buffer 내에 해당 값을 채워넣을때 0으로 초기화 되었었던 `full` 을 하나씩 늘려간다. 소비자는 그와 반대대는 순서로 하면 된다. 

함수로 구현하게 된다면 이렇게 된다. 
```c
get(){
	lock.acquire();
	while(front == tail) {
		empty.wait(&lock)
	}
	item = buf[front%MAX];
	front++;
	full.signal(&lock);
	lock.release();
	return item;
}

put(itme){
	lock.acquire();
	while((tail-fount) == MAX){
		full.wait(&lock)
	}
	buf[tail%MAX] = item;
	tail++;
	empty.signal(lock);
	lock.release
}
```
근데, get에서 더이상 뽑을 것이 없을 때 (`frount == tail`) 일때, 왜 while 로 체크를 할까? if 로 체크를 하는 것은 안될까? 

### Mesa / Hoare style sementic
만약 producer 가 데이터를 넣고 signal 로 consumer 가 소비할 수 있게 깨워준다. 그러면 producer 는 계속 실행되어야할까? 아니면, 멈추가 consumer 에게 프로세서를 넘겨줘야할까? producer 가 계속 실행하는 환경을 mesa style sementic 이고, consumer 에게 락과 프로세서를 모두 넘겨주는 환경인 hoare style 이다. Mesa 의 경우 producer 계속 실행중인 환경이기 때문에, 대기하는 동안 다른 스레드에 의해 condition 이 깨질 가능성이 있다. 그래서 깨어났을때 다시 한번 체크하기 위해서 `while` 을 사용해야한다. 
##### Under Mesa

| P1    | P2    | P3       | P4    | C1      | Count |
| ----- | ----- | -------- | ----- | ------- | ----- |
|       |       |          |       |         | 0     |
| put 1 |       |          |       |         | 1     |
|       | put 1 |          |       |         | 2     |
|       |       | acquire  |       |         | 2     |
|       |       | wait     |       |         | 2     |
|       |       |          |       | acquire | 2     |
|       |       |          |       | get 1   | 1     |
|       |       |          |       | signal  | 1     |
|       |       |          |       | release | 1     |
|       |       |          | put 1 |         | 2     |
|       |       | No space |       |         | X     |

##### Under Hoare

| P1    | P2    | P3       | P4    | C1               | Count |
| ----- | ----- | -------- | ----- | ---------------- | ----- |
|       |       |          |       |                  | 0     |
| put 1 |       |          |       |                  | 1     |
|       | put 1 |          |       |                  | 2     |
|       |       | acquire  |       |                  | 2     |
|       |       | wait     |       |                  | 2     |
|       |       |          |       | acquire          | 2     |
|       |       |          |       | get 1            | 1     |
|       |       |          |       | signal + release | 1     |
|       |       | put 1    |       |                  | 1     | 
|       |       |          | No Space |                  | 2     |

Hoares-style 에서는 시그널을 받은 waiter 가 즉시 일어나지만, Mesa 에서는 아니기 때문에, while 로 체크를 해서 만약에 자리가 없다면, 다시 empty 에 wait 를 해야한다. 



## Reader-Writer Problem
어떠한 데이터베이스에 대해서 읽기와 쓰기가 일어난다고 생각해보자. 모든 스레드가 읽기만 한다면 문제가 없지만, 쓰기가 끼어드는 순간 원하는 동작을 안하게 될 가능성이 높다. 첫번째 케이스는 first reader-writer 케이스이다. 쓰기가 공유 자원을 잡은 상태가 아니라면, 읽기는 무조건 먼저 들어가는 방법이다. 두번째 케이스는 seconde reader-writer 케이스다. 쓰기가 준비가 되자마자 cpu 를 뺏어서 쓰는 구조이다. 두 케이스 모두 읽기와 쓰기 둘중의 하나는 starvation 에 빠질 가능성이 있다. 

첫번째 문제 대한 해답은 다음 방법으로 해결 가능하다. 
```c
semaphore rw_mutex = 1;
semaphore mutex = 1;
int read_count = 0;

// writer 
do {
	wait(rw_mutex); // 쓸 수 있는 상태가 될때까지 rw mutex 를 대기한다
	...
	/* writing is performed */
	...
	signal(rw_mutex); // 다 쓰고 나서 signal 을 준다 
} while (true);


do {
	wait(mutex); // read_count 가 임계영역이니 wait 를 걸고 들어간다
	read_count++;
	if (read_count == 1) // 하나의 스레드가 읽고 있다면 rw_mutex 를 대기 시킨다.
		wait(rw_mutex); 
	signal(mutex); // 읽기 스레드가 하나 추가되었으니, 임계영역을 나가며 시그널을 준다
	...
	/* reading is performed */
	... 
	wait(mutex); // 다시 P
	read_count--; // 다 읽었으니 read_count 를 줄여준다
	if (read_count == 0) // 만약에 read_count 가 0에 도달하면 더이상 읽기 스레드가 없으니
		signal(rw_mutex);// 쓰기를 할 수 있게 시그널을 준다
	signal(mutex); // 다시 V
} while (true);
```
이런 식으로 하게 되면 읽기가 모두 끝나야 쓸 수 있는 상태로 reader-writer problem 해결이 가능하다. 이러한 방식의 해결책은 읽기만 하는 스레드와 쓰기만 하는 스레드의 구분이 명확한 시스템과 읽기가 intensive 하게 일어하는 시스템에서 유리하다. 

그러면 writer-preference 시스템에서는 어떤식으로 해야할까?
```c
int readcount, writercount = 0
semaphore rwmutex, wmutex, readTry, resouce = 1

read() {
	readTry.P(); // reader 의 진입을 위해 P
	rmutex.P(); // 다른 읽기의 진입을 막기 위해 P
	readcounter++;
	if (readcount == 1) // 내가 첫번째 읽기인지 체크
		resource.P() // 그러하다면, 리소스의 P 한다
	rmutex.P() // 다른 읽기에게 entry section 의 진입을 허용해준다
	readTry.P() // 읽기 시도를 완료했다고 표시한다
	/* reading resource */

	rmutex.P() // exit section 을 예약
	readcount--;
	if (readcount == 0) // 내가 마지막 읽기라면, 
		resource.V() // 리소스의 락을 해제한다
	rmutex.V() // exit section 을 해제
}

writer(){
	/* <Entry section> */
	wmutex.P() // entry 섹션 예약
	writecount++; // 쓰기 스레드 생성을 표시
	if(writecounter == 1) // 내가 첫 쓰기라면,
		readTry.P()       // 읽기 시도가 아예 못 들어오게 막는다
	wmutex.V()            // entry 섹션 해제

	resource.P();         // 리소스 예약
	/* <Critical Section> */ 
	resource.V();         // 다른 스레드들도 쓸 수 있게 해제

	/* <Exit Section> */
	wmutex.P();           // exit 섹션 예약
	writecount--;         // 쓰기 스레드의 작업 종료를 표시
	if(wriecount == 0)    // 내가 마지막 쓰기라면, 
		readTry.V();      // 읽기 시도를 열어준다
	wmutex.V();           // exit 섹션 해제	
}
```

하지만 여전히 reader 의 starvation 문제가 있다. 이러한 조건을 만족하는 문제를 third reader-writer problem 이다: 어떠한 r,w 도 굶어서는 안된다. 이에 대한 해결책은 FIFO 방식으로 온 요청의 순서대로 처리하는 것이다. 
```c
int readcount = 0
semaphore resource; // binary 세마포어
semaphore rmutex;   // readcount 의 프로텍션
semaphore serviceQueue // 요청의 순서를 관리

reader(){
	/* <Entry section> */
	serviceQueue.P()    // 작업을 하기 위해 줄서기
	rmutex.P();         // 다른 읽기의 진입을 막기 위해 P
	readcounter++;
	if (readcount == 1) // 내가 첫번째 읽기인지 체크
		resource.P()    // 그러하다면, 리소스의 P 한다
	serviceQueue.V()    // 다음으로 줄 선 사람에게 진입을 혀용
	rmutex.P()          // 다른 읽기에게 entry section 의 진입을 허용해준다

	/* <Critical Section> */ 
	
	/* <Exit Section> */
	rmutex.P() // exit section 을 예약
	readcount--;
	if (readcount == 0) // 내가 마지막 읽기라면, 
		resource.V() // 리소스의 락을 해제한다
	rmutex.V() // exit section 을 해제
}

writer() {
	/* <Entry section> */
	serviceQueue.P();    // 작업을 위해 줄서기
	resource.P();        // 리소스에 접근 요청
	serviceQueue.V();    // 다음 줄 선 사람의 진입을 허용

	/* <Critical Section> */ 
	
	/* <Exit Section> */
	resource.V()         // 다른 스레드의 리소스 접근을 허용
}
```

## Monitor
하지만, 이런식으로 직접 락을 들고다니며 키고 끄면 굉장히 실수하기가 좋다. 따라서 일부 프로그래밍 언어에서는 이른 추상화 하여 shared object 자체를 지원하기도한다. 
![](images/50bc587f14ac3e73127738b77ce092db.png)
이렇게 shared data, cond var, operations (functions), initalization functison, entry queue (lock queue) 로 구현된다. 

이러한 shared object 를 이용해 직접 락을 컨트롤하지 않고 implicit 하게 shared object 에 접근할 수 있다. 

이 예시는 단일 인스턴스의 리소스를 할당하는 모니터다
```c
monitor ResourceAllocator{
	boolean busy;
	condition x;

	void acquire(int time){
		if (busy)
			x.wait(time)
		busy = true;
	}

	void release(){
		busy = false;
		x.signal;
	}

	initialization_code(){
		busy = false;
	}
}
```
이런식으로 한다면, 개발자가 
1. 모니터의 public method 를 적절한 순서대로 호출하고
2. mutual-exclusion gateway 를 우회하고 데이터에 접근하지 않는한
공유 자원을 안전하게 추상화 하여 쓸 수 있다. 