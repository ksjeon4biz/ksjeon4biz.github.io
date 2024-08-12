---
layout: post
title:  "Garbage Collection"
date:   2024-08-12 21:30:00 +0900
categories: architecture
---

Full Garbage Collection (Full GC)은 JVM(Java Virtual Machine)에서 메모리 관리를 위해 수행하는 가비지 컬렉션의 한 유형으로, 힙(Heap) 메모리 전체를 대상으로 수행됩니다. Full GC는 전체 힙 공간에서 사용되지 않는 객체들을 수집하고, 가능한 경우 메모리를 압축하여 연속된 공간을 확보합니다. 이는 애플리케이션의 성능에 큰 영향을 미칠 수 있기 때문에, Full GC의 이해와 관리가 중요합니다.

### Full GC의 특징
1. 전체 힙 스캔: Full GC는 힙 메모리의 모든 영역(Young Generation과 Old Generation)을 검사합니다. 경우에 따라 Permanent Generation(또는 Metaspace)도 포함됩니다.

1. 장시간 소요: Full GC는 전체 힙을 대상으로 하기 때문에, 다른 가비지 컬렉션 방식보다 더 많은 시간이 소요됩니다. 이로 인해 애플리케이션의 일시적인 멈춤(Stop-the-World, STW)이 발생합니다.

1. 메모리 압축: Full GC는 힙 메모리를 압축하여 단편화(Fragmentation)를 줄이고 연속된 빈 공간을 확보할 수 있습니다.

1. Stop-the-World 이벤트: Full GC가 실행되는 동안 모든 애플리케이션 스레드가 일시적으로 중단됩니다. 이로 인해 응답 시간이 증가하고 성능 저하가 발생할 수 있습니다.

### Full GC의 발생 원인
1. 메모리 부족: Young Generation과 Old Generation에서 더 이상 객체를 할당할 메모리가 부족할 때 Full GC가 발생할 수 있습니다.

1. Permanent Generation/Metaspace 부족: 클래스 메타데이터를 저장하는 Permanent Generation 또는 Metaspace가 부족할 때도 Full GC가 발생합니다.

1. System.gc() 호출: 코드에서 명시적으로 System.gc()를 호출하면 Full GC가 발생할 수 있습니다. 이는 가비지 컬렉터에게 전역 가비지 컬렉션을 수행하도록 요청하는 것입니다.

1. Promotion 실패: Young Generation에서 Old Generation으로 객체를 승격할 공간이 부족할 때 Full GC가 발생할 수 있습니다.
메타스페이스와 클래스 로딩 문제: 많은 클래스를 로드하거나 언로드하는 상황에서도 Full GC가 발생할 수 있습니다.

### Full GC의 영향
1. 성능 저하: Full GC는 모든 애플리케이션 스레드를 중단시키기 때문에 응답 시간에 영향을 미치고, 일시적인 성능 저하를 초래할 수 있습니다.

1. 장애: 빈번한 Full GC는 애플리케이션의 성능을 크게 저하시킬 수 있으며, 경우에 따라 OutOfMemoryError와 같은 메모리 관련 장애를 발생시킬 수 있습니다.

### Full GC 최적화 방법
1. 메모리 설정 조정: JVM의 힙 크기, Young Generation과 Old Generation의 비율, Metaspace 크기 등을 적절히 설정합니다.

1. GC 튜닝: 적절한 가비지 컬렉터를 선택하고, GC 튜닝 옵션을 설정하여 Full GC의 빈도와 지속 시간을 줄일 수 있습니다. 예를 들어, G1 Garbage Collector를 사용하면 Full GC의 영향을 줄일 수 있습니다.

1. 메모리 누수 방지: 애플리케이션에서 메모리 누수가 발생하지 않도록 코드 품질을 개선합니다. 이는 사용되지 않는 객체가 메모리에서 제거되지 않고 남아 있어 Full GC를 유발할 수 있습니다.

1. 객체 수명 관리: 객체의 수명을 최적화하여 Young Generation에서 빠르게 수집될 수 있도록 합니다.
1. System.gc() 호출 최소화: 코드에서 명시적으로 System.gc()를 호출하는 것을 피합니다.

### GC 로깅과 모니터링
JVM에서 GC 로깅 옵션을 활성화하여 가비지 컬렉션 활동을 모니터링할 수 있습니다. 예를 들어, -Xlog:gc* 옵션을 사용하여 GC 로그를 생성하고, 이를 분석하여 Full GC 발생 원인을 파악할 수 있습니다.
모니터링 도구(JVisualVM, JConsole, Grafana 등)를 사용하여 JVM 메모리 사용량과 가비지 컬렉션 활동을 실시간으로 관찰할 수 있습니다.
Full GC는 JVM에서 중요한 역할을 하지만, 잘못 관리되면 성능에 큰 영향을 미칠 수 있습니다. 따라서 Full GC의 동작 원리와 최적화 방법을 이해하고, 이를 효과적으로 관리하는 것이 중요합니다.


