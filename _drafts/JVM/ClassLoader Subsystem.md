## ClassLoader Subsystem

JVM은 RAM에 있는데 실행 동안 클래스 로더 서브시스템을 사용해서 클래스 파일을 RAM으로 가져온다. 이것을 `동적 클래스 로딩 기능`이라고 하는데 처음으로 클래스를 참조할 때 클래스 파일을 로드하고 링크하고 초기화 한다. 이 과정은 런타임에 일어난다.

<img width="678" alt="image" src="https://user-images.githubusercontent.com/51963264/198336538-00180211-7f05-44a6-bdf9-249c72251607.png">

클래스로더 서브시스템의 프로세스도 크게 3가지로 나눌수 있다.

1. 로딩(Loading)
2. 연결(Linking)
3. 초기화 (Initialize)
