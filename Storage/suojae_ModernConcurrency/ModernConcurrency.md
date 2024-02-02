

<br/>

# Modern concurrency on apple platforms

<br/>

![image](https://github.com/iOS-BookClub/BookClub/assets/126137760/b1d30c7f-009c-40ab-8a00-0c0529bcb15f)


<br/>


#

### Chapter01. Introduction

```swift
// 뮤텍스 사용예시
class Library {
    private var lock = NSLock()
    var book: String = "Swift's Guide to Magic"

    func accessBook() {
        lock.lock() // 마법사가 책을 사용하기 전에 잠금
        print("Reading \(book)")
        // 책을 읽는 코드
        lock.unlock() // 마법사가 책 사용을 마치고 잠금 해제
    }
}

let library = Library()
DispatchQueue.global().async {
    library.accessBook()
}
DispatchQueue.global().async {
    library.accessBook()
}
```

```swift
// 세마포어 사용 예시
class MagicalLibrary {
    private let semaphore = DispatchSemaphore(value: 2) // 동시에 2명의 마법사가 접근 가능
    var books = ["Swift Spell Book", "Objective-C Charms"]

    func accessBook(index: Int) {
        semaphore.wait() // 마법사가 책에 접근하기 전에 대기
        let book = books[index]
        print("Reading \(book)")
        // 책을 읽는 코드
        semaphore.signal() // 마법사가 책 사용을 마치고, 다음 마법사에게 신호
    }
}

let magicalLibrary = MagicalLibrary()
DispatchQueue.global().async {
    magicalLibrary.accessBook(index: 0)
}

DispatchQueue.global().async {
    magicalLibrary.accessBook(index: 1)
}
```

```swift
// NSOperationQueue를 통해 간단한 멀티스레딩 처리
let operationQueue = OperationQueue()

let operation1 = BlockOperation {
    print("Operation 1: Conjuring spells")
}

let operation2 = BlockOperation {
    print("Operation 2: Mixing potions")
}

operationQueue.addOperation(operation1)
operationQueue.addOperation(operation2)
```

<br/>

>In the context of multithreading, a deadlock occurs when two different processes are waiting on the other to finish

<br/>

>The deadlock problem has many established solutions. Mutex and Semaphores being the most used ones. 

<br/>

> A mutex will signal other processes that a process is currently using some resource by adding a lock to it and preventing other processes from grabbing until that lock is freed. 

<br/>

> Mutex and Semaphores sound similar, but the key difference lies in what kind of resource they are locking.

<br/>

> In general, a mutex can be used to protect anything that doesn’t have any sort of execution on its own, such as files, sockets, and other filesystem elements. Semaphores can be used to protect execution in your program itself such us shared code execution paths. 

<br/>


>The use of semaphores will allow threads to synchronize, as they will be able to coordinate the resources amongst each other so they both have their fair usage of them.

<br/> 

>Deadlocks and race conditions are very similar. The main difference, in deadlocks, is we have multiple processes waiting for the other to finish, whereas in race conditions we have both processes writing data and corrupting it because no processes know that the resource in question is being used by someone else.

<br/>

>If you want to work with multithreading, you will end up creating multiple NSThread objects, and you are responsible for managing them all.

<br/>

>Each NSThread object has properties you can use to check the status of the thread, such as executing, cancelled, and finished.

<br/>

>With the GCD, you will never concern yourself with managing threads manually. It truly is a high-level framework in that sense"

<br/>

>Sitting at a higher level than the GCD , the NSOperation APIs are a high-level tool to do multithreading.

<br/>

>You will never have to manage locks or semaphores with it, and in turn the API is simple.


<br/>

# 

### Chapter02. Introducing async/await

```swift
import Foundation

var callback: (()->Void)?
var test: Task<String, Never>?

// 첫 번째 비동기 작업을 나타내는 함수
func fetchData1() async -> String {
    print("fetchData1 시작")
    // 비동기 작업을 시뮬레이션하기 위해 1초 대기
    try? await Task.sleep(nanoseconds: 1_000_000_000)
    print("fetchData1 완료")
    return "Data from fetchData1"
}

// 두 번째 비동기 작업을 나타내는 함수
func fetchData2() async -> String {
    print("fetchData2 시작")
    // 비동기 작업을 시뮬레이션하기 위해 1초 대기
    try? await Task.sleep(nanoseconds: 1_000_000_000)
    print("fetchData2 완료")
    return "Data from fetchData2"
}

func fetchData3() -> String {
    print("fetchData3 시작")
    print("fetchData3 완료")
    return "Data from fetchData3"
}

// 비동기 작업을 순차적으로 처리하는 함수
func performAsyncTasks() async {
    print("performAsyncTasks 시작")
    
    test = Task { () -> String in
        // 첫 번째 fetchData1의 완료를 기다림
        let data1 = await fetchData1()
        print(data1)
        
        // 두 번째 fetchData2의 완료를 기다림
        let data2 = await fetchData2()
        print(data2)
        
        return "비동기 작업 완료"
    }
    
    let data3 = fetchData3()
    print(data3)
}

// 비동기 함수 실행
Task {
    await performAsyncTasks()
    await print(test?.value ?? "")
}

RunLoop.main.run()

/*
performAsyncTasks 시작
fetchData3 시작
fetchData3 완료
Data from fetchData3
fetchData1 시작
fetchData1 완료
Data from fetchData1
fetchData2 시작
fetchData2 완료
Data from fetchData2
비동기 작업 완료
*/
```

<br/>

> Task has an initializer with a closure, and that closure itself is a concurrent context.

<br/>

>The same thread that called fetchUserInfo() will be free to do other work, and that other work is none of your concern since it’s handled by the system. It could be another task within your app, or it would be outside of your realm. Until control is returned to you, you cannot continue executing statements. 

<br/> 

> MainActor global actor in the real world for the first time. This will ensure that whenever a property is updated, it will happen on the main thread. 

<br/>

# 

### Chapter03. Continuations

>In short, a Continuation is whatever happens after an async call is finished. When we are using async/await , a continuation is simply everything below an await call. 

<br/> 

```swift
import Foundation

func printThreadInfo(context: String) {
    print("\(context): 현재 스레드 - \(Thread.current)")
}

// 현재 스레드 정보를 출력합니다. (메인 스레드)
printThreadInfo(context: "메인 컨텍스트")

// Task를 사용하여 새로운 작업을 시작합니다.
Task {
    printThreadInfo(context: "Task")
    // 'Task' 안에서는 메인 스레드 또는 메인 스레드와 유사한 환경에서 실행됩니다.
}

// Task.detached를 사용하여 새로운 작업을 시작합니다.
Task.detached {
    printThreadInfo(context: "Task.detached")
    // 'Task.detached' 안에서는 완전히 새로운 스레드에서 실행됩니다.
}

// 비동기적으로 실행되기 때문에, 메인 스레드가 마지막에 종료되지 않도록 대기합니다.
sleep(1)

/*
메인 컨텍스트: 현재 스레드 - <NSThread: 0x600003b08000>{number = 1, name = main}
Task: 현재 스레드 - <NSThread: 0x600003b08000>{number = 1, name = main}
Task.detached: 현재 스레드 - <NSThread: 0x600003b14400>{number = 3, name = (null)}
*/
```

>  The process of storing the images in a local cache can be a detached task, that way if the task is cancelled but the image is downloaded, the cache-saving operation will proceed without an issue.

<br/>




