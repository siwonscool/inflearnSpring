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
    이제는 VIP 고객이면 10% 할인율을 적용하여 10000원을 주문하면 1000원, 20000원을 주문하면 2000원을 할인하는 것으로요
    
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
* AppConfig 
