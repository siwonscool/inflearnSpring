인프런 Spring 강의(김영한님)를 들으며  
=============
## 1. 개요
mealkit 프로젝트의 경우 Spring boot의 단편적인 사용법을 숙지하고 진행한 프로젝트여서 Spring의 이해도가 떨어진다고 생각하고, 
객체지향의 SOLID 5원칙을 공부하며 Spring 프레임워크에서는 어떤식으로 활용이 될것인가 대한 궁금증을 해결하기 위해 이 강의를 수강하였다.

웹개발자로써 내가짠 코드가 어떻게하면 더 클린한 코드가 될수있을지 고민하고 싶었다.


## 2. 요구사항을 기반으로 만든 예제

### 2.1 개발환경 : https://start.spring.io 에서 간단한 스프링 부트 예제 생성후 빌드 (Gradle 전체 설정은 build.gradle 파일 참조)

  * Project : Gradle Project 
  * Spring Boot : 2.3.x
  * Language : JAVA
  * Packaging : JAR
  * JAVA : 8


### 2.2 비즈니스 요구사항과 설계

#### 회원
  
    1. 회원가입하고 조회가 가능하다
    2. 회원은 일반등급(BASIC)과 VIP 2가지 등급이 있다.
    3. 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)

#### 주문과 할인 정책

    1. 회원은 상품을 주문할 수 있다.
    2. 회원등급에 따라 할인정책을 적용한다.
    3. 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액할인 (추후 변경될 수 있음)
    4. 할인 정책은 변경가능성이 높고, 오픈 직전까지 고민을 미루고 싶다. (ㅋㅋ)

### 2.3 회원 도메인 설계

