# Chapter1 도메인 모델 시작하기

## 1.1 도메인이란

- 도메인은 소프트웨어로 해결하고자 하는 문제 영역이라고 할 수 있다.
    - 도메인의 예시는 온라인 서점 소프트웨어로 들 수 있다. 온라인 서점 소프트웨어는 온라인으로 책을 판매하는 데 필요한 상품 조회, 구매, 결제, 배송 추적 등의 기능을 제공해야 한다.
- 도메인은 여러 하위 도메인으로 구성된다.
- 소프트웨어가 도메인의 모든 기능을 제공하진 않고, 고정된 하위 도메인이 존재하는 것도 아니다.

## 1.2 도메인 전문가와 개발자 간 지식 공유

- 코딩에 앞서 요구사항을 올바르게 이해하는 것이 중요하다.
    - 요구사항을 잘못 이해하면, 변경하거나 다시 만들어야 할 코드가 만들어지고 소프트웨어를 만드는데 실패하거나 일정이 크게 밀리는 경우가 많다.
    - 요구사항을 올바르게 이해하려면? 개발자와 전문가가 직접 대화하는 것이다.
    - 도메인 전문가 만큼은 아니겠지만 이해관계자와 개발자도 도메인 지식을 갖춰야 한다. 제품 개발과 관련된 도메인 전문가, 관계자, 개발자가 같은 지식을 공유하고 직접 소통할수록 도메인 전문가가 원하는 제품을 만들 가능성이 높아진다.
- 잘못된 요구사항이 들어가면 잘못된 제품이 나온다.
    - 전문가나 관리자가 요구한 내용이 항상 올바르것은 아니며 때론 본인들이 실제로 원하는 것을 정확하게 표현하지 못할 때도 있다. 그래서 개발자는 요구사항을 이해할 때 왜 이런 기능을 요구하는지 또는 실제로 원하는 게 무엇인지 생각하고 전문가와 대화를 통해 진짜로 원하는 것을 찾아야 한다.

## 1.3 도메인 모델

- 도메인 모델은 특정 도메인 자체를 이해하기 위해 해당 도메인을 개념적으로 표현한 것이다.
- 도메인 모델링은 도메인이 제공하는 기능과 데이터 구성 파악 등 상황에 따라 여러 방식으로 표현 될 수 있다.
    - 객체를 이용한 도메인 모델링
        - 주문 도메인
            - 상품을 몇개 살지 선택하고 배송지 입력
            - 선택한 상품 각격을 이용해서 총 지불 금액을 계산하고, 금액 지불을 위한 결제 수단을 선택함
            - 주문한 뒤에도 배송 전이면 배송지 주소를 변경하거나 주문을 취소 할 수 있음
        - 도메인 모델 (Order)
            - 주문번호 (orderNumber)와 지불할 총금액 (totalAmounts)가 있음
            - 배송정보 (ShippingInfo)를 변경(changeShipping) 할 수 있음
            - 주문을 취소 (cancel) 할 수 있음
    - 상태 다이어그램을 이용해서 주문의 상태 전이를 모델링
- 하위 도메인과 모델
    - 각 하위 도메인이 다루는 영역은 서로 다르기 때문에 같은 용어라도 하위 도메인 마다 의미가 달라질 수 있다.
    - 도메인에 따라 용어 의미가 결정되므로 여러 하위 도메인을 하나의 다이어그램에 모델링하면 안 된다.
    - 모델의 각 구성요소는 특정 도메인으로 한정할 때 비로소 의미가 완전해지기 때문에 각 하위 도메인마다 별도로 모델을 만들어야 한다.
        - ex. 카탈로그 하위 도메인 모델과 배송 하위 도메인 모델은 따로 만들어야 한다. 
        카탈로그의 상품과 배송의 상품 의미는 다르기 때문에 함께 제공하면, 
        카탈로그의 상품을 제대로 이해하는 데 방해가 된다.

## 1.4 도메인 모델 패턴

