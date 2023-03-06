## JVM Architecture

JVM 기반 언어의 compiler는 .class 라는 바이트코드(`Byte Code`)로 변환시켜주는데 Byte Code 는 기계어(`Native Code`)가 아니므로 OS 에서 바로 실행이 되지 않는다.그래서 바이트코드를 실행하기 위한 소프트웨어가 필요한데 그것이 바로 JVM(`Java Virtual Machine`)이다.

### JVM 컴파일 과정

<img width="577" alt="image" src="https://user-images.githubusercontent.com/51963264/198292023-bc3cc409-6126-4c7b-a073-c97914af8370.png">

JVM의 각 구성요소들이 어떤 일을 하는지는 다음에 다뤄보기로 하고 JVM이 어떤 식으로 컴파일하는지에 대한 대략적인 과정은 다음과 같다.

1. 컴파일러(Compiler)가 소스파일을 컴파일한다. 이때 나오는 파일은 바이트 코드(.class)파일로 아직 컴퓨터가 읽을 수 없는 JVM이 이해할 수 있는 코드입니다.

2. 컴파일된 바이크 코드를 JVM의 클래스로더(Class Loader)에게 전달합니다.

클래스 로더는 동적로딩(Dynamic Loading)을 통해 필요한 클래스들을 로딩 및 링크하여 런타임 데이터 영역(Runtime Data area), 즉 JVM의 메모리에 올립니다.

3. 실행엔진(Execution Engine)은 JVM 메모리에 올라온 바이트 코드들을 명령어 단위로 하나씩 가져와서 실행합니다. 이때, 실행 엔진은 두가지 방식으로 변경합니다.

 결국 운영체제가 읽을 수 있는 기계어 변환이 최종 목적이기 때문에 JVM은 운영체제에 맞게 설치되어야 한다.

### JVM 구성요소

<img width="538" alt="image" src="https://user-images.githubusercontent.com/51963264/196954440-e4653e9d-9ffe-4cce-80e2-c5b2279410be.png">

일반적으로 JVM의 구성은 다음과 같이 크게 세가지로 나눌 수 있다

- `ClassLoader Subsystem`:   
런타임 중에 JVM의 메소드 영역에 동적으로 Java 클래스를 로드하는 역할을 담당

- `Runtime Data Area`:
JVM이 프로그램을 수행하기 위해 운영체제로부터 할당 받는 메모리 영역

- `Execution Engine`:   
RuntimeDataArea에 할당된 byteCode를 실행하는 역할을 담당
