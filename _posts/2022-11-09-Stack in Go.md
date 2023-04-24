---
layout: post
title: (자료구조 1) Stack in Go 
subtitle: 구조체와 리시버를 이용해 스택 구현
banner:
  image: https://velog.velcdn.com/images/decoy-duck/post/498cc390-d45c-4b4d-bad9-86c36f214efa/image.png
  opacity: 0.618
  background: "#000"
  heading_style: "font-size: 4.25em; font-weight: bold; "
author: 오재문
categories: [Data Structure]
tags: [Stack,Go]
comments: true

---

스택의 입출력은 맨 위에서만 일어나기 때문에 처음이나 중간에 데이터를 삭제하거나 추가 할 수 없습니다. 가장 최근에 들어온 순으로 제거가 되는 후입선출 `LIFO`List In First Out 로 작동하는 지료 구조이며 이러한 특성으로 문자열 뒤집기나 역추적 문제에서 유용하게 사용할 수 있습니다. 또한 스택을 이용하면 monotonic stack이라는 강력한 테그닉을 구사할 수 있는데, 기회가 된다면 다뤄보도록 하겠습니다.

---

## 스택의 작동방식

먼저 스택에 저장되는 데이터를 `요소 element`라고 합니다. 스택은 기본적으로 push와 pop이라는 연산을 사용하여 요소를 추가하거나 삭제합니다. push(element)를 수행하게 되면 비어있는 스택에 elememt가 추가됩니다. 여기에 한번 더 push 연산을 수행하면 처음 요소 위에 새로운 요소가 추가되며 pop 연산을 하면 가장 위에 쌓인 요소가 삭제 됩니다.

