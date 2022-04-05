# 💸 은행 창구 매니저 프로젝트

* 팀 프로젝트(2인)
* 프로젝트 기간: 2021.12.20 ~ 2021.12.31

# 목차

- [키워드](#키워드)
- [프로젝트 소개](#%EF%B8%8F-프로젝트-소개)
- [프로젝트 주요기능](#-프로젝트-주요기능)
- [Trouble Shooting](#-trouble-shooting)
    + ["참조타입을 활용한 연결리스트"](#참조타입을-활용한-연결리스트)
    + ["와일드카드 패턴으로 생성한 인스턴스의 참조 카운트 상태는?"](#와일드카드-패턴으로-생성한-인스턴스의-참조-카운트-상태는)
    + ["지레짐작하여 설계하지 말기"](#지레짐작하여-설계하지-말기)
    + ["Thread Safe하게 코드짜보기"](#thread-safe하게-코드짜보기)
    + ["뷰의 요소를 비동기적으로 업데이트 하기"](#뷰의-요소를-비동기적으로-업데이트-하기)
    + ["멈추지 않는 타이머"](#멈추지-않는-타이머)
- [새롭게 알게된 것](#-새롭게-알게된-것)
    + ["코드에서 경쟁 상태를 확인하는 방법"](#코드에서-경쟁-상태를-확인하는-방법)
    + ["Core Foundation vs Foundation"](#core-foundation-vs-foundation)
    + ["구조체 프로퍼티는 클로저 내부에서 왜 값을 변경할 수 없는가?"](#구조체-프로퍼티는-클로저-내부에서-왜-값을-변경할-수-없는가)
    + ["DispatchQueue는 중간에 작업을 중지시킬 수 있을까?"](#dispatchqueue는-중간에-작업을-중지시킬-수-있을까)

# 키워드

- `Queue`
    - `LinkedList`
- `Generic`
- `Protocol`
- `Delegate Pattern`
- `SOLID : SRP`
- `Wildcard Pattern`
- `Strong Reference Cycle`
- `Core Foundation` vs `Foundation`
- `CFAbsoluteTime` `Date`
- `GCD`
    - `DispatchQueue`
    - `Semaphore`
    - `DispatchGroup`
    - `async` `sync`
    - `Serial` `Concurrent`
- `UI 구성 (only Code)`
- `Timer` `RunLoop`
- `Custom View`
- `Auto Layout`

</br>

## ⭐️ 프로젝트 소개

은행 창구 업무 스케줄을 관리하는 기능을 동시성 프로그래밍을 활용하여 구현한 앱이에요. 🏦

손님이 입장한 순서대로 대기표를 나눠주고, 두개의 창구에서 맡은 업무를 **동시에 순서대로** 처리해요 ! 💪🏻

</br>

## ✨ 프로젝트 주요기능

> ### 👩🏻‍💻 고객 추가버튼을 누르면 업무를 대기순서대로 시작하고, 모든 업무를 마치면 업무시간 타이머가 일시정지해요.

<img src="https://i.imgur.com/1kzy1SC.gif" width=30%>

> ### 🏃🏻‍♀️ 업무중에도 고객을 더 추가할 수도 있어요.

<img src="https://i.imgur.com/pSgdvCq.gif" width=30%>

> ### ❌ 초기화 버튼을 누르면, 하던 업무를 중지하고 은행 업무를 종료해요.

<img src="https://i.imgur.com/tlTWuH3.gif" width=30%> 
 
</br>

## 🛠 Trouble Shooting

### "참조타입을 활용한 연결리스트"

- `상황` append를 할때 연결된 리스트들의 가장 마지막 부분에 새로운 데이터를 넣어줘야하는데, 요소를 탐색하기 위해 반복적으로 포인터를 추적하는 작업을 해야하는 점을 깔끔하게 해결해보고 싶었다.
- `이유` Queue를 위한 LinkedList인 점도 있고, 또 각 요소를 탐색하기 위해 반복적으로 포인터를 추적하게 된다면 시간복잡도가 O(n)이기 때문에 데이터 접근 방식이 매우 비효율적인 점을 해결하고 싶었다.
- `해결`  클래스의 참조를 활용하여 연결리스트를 연결하도록 구현하였다.
    
    ```swift
    func append(_ value: Element) {
        let newNode = Node(value: value)
            
        if isEmpty {
            head = newNode
            tail = newNode
        } else {
            tail?.next = newNode
            tail = newNode
        }
    }
    ```
<img src="https://i.imgur.com/C3uId1Z.png" width=80%>

#

### "와일드카드 패턴으로 생성한 인스턴스의 참조 카운트 상태는?"

- `상황` 기존에 와일드카드 패턴으로 인스턴스 생성한 타입이 Delegate를 채택하고 있던 형태였다. 이후 순환참조 문제가 우려되어 각 타입들마다 delegate 프로퍼티에 weak 키워드를 붙여주었다. 
- `이유` 그런데 weak 키워드를 붙여주니 해당 타입에서 출력해주었던 메소드가 실행되지 않았다. init과 deinit을 통해 디버깅을 해보니 와일드카드 패턴으로 생성한 인스턴스가 생성과 동시에 해제되는 것을 확인할 수 있었다. `와일드카드 패턴`은 **값을 해체하거나 무시하는 패턴**중 하나이므로 `weak 키워드`가 추가됨과 동시에 retain count가 올라가지 않기 때문에 생성과 동시에 해제되는 것이였다.
- `해결` 사용하지 않는 viewController(은행원, 은행)를 상수에 담으면 xcode에서 경고 메세지가 발생한다. 그럼에도 불구하고 와일드 카드 패턴을 사용해서 순환참조 문제를 해결할 수 있다고 판단되어 weak 키워드를 제거하고 와일드카드 패턴을 사용하기로 결정했다.
    
    ```swift
    final class Bank {
        private let bankClerk: BankClerk
        private var customerQueue = Queue<Customer>()
        var delegate: BankDelegate? // weak 키워드 제거
    ...
    }
    
    func run() {
        let bankClerk = BankClerk()
        let bank = Bank(bankClerk: bankClerk)
        let bankManager = BankManager(bank: bank)
        let _ = BankClerkViewController(bankClerk: bankClerk)
        let _ = BankViewController(bank: bank) 
    ...
    }
    ```

#

### "지레짐작하여 설계하지 말기"

<img src="https://i.imgur.com/RFm5zea.png" width=60%>

* `상황`
    * 팀원인 허황과 나는 콘솔앱 다음 스텝에 있을 ViewController를 추가하는 작업까지 추측하여 미리 콘솔앱을 구현할 때부터 Delegate 패턴까지 고려하여 설계를 하였다. (위 사진은 당시 작성했던 UML이다.)
    * 리뷰어에게 코드리뷰를 받고 다시 살펴보며 고민해본 결과, 콘솔앱에서 불필요한 역할 분리가 일어나서 오히려 코드 가독성이 매우 떨어졌다.
    
    <img src="https://i.imgur.com/Jxv8sVw.png" width=60%>
    
* `이유`
    * 당장 만들어야할 기능이 아니라 추후 추가될 기능까지 추측하여 설계한 탓이 큰 문제점인 것 같다.
* `해결`
    * 따라서 다음 기능이 어떻게 추가될 지 알 수 없는 상태에서는 최소한의 기능만 빠르게 작업해버릇하는 습관을 들여야겠다는 생각이 들었고, 나중에 기능 요구사항이 추가된다면, 거기에 맞게 다시 설계를 해야겠다는 생각이 들었다.
    * 추측하여 설계한 부분을 다 [수정](https://github.com/leeari95/ios-bank-manager/commit/f390adfb033eca8bc2a013188d2d63356d50d6be)하였다. 추후 기능 요구사항이 나온다면, 그때 다시 설계해보기로 하였다.
* `느낀점`
    * 다음 스텝을 지레짐작해서 개발하지 말자
    * 그냥 요구사항에 맞춰 최소한의 기능을 빨리 만들어보자.
    * 시도하고 실패하고 돌아가는 과정을 반복하며 개선해나가는 방향으로 진행하자.

# 

### "Thread Safe하게 코드짜보기"

* `상황`
    - 은행원의 수만큼 DispatchQueue를 만들어주고 있고, 그 안에서 dequeue를 하기 위해 고객의 큐에 접근하고 있다.
        - 여러 스레드에서 고객의 큐를 접근할 수 있는 가능성이 있다.
        - 그러나 현재 우리 코드에서는 race condition없이 잘 작동한다. 이유가 무엇일까?
* `이유`
    ```
    1. 일단 은행원의 수(DispatchQueue)가 적다. 3개뿐이다. 
       따라서 스레드는 3개 이상 생기지 않는다.
    2. 그리고 각 업무를 할때 스레드를 잠재운다. 딜레이가 있다.
    3. 고객의 숫자도 적은편이다.
    ```
    - 따라서 결론은 현재 셋팅(`은행원 수`, `고객 수`, `딜레이`) 때문에 race condition이 발생하지 않는다는 판단이 들었다.
    - 현재는 race condtion이 발생하지 않고 정상적으로 작동하지만, 결국에 `Queue`는 `Thread-Safe` 하지 않기 때문에, race condition이 발생하지 않을 것이라고도 장담할 수 없다는 결론이다.
        - 따라서 DispatchSemaphore를 사용해서 접근할 수 있는 스레드의 수를 제한해주는 것이 안전할 것 같다는 생각이 들었다.
        - 하지만 이 방법도 테스트 결과 `customers.dequeue` 작업들이 for문으로 인해 작업이 쌓여있고, 큐의 접근을 세마포어로 제한하고 있으니, 제한하는 동안 쌓인 작업들이 제한이 풀리고 실행되면 비어있는 큐에 접근할 수 있기 때문에 fatalError가 발생할 수 있다.
* `해결`
    - 현재 global()로 동시성큐를 사용하고 있는데, 이 부분을 Serial Queue로 만들어서 질서를 지키게 해주면 매우 안전할 것 같다. 👍
        - 테스트 결과, race condition 발생 하지 않았다!!!!!
* **개선된 코드**
- ![](https://i.imgur.com/R2jT5ox.png)
    * 멤버변수로 Serial Queue를 생성해주고 DispatchQueue.global() 대신 직접 생성해준 bankerQueue를 사용하도록 개선하였다.
        * DispatchQueue가 지역변수가 아니라 멤버변수여야하는 이유는 뭘까?
            * DispatchQueue에 비동기로 큐를 보내고 있는 부분이 for문에 의해 4번 불린다고 했을 때, 지역변수로 DispatchQueue가 4개, 즉 각각의 작업마다 생성될테니 실질적으로 동시에 접근하는 것을 막아주지 못할 것이다.
            * 따라서 for문 내부에서 Queue를 생성하는 것이 아니라 바깥에서 생성해서, 그 해당 큐를 사용하는 방식으로 활용해야 한다.
            * Semaphore를 활용할 때에도 위와 마찬가지로 멤버변수여야 한다!!!!

#

### "뷰의 요소를 비동기적으로 업데이트 하기"

- `상황` UIButton에 View의 요소를 업데이트 하도록 기능을 추가하고 테스트 해보았으나, 버튼이 클릭됨가 동시에 화면이 멈추고 에러가 나면서 뷰가 업데이트 되지 않는 상황을 마주했다.

<img src="https://i.imgur.com/0S10HBs.png" width=30%>

- `이유` 비동기 프로그래밍시 뷰를 그리는 작업은 main thread에서 처리해주어야 하는데, 따로 처리해주지 않아서 해당 에러가 발생했던 것 같다.
- `해결` 뷰를 그리는 작업을 main thread에서 처리하도록 DispatchQueue.main.async 클로저 구문안에 넣어주었다.

#

### "멈추지 않는 타이머"

- `상황` 고객 추가버튼을 여러번 클릭하고 초기화를 누르게 되면 타이머가 멈추지 않았다.

<img src="https://i.imgur.com/fvZWEq1.gif" width=30%>

- `이유` 타이머가 추가버튼 클릭시 계속 추가되면서 추가된 타이머가 끝나지 않고 계속 실행되는 듯 했다.
- `해결` 초기화 버튼을 클릭할 때, 대기중인 고객이 없을 때 타이머를 멈추면서 타이머에 nil을 대입해주었고, 타이머가 작동중에는 타이머를 추가하지 않도록 하는 로직을 추가해주니 해결되었다.

<img src="https://i.imgur.com/VsvkGMM.gif" width=30%>


</br>

## 🔥 새롭게 알게된 것

### "코드에서 경쟁 상태를 확인하는 방법"

> 동시성 프로그래밍을 적용해보다가, 동시에 여러 스레드가 접근하는 일이 발생하고 있는지 디버깅 해보고 싶어서 알아보다가 알게되었다.

* Xcode에서 ..
    * `Product > scheme > editScheme > Run > Diagnostics > Thread Sanitizer`
    * `Thread Sanitizer` 이걸 체크하면 빌드를 돌리고 나서 thread safe 하지 않은 상황이 발생할 수 있는 가능성을 엑스코드에서 체크해준다.
    * 하지만 사용해보니 완벽하게 체크해주는 건 아닌 것 같다. 그냥 도와주는 기능이라고 생각하고 사용해야할 것 같다.

![](https://i.imgur.com/aEg0Qbv.png)

> Thread Safe하게 코드를 작성하려면?

* 공유자원을 읽고 쓰는 작업을 Thread safe하게 Shemaphore를 사용해서 하나의 thread만 접근 할 수 있도록 하는 방법이 있다.
    * 하지만 이 방법은 완벽하게 제어하기는 무리가 있다. 오히려 공유자원을 lock으로 처리하다가 교착 상황 발생할 가능성이 높다고 한다.
* Serial Queue sync로 보내서 처리하는 방법도 있다.
    * 그러면 들어온 task에 순서가 생기기 때문에 다수의 스레드에서 동시에 값을 접근하지 못하게 하는 상황이 된다.
    * sync로 사용하는 이유는 Serial queue로 보낸 작업을 기다림으로써 공유자원의 제대로 된 값을 얻기 위함이다.

#

### "Core Foundation vs Foundation"

> CFAbsoluteTime 타입 대신에 Date 타입을 활용하여 연산시간 측정하도록 수정하였다. Core Foundation 내장함수보다 Foundation에 있는 기능을 활용하는 것이 더 옳다고 판단되었다.

* 코어 파운데이션에 있는 기능은 Foundation에서 래핑하여 구현되어져있다.
* 보통 앱개발을 할 때에는 Foundation의 기능 없이 개발하기에는 어려움이 있기 때문에, Core Foundation에 내장되어있는 기능보다는 Foundation에 내장되어있는 기능을 사용하는 것을 선호하는 편이다.

#

### "구조체 프로퍼티는 클로저 내부에서 왜 값을 변경할 수 없는가?"

- `상황` 프로젝트에서  `Bank 타입`은 `구조체`이고, `numberOfCustomers 프로퍼티`를 가지고 있다. 은행원이 일을 할 때, 처리한 고객의 수를 `Dispatch.global().async` 클로저 내에서 카운트(`numberOfCustomers`) 해주도록 하려고 했으나 아래와 같은 에러가 발생하였다.
- `이유`  `Escaping closure`의 경우 구조체에서는 캡쳐가 불가능하기 때문에, 프로퍼티를 변경하려고 하면 아래와 같은 에러가 발생한다.

```swift
// 간단한 코드 예시
struct SomeStruct {
    var num = 0
    
    private mutating func test() {
        let closure = { // Escaping closure captures mutating 'self' parameter
            self.num += 1
        }
        closure()
    }
}
```
- `Escaping closure`의 경우 구조체에서는 캡쳐가 불가능하기 때문에, 프로퍼티를 변경하려고 하면 위와 같은 에러가 발생한다.
    
    > class와 같은 참조 타입이 아닌 Struct, enum과 같은 값타입에서는 mutating reference의 캡쳐를 허용하지 않기 때문에 self 사용이 불가능 하다.
    > 
- `해결` `Bank 타입`을 `클래스`로 바꿔주었다.

#

### "DispatchQueue는 중간에 작업을 중지시킬 수 있을까?"
* cancel 메서드를 호출하더라도 작업이 작업 중이라면 중지시킬 수 없다.
* 우회하는 방법이 존재하지만 GCD가 작업을 취소하거나 하는 것은 할 수는 없다.
* Operation Queue를 사용하면 작업 중인 작업을 중지 할 수 있다. [[참고 링크]](https://developer.apple.com/documentation/foundation/operationqueue/1417849-cancelalloperations)

</br>

[![top](https://img.shields.io/badge/top-%23000000.svg?&amp;style=for-the-badge&amp;logo=Acclaim&amp;logoColor=white&amp;)](#-은행-창구-매니저-프로젝트)

<details>
<summary>[학습 기록 흔적]</summary>
<div markdown="1">

# 목차

- [STEP 1 : 큐 타입 구현](#step-1--큐-타입-구현)
    + [고민했던 것](#1-1-고민했던-것)
    + [의문점](#1-2-의문점)
    + [Trouble Shooting](#1-3-trouble-shooting)
    + [배운 개념](#1-4-배운-개념)
    + [PR 후 개선사항](#1-5-pr-후-개선사항)
- [STEP 2 : 타입 구현 및 콘솔앱 구현](#step-2--타입-구현-및-콘솔앱-구현)
    + [고민했던 것](#2-1-고민했던-것)
    + [의문점](#2-2-의문점)
    + [Trouble Shooting](#2-3-trouble-shooting)
    + [배운 개념](#2-4-배운-개념)
    + [PR 후 개선사항](#2-5-pr-후-개선사항)
- [STEP 3 : 다중 처리](#step-3--다중-처리)
    + [고민했던 것](#3-1-고민했던-것)
    + [의문점](#3-2-의문점)
    + [Trouble Shooting](#3-3-truoble-shooting)
    + [배운 개념](#3-4-배운-개념)
- [STEP 4 : UI 구현](#step-4--ui-구현)
    + [고민했던 것](#4-1-고민했던-것)
    + [의문점](#4-2-의문점)
    + [Trouble Shooting](#4-3-trouble-shooting)
    + [배운 개념](#4-4-배운-개념)

# STEP 1 : 큐 타입 구현

은행에 도착한 고객이 임시로 대기할 대기열 타입을 구현합니다.

## 1-1 고민했던 것

- 양방향 LinkedList vs 단방향 LinkedList
    - 서로 이야기를 나누면서 단방향으로도 충분히 큐를 구현할 수 있다는 생각이 듦.
    - 노드를 클래스로 만들고 참조를 활용하여 모든 노드를 연결할 수 있게 구현하였다.
- 테스트를 위한 Mock 타입 생성
    - 테스트 진행시 `Node`, LinkedList의 `head`, `tail`의 요소를 접근하여 테스트 결과를 확인했다.
    - 해당 요소들은 외부에서 접근하면 안된다는 판단이 들어서 따로 MockLinkedList를 만들어주어 원활한 테스트를 할 수 있도록 구현하였다.
- attribute를 활용
    - 반환 값을 유의미하게 사용하지 않고 버려도되는 remove 관련 메소드에 속성 `@discardableResult`을 부여하여 컴파일러 경고가 발생하지 않도록 하였다.

## 1-2 의문점

- if vs guard
    
    ```swift
    // 1. if 구문
    func removeFirst() -> Element? {
        if isEmpty {
            return nil
        }
        let result = head?.value
        head = head?.next
        
        return result
    }
    ```
    
    ```swift
    // 2. guard 구문
    func removeFirst() -> Element? {
        guard isEmpty == false else {
            return nil
        }
        let result = head?.value
        head = head?.next
        
        return result
    }
    ```
    
- if문으로 구현 시 조건문이 깔끔하게 isEmpty로 맞아떨어지기 때문에 코드의 가독성이 좋다고 생각했다.
    - guard문이든 if문이든 메소드 시작부문에서 return을 활용하여 메소드를 탈출하는건 동일하기 때문에 어떤 구문을 사용하든 상관없다는 생각이 들었다.
- guard문
    - guard 구문의 기능은 코드 블럭의 빠른 종료 기능을 가지고 있어 함수 전체 흐름을 봤을 때 guard 구문이 가독성이 더 좋다고 생각했다. guard문만 보더라도 함수를 탈출하는 구문이라고 읽혀지기 때문이다.
    - if문 처럼 조건문이 깔끔하지는 못하게 되었지만, 함수를 탈출하는 부분을 guard문으로 일관성 있게 구성할 수 있게 되었다.

## 1-3 Trouble Shooting

### [1] 참조타입을 활용한 연결리스트

- `상황` append를 할때 연결된 리스트들의 가장 마지막 부분에 새로운 데이터를 넣어줘야하는데, 요소를 탐색하기 위해 반복적으로 포인터를 추적하는 작업을 해야하는 점을 깔끔하게 해결해보고 싶었다.
- `이유` Queue를 위한 LinkedList인 점도 있고, 또 각 요소를 탐색하기 위해 반복적으로 포인터를 추적하게 된다면 시간복잡도가 O(n)이기 때문에 데이터 접근 방식이 매우 비효율적인 점을 해결하고 싶었다.
- `해결`  클래스의 참조를 활용하여 연결리스트를 연결하도록 구현하였다.
    
    ```swift
    func append(_ value: Element) {
        let newNode = Node(value: value)
            
        if isEmpty {
            head = newNode
            tail = newNode
        } else {
            tail?.next = newNode
            tail = newNode
        }
    }
    ```

![](https://i.imgur.com/C3uId1Z.png)
    

## 1-4 배운 개념

- Linked-list 자료구조의 이해
- Queue 자료구조의 이해

## 1-5 PR 후 개선사항

- 테스트 코드 내부에서 override한 메소드 내부에 super 메소드를 호출하도록 수정
- LinkedList를 테스트하기 위한 Mock 객체 삭제
    - LinkedList 타입에서 접근할 수 있는 프로퍼티, 메소드를 활용하여 테스트를 하도록 리팩토링
- 테스트 메소드 이름을 적절하게 수정
- QueueTests에서 `setUp()` 메소드를 활용하도록 수정


[![top](https://img.shields.io/badge/top-%23000000.svg?&amp;style=for-the-badge&amp;logo=Acclaim&amp;logoColor=white&amp;)](#목차-1)

---

# STEP 2 : 타입 구현 및 콘솔앱 구현

은행과 고객의 타입을 구현하고 콘솔앱을 구현한다.

## 2-1 고민했던 것

- 구현한 타입들 각각 하나의 책임만 갖도록 고민해보았다.
    - `출력문`까지도 은행원, 은행이 해야할 일이 아니라고 판단되었기 때문에 따로 출력만 하는 타입(ViewController)을 만들어 `Delegate 패턴`을 활용하여 분리해주었다.
    - 요구사항 예시에 맞춰 출력을 해주기 위해 NumberFormatter를 활용하는 부분이 필요했는데, 이 부분은 은행에서 직접 처리하는 건 적합하지 않다는 생각이 들었다. 따라서 따로 타입으로 빼주어 static 메소드로 구현해주었다.
- 순환참조
    - `Delegate 패턴`을 사용하기 위해서 `모델 타입` 별로 `viewController`를 만들어주었다.
    - `모델 타입`은 `delegate`로 `viewController`를 가지고 있고 `viewController`은 자기 자신을 `delegate`로 넘겨주기 위해 `모델 타입`을 가지고 있어 `순환 참조`가 발생한다.
    - `모델 타입`의 `delegate`를 weak 키워드(약한 참조)를 사용해 `순환참조`를 해결할 수 있다.
- Delegate 패턴 작성시 네이밍에 대한 고민을 해보았다.
    - 구글의 [Swift Style Guide](https://google.github.io/swift/#delegate-methods)를 참고하여 네이밍을 하였다.
        
        > The term “delegate’s source object” refers to the object that invokes methods on the delegate. For example, a `UITableView` is the source object that invokes methods on the `UITableViewDelegate` that is set as the view’s `delegate` property.
        > 

## 2-2 의문점

- Delegate 패턴 구현을 위해 기존 모델 타입에 변경사항이 생겼는데 이게 올바른 것인지 잘 판단이 서질 않는다.
    - 기존 구조체였던 타입들을 `class`로 변경해준 부분이 추후 retain count 추적 비용이 발생할 것이라는 우려가 생겼다.
- Delegate 패턴 사용시 강한 순환 참조 발생을 예방하기 위해 weak 키워드를 사용해주었더니 와일드카드 패턴으로 생성한 인스턴스가 생성과 동시에 해제가 되었다.
    - 인스턴스를 생성한 후 참조를 하지 않는다면 weak 키워드 보다는 와일드카드 패턴을 사용하는 것이 적합한 것일까?
- 단일 책임 원칙에 맞게 역할을 Delegate 패턴을 활용하여 분리해주었다. 하지만 확장성을 고려했을 때 이게 올바르게 설계한 것이 맞는지 너무 오버 엔지리어닝한 부분은 아닌지 판단이 어렵다.

## 2-3 Trouble Shooting

### [1] 와일드카드 패턴으로 생성한 인스턴스의 참조 카운트 상태는?

- `상황` 기존에 와일드카드 패턴으로 인스턴스 생성한 타입이 Delegate를 채택하고 있던 형태였다. 이후 순환참조 문제가 우려되어 각 타입들마다 delegate 프로퍼티에 weak 키워드를 붙여주었다.
    
    ```swift
    final class Bank {
        private let bankClerk: BankClerk
        private var customerQueue = Queue<Customer>()
        weak var delegate: BankDelegate?
    ...
    }
    
    func run() {
        let bankClerk = BankClerk()
        let bank = Bank(bankClerk: bankClerk)
        let bankManager = BankManager(bank: bank)
        let _ = BankClerkViewController(bankClerk: bankClerk)
        let _ = BankViewController(bank: bank) // 
    ...
    }
    ```
    
- `이유` 그런데 weak키워드를 붙여주니 해당 타입에서 출력해주었던 메소드가 실행되지 않았다. init과 deinit을 통해 디버깅을 해보니 와일드카드 패턴으로 생성한 인스턴스가 생성과 동시에 해제되는 것을 확인할 수 있었다. `와일드카드 패턴`은 값을 해체하거나 무시하는 패턴중 하나이므로 `weak 키워드`가 추가됨과 동시에 retain count가 올라가지 않기 때문에 생성과 동시에 해제되는 것이였다.
- `해결` 사용하지 않는 viewController(은행원, 은행)를 상수에 담으면 xcode에서 고메세지가 출력된다. 와일드 카드 패턴을 사용해서 순환참조 문제를 해결할 수 있다고 판단되어 weak 키워드를 제거하고 와일드카드 패턴을 사용하기로 결정했다.
    
    ```swift
    final class Bank {
        private let bankClerk: BankClerk
        private var customerQueue = Queue<Customer>()
        var delegate: BankDelegate?
    ...
    }
    
    func run() {
        let bankClerk = BankClerk()
        let bank = Bank(bankClerk: bankClerk)
        let bankManager = BankManager(bank: bank)
        let _ = BankClerkViewController(bankClerk: bankClerk)
        let _ = BankViewController(bank: bank) 
    ...
    }
    ```
    

## 2-4 배운 개념

- 타입 추상화 및 일반화
- Delegate 패턴 활용
- SOLID의 단일 책임 원칙
- 와일드카드 패턴과 weak 키워드의 관계

## 2-5 PR 후 개선사항

- 개발할 때 염두해야할 것
    - 기능을 선택할 때에는 비슷한 기능은 무엇이 있는지 탐색하기.
    - 선택한 기능이 다른 기능보다 나은점은 뭐가 있는지 생각해보기
    - 기능을 찾아 사용할 때 선택한 이유를 생각해보기
- `CFAbsoluteTime` 타입 대신에 Date 타입을 활용하여 연산시간 측정하도록 수정
    - `Core Foundation` 내장함수보다 `Foundation`에 있는 기능을 활용하려는 의도
        - Core Foundation vs Foundation
            - 코어 파운데이션에 있는 기능은 Foundation에서 래핑하여 구현되어져있다.
            - 보통 앱개발을 할 때에는 Foundation의 기능 없이 개발하기에는 어려움이 있기 때문에, Core Foundation에 내장되어있는 기능보다는 Foundation에 내장되어있는 기능을 사용하는 것을 선호하는 편이다.
- `Delegate` 패턴과 `ViewController` 타입 모두 삭제
    - 다음 STEP를 지레짐작하지 말고 현재 STEP에 충실하자.
    - 시도하고 실패하고 돌아가는 과정을 반복하며 개선해나가는 방향으로 진행하자.
- `Message`의 네이밍을 명확하게 개선
    - Menu의 프로퍼티를 활용하여 `menuList`의 프로퍼티 내부 하드코딩을 개선
- `NumberFormatter`를 활용하였던 부분을 Double을 확장하여 개선
    - `String(format:)`을 활용하여 description을 반환하도록 구성

[![top](https://img.shields.io/badge/top-%23000000.svg?&amp;style=for-the-badge&amp;logo=Acclaim&amp;logoColor=white&amp;)](#목차-1)

---

# STEP 3 : 다중 처리

은행원 여러명이 업무를 처리할 수 있도록 구현합니다.

## 3-1 고민했던 것

### 1. 다중 처리

- 여러 은행원이 동시에 고객의 일을 처리하는 로직을 고민했다.
1. `global() 큐`에 은행원의 수 만큼 `task`를 만들고 각각의 `task`에서 고객을 `dequeue`해서 처리하는 방식
2. 하나의 while 구문에서 dequeue 해주고 고객의 은행 업무에 따라 예금 고객은 `DispatchQueue.global()`에 세마포어를 이용해 `은행원 수 == 접근 가능한 스레드 수`  만드는 방식
    - `DispatchSemaphore`의 `value`를 1이 아닌 2나 3으로 줄 경우 `Race Condition`이 발생할 수 있다는 가능성이 존재하기 때문에 해당 방법은 적절하지 못하다는 생각이 들었다.
    - `Banker`가 가지고 있는 `work()` 메소드는 현재 공유자원에 접근하고 있지 않지만, 추후 해당 메소드가 공유자원에 접근하지 않을거라는 보장이 없다라는 생각이 들었다.
- 추후 은행원의 수가 변경되더라도 대응할 수 있도록 은행원 수 만큼 `DispatchQueue.global()`에 `task`를 만드는 `1번 방식`으로 구현했다.
    - `은행원 수` == `DispatchQueue의 수`

### 2. 대기번호 오름차순으로 업무를 할 수 있도록 처리

- 프로젝트 실행예시에는 대기번호 순으로 실행되고 있지 않은 점을 발견하게 되었다.
    - 이 부분을 야곰에게 질문해보았고, 순차적으로 업무를 할 수 있도록 구현해보라고 하셔서 고민하게 되었다.
- 고민한 결과, 대기번호 순으로 `차례대로 업무가 처리되야 된다고 생각`했기 때문에 대출업무와 예금업무를 보는 고객들을 `두개의 고객큐`로 나누어 관리하도록 구현했다.

### 3. CustomStringConvertible을 사용하지 않고 연산프로퍼티를 사용

- `Banking`의 경우 `rawValue`를 `String`으로 가지고 있는 형태인데, 이 부분을 은행원이 어떤 업무를 처리했는지 알려주기 위해 사용하려는 의도로 구현하게 되었다. `프로토콜 채택`과 `연산프로퍼티` 중에 어떤 방식으로 활용할 지 고민해보았다.
    - `CustomStringConvertible`의 경우 인스턴스를 원하는 형태의 문자열로 반환하고 싶을 때 채택하여 사용하는 프로토콜로 우리가 활용하고자 하였던 부분이랑은 적합하지 않다는 생각이 들었다.
    - 따라서 description이라는 연산프로퍼티를 활용하여 rawValue를 반환하도록 구현해주었다.

## 3-2 의문점

### DispatchSemaphore의 value가 1 이상이라면 Race Condition이 무조건 발생할까?

- 처음에 while문 안에 `DispatchQueue`를 호출하고 `DispatchSemaphore`의 `value`를 `3`으로 주어 3개의 스레드만 접근할 수 있도록 설정해주었다. 그런데 여러번 돌려도 Race Condition이 발생하지 않았다.
- 은행원의 수(`DispatchSemaphore value`)와 고객의 수를 늘려주고 테스트를 해보았더니 Race Condition이 발생하였다.
    - 테스트를 하면서 알게된 부분은 `Banker`의 `work()` 메소드가 공유자원에 접근하지 않기 때문에 발생하지 않았던 것이였다. 프로퍼티 `count`를 만들어주고 그 count에 접근하도록 구현해주었더니 Race Condition이 발생하는 상황을 확인할 수 있었다.
- 이 의문점을 풀면서 결국 `Semaphore의 value를 여러개 주는 것`은 공유자원에 접근할 수 있는 상황이라면 `Thread-safe하지 않은 것`이고, 따라서 은행원의 수를 `DispatchSemaphore`의 `value`에 맞춰 설계를 하는 것은 적절하지 못하다는 결론이 났다.

## 3-3 Truoble Shooting

### 구조체 프로퍼티는 클로저 내부에서 왜 변경할 수 없는가?

- `상황` 프로젝트에서  `Bank 타입`은 `구조체`이고, `numberOfCustomers 프로퍼티`를 가지고 있다. 은행원이 일을 할 때, 처리한 고객의 수를 `Dispatch.global().async` 클로저 내에서 카운트(`numberOfCustomers`) 해주도록 하려고 했으나 위와 같은 에러가 발생하였다.
- `이유`  `Escaping closure`의 경우 구조체에서는 캡쳐가 불가능하기 때문에, 프로퍼티를 변경하려고 하면 위와 같은 에러가 발생한다.

```swift
// 간단한 코드 예시
struct SomeStruct {
    var num = 0
    
    private mutating func test() {
        let closure = { // Escaping closure captures mutating 'self' parameter
            self.num += 1
        }
        closure()
    }
}
```

- `Escaping closure`의 경우 구조체에서는 캡쳐가 불가능하기 때문에, 프로퍼티를 변경하려고 하면 위와 같은 에러가 발생한다.
    
    > class와 같은 참조 타입이 아닌 Struct, enum과 같은 값타입에서는 mutating reference의 캡쳐를 허용하지 않기 때문에 self 사용이 불가능 하다.
    > 
- `해결` `Bank 타입`을 `클래스`로 바꿔주었다.

## 3-4 배운 개념

- 동기(Synchronous)와 비동기(Asynchronous)의 이해
- 동시성 프로그래밍 개념의 이해
- 동시성 프로그래밍을 위한 기반기술(GCD, Operation) 등의 이해
- 스레드(Thread) 개념에 대한 이해
- GCD를 활용한 동시성 프로그래밍 구현
- 동기(Synchronous)와 비동기(Asynchronous) 동작의 구현 및 적용

## 3-5 PR 후 개선사항

- Bank 타입 리팩토링
    - 은행원 수를 나타내는 프로퍼티명 number를 count로 수정
    - Bank타입을 class가 아닌 struct로 수정
    - `workBanker()` 메소드 내부에 while문을 while let으로 수정
- Banker 타입의 `workSpeed()` 메소드를 제거
- LinketList의 `removeFirst()` 메소드 내부를 Thread-Safe하게 개선
    - Thread Safe
        - 기존 로직은 은행원 수만큼 global() 큐에 Task를 만드는 방식이여서 하나의 고객 큐에 여러 스레드가 접근해 `Race Condition`이 발생할 가능성이 있음.
        - 여러 스레드가 하나의 고객 큐에 접근해도 LinkedList에서 첫 번째 요소를 반환하는 메서드를 하나의 스레드에서만 접근할 수 있도록 `SerialQueue.sync`를 활용하여 `Race Condition`을 해결

[![top](https://img.shields.io/badge/top-%23000000.svg?&amp;style=for-the-badge&amp;logo=Acclaim&amp;logoColor=white&amp;)](#목차-1)

---

# STEP 4 : UI **구현**

콘솔앱을 UI앱으로 구현합니다.

### UI 구성 설계

![](https://i.imgur.com/r9Gxh6p.png)


## 4-1 고민했던 것

- 화면 전체를 스택뷰로 구성할 수 있도록 고민해보았다.
- 동일한 구성을 가지고 있는 뷰를 Custom View로 분리할 수 있도록 고민해보았다.
    - 동일한 요소가 있는 뷰를 커스텀하여 재사용할 수 있도록 구현
- 뷰를 그릴 때 main Thread에서 바로 처리할 수 있는 방법을 고민해보았다.
    - delegate를 활용하여 ViewController가 뷰를 그리는 시점에 DispatchQueue.main.async를 활용하여 비동기적으로 뷰를 그릴 수 있도록 구현
- 각종 예외사항에 대해 대처할 수 있도록 어떤 로직을 추가해야하는지 고민해보았다.
    - 고객 10명 추가 버튼을 클릭시 은행이 일을 하고 있다면 고객만 추가할 수 있도록 로직 추가
    - 고객을 추가할 때 먼저 일이 끝나서 사라진 Thread(은행원)가 있는 경우 다시 일을 할 수 있도록 DispatchQueue를 추가하는 로직 추가 (프로퍼티 옵저버 활용)

## 4-2 의문점

- Bool 타입 네이밍을 할때 부정문을 사용해도 될까?
    - Bool 타입 변수를 만들 때 부정문을 사용할 경우 아래 guard 구문을 한국말로 표현하면 `타이머 인스턴스가 존재하지 않지 않는다면 함수를 종료 시켜라` 라고 표현할 수 있다.
        - 다시 말해  `타이머 인스턴스가 존재한다면 함수를 종료시켜라`로 다시 한번 생각해야한다.
    - 하지만 코드는 간결하게 작성할 수 있다.
        
        ```swift
        // 타이머의 인스턴스가 존재할 경우 guard 구문에서 함수를 종료함.
        func open() {
            let isNotWorking = timer == nil
            guard isNotWorking else {
                return
            }
            startTimer()
            
            // ... 
        }
        ```
        
    - Bool 타입 변수를 만들 때 부정문을 사용하지 않는 경우 뒤에 false 구문이 추가된다.
        
        ```swift
        // 타이머의 인스턴스가 존재할 경우 guard 구문에서 함수를 종료함.
        func open() {
            let isWorking = timer != nil
            guard isWorking == false else {
                return
            }
            startTimer()
            
            // ... 
        }
        ```
        
    - 프로퍼티로 한번에 깔끔하게 만드는 것이 좋은건지 아니면 false문이라도 추가하여 읽기 수월하도록 구성해주어야 하는게 맞는건지 의문이다.
    
- DispatchQueue는 작업하고 있는 것을 중간에 중지시킬 수 있을까?
    - cancel 메서드를 호출하더라도 작업이 작업 중이라면 중지시킬 수 없다.
    - 우회하는 방법이 존재하지만 GCD가 작업을 취소하거나 하는 것은 할 수는 없다.
    - Operation Queue를 사용하면 작업 중인 작업을 중지 할 수 있다. [[참고 링크]](https://developer.apple.com/documentation/foundation/operationqueue/1417849-cancelalloperations)
- Custom View 클래스 내부에 프로퍼티를 가져도 되는가?
    - 필요하다면 사용해도 무관하다.
    - 특정 뷰를 찾을 때 Int, String처럼 중복된 값이 들어올 경우 여러 뷰를 찾아올 수도 있다.
    - UUID를 활용하는 활용하면 고유한 값을 가질 수 있다.

## 4-3 Trouble Shooting

### 1. 뷰의 요소를 비동기적으로 업데이트 하기

- `상황` UIButton에 View의 요소를 업데이트 하도록 기능을 추가하고 테스트 해보았으나, 버튼이 클릭됨가 동시에 화면이 멈추고 에러가 나면서 뷰가 업데이트 되지 않는 상황을 마주했다.
* ![](https://i.imgur.com/0S10HBs.png)
- `이유` 비동기 프로그래밍시 뷰를 그리는 작업은 main thread에서 처리해주어야 하는데, 따로 처리해주지 않아서 해당 에러가 발생했던 것 같다.
- `해결` 뷰를 그리는 작업을 main thread에서 처리하도록 DispatchQueue.main.async 클로저 구문안에 넣어주었다.

### 2. 멈추지 않는 타이머

- `상황` 고객 추가버튼을 여러번 클릭하고 초기화를 누르게 되면 타이머가 멈추지 않았다.
* ![https://i.imgur.com/fvZWEq1.gif](https://i.imgur.com/fvZWEq1.gif)
- `이유` 타이머가 추가버튼 클릭시 계속 추가되면서 추가된 타이머가 끝나지 않고 계속 실행되는 듯 했다.
- `해결` 초기화 버튼을 클릭할 때, 대기중인 고객이 없을 때 타이머를 멈추면서 타이머에 nil을 대입해주었고, 타이머가 작동중에는 타이머를 추가하지 않도록 하는 로직을 추가해주니 해결되었다.
* ![https://i.imgur.com/VsvkGMM.gif](https://i.imgur.com/VsvkGMM.gif)

## 4-4 배운 개념

- 동시성 프로그래밍 중 UI 요소 업데이트의 주의점 이해
- 커스텀 뷰 구현
- 스택뷰 활용
- 코드로 UI 구성(오토레이아웃)
- 타이머 활용방법과 Run Loop

[![top](https://img.shields.io/badge/top-%23000000.svg?&amp;style=for-the-badge&amp;logo=Acclaim&amp;logoColor=white&amp;)](#목차-1)


</div>
</details>
