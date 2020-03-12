---
layout: post
title: "Java에서 Thread 사용하기 - 2"
excerpt: "Java에서 Thread 사용하기 - 2(Thread 정지/동기화/협업)"
---

## Thread 종료
Thread는 자신의 메소드가 모두 실행되면 종료되면 종료된다.   
하지만 그 전에 종료시키는 법도 있다.
 
기존에는 Thread.stop() 가 있었지만 현재는 사용하지 않는다.

- interrupt로 종료

- flag를 두어 종료

flag를 두어 종료시키는 방법은 run()안에 루프를 두고, 특정 조건이되면 flag값을 바꿔주어 종료시키는 것이다.

```java
public class TerminateThread extends Thread{

    private boolean flag = false;

    public TerminateThread(String name){
        super(name);
    }

    public void run(){
        while (!flag){
            try {
                sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println(getName() + " end");
    }

    public void setFlag(boolean flag){
        this.flag = flag;
    }

    public static void main(String[] args) throws IOException {
        TerminateThread threadA = new TerminateThread("A");       
        threadA.start();       
        int in;
        while (true){
            in = System.in.read();
            if ( in == 'A'){
                threadA.setFlag(true);
                break;
            }
        }
        System.out.println("main end");
    }
}
``` 

전역변수로 `flag`를 두어 A가 들어오면 `flag`를 true로 설정 하도록 하였다.
그렇게 되면 run()에서 while이 수행중이던 스레드가 종료되고, 그런 뒤 main 스레드가 종료된다.


## 자바에서 동기화 구현

- synchronized 수행문
- synchronized(참조형 형식) { // } : 참조형 수식에 해당하는 객체에  lock을 걺

자바에서 동기화를 구현하는 법은 두가지이다.   
synchronized 키워드를 메서드에 붙여주거나 synchronized() 메서드로 블럭을 감싸주는 것이다. 

```java
class MoneyController {
    private int money = 5000;
    public void plusMoney(int money){
        int nowMoney = getMoney();
        try {
            sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        setMoney(nowMoney + money);
    }
    public void minusMoney(int money){
        int nowMoney = getMoney();
        try {
            sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        setMoney(nowMoney - money);
    }
    public int getMoney(){
        return money;
    }
    public void setMoney(int money){
        this.money = money;
    }
}

class ThreadA extends Thread{
    public void run() {
        SynchronizedTest.moneyController.plusMoney(2000);
        System.out.println(SynchronizedTest.moneyController.getMoney());
    }
}

class ThreadB extends Thread{
    public void run() {
        SynchronizedTest.moneyController.minusMoney(3000);
        System.out.println(SynchronizedTest.moneyController.getMoney());
    }
}

public class SynchronizedTest{

    public static MoneyController moneyController = new MoneyController();
    public static void main(String[] args) {
        ThreadA threadA = new ThreadA();
        ThreadB threadB = new ThreadB();
        threadA.start();
        threadB.start();
    }
}
```

돈을 관리해주는 클래스 MoneyController가 있다. 처음 5000원을 갖고 파라미터로 받은 돈을 plus/minus해준다.    
돈을 plus해줄 때는 3초 minus해줄 때는 1초가 걸린다. 

ThreadA에서는 2000원을 plus해주고 ThreadB에서는 3000원을 minus해준다.  
정상적인 처리라면 5000+2000-3000이므로 4000원이 남아야한다.

하지만 동기화 처리가 되어있지 않기 때문에 7000원이 남게된다.

이때 문제를 해결하기 위해서는 plusMoney()/minusMoney() 앞에 `synchronized` 를 붙여준다.  
또는 plusMoney()/minusMoney() 내에서 synchronized(this){} 으로 감싸도 된다.
```java
    public void plusMoney(int money){
        synchronized(this){
            int nowMoney = getMoney();
            try {
                sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            setMoney(nowMoney + money);
        }   
    }
```
Thread 내에서도 아래와 같이 사용 가능하다.
```java
class ThreadA extends Thread{
    public void run() {
        synchronized(SynchronizedTest.moneyController){ //공유하는 객체
            SynchronizedTest.moneyController.plusMoney(2000);
            System.out.println(SynchronizedTest.moneyController.getMoney());
        }
    }
}
```


## Thread간의 협업
- wait() : 리소스가 더 이상 유효하지 않은 경우 리소스가 사용 가능할 때 까지 thread를 non-runnable 상태로 전환
- notify() : wait()하고 있는 thread 중 **하나**의 thread를 runnable 한 상태로 깨움
- notifyAll() : wait()하고 있는 **모든** thread가 runnable한 상태가 되도록 함

notify()보다 notifyAll()을 사용하기를 권장한다.
특정 thread가 통지를 받도록 제어 할 수 없으므로 모두 깨운 후 scheduler에 CPU를 점유하는 것이 좀 더 공평하다고 한다.