![image](https://user-images.githubusercontent.com/51963264/232715325-290087c0-4a9b-4b14-95b5-966914f141a7.png)


## 스택의 구현 

> 참고로 Go에서는 OOP를 직접적으로 지원해주지 않습니다. 그러나 Go언어만의 독특한 방법으로 OOP를 구현할 수는 있습니다. 처음 Go를 접한다면 이 부분에서 대부분 헤매일 것입니다. 앞으로 자료구조를 구현하면서 다양한 구현 패턴을 익혀보겠습니다. 

구현해야할 기능은 다음과 같이 정리할 수 있습니다.

```GO
package stack

// Push 요소를 스택에 추가 합니다.
func Push(item T) {}
// Empty 스택이 비어 있다면 , true를 리턴합니다.
func Empty() bool {}
// Pop 스택의 맨 위의 요소 리턴하고 삭제합니다.
func  Pop() T {}
// Peek 맨 위 요소를 리턴합니다.
func Peek() T {}
// Size 스택에 있는 요소의 갯수를 리턴합니다.
func  Size() int {}

```

위의 함수만으로는 스택이라고 말할 수 없습니다. 스택을 만들기 위해서는 스택의 행위뿐만아니라 스택의 상태값를 저장할 공간과 스택이 무엇인지 정의할 무언가가 필요합니다. 다른 프로그래밍언어라면 `클래스 class`를 사용해 스택이 무엇인지 정의 할 수 있지만 Go에서는 클래스를 지원하지 않습니다. 대신 구조체를 이용해 클래스와 비슷한 역할을 할 수 있습니다. 그리고 그 구조체와 위의 함수를 연결하여 메소드를 만들고 거기다 생성자 함수를 추가하면 스택이라고 불릴만한 무언가를 사용할 수 있습니다. 

그래서 순서를 정리하면 다음과 같습니다.

1. 구조체를 정의한다.
2. 생성자 함수를 만든다.
3. 구조체와 메서드를 연결한다.

### 1. 구조체 정의

가장 먼저 구조체를 정의합니다. 구조체는 여러 데이터를 하나의 타입으로 묶어 객체처럼 주고 받을 수 있습니다. Go 언어는 첫글자의 대소문자를 이용해서 접근을 제어를 할 수있는데 소문자일 경우 패키지 밖에서는 접근할 수없습니다. 넓은 범위에서 접근하게 되면 보안상 좋지 않기 때문에 소문자로 지정합니다.

```Go
type stack[T any] struct {
	nodes []T
}

```

### 2. 생성자 함수

구조체의 필드에 직접 접근하는 것은 좋은 방법이 아닙니다. 때문에 되도록 좁은 범위에서 접근 가능하도록 하는 것이 좋습니다. 하지만 그렇게 되면 패키지 외부에서는 스택을 생성할 수 없습니다. 이 문제를 해결하기 위해 생성자 함수를 사용합니다.

```Go
/ NewStack 생성
func New[T any]() *stack[T] {
	return &stack[T]{
		nodes: make([]T, 0),
	}
}
```

이렇게 하면 생성자함수를 통해서 패키지 외부에서도 스택을 안전하게 생성할 수 있으면서 외부에서 상태값을 변경하는것을 막을 수 있습니다.그리고 실제로 생성자함수가 제대로 스택을 생성하는지 테스트해보겠습니다.

```Go
stack_test.go


func TestNewStack(t *testing.T) {
	s := New[int]()
	if reflect.TypeOf(s) != reflect.TypeOf(&stack[int]{}) {
		t.Errorf("NewStack error")
	}
}
```
### 3. 메소드 

구조체와 연결된 함수를 메소드라고 합니다. Go에서는 함수와 구조체를 연결하기 위해 리시버라는 것을 이용합니다. 리시버를 이용해서 구조체의 필드에 접근할 수 있습니다. 그리고 요구사항에 맞게 로직을 채워 넣으면 메소드가 완성됩니다. 

로직을 작성하는 것은 매우 쉽습니다. 대부분의 경우 테스트를 생략해도 괜찮지만 테스트 코드를 작성하고 실행하는 개발 습관을 만드는 것이 앞으로 있을 복잡한 로직을 작성할 때 도움이 될 것같습니다. 전에 테스트 코드를 먼저 작성해보겠습니다. 

```Go

func TestStack_Push(t *testing.T) {
	s := New[int]()
	s.Push(1)
	if len(s.nodes) != 1 {
		t.Errorf("push error")
	}
}

```
num이 아닌 다른 수가 들어가거나 아예 들어가지 않을 경우 에러를 출력하는 테스트 코드를 작성했습니다. 아직 로직을 작성하지 않았기 때문에 곧바로 실행할 경우 테스트는 실해하게 됩니다. 이렇게 테스트 코드를 먼저 작성하는 이유는 로직을 먼저 작성할 경우 테스트 통과를 쉽게 하기 위해 테스트 코드 작성을 수비적으로 할 수 있기 때문에 로직 작성전에 테스트 코드를 먼저 작성하는 것이 핵심입니다. 

```Go

// Push 요소 추가
func (this *stack[T]) Push(item T) {
	this.nodes = append(this.nodes, item)
}

```
로직을 작성하고 테스트를 통해 기능이 완벽하게 수행되는지 확인할 수 있습니다. 이런 식으로 나머지 로직도 하나씩 테스트 코드를 먼저 작성하는 것으로 만들어 나갈 수 있습니다. 한번에 많은 테스트 코드를 작성할 경우 심리적으로 부담이 될 수 있기 때문에 처음 작게 시작하고 점진적으로 테스트 코드를 더해나가는 것이 좋습니다. 

그렇게 해서 완성된 전체로직은 다음과 같습니다.

```GO

type stack[T any] struct {
	nodes []T
}

// NewStack 생성
func New[T any]() *stack[T] {
	return &stack[T]{
		nodes: make([]T, 0),
	}
}

// Push 요소 추가
func (this *stack[T]) Push(item T) {
	this.nodes = append(this.nodes, item)
}

// Empty 비어 있는지 확인
func (this *stack[T]) Empty() bool {
	return len(this.nodes) == 0
}

// Pop 요소 삭제
func (this *stack[T]) Pop() T {
	result := this.nodes[len(this.nodes)-1]
	this.nodes = this.nodes[:len(this.nodes)-1]
	return result
}

// Peek 맨 위 요소 확인
func (this *stack[T]) Peek() T {
	return this.nodes[len(this.nodes)-1]
}

// Size 크기
func (this *stack[T]) Size() int {
	return len(this.nodes)
}

```

## 검증

테스트를 통과했다면 이제 실제로 알고리즘 문제를 풀어보면서 이상이 없는지 확인 해 보겠습니다. 적용해볼 문제는 [10828. 스택](https://www.acmicpc.net/problem/10828)으로 스택을 이용하면 간단히 풀 수있는 문제 입니다. 문제의 요구사항은 다음과 같이 요약할 수 있습니다.

- push X: 정수 X를 스택에 넣는 연산이다.
- pop: 스택에서 가장 위에 있는 정수를 빼고, 그 수를 출력한다. 만약 스택에 들어있는 정수가 없는 경우에는 -1을 출력한다.
- size: 스택에 들어있는 정수의 개수를 출력한다.
- empty: 스택이 비어있으면 1, 아니면 0을 출력한다.
- top: 스택의 가장 위에 있는 정수를 출력한다. 만약 스택에 들어있는 정수가 없는 경우에는 -1을 출력한다.

문제를 풀 수 있는 방법은 여러 가지있을 수 있지만 다음과 같이 해결 할 수 있습니다.

```Go
...
var n int
fmt.Fscanln(r, &n)
stack := stack.New[int]()
for i := 0; i < n; i++ {
	var command string
	fmt.Fscan(r, &command)
	switch command {
	case "push":
		var num int
		fmt.Fscan(r, &num)
		stack.Push(num)
	case "pop":
		if stack.Empty() {
			fmt.Fprintln(w, -1)
		} else {
			fmt.Fprintln(w, stack.Pop())
		}
	case "size":
		fmt.Fprintln(w, stack.Size())
	case "empty":
		if stack.Empty() {
			fmt.Fprintln(w, 1)
		} else {
			fmt.Fprintln(w, 0)
		}
	case "top":
		if stack.Empty() {
			fmt.Fprintln(w, -1)
		} else {
			fmt.Fprintln(w, stack.Peek())
		}
	}
}
```
<img width="720" alt="image" src="https://user-images.githubusercontent.com/51963264/233943863-bc561e16-0960-45b4-bbc7-12a80940943b.png">

---

## 마치며

스택 구현을 통해 구조체를 정의히고 함수와 구조체를 연결하여 메소드를 만들어보았습니다. 그리고 생성자 함수를 통해 안전하게 스택을 생성하는 방법도 익혀 보았습니다. 자료구조는 여러가지 방법으로 구현할 수있습니다. 그리고 훨씬 더 복잡한 로직을 요구하는 자료구조 또한 존재합니다. 앞으로 다양한 자료구조를 구현해보면서 위와 같은 요구사항들을 Go로 어떻게 해결할 수있는지 배워나가 보도록 하겠습니다.
