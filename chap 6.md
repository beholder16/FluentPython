# Chapter6 일급 함수 디자인 패턴
패턴에 참여하는 일부 클래스의 객체를 간단한 함수로 교체하면, 획일적으로 반복되는 코드의 상당 부분을 줄일 수 있다. 
함수 객체를 이용해서 전략 패턴을 리팩토링하고, 비슷한 방법으로 명령 패턴을 단순화할 수 있다. 
## 6.1 사례 : 전략 패턴의 리팩토링
전략패턴 : 파이썬에서 함수를 일급 객체로 사용하면 더욱 간단해질 수 있는 디자인 패턴의 대표적인 사례
### 6.1.1 고전적인 전략
일련의 알고리즘을 정의하고 각각을 하나의 클래스 안에 넣어서 교체하기 쉽게 만든다. 전략을 이용하면 사용하는 클라이언트에 따라 알고리즘을 독립적으로 변경할 수 있다. 

1. 충성도 포인트가 1000점 이상인 고객은 전체 주문에 대해 5% 할인을 적용한다.
2. 하나의 주문에서 20개 이상의 동일 상품을 구입하면 해당 상품에 대해 10% 할인을 적용한다.
3. 서로 다른 상품을 10종류 이상 주문하면 전체 주문에 대해 7% 할인을 적용한다.

```py
from abc import ABC, abstractmethod
from collections import namedtuple

Customer = namedtuple('Customer', 'name fidelity')

class LineItem:
    
    def __init__(self, product, quantity, price):
        self.product = product
        self.quantity = quantity
        self.price = price
        
    def total(self):
        return self.price * self.quantity
    
class Order: # 콘텍스트
    
    def __init__(self, customer, cart, promotion = None):
        self.customer = customer
        self.cart = list(cart)
        self.promotion = promotion
        
    def total(self):
        if not hasattr(self, '__total'): #뭐지?
            self.__total = sum(item.total() for item in self.cart)
        return self.__total
    
    def due(self):
        if self.promotion is None:
            discount = 0
        else:
            discount = self.promotion.discount(self)
        return self.total() - discount
    
    def __repr__(self):
        fmt = '<Order total: {:.2f} due: {:2f}>'
        return fmt.format(self.total(), self.due())
    
    
class Promotion(ABC): # 전략 : ABC
    
    @abstractmethod
    def discount(self, order):
        """할인액을 구체적인 숫자로 반환한다."""
        
        
class FidelityPromo(Promotion): # 첫 번째 구체적인 전략
    """충성도 포인트가 1000점 이상인 고객에게 전체 5% 할인 적용"""
    
    def discount(self, order):
        return order.total() * .05 if order.customer.fidelity >= 1000 else 0
    
class BulkItemPromo(Promotion): # 두 번째 구체적인 전략
    """20개 이상의 동일 상품을 구입하면 10% 할인 적용"""
    
    def discount(self, order):
        discount = 0
        for item in order.cart:
            if item.quantity >= 20:
                discount += item.total() * .1
        return discount
    
class LargeOrderPromo(Promotion):
    """10종류 이상의 상품을 구입하면 전체 7% 할인 적용"""
    
    def discount(self, order):
        distinct_items = {item.product for item in order.cart}
        if len(distinct_items) >= 10:
            return order.total() * .07
        return 0
```
```py
joe = Customer('John Doe', 0) # 고객 두명의 충성도 점수 joe는 0점, ann은 1100점
ann = Customer('Ann Smith', 1100)
cart = [LineItem('banana', 4, .5), LineItem('apple', 10, 1.5), LineItem('watermellon', 5, 5.0)] # 한 쇼핑 카트의 항목 종류가 3가지

Order(joe, cart, FidelityPromo()) # FidelityPromo 할인은 joe에게 아무 할인을 해주지 않음

Order(ann, cart, FidelityPromo()) # ann은 충성도 점수가 1000점이 넘으므로 5%할인을 받는다. 

banana_cart = [LineItem('banana', 30, .5), LineItem('apple', 10, 1.5)] # banana cart에는 banana상품이 30개, apple 상품이 10개 들어있다.

Order(joe, banana_cart, BulkItemPromo()) # BulkItemPromo 할인 덕분에 joe는 바나나에 대해 1.5 달러를 할인 받는다. 

long_order = [LineItem(str(item_code), 1, 1.0) for item_code in range(10)] # long_order에는 각각 1달러인 10개의 서로 다른 상품이 있다.

Order(joe, long_order, LargeOrderPromo()) # LargeOrderPromo 덕분에 joe는 전체 주문에 대해 7%를 할인 받는다. 

Order(joe, cart, LargeOrderPromo())
```

