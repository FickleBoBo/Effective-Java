# finalizer와 cleaner 사용을 피하라

## finalizer와 cleaner
Java에서는 객체 소멸자가 두 가지 존재한다. 
1. finalizer
Java 9부터 deprecated
2. cleaner
Java 9 부터 finalizer의 대안으로 도입

cleaner가 finalizer보다 덜 위험하지만 예측할 수 없고, 느리고, 일반적으로 불필요하다. 그렇다면 불필요한 이유를 알아보자.

### 1. 실행 시점과 여부를 보장할 수 없다.

```Java
public class FileResource {
    private FileInputStream fis;
    
    public FileResource(String path) throws IOException {
        fis = new FileInputStream(path);
    }
    
    @Override
    protected void finalize() throws Throwable {
        fis.close(); 
    }
}
```
위와 같은 예시에서 GC가 언제 실행될지 모르기 떄문에 finalizer도 언제 호출될지 알 수 없다. 따라서 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner을 사용하면 안된다. 데이터베이스 같은 공유 자원에서 락 해제를 객체 소멸자에 맡겨 놓으면 분산 시스템이 서서히 멈추기 때문이다. 

### 2. finalizer는 GC의 효율을 떨어트릴 수 있다

- 성능 저하: finalizer가 있는 객체는 GC가 처리하는 과정이 훨씬 복잡해진다.
일반 객체는 GC가 한 번에 회수하지만 finalizer 객체는 여러 GC 사이클에 걸쳐서 회수된다.
  
- GC는 finalizer 객체를 다음과 같이 처리한다. 

> GC가 finalizer 객체를 발견 -> finalization queue에 등록 -> finalizer 스레드가 finalize() 실행 대기 -> 다음 GC 사이클에서야 메모리 회수
  
따라서 finalizer 큐에 객체가 쌓이면서 OutOfMemoryError 발생이 가능하다.

### 3. finalizer 공격에 노출될 수 있다.
생성자나 직렬화 과정에서 예외를 발생시켜 객체 생성에 실패하더라도 생성 실패한 객체의 finalizer는 실행되어 완전히 생성되지 않은 객체에 접근이 가능해져 보안의 위험이 발생한다.

## finalizer와 cleaner의 대안
1. AutoCloseable의 구현
2. 인스턴스를 사용하고 난 다음에 `close()` 메서드 사용
3. try-with-resources 

```Java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();
    
    private static class State implements Runnable {
        int numJunkPiles; 
        
        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }
        
        @Override
        public void run() {
            System.out.println("방 청소");
            numJunkPiles = 0;
        }
    }
    
    private final State state;
    private final Cleaner.Cleanable cleanable;
    
    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }
    
    @Override
    public void close() {
        cleanable.clean();
    }
}
```

이 코드는 Room의 cleaner를 단지 안전망으로 사용한 코드이다. 모든 Room 생성을 try-with- resourrces 블록으로 감싼다면 자동 청소는 전혀 필요하지 않다. 

```Java
try (Room room = new Room(7)) {
    System.out.println("방 사용");
}
```
잘 짜인 코드의 예시이다. try 블록이 끝나면 자동으로 room.close()를 호출하고 -> cleanable.clean() -> state.run()을 호출하게 되어 정상적으로 방청소를 출력하게 된다. 

```Java
new Room(99);
```
반면 다음과 같이 사용한다면 쓰레기가 99개인 state가 생성되고 cleaner에 등록된다. -> Room의 객체 참조가 없어서 GC의 대상이 되지만 언제 GC가 실행될지는 보장이 되지 않기떄문에 사용하는 것을 지양해야한다. 

## 결론
- finalizer는 Deprecated이며, cleaner도 느리고 비결정적이라 일반 코드에서 불필요하다.
- 자원 해제의 유일한 확실한 방법은 close() 이며 try-with-resources를 통해 결정적 해제를 보장하라.
- Cleaner는 프로세스 종료 시 보장되지 않고 성능 비용이 있으므로 예외적으로만 안전망으로 사용한다.