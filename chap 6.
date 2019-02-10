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