### 6.1.2 함수지향 전략
구체적인 전략 부분을 함수로 변경하고 Promotion Class를 제거

```py
from collections import namedtuple

Customer = namedtuple('Customer', 'name fidelity')

class LineItem:
    
    def __init__(self, product, quantity, price):
        self.product = product
        self.quantity = quantity
        self.price = price
    
    def total(self):
        return self.price * self.quantity
    
    
class Order: # 콘텍스트
    
    def __init__(self, customer, cart, promotion = None):
        self.customer = customer
        self.cart = list(cart)
        self.protmotion = promotion # promotion은 Order class에서 긁어오기
    
    def total(self):
        if not hasattr(self, '__total'):
            self.__total = sum(item.total() for item in self.cart) # 왜 굳이 double under score를?
        return self.__total
    
    def due(self):
        if self.promotion is None:
            discount = 0
        else:
            discount = self.promotion(self) # 이 시점에서 promotion 적용. 할인액을 계산하려면 self.promotion()함수를 호출하면 됨 
        
    def __repr__(self):
        fmt = '<Order total: {:2f} due: {:2f}>'
        return fmt.format(self.total(), self.due())
    
# 추상 클래스는 제거    
    
    
def fidelity_promo(order): # 구체적인 전략은 함수로 구현
    """충성도 포인트가 1000점 이상인 고객에게 전체 5%할인 적용"""
    return order.total() * .05 if order.customer.fidelity >= 1000 else 0

def bulk_item_promo(order):
    """20개 이상의 동일 상품을 구입하면 10% 할인 적용"""
    discount = 0
    for item in order.cart:
        if item.quantity >= 20:
            discoutn += item.total() * .1
    return discount

def large_order_promo(order):
    """10종류 이상의 상품을 구입하면 전체 7% 할인 적용"""
    distinct_items = {item.product for item in order.cart}
    if len(distinct_items) >= 10:
        return order.total() * .07
    return 0
    
```

### 6.1.3 최선의 전략 선택하기 : 단순한 접근법

```py
# 함수 리스트를 반복해서 최대 할인액을 찾아내는 best_promo()함수

promos = [fidelity_promo, bulk_item_promo, large_order_promo] # 함수로 구현된 전략들을 list로

def best_promo(order): # 다른 *_promo 함수들처럼 Order 객체를 인수로 받음
    """최대로 할인받을 금액을 반환한다."""
    return max(promo(order) for promo in promos) # 제너레이터 표현식을 써서 promos에 있는 각 함수를 order에 적용하고 최대 할인액을 계산
```

### 6.1.4 모듈에서 전략찾기
globals()

현재 전역 심벌 테이블을 나타내는 딕셔너리 객체를 반환한다. 이 딕셔너리는 언제나 현재 모듈에 대한 내용을 담고 있다. 
(함수나 메서드 안에서 호춣할 때, 함수를 호출한 모듈이 아니라 함수가 정의된 모듈을 나타낸다.)
```py
promos = [globals()[name] for name in globals() if name.endswith('_promo') and name != 'best_promo']

def best_promo(order):
    """최대로 할인받을 금액을 반환한다"""
    return max(promo(order) for promo in promos)
```
