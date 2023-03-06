## OOP

자바의 경우 클래스 밖의 함수는 존재할수 없기 때문에 자바의 함수는 `메서드`라 부르지만 Go의 경우 메소드 뿐만 아니라 함수도 존재 한다.

>💡 용어 정리
>
>함수(function) - 처리의 묶음   
객체(object) - 변수의 묶음   
구조체(struct) - 객체를 선언하는 방법   
리시버(reciever) - 객체와 함수를 연결하는 매개체  
메소드(method) - 객체에 연결된 함수    
value receiver - 객체를 value로 가져와서 사용하는 리시버   
pointer receiver - 객체를 referene(pointer)로 가져와서 사용하는 리시버 

### Go에서의 메서드

 Go는 OOP를 직접적으로 지원하지 않는다. 그래서 Go 언어에서는 class가 아닌 struct와 interface를 통해 구현할 수 있다. struct 필드만을 가지며, 메서드는 별도로 분리되어 정의해야 되는데 그역할은 interface가 담당한다.

>💡 Go에서의 OOP
>
>클래스를 구조체로 구현함수로 생성자 Constructor구현   
함수를 구조체 리시버 Receiver로 연결하여 메서드 Method를 구현   
상속 대신 임베딩 Embedding으로 구조체를 재사용   
인터페이스로 다형성 Polymorphism을 구현   


메서드는 함수의 한 종류로써, 타입에 속한 함수를 메서드라고 하며 어떤 타입도 메서드를 가질 수 있습니다.

Go에서 메서드는 다음과 같이 정의한다.

```go
type Node struct{
    a int
}

func (s *Node) 메서드명 (변수명 변수타입){
  //To do
}
```

### receiver

 리시버(receiver)는 값 뿐만 아니라 포인터로 선언 할수 있는데 이것을 pointer receiver라 부르며 값으로 선언했을 경우 value receiver로 구분해서 부른다. 







