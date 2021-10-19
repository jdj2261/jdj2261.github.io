---
layout: post
title: Effective Python 2nd - 메타클래스와 애트리뷰트
date: 2021-10-19 11:48:00 +09:00
category: Python
---

Chapter 6. 메타클래스와 애트리뷰트를 읽으면서 정리합니다.

새로 알게 되었거나 중요하다고 생각되는 부분 위주로 정리하고자 합니다.

# Effective Python 2nd

## Chapter 6. 메타클래스와 애트리뷰트

### 44. 세터와 게터 메서드 대신 평범한 애트리뷰트를 사용하라

- 새로운 클래스 인터페이스를 정의할 때는 간단한 공개 애트리뷰트에서 시작하고, 세터나 게터 메서드를 가급적 사용하지 말라

  ~~~python
  class OldResistor:
      def __init__(self, ohms):
          self._ohms = ohms
  
      def get_ohms(self):
          return self._ohms
  
      def set_ohms(self, ohms):
          self._ohms = ohms
  ~~~

- @property로 이를 구현하자

  ~~~python
  class Resistor:
      def __init__(self, ohms):
          self.ohms = ohms
          self.voltage = 0
          self.current = 0
          
  class VoltageResistance(Resistor):
      def __init__(self, ohms):
          super().__init__(ohms)
          self._voltage = 0
  
      @property
      def voltage(self):
          return self._voltage
  
      @voltage.setter
      def voltage(self, voltage):
          self._voltage = voltage
          self.current = self._voltage / self.ohms
  ~~~

- @propert 메서드가 빠르게 실행되도록 유지하라.

  느리거나 복잡한 작업의 경우 프로퍼티 대신 일반 메서드를 사용하라.

### 45. 애트리뷰트를 리팩터링하는 대신 @property를 사용하라



