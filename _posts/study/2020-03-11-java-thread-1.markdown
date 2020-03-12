---
layout: post
title: "Java에서 Thread 사용하기 - 1"
excerpt: "Java에서 Thread 사용하기 - 1(정의/사용법/Status/interrupt)"
---

Thread에 대해 알아보자. 학교다닐 때 배웠지만 사용하지 않아서 기억이 가물가물하다.  
그래서 포스팅하여 기록해보려고 한다. 

## Process

- 실행중인 프로그램
- OS로 부터 메모리를 할당 받음

## Thread

- 실제 프로그램이 수행되는 작업의 최소 단위
- 하나의 프로세스는 하나 이상의 Thread를 가지게 됨

## Thread를 사용하는 방법 2가지
1. extends Thread
```java
public class ThreadTest extends Thread {
    public void run(){
        //
    }
}
public static void main(String[] args) {
    ThreadTest test = new ThreadTest();
    test.start();
}
```
2. implements Runnable
```java
public class ThreadTest implements Runnable {
    public void run(){
        //
    }
}
public static void main(String[] args) {
    ThreadTest runner = new ThreadTest();
    Thread thread = new Thread(runner);
    runner.start();
}
```

## Multi Thread
- 동시에 여러 개의  Thread가 수행되는 프로그래밍
- Thread는 각각의 작업공간(context)를 가짐
- 공유 자원이 있는 경우 race condition 이 발생
- critical section에 대한 동기화(synchronization)의 구현이 필요


## Thread Status

![thread status](../../images/20200311/thread_status.jpg){:width="700"}

Thread는 `Start`되면 `Runnable`상태가 된다. 이때 CPU의 분배에 따라 `Run`상태가 된다.
Thread가 종료되면 `Dead`상태가 된다.

이때 `Runnable`상태에서 CPU의 점유권을 상실한 상태가 `Not Runnable` 이다.

`Runnable`에서 `Not Runnable`가 되는 방법은 3가지가 있다.
1. sleep()
2. wait()
3. join()

그렇다면 반대로 `Not Runnable`에서 `Runnable`로 변경하는 방법은
1. sleep() - 정해준 시간(밀리초)이 지나면
2. wait() - notify() 또는 notifyAll()
3. join() - 다른 스레드가 종료되면


## interrupt
- 다른 thread에 예외를 발생시키는 interrupt를 보냄
- thread가 sleep(), wait(), join() 메서드에 의해 블럭킹되었다면 interrupt에 의해 다시  runnable상태가 될 수 있음

```java
public class InterruptTest extends Thread {
    public void run(){
        try {
            while(true){
                System.out.println("호출");
                sleep(1000);    
            }
        } catch (InterruptedException e) {
            System.out.println(e);           
        }
    }
    public static void main(String[] args){
        InterruptTest test = new InterruptTest();
        test.start();
        try {
            while(true){
                test.sleep(5000);    
            }
        } catch (InterruptedException e) {}
        test.interrupt();
    }
}
```
sleep중일 때 interrupt() 메서드를 수행하면 InterruptedException가 발생하고 run()의 catch에서 잡아준다.  
그렇게되면 run()을 빠져나오게되고 스레드는 종료된다.