---
---



Garbage Collector(GC)가 메모리를 해제하지 못하는 경우는 다양한 상황에서 발생할 수 있습니다. 이러한 경우는 주로 객체가 여전히 참조되고 있어서, GC가 해당 객체를 "사용 중"이라고 판단하기 때문입니다. 대표적인 상황은 다음과 같습니다:

### 1. 강한 참조(Strong Reference)
객체가 여전히 다른 객체에 의해 강하게 참조되고 있는 경우, GC는 해당 객체를 해제하지 않습니다. 이는 가장 일반적인 경우입니다.

```java
Object a = new Object();
Object b = a;  // b가 a를 강하게 참조
a = null;      // a는 null이 되었지만, b가 여전히 참조 중
// 이 경우 GC는 객체를 해제하지 않음
```

### 2. 메모리 누수(Memory Leak)
의도하지 않은 객체 참조로 인해 객체가 해제되지 않는 경우입니다. 이 경우 객체는 더 이상 필요하지 않지만, 참조가 남아 있어 GC가 이를 해제하지 않습니다.

```java
public class MemoryLeakExample {
    private List<Object> objectList = new ArrayList<>();

    public void addObject() {
        Object obj = new Object();
        objectList.add(obj);  // objectList가 obj를 계속 참조
    }

    public void removeObject() {
        if (!objectList.isEmpty()) {
            objectList.remove(0);  // 일부 객체만 제거
        }
    }
}
```
위의 예에서는 objectList가 객체를 계속 참조하고 있어 메모리 누수가 발생할 수 있습니다.

### 3. 순환 참조(Circular Reference)
객체들이 서로를 참조하는 경우입니다. 현대의 대부분의 GC 알고리즘은 이를 잘 처리하지만, 일부 경우에는 문제가 발생할 수 있습니다.

```java
class Node {
    Node next;
    Node prev;
}

Node n1 = new Node();
Node n2 = new Node();
n1.next = n2;
n2.prev = n1;
// n1과 n2는 서로를 참조
```

### 4. 정적 참조(Static Reference)
정적 변수는 애플리케이션이 종료될 때까지 유지됩니다. 따라서 정적 변수가 객체를 참조하고 있으면, 해당 객체는 GC에 의해 해제되지 않습니다.

```java
public class StaticReference {
    private static Object staticObject = new Object();
}
```

### 5. 전역 객체(Global Object)
전역 변수 또는 Singleton 패턴으로 유지되는 객체는 애플리케이션의 전체 수명 동안 유지될 수 있습니다.

```java
public class GlobalReference {
    private static final GlobalReference instance = new GlobalReference();
    private Object object = new Object();
    
    public static GlobalReference getInstance() {
        return instance;
    }
}
```

### 6. JNI와의 상호작용
Java Native Interface(JNI)를 통해 네이티브 코드를 사용할 때, 네이티브 코드가 Java 객체를 참조하면 GC가 이를 인식하지 못할 수 있습니다.

```c
JNIEXPORT void JNICALL Java_MyClass_nativeMethod(JNIEnv *env, jobject obj) {
    // 네이티브 코드에서 obj를 참조
}
```

### 7. 리소스 누수(Resource Leak)
파일, 소켓, 데이터베이스 연결 등과 같은 시스템 리소스를 적절히 해제하지 않으면, 메모리 누수가 발생할 수 있습니다.

```java
try (FileInputStream fis = new FileInputStream("file.txt")) {
    // 파일을 읽음
} catch (IOException e) {
    e.printStackTrace();
}
// try-with-resources 구문을 사용하지 않으면 리소스 누수가 발생할 수 있음
```

### 8. 클로저와 람다(Closure and Lambda)
클로저나 람다 표현식이 외부 변수를 참조할 경우, 이 변수는 클로저가 존재하는 한 해제되지 않습니다.

```java
Runnable r = () -> {
    // 외부 변수 참조
};
```

이러한 경우들을 피하기 위해, 적절한 코드 작성과 메모리 관리를 통해 객체 참조를 명확히 하고, 필요하지 않은 객체는 참조를 해제해야 합니다. 또한, 메모리 누수를 방지하기 위해 자주 사용하는 패턴이나 도구들을 사용하는 것이 좋습니다.


