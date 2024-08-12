---
layout: post
title:  "제어 반전(Inversion of Control, IoC)"
date:   2024-08-12 22:50:00 +0900
categories: architecture
---

제어 반전(Inversion of Control, IoC)은 객체 지향 프로그래밍에서 객체의 생성 및 의존성 관리 책임을 애플리케이션 코드가 아닌 프레임워크 또는 외부 컨테이너에 맡기는 디자인 원칙입니다. IoC의 주요 목적은 객체들 간의 결합도를 낮추어 시스템의 유연성과 재사용성을 높이는 것입니다.

### 제어 반전의 주요 개념
1. #### 제어의 흐름 반전:
* 전통적인 방식에서는 애플리케이션 코드가 라이브러리를 호출하여 기능을 수행합니다. 반면에 IoC에서는 프레임워크가 애플리케이션 코드를 호출합니다.
* 예: 전통적인 방식에서는 애플리케이션 코드가 특정 객체를 생성하고 관리하지만, IoC에서는 프레임워크가 이러한 객체의 생명 주기를 관리합니다.

1. #### 의존성 주입(Dependency Injection, DI):
* IoC의 가장 일반적인 구현 방식 중 하나입니다.
* 객체가 필요로 하는 의존성을 외부에서 주입합니다. 이를 통해 객체는 의존성을 직접 생성하거나 찾지 않고, 주입받은 의존성을 사용합니다.

### 의존성 주입의 유형
1. #### 생성자 주입(Constructor Injection):
의존성을 생성자를 통해 주입합니다.

    ```java
    public class Service {
        private final Repository repository;
        
        public Service(Repository repository) {
            this.repository = repository;
        }
    }
    ```

1. #### 세터 주입(Setter Injection):
의존성을 세터 메서드를 통해 주입합니다.

    ```java
    public class Service {
        private Repository repository;
        
        public void setRepository(Repository repository) {
            this.repository = repository;
        }
    }
    ```

1. #### 인터페이스 주입(Interface Injection):
의존성을 주입받기 위한 메서드를 정의한 인터페이스를 사용하여 주입합니다.

    ```java
    public interface Injectable {
        void inject(Repository repository);
    }

    public class Service implements Injectable {
        private Repository repository;
        
        @Override
        public void inject(Repository repository) {
            this.repository = repository;
        }
    }
    ```

### 제어 반전의 장점
1. #### 결합도 감소:
* 객체들 간의 결합도를 낮추어 더 유연하고 변경하기 쉬운 코드를 작성할 수 있습니다.
* 의존성 변경이 필요할 때 최소한의 수정으로 가능해집니다.

1. #### 재사용성 향상:
* 의존성이 주입되므로 동일한 코드를 다양한 상황에서 재사용할 수 있습니다.

1. #### 테스트 용이성:
* 의존성을 외부에서 주입받으므로, 테스트 시에 목 객체(Mock Object)나 스텁(Stub)을 쉽게 주입하여 테스트할 수 있습니다.

1. #### 유지보수성 개선:
* 객체 생성과 의존성 관리를 중앙에서 통제하므로 코드의 유지보수가 더 쉬워집니다.
* 코드의 변경 사항이 한 곳에 집중되므로, 버그가 발생할 가능성이 줄어듭니다.

### 제어 반전의 구현 예시
스프링 프레임워크는 IoC를 구현한 대표적인 프레임워크입니다. 스프링 컨테이너는 객체의 생성과 의존성 주입을 관리합니다.

#### 스프링에서의 예제
1. #### 설정 클래스:

    ```java
    @Configuration
    public class AppConfig {
        @Bean
        public Repository repository() {
            return new RepositoryImpl();
        }
        
        @Bean
        public Service service() {
            return new Service(repository());
        }
    }
    ```

1. #### 서비스 클래스:

    ```java
    public class Service {
        private final Repository repository;
        
        @Autowired
        public Service(Repository repository) {
            this.repository = repository;
        }
    }
    ```

1. #### 애플리케이션 실행:

    ```java
    public class Main {
        public static void main(String[] args) {
            ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
            Service service = context.getBean(Service.class);
            // service 사용
        }
    }
    ```

### 결론
제어 반전은 객체 지향 설계에서 중요한 원칙으로, 객체의 생성과 의존성 관리를 프레임워크나 외부 컨테이너에 맡김으로써 결합도를 낮추고 유연성과 재사용성을 높이는 데 큰 도움이 됩니다. 이를 통해 코드의 유지보수성과 테스트 용이성을 크게 개선할 수 있습니다.