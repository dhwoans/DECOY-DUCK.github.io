## Runtime Data Area

<img width="646" alt="image" src="https://user-images.githubusercontent.com/51963264/198342004-6281b1e1-4797-4cfb-af93-1f06843b9926.png">

Runtime Data Area은 JVM이 작업을 수행하기 위해 OS로부터 할당받은 메모리 영역이다. 5가지 영역으로 구분할 수 있다.

>- `Stack Area`:   
Method 내에서 사용되는 값들(매개변수, 지역변수, 리턴값 등)이 저장되는 영역
>- `PC Register`:   
CPU의 Register와 역할이 비슷하다. 현재 수행중인 JVM 명령의 주소값이 저장된다. (각 Thread별로 하나씩 생성)
 
>- `Native Method Stack`:   
다른 언어(C/C++ 등)의 메소드 호출을 위해 할당되는 영역
>- `Heap Area`:   
new 명령어로 생성된 인스턴스와 객체가 저장되는 구역
>- `Method Area` :      
클래스, 변수, Method, static변수, 상수 정보 등이 저장되는 영역

### Data Area 구성요소

<img width="411" alt="image" src="https://user-images.githubusercontent.com/51963264/198346022-d737e394-e184-426c-b7a9-5e936f97548d.png">

크게 Thread shared data areas 와 Per-thread data areas 로 나눌 수 있다. 

Per-thread data areas 

- PC Register 
- JVM Stack 
- Native Method Stack 

각 Thread별로 하나씩 생성되며 Thread 종료시 소멸된다.

Thread shared data areas

- Heap Aread
- Method Area

모든 Thread가 공유하며 JVM 종료시 소멸된다.


1. `PC(Program Counter) Register` 

 PC Register는 각 Thread 별로 하나씩 존재하며 현재 수행중인  JVM 명령의 주소값이 저장을 저장한다. 이때 PC Register는 CPU 내의 기억장치인 레지스터와 비슷한 역할을 하지만 Register-Base가 아닌 Stack-Base로 동작한다. 
 
 만일 Native Method를 수행한다면 PC Register는 Undefined 상태가 된다. 이 PC Register에 저장되는 Instruction 주소는 Native Pointer 일 수도 있고 Method Bytecode 일 수도 있다. Native Method를 수행할 때에는 JVM을 거치지 않고 API를 통해 바로 수행하게 된다. 이는 Java는 Platform에 종속받지 않는다는 것을 보여준다.

2. `JVM Stack Area`

 JVM Stacks은 Thread의 수행 정보를 Frame을 통해서 저장하게 된다. Thread가 시작될 때 생성되며, 각 Thread 별로 생성되기 때문에 다른 Thread는 접근할 수 없다. Method가 호출되면 Method와 Method 정보는 Stack에 쌓기에 되며, Method 호출이 종료될 때 Stack point에서 제거된다. Method 정보는 해당 Method의 매개변수, 지역변수, 임시변수 그리고 어드레스(메소드 호출한 주소) 등을 저장하고 Method 종료 시 메모리 공간이 사라진다.

3. `Native Method Stack`   

 Java 외의 언어로 작성된 네이티브 코드들을 위한 메모리 영역으로 JNI(Java Native Interface)를 통해 호출되는 C/C++ 등의 코드를 수행하기 위한 Stack이다. Natvie Method를 위해 Native Method Stack이라는 메모리 공간을 갖는다. Application에서 Natvie Method를 호출하게 되면 Native Method Stack에 새로운 Stack Frame이 생성된다. 이는 JNI를 이용하여 JVM 내부에 영향을 주지 않기 위함이다.

4. `Method Area`  

 모든 쓰레드가 공유하는 메모리 영역이다. Method Area는 클래스, 인터페이스, 메소드, 필드, Static 변수 등 바이트 코드 등을 보관한다.

5.  `Heap Area`

 힙 영역은 모든 자바 클래스의 인스턴스(instance)와 배열(array)이 할당되는 곳으로, 런타임(run time) 데이터를 저장하는 영역이다. 힙 영역은 JVM이 시작될 때 생성되어 애플리케이션이 실행되는 동안 크기가 커졌다 작아졌다 한다