

<br/>

# Modern concurrency on apple platforms

<br/>

![image](https://github.com/iOS-BookClub/BookClub/assets/126137760/b1d30c7f-009c-40ab-8a00-0c0529bcb15f)


<br/>


#

### Chapter01. Introduction

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

> 

<br/> 

# 

### Chapter01. 예제: 홈 미디어 플레이어

<br/> 

# 

### Chapter05. 모델링 관련 조언

<br/> 

# 

### Chapter06. 엔지니어가 사용하는 모델

<br/> 

# 

### Chapter07. 소프트웨어 아키텍처의 개념 모델

<br/> 

# 

### Chapter08. 도메인 모델

<br/> 

# 

### Chapter09. 디자인 모델