- 도메인 모델은 아키텍처 상의 도메인 계층을 객체지향 기법으로 구현하는 패턴을 말한다.
- `도메인 모델에게 필요한 코드`는 `도메인 객체 내부에 구현하는 방식`을 말한다.
    - `order.update()`
        
        ```java
        public class Order {
        	private OrderState state;
        	private ShippingInfo shippingINfo;
        	
        	public void changeShippingInfo(ShippingInfo newShippingInfo) {
        		if (!state.isShippingChangeable()) {
        			throw new IllegalStateException("can't change shipping in " + state);
        		}
        			this.shippingInfo = newShippingInfo;
        	}
        		...
        }
        
        public enum OrderState {
        	PAYMENT_WAITING {
        		public booolean isShippingChangeable() {
        			return true;
        		}
        	},
        	PREPARING {
        		public boolean isShippingChangeable() {
        			return true;
        		}
        	},
        	SHIPED, DELIVERING, DELIVERY_COMPLETED;
        
        	public boolean isShippingChangeable() {
        		return false;
        	}
        }
        ```
        
        - 주문 상태를 표현하는 OrderState는 배송지를 변경할 수 있는 검사하는 메서드 제공함
        - 실제 배송지 정보를 변경하는 Order 클래스의 changeShippingInfo() 메서드는 orderState의 isShippingChangeable() 메서드를 이용해서 변경 가능한 경우에만 배송지를 변경함
        
        ```java
        public class Order {
        	private OrderState state;
        	private ShippingInfo shippingINfo;
        	
        	public void changeShippingInfo(ShippingInfo newShippingInfo) {
        		if (!state.isShippingChangeable()) {
        			throw new IllegalStateException("can't change shipping in " + state);
        		}
        			this.shippingInfo = newShippingInfo;
        	}
        	
        	private boolean isShippingChangeable() {
        		return state == OrderState.PAYMENT_WAITING || 
        			state == OrderState.PREPARING;
        	}
        		...
        }
        
        public enum OrderState {
        	PAYMENT_WAITING, PREPARING, SHIPPED, DELIVERING, DELIVERY_COPLETED;
        }
        ```
        
        - 큰 틀에서 보면 OrderState는 Order에 속한 데이터이므로 배송지 정보 변경 가능 여부를 판단하는 코드를 Order로 이동할 수 있음
        - 배송지 변경이 가능한지를 판단할 규칙이 주문 상태와 다른 정보를 함께 사용한다면 OrderState만으로는 배송지 변경 가능 여부를 판단할 수 없으므로 Order에서 로직을 구현해야 함
- 처음부터 완벽한 도메인 모델 설계는 매우 힘들기 때문에, 전반적인 개요를 알 수 있는 수준으로 작성하고 구현 과정에서 점진적으로 발전시켜 나가도록 한다.

## 1.5 도메인 모델 도출

- 도메인을 모델링 할 때 기본이 되는 작업은 모델을 구성하는 핵심 구성요소, 규칙, 기능을 찾는 것이다.
    - 기획서를 보고 필요한 기능, 이슈 포인트 등을 고려하여 설계한다.
- 문서화 자체도 중요하지만, 코드 자체도 도메인 관점에서 잘 표현해야 가독성도 높아지며 문서로서 코드가 의미를 갖는다.

## 1.6 엔티티와 밸류

- 모델은 크게 엔티티와 밸류로 구분한다.
- 엔티티
    - 엔티티의 가장 큰 특징은 식별자를 가진다는 것이다. (예시 - 주문번호)
        - 엔티티의 식별자는 바뀌지 않고 고유하기 때문에 두 엔티티 객체의 식별자가 같으면 두 엔티티는 같다고 판단 할 수 있다.
    - 엔티티의 식별자를 생성하는 시점은 도메인의 특징과 사용하는 기술에 따라 달라진다.
        - 특정 규칙에 따라 생성
        - UUID 또는 Nano ID와 같은 고유 식별자 생성기 사용
        - 값을 직접 입력(예시 - 회원의 아이디나 이메일)
        - 일련번호 사용(시퀀스나 DB의 자동 증가 컬럼 사용)
- 밸류
    - 밸류 타입은 별도의 식별자가 없고, 객체 자체로 의미를 명확히 표현 가능하게 한다.
        - 예시 - Address, Money 등
    - 밸류 타입의 장점은 밸류 타입을 위한 기능을 추가할 수 있다는 점이다.
        - 예시 - 전체 금액 구하기 : price * quantity → price.multiply(quantity)
    - 밸류 객체의 데이터를 변경할 때는 기존 데이터를 변경하기보다는 변경한 데이터를 갖는 새로운 밸류 객체를 생성하는 방식을 선호한다.
        
        ```java
        public class Money {
        	private int value;
        
        	public Money add(Money money) {
        		return new Money(this.value + money.value);
        	}
        
        //value를 변경할 수 있는 메서드 없음
        }
        ```
        
- 객체를 불변(immutable)하게 구현하는 것은 아래와 같은 장점을 가진다.
    - 변경에 불가하여 안전한 설계가 가능함(사이드 이펙트 차단)
    - 참조 투명성과 스레드에 안전한 특징을 가짐
- 명확한 이유가 있는게 아니라면 setter 사용을 지양하고, 필요한 필드를 모두 포함하여 생성자로 강제하도록 한다.
    - 도메인 객체가 불완전한 상태로 사용되는 것을 막을 수 있다.(특정 필드를 채우지 않고 객체 생성하는 경우)

## 1.7 도메인 용어와 유비쿼터스 언어

- 도메인에서 사용하는 용어를 코드에 반영하여, 코드만 읽어도 자연스럽게 이해하는 방향으로 가야한다.
    - 예시 - Enum 사용시 STEP1, STEP2와 같이 사용하지 않고 명확하게 READY, PAID 등으로 표현