![image](https://user-images.githubusercontent.com/67731034/149877613-c2b5fd38-c90c-438c-8b44-9aab286d9735.png)

![image](https://user-images.githubusercontent.com/67731034/149877686-75521f14-b80c-4a57-898b-05796358a9fb.png)

![image](https://user-images.githubusercontent.com/67731034/149877734-738961be-278e-469d-a715-5f13c4923158.png)

![image](https://user-images.githubusercontent.com/67731034/149877752-0d065b12-9d5a-4778-86b2-e7085307bdd0.png)

#### 회원 도메인 설계의 문제점

    1. 다른 저장소로 변경할때 (MemoryMemberRepository -> DbMemberRepository) OCP 원칙을 잘 준수 하였는가?
    2. DIP를 잘 지키고 있는가?
    
    답) 의존관계가 인터페이스 뿐만 아니라 구현까지 모두 의존하는 경향이 있음

### 2.4 주문과 할인 도메인 설계

![image](https://user-images.githubusercontent.com/67731034/149880772-3eba9690-1a87-4021-8012-f6fecf40279e.png)

![image](https://user-images.githubusercontent.com/67731034/149880818-9fa2d102-8919-4ff9-99a6-2d5abea32a8b.png)

![image](https://user-images.githubusercontent.com/67731034/149880848-3cf26cc7-959e-4cf1-b91d-69a2f6071aa5.png)

![image](https://user-images.githubusercontent.com/67731034/149880870-5729bc41-9a1b-4fe1-862e-c6e4e68f13e9.png)

![image](https://user-images.githubusercontent.com/67731034/149880951-980fbb03-f42a-41ce-8fdf-5ef41f6b0b5d.png)

![image](https://user-images.githubusercontent.com/67731034/149880982-86645664-55b8-4cd0-aa18-3512250a3a2f.png)


## 3. 객체지향원리를 적용하여 요구사항 변경에도 유연한 코드 작성

### 3.1 새로운 할인 정책
    
    서비스 오픈 직전에 할인 정책을 고정금액이 아닌 합리적으로 금액당 할인하는 정률% 할인으로 변경하고싶습니다.
    예를들어 VIP 고객의 경우 10000원을 주문하던 20000원을 주문하던 1000원이 할인되었다면,
    이제는 VIP 고객은 10% 할인율을 적용하여 10000원을 주문하면 1000원, 20000원을 주문하면 2000원을 할인하는 것으로요
    
![image](https://user-images.githubusercontent.com/67731034/149882364-0418f49e-db03-44c6-b351-865cc1e41a7a.png)

### 3.2 새로운 할인 정책과 문제점

#### 할인 정책을 교체하려면 클라이언트인 OrderServiceImpl 클래스를 수정하여야 한다.

```java
public class OrderServiceImpl implements OrderService {
// private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
 private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
```
#### 문제점 발견

    1. 역할과 구현을 충실하게 구분하였다 (O)
    2. 다형성도 활용하여 인터페이스와 구현체를 분리하였다 (O)
    3. OCP, DIP 설계원칙을 준수하였다. (X)
      
      OCP : 현재 코드는 기능을 확장하여 변경하면 현재 클라이언트의 코드에 영향을 주게된다.
      DIP : 클래스 의존관계를 살펴보면 인터페이스 뿐만 아니라 구현 클래스에도 의존하고 있다.
      
![image](https://user-images.githubusercontent.com/67731034/149884056-42de50cc-082d-4342-a3c4-4e8abd1af19f.png)

![image](https://user-images.githubusercontent.com/67731034/149884086-b2b47c3e-c39b-45db-ae20-5d0b6e957d13.png)

#### 해결방안

이러한 문제를 해결하려면 클라이언트의 OrderServiceImpl 에 DiscountPolicy의 구현객체를 _대신 생성하고 주입해야 한다._

### 3.3 관심사의 분리

#### AppConfig 두두등장

* 애플리케이션의 전체 동작 구조를 구성(config)하기 위해서, 구현객체를 생성하고, 연결을 책임지는 별도의 설정클래스를 생성한다.
* AppConfig 는 애플리케이션의 동작에 필요한 구현 객체를 생성한다.
  * MemberServiceImpl
  * MemoryMemberRepository
  * OrderServiceImpl
  * FixDiscountPolicy
* AppConfig 는 생성한 객체의 인스턴스의 참조를 생성자 주입을 통해 수행한다.
  * MemberServiceImpl -> MemoryServiceRepository
  * OrderServiceImpl -> MemoryServiceRepository, FixDiscountPolicy

![image](https://user-images.githubusercontent.com/67731034/149891696-2eb814ae-9124-4022-a7a9-dbae1bc9e08c.png)

### 3.4 새로운 구조와 할인 정책 적용

* AppConfig 의 등장으로 정액할인 정책을 정률할인 정책으로 바꿔보자
  * DiscountPolicy() 메소드의 반환값을 변경해주자

![image](https://user-images.githubusercontent.com/67731034/149895075-392e9f16-93b1-4b49-86b4-341aff5bbb2b.png)

![image](https://user-images.githubusercontent.com/67731034/149895114-dcbd2dfb-05bf-4bd7-a3c9-0d1451d5aef2.png)

![image](https://user-images.githubusercontent.com/67731034/149895167-d311ae2f-cce3-4579-8a16-94344ede5bd8.png)

### 3.5 좋은 객체 지향 설계의 5가지 원칙 적용 (SOLID 5원칙 개념 정리 : https://dev-cool.tistory.com/18)

#### SRP 단일 책임 원칙

> 한 클래스는 하나의 책임만 가져야 한다.

    1. 클라이언트 객체는 직접 구현 객체를 생성하고, 연결하고, 실행하는 다양한 책임을 가지고 있음
    2. SRP 원칙에 따르면 책임을 분리해야함
    3. 구현객체를 생성하고 연결하는 역할을 AppConfig 가 담당
    4. 클라이언트 객체는 실행하는 책임만 담당


#### DIP 의존관계 역전 원칙

> 프로그래머는 “추상화에 의존해야지, 구체화에 의존하면 안된다.” 의존성 주입은 이 원칙을 따르는 방법 중 하나다.

    1. 새로운 할인 정책을 개발하고, 적용하니 클라이언트 코드도 함께 변경해야 했다. `OrderServiceImpl` 에서 
       DIP를 지키며 `DiscountPolicy` 인터페이스에 의존하는것 같았지만 `FixDiscountPolicy` 에도 의존하는 것을 확인할 수 있다.

    2. `DiscountPolicy` 인터페이스만 `OrderServiceImpl` 에 의존하기 위해서 AppConfig 에서 `FixDiscountPolicy` 
       인스턴스를 클라이언트 대신 생성해서 클라이언트 코드에 의존성을 주입하였다.

#### OCP 

> 소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다.

    1. 객체지향의 다형성을 사용해서 DIP 를 지킴
    2. 애플리케이션을 사용영역과 구성영역으로 나눔
    3. AppConfig 가 의존 관계를 `FixDiscountPolicy`에서 `RateDiscountPolicy`로 변경하여서 클라이언트 코드에 
       주입하므로 클라이언트의 코드에는 변경사항이 없다.
       

### 3.6 IoC, DI 그리고 컨테이너

#### 제어의 역전 IoC(Inversion of Control)

    1. 기존 프로그램은 개발자 입장에서 클라이언트 구현 객체가 필요한 서버 구현 객체를 생성, 연결, 실행하였다.
    2. AppConfig 가 등장한 이후 구현 객체는 자신의 로직을 실행하는 역할만 담당한다. 이제 프로그램의 제어흐름을
       AppConfig가 가져가는 것이다.
    3. 이제는 프로그램의 제어 흐름의 권한은 모두 AppConfig에 있다 심지어 `OrderServiceImpl` 도 AppConfig가 생성한다.
    
#### 의존관계 주입 DI(Dependency Injection)
    
    1. 의존관계는 정적인 클래스 의존관계와, 동적인 객체(인스턴스) 의존과계 둘을 분리해서 생각해야 한다.
    
![image](https://user-images.githubusercontent.com/67731034/149902574-9c0d1baf-fc4b-4c82-98a1-afa9ead2d727.png)

![image](https://user-images.githubusercontent.com/67731034/149902607-f8490c1d-c1ed-4b89-8aff-22289b582d29.png)

![image](https://user-images.githubusercontent.com/67731034/149902657-7db9d494-b620-4910-b2fa-0e30160e6a7e.png)

#### IoC 컨테이너, DI 컨테이너

    1. AppConfig 처럼 객체를 생성하고 관리하면서 의존관계를 연결해주는 것을 IoC 컨테이너, DI 컨테이너 라고 한다.

### 3.7 스프링으로 전환하기