---
---


메모리 누수를 방지하고 관리하기 위해 사용할 수 있는 패턴과 도구는 여러 가지가 있습니다. 여기서는 몇 가지 주요 패턴과 도구를 소개하겠습니다.

### 메모리 누수 방지에 적합한 패턴
1. #### try-with-resources (Java)
자동으로 자원을 해제하는 구문입니다. 주로 파일, 네트워크 연결, 데이터베이스 연결 등을 사용할 때 유용합니다.

    ```java
    try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
        // 파일을 읽음
    } catch (IOException e) {
        e.printStackTrace();
    }
    ```

1. #### 싱글톤 패턴 (Singleton Pattern)
전역 객체를 하나만 유지하여 메모리 사용을 최소화합니다. 그러나 제대로 관리하지 않으면 메모리 누수를 일으킬 수 있으므로 주의가 필요합니다.

    ```java
    public class Singleton {
        private static Singleton instance;
        
        private Singleton() {}

        public static Singleton getInstance() {
            if (instance == null) {
                instance = new Singleton();
            }
            return instance;
        }
    }
    ```

1. #### 약한 참조 (Weak Reference)
강한 참조 대신 약한 참조를 사용하여 GC가 객체를 수거할 수 있도록 합니다. 이는 캐시를 구현할 때 유용합니다.

    ```java
    WeakReference<MyObject> weakRef = new WeakReference<>(new MyObject());
    MyObject obj = weakRef.get(); // 필요 시에만 객체에 접근
    ```

1. #### 이벤트 리스너 해제
이벤트 리스너를 사용한 후에는 반드시 해제하여 메모리 누수를 방지합니다.

    ```java
    button.addActionListener(new ActionListener() {
        @Override
        public void actionPerformed(ActionEvent e) {
            // 이벤트 처리 코드
        }
    });

    // 리스너 해제
    button.removeActionListener(listener);
    ```

1. #### Flyweight 패턴
공유 가능한 객체를 재사용하여 메모리 사용을 줄입니다. 주로 많은 작은 객체를 사용하는 경우에 유용합니다.

    ```java
    public class Flyweight {
        private static Map<String, Flyweight> pool = new HashMap<>();

        private Flyweight() {}

        public static Flyweight getInstance(String key) {
            if (!pool.containsKey(key)) {
                pool.put(key, new Flyweight());
            }
            return pool.get(key);
        }
    }
    ```

### 메모리 누수 방지에 유용한 도구
1. #### VisualVM (Java)
Java 애플리케이션의 메모리 사용을 모니터링하고, 메모리 누수를 감지하는 데 사용됩니다.

    ```shell
    jvisualvm
    ```

1. #### Eclipse Memory Analyzer (MAT)
힙 덤프를 분석하여 메모리 누수를 식별하고, 문제의 근원을 찾아줍니다.

    ```shell
    // 힙 덤프 생성
    jmap -dump:file=heapdump.hprof <pid>
    // MAT에서 힙 덤프 분석
    ```

1. #### JProfiler
Java 애플리케이션의 성능, 메모리 사용, 스레드 동작 등을 분석할 수 있는 상용 도구입니다. 메모리 누수 탐지를 위한 강력한 기능을 제공합니다.

1. #### YourKit Java Profiler
메모리 분석 및 성능 튜닝을 위한 또 다른 상용 도구입니다. 사용하기 쉽고 강력한 기능을 갖추고 있습니다.

1. #### Valgrind (C/C++)
메모리 누수, 잘못된 메모리 사용 등을 검출할 수 있는 도구입니다. 주로 C/C++ 애플리케이션에 사용됩니다.

    ```shell
    valgrind --leak-check=full ./your_program
    ```

1. #### LeakCanary (Android)
Android 애플리케이션에서 메모리 누수를 쉽게 감지하고 디버깅할 수 있는 오픈 소스 라이브러리입니다.

    ```java
    // Application 클래스에 추가
    class MyApplication : Application() {
        override fun onCreate() {
            super.onCreate()
            LeakCanary.install(this)
        }
    }
    ```

### 결론
메모리 누수를 방지하기 위해서는 적절한 코드 작성 패턴을 따르고, 메모리 관리를 위한 도구를 사용하는 것이 중요합니다. 위에서 소개한 패턴과 도구를 활용하면 메모리 누수를 줄이고, 애플리케이션의 성능을 최적화할 수 있습니다. 지속적인 모니터링과 분석을 통해 잠재적인 메모리 누수를 조기에 발견하고 수정하는 것이 필요합니다.