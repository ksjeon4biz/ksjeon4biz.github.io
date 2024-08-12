---
layout: post
title:  "디자인패턴 - Singleton"
date:   2024-08-12 23:30:00 +0900
categories: architecture
---

Singleton Pattern은 인스턴스가 오직 하나만 생성되도록 보장하는 디자인 패턴입니다. 이 패턴은 특정 상황에서 유용하게 사용될 수 있습니다. 다음은 Singleton Pattern이 필요한 요구사항과 예시를 설명합니다.

### 필요한 요구사항
1. #### 유일한 자원 접근 관리
어플리케이션 내에서 공통된 자원에 대한 접근을 제한하고, 오직 하나의 인스턴스만을 통해 접근할 필요가 있을 때 Singleton Pattern이 유용합니다. 예를 들어, 설정 매니저, 로깅 시스템, 데이터베이스 연결, 캐시 매니저 등이 해당됩니다.

1. #### 공유 리소스의 중앙 집중화
여러 컴포넌트가 공유하는 중요한 상태나 데이터를 관리할 때 Singleton 패턴을 사용하여 중앙 집중화된 접근을 제공할 수 있습니다. 이는 데이터의 일관성과 동기화를 보장하는 데 도움이 됩니다.

1. #### 런타임 환경의 상태 관리
런타임에서 상태를 공유하고, 그 상태를 변경해야 할 필요가 있을 때 Singleton 패턴을 사용하여 모든 구성 요소가 동일한 상태를 참조하도록 할 수 있습니다.

### 예시
#### 1. 로깅 시스템
로깅은 어플리케이션에서 중요한 역할을 합니다. 여러 모듈에서 로깅을 사용할 수 있으며, 모든 로그 메시지는 단일 위치에서 관리되어야 합니다. Singleton 패턴을 사용하여 로깅 시스템의 인스턴스가 하나만 존재하도록 하고, 모든 모듈은 이 인스턴스를 통해 로그를 기록하도록 설계할 수 있습니다.

```java
public class Logger {
    private static Logger instance;
    
    private Logger() {
        // private constructor to prevent instantiation
    }
    
    public static Logger getInstance() {
        if (instance == null) {
            instance = new Logger();
        }
        return instance;
    }
    
    public void log(String message) {
        // log message implementation
        System.out.println("[LOG] " + message);
    }
}

// Usage example
public class MyClass {
    private Logger logger = Logger.getInstance();
    
    public void doSomething() {
        logger.log("Doing something...");
    }
}
```

위 예제에서 Logger 클래스는 Singleton 패턴을 구현하여 하나의 인스턴스만을 반환하도록 하고, 모든 로그 메시지는 단일 인스턴스를 통해 기록됩니다.

#### 2. 데이터베이스 연결
데이터베이스 연결은 어플리케이션에서 자주 사용되는 리소스입니다. 여러 곳에서 데이터베이스에 접근해야 할 때, Singleton 패턴을 사용하여 하나의 데이터베이스 연결 객체만을 유지하고 모든 모듈이 이를 공유하여 데이터베이스 연결을 효율적으로 관리할 수 있습니다.

```java
public class DatabaseConnection {
    private static DatabaseConnection instance;
    
    private DatabaseConnection() {
        // private constructor to prevent instantiation
    }
    
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();
            // initialize database connection
        }
        return instance;
    }
    
    public void executeQuery(String query) {
        // execute query implementation
        System.out.println("Executing query: " + query);
    }
}

// Usage example
public class DataAccessObject {
    private DatabaseConnection dbConnection = DatabaseConnection.getInstance();
    
    public void fetchData() {
        dbConnection.executeQuery("SELECT * FROM data");
    }
}
```

위 예제에서 DatabaseConnection 클래스는 Singleton 패턴을 사용하여 하나의 데이터베이스 연결 인스턴스만을 생성하고, 이를 DataAccessObject 클래스에서 공유하여 데이터베이스 쿼리를 실행합니다.

이와 같이 Singleton 패턴은 특정 자원에 대한 접근을 제어하고, 인스턴스의 유일성을 보장하여 어플리케이션의 일관성과 효율성을 높이는 데 유용합니다.
