---
layout: post
title:  "디자인패턴 - Factory"
date:   2024-08-12 23:30:00 +0900
categories: architecture
---

Factory Pattern은 객체 생성을 캡슐화하여 클라이언트가 직접 객체를 생성하는 대신, 팩토리 메서드를 통해 객체를 생성하고 반환하는 디자인 패턴입니다. 이 패턴은 특정 상황에서 유용하게 사용될 수 있습니다. 다음은 Factory Pattern이 필요한 요구사항과 예시를 설명합니다.

### 필요한 요구사항
1. #### 객체 생성 로직의 캡슐화
객체 생성이 복잡하거나 다양한 조건에 따라 다른 객체를 생성해야 할 때, 이러한 생성 로직을 단일화하여 관리할 필요가 있습니다. Factory Pattern은 이러한 복잡성을 감추고 객체 생성을 단순화할 수 있습니다.

1. #### 클래스와 클라이언트 코드의 결합도 감소
클라이언트 코드가 구체적인 클래스에 의존하지 않고, 팩토리를 통해 객체를 생성할 수 있도록 함으로써, 코드의 유연성과 확장성을 높이고, 유지보수를 쉽게 만들 수 있습니다.

1. #### 객체 생성의 중앙 관리
애플리케이션에서 객체 생성을 중앙에서 관리하고, 이를 통해 생성된 객체의 라이프사이클을 관리할 필요가 있을 때 Factory Pattern이 유용합니다.

### 예시
#### 1. Shape Factory (도형 팩토리)
도형 팩토리는 다양한 종류의 도형 객체를 생성하는 팩토리입니다. 각 도형은 공통된 인터페이스를 구현하고 있지만, 구체적인 구현은 다를 수 있습니다.

```java
// Shape 인터페이스
public interface Shape {
    void draw();
}

// 구체적인 도형 클래스들
public class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Inside Circle::draw() method.");
    }
}

public class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Inside Rectangle::draw() method.");
    }
}

public class Square implements Shape {
    @Override
    public void draw() {
        System.out.println("Inside Square::draw() method.");
    }
}

// Shape 객체를 생성하는 팩토리 클래스
public class ShapeFactory {
    // 팩토리 메서드 - 도형 객체를 생성하고 반환
    public Shape getShape(String shapeType) {
        if (shapeType == null) {
            return null;
        }
        if (shapeType.equalsIgnoreCase("CIRCLE")) {
            return new Circle();
        } else if (shapeType.equalsIgnoreCase("RECTANGLE")) {
            return new Rectangle();
        } else if (shapeType.equalsIgnoreCase("SQUARE")) {
            return new Square();
        }
        return null;
    }
}

// 클라이언트 코드
public class Client {
    public static void main(String[] args) {
        ShapeFactory shapeFactory = new ShapeFactory();
        
        // 원 객체 생성 및 사용
        Shape circle = shapeFactory.getShape("CIRCLE");
        circle.draw();
        
        // 사각형 객체 생성 및 사용
        Shape rectangle = shapeFactory.getShape("RECTANGLE");
        rectangle.draw();
        
        // 정사각형 객체 생성 및 사용
        Shape square = shapeFactory.getShape("SQUARE");
        square.draw();
    }
}
```

위 예시에서 ShapeFactory 클래스는 팩토리 메서드 getShape()를 통해 클라이언트 코드가 원하는 종류의 도형 객체를 생성하고 반환합니다. 클라이언트는 팩토리를 통해 객체를 생성하므로, 구체적인 도형 클래스에 직접적으로 의존하지 않고도 도형 객체를 사용할 수 있습니다.

#### 2. Database Connection Factory (데이터베이스 연결 팩토리)
데이터베이스 연결 객체를 생성하는 팩토리 예시입니다. 다양한 데이터베이스 유형에 따라 다른 연결 객체를 생성할 수 있습니다.

```java
// 데이터베이스 연결 인터페이스
public interface DatabaseConnection {
    void connect();
    void disconnect();
}

// MySQL 데이터베이스 연결 구현
public class MySqlConnection implements DatabaseConnection {
    @Override
    public void connect() {
        System.out.println("Connecting to MySQL database...");
        // MySQL 연결 로직
    }
    
    @Override
    public void disconnect() {
        System.out.println("Disconnecting from MySQL database...");
        // MySQL 연결 해제 로직
    }
}

// Oracle 데이터베이스 연결 구현
public class OracleConnection implements DatabaseConnection {
    @Override
    public void connect() {
        System.out.println("Connecting to Oracle database...");
        // Oracle 연결 로직
    }
    
    @Override
    public void disconnect() {
        System.out.println("Disconnecting from Oracle database...");
        // Oracle 연결 해제 로직
    }
}

// 데이터베이스 연결 객체를 생성하는 팩토리 클래스
public class DatabaseConnectionFactory {
    public DatabaseConnection getConnection(String dbType) {
        if (dbType == null) {
            return null;
        }
        if (dbType.equalsIgnoreCase("MYSQL")) {
            return new MySqlConnection();
        } else if (dbType.equalsIgnoreCase("ORACLE")) {
            return new OracleConnection();
        }
        return null;
    }
}

// 클라이언트 코드
public class DatabaseClient {
    public static void main(String[] args) {
        DatabaseConnectionFactory factory = new DatabaseConnectionFactory();
        
        // MySQL 데이터베이스 연결 생성 및 사용
        DatabaseConnection mysqlConnection = factory.getConnection("MYSQL");
        mysqlConnection.connect();
        // 사용 후 연결 해제
        mysqlConnection.disconnect();
        
        // Oracle 데이터베이스 연결 생성 및 사용
        DatabaseConnection oracleConnection = factory.getConnection("ORACLE");
        oracleConnection.connect();
        // 사용 후 연결 해제
        oracleConnection.disconnect();
    }
}
```

위 예시에서 DatabaseConnectionFactory 클래스는 팩토리 메서드 getConnection()을 통해 클라이언트가 요청한 데이터베이스 유형에 따라 적절한 DatabaseConnection 객체를 생성하고 반환합니다. 클라이언트는 데이터베이스 종류에 대한 구체적인 구현에 의존하지 않고도 데이터베이스 연결을 관리할 수 있습니다.

Factory Pattern은 객체 생성을 중앙화하고 캡슐화하여 코드의 유연성과 확장성을 높이는 데 유용합니다. 이는 특히 객체 생성 로직이 복잡하거나 다양한 상황에 따라 다른 객체를 생성해야 할 때 유용하게 사용됩니다.