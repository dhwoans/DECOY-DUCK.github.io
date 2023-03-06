## Test Unit

Go는 간편하게 사용할 수 있는 테스트 패키지를 내장하고 있는데, "go test" 명령을 실행하여 테스트 코드들을 실행할 수 있다. "go test"는 현재 폴더에 있는 *_test.go 파일들을 테스트 코드로 인식하고, 이들을 일괄적으로 실행한다. 

>- 파일명을 *_test.go로 지을 것.
>- 테스트 함수명을 func TestXxx(t *testing.T)의 형태로 지을 것.

테스트 파일은 "testing" 이라는 표준패키지를 사용하는데, 먼저 testing 패키지를 import 하고, 테스트 메서드를 작성한다. 

```go
import "testing"
```

테스트 메서드는 TestXxx와 같은 특별한 메서드명을 갖는데, 앞의 Test는 해당 메서드가 테스트 메서드임을 알리는 것이고 Xxx는 임의의 메서드명으로 처음 글자는 항상 대문자이어야 한다. 

메서드의 ProtoType은 아래와 같이 testing.T 포인터를 하나 입력으로 받으며 출력은 없다. 테스트 에러를 표시하기 위해 testing.T 의 Error(), Fail() 등의 메서드들을 사용한다.

```go
func TestStack_Push(t *testing.T) {
	s := NewStack[int]()
	s.Push(1)

	if len(s.nodes) != 1 {
		t.Errorf("push error")
	}
}
```

 t.Error(), t.Fatal() 을 호출하면 에러 내용을 출력할 수 있다

 ### lifecycle

 junit과 마찬가지로 라이프사이클 메서드가 존재한다.
     
 - 전처리

    - `SetupSuite` : suite에서 전체 테스트 실행 전에 한번만 실행된다
    - `SetupTest` : suite에서 각 테스트 실행 전에 실행된다
    - `BeforeTest` : 테스트가 실행하기 전에 suiteName, testName 인자를 받아 실행하는 함수이다

- 후처리

    - `AfterTest` : 테스트가 실행후에 suiteName testName 인자를 받아 실행하는 함수이다
    - `TearDownTest` : suite에서 각 테스트 실행후에 실행된다
    - `TestDownSuite` : suite에서 모든 테스트가 실행된 후에 실행된다

### testify

>Go code (golang) set of packages that provide many tools for testifying that your code will behave as you intend.

Java나 Kotlin만 하더라도 mockito나 mockK , assertJ와 같은 가독성 있는 코드를 짤수 있게 하거나 테스트 대역을 쉽게 쓸수있도록 하는 유용한 라이브러리를 제공하고 있는데 testify는 Go에서 그 역할을 하고 있다.

크게 3가지의 기능을 제공한다.

- Easy assertion
- Mock
- Test suite interface and function


