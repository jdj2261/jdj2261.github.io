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

- @property를 사용해 데이터 모델을 점진적으로 개선하라
- @property 매서드를 너무 과하게 쓰고 있다면, 클래스와 클래스를 사용하는 모든 코드를 리팩터링하는 것을 고려하라

### 46. 재사용 가능한 @property 메서드를 만들려면 디스크립터를 사용하라

- @property 메서드의 동작과 검증 기능을 재사용하고 싶다면 디스크립터 클래스를 만들라.

- 디스크립터 클래스를 만들 때는 메모리 누수를 방지하기 위해 WeakKeyDictionary를 사용하라

  ~~~python
  from weakref import WeakKeyDictionary
  class Grade:
      def __init__(self):
          self._values = WeakKeyDictionary()
  
      def __get__(self, instance, instance_type):
          if instance is None:
              return self
          return self._values.get(instance, 0)
  
      def __set__(self, instance, value):
          if not (0 <= value <= 100):
              raise ValueError(
                  '점수는 0과 100 사이입니다')
          self._values[instance] = value
  
  class Exam:
      # 클래스 애트리뷰트
      math_grade = Grade()
      writing_grade = Grade()
      science_grade = Grade()
  
  first_exam = Exam()
  first_exam.writing_grade = 82
  second_exam = Exam()
  second_exam.writing_grade = 75
  print(f'첫 번째 쓰기 점수 {first_exam.writing_grade} 맞음')
  print(f'두 번째 쓰기 점수 {second_exam.writing_grade} 맞음')
  ~~~

### 47. 지연 계산 애트리뷰트가 필요하면 `__getatrr__`, `__getattribute__`,` __setattr__` 을 사용하라

- 객체의 애트리뷰트를 지연해 가져오거나 저장할 수 있다.

- `__getatrr__` 은 애트리뷰트가 존재하지 않을 때 호출되지만`__getattribute__` 는 애트리뷰트를 읽을 때마다 항상 호출된다.

- 무한 재귀를 피하려면 super()에 있는 메서드를 사용해 인스턴스 애트리뷰트에 접근하라.

  ~~~python
  class DictionaryRecord:
      def __init__(self, data):
          self._data = data
  
      def __getattribute__(self, name):
          print(f'* 호출: __getattribute__({name!r})')
          data_dict = super().__getattribute__('_data')
          return data_dict[name]
  
  data = DictionaryRecord({'foo': 3})
  print('foo: ', data.foo)
  ~~~

### 48. `__init_subclass__`를 사용해 하위 클래스를 검증하라

- 메타클래스의 __new__ 메서드는 class 문의 모든 본문이 처리된 직후에 호출된다.

  ~~~python
  class Meta(type):
      def __new__(meta, name, bases, class_dict):
          print(f'* 실행: {name}의 메타 {meta}.__new__')
          print('기반클래스들:', bases)
          print(class_dict)
          return type.__new__(meta, name, bases, class_dict)
  
  class MyClass(metaclass=Meta):
      stuff = 123
  
      def foo(self):
          pass
  
  class MySubclass(MyClass):
      other = 567
  
      def bar(self):
          pass
    
  # * 실행: MyClass의 메타 <class '__main__.Meta'>.__new__
  # 기반클래스들: ()
  # {'__module__': '__main__', '__qualname__': 'MyClass', 'stuff': 123, 'foo': <function # MyClass.foo at 0x1060a9040>}
  # * 실행: MySubclass의 메타 <class '__main__.Meta'>.__new__
  # 기반클래스들: (<class '__main__.MyClass'>,)
  # {'__module__': '__main__', '__qualname__': 'MySubclass', 'other': 567, 'bar': <function MySubclass.bar at 0x1060a90d0>}
  ~~~

- 메타클래스를 사용해 클래스가 정의된 직후이면서 클래스가 생성되기 직전에 클래스 정의를 변경할 수 있다.

  하지만 원하는 목적을 달성하기에 너무 복잡하다.

  ~~~python
  class ValidatePolygon(type):
      def __new__(meta, name, bases, class_dict):
          # Polygon 클래스의 하위 클래스만 검증한다
          if bases:
              if class_dict['sides'] < 3:
                  raise ValueError('다각형 변은 3개 이상이어야 함')
          return type.__new__(meta, name, bases, class_dict)
  
  class Polygon(metaclass=ValidatePolygon):
      sides = None # 하위 클래스는 이 애트리뷰트에 값을 지정해야 한다
      @classmethod
      def interior_angles(cls):
          return (cls.sides - 2) * 180
  
  class Triangle(Polygon):
      sides = 3
  
  class Rectangle(Polygon):
      sides = 4
  
  class Nonagon(Polygon):
      sides = 9
  
  assert Triangle.interior_angles() == 180
  assert Rectangle.interior_angles() == 360
  assert Nonagon.interior_angles() == 1260
  
  class Line(Polygon):
     print('sides 이전')
     sides = 2
     print('sides 이후')
  
  #sides 이전
  #sides 이후
  #    raise ValueError('다각형 변은 3개 이상이어야 함')
  #ValueError: 다각형 변은 3개 이상이어야 함
  ~~~

- `__init_subclass__` 를 사용해 하위 클래스가 정의된 직후, 하위 클래스 타입이 만들어지기 직전에 요건을 확인하라

  ~~~python
  class ValidatePolygon(type):
      def __new__(meta, name, bases, class_dict):
          # 루트 클래스가 아닌 경우만 검증한다
          if not class_dict.get('is_root'):
              if class_dict['sides'] < 3:
                  raise ValueError('다각형 변은 3개 이상이어야 함')
          return type.__new__(meta, name, bases, class_dict)
  
  class Polygon(metaclass=ValidatePolygon):
      is_root = True
      sides = None  # 하위 클래스에서 이 애트리뷰트 값을 지정해야 한다
  
  class ValidateFilledPolygon(ValidatePolygon):
      def __new__(meta, name, bases, class_dict):
          # 루트 클래스가 아닌 경우만 검증한다
          if not class_dict.get('is_root'):
              if class_dict['color'] not in ('red', 'green'):
                  raise ValueError('지원하지 않는 color 값')
          return super().__new__(meta, name, bases, class_dict)
  
  class FilledPolygon(Polygon, metaclass=ValidateFilledPolygon):
      is_root = True
      color = None  # 하위 클래스에서 이 애트리뷰트 값을 지정해야 한다
  
  class GreenPentagon(FilledPolygon):
      color = 'green'
      sides = 5
  
  greenie = GreenPentagon()
  assert isinstance(greenie, Polygon)
  ~~~

  

- `super().__init_subclass__` 를 호출해 여러 계층에 걸쳐 클래스를 검증하고 다중 상속을 처리하라

  ~~~python
  class Top:
      def __init_subclass__(cls):
          super().__init_subclass__()
          print(f'{cls}의 Top')
  
  class Left(Top):
      def __init_subclass__(cls):
          super().__init_subclass__()
          print(f'{cls}의 Left')
  
  
  class Right(Top):
      def __init_subclass__(cls):
          super().__init_subclass__()
          print(f'{cls}의 Right')
  
  
  class Bottom(Left, Right):
      def __init_subclass__(cls):
          super().__init_subclass__()
          print(f'{cls}의 Bottom')
  ~~~

### 49. `__init_subclass__` 를 사용해 클래스 확장을 등록하라

- 클래스 등록은 파이썬 프로그램을 모듈화할 때 유용한 패턴이다.

  ~~~python
  import json
  class Serializable:
      def __init__(self, *args):
          self.args = args
  
      def serialize(self):
          return json.dumps({'args': self.args})
  
  class Point2D(Serializable):
      def __init__(self, x, y):
          super().__init__(x, y)
          self.x = x
          self.y = y
  
      def __repr__(self):
          return f'Point2D({self.x}, {self.y})'
  
  point = Point2D(5, 3)
  print('객체:', point)
  print('직렬화한 값:', point.serialize())
  
  #객체: Point2D(5, 3)
  #직렬화한 값: {"args": [5, 3]}
  ~~~

- 메타클래스를 사용하면, 프로그램 안에서 기반 클래스를 상속한 하위 클래스가 정의될 때마다 등록 코드를 자동으로 실행할 수 있다.

  ~~~python
  class BetterSerializable:
      def __init__(self, *args):
          self.args = args
  
      def serialize(self):
          return json.dumps({
              'class': self.__class__.__name__,
              'args': self.args,
          })
  
      def __repr__(self):
          name = self.__class__.__name__
          args_str = ', '.join(str(x) for x in self.args)
          return f'{name}({args_str})'
    registry = {}
  
  def register_class(target_class):
      registry[target_class.__name__] = target_class
  
  def deserialize(data):
      params = json.loads(data)
      name = params['class']
      target_class = registry[name]
      return target_class(*params['args'])
  ~~~

  ~~~python
  class Meta(type):
      def __new__(meta, name, bases, class_dict):
          cls = type.__new__(meta, name, bases, class_dict)
          register_class(cls)
          return cls
  
  class RegisteredSerializable(BetterSerializable,
                               metaclass=Meta):
      pass
  
  class Vector3D(RegisteredSerializable):
      def __init__(self, x, y, z):
          super().__init__(x, y, z)
          self.x, self.y, self.z = x, y, z
  ~~~

- 표준적인 메타클래스 방식보다는 `__init_subclass` 가 더 낫다.

  ~~~python
  class BetterRegisteredSerializable(BetterSerializable):
      def __init_subclass__(cls):
          super().__init_subclass__()
          register_class(cls)
  
  
  class Vector1D(BetterRegisteredSerializable):
      def __init__(self, magnitude):
          super().__init__(magnitude)
          self.magnitude = magnitude
  ~~~

### 50. `__set_name__` 으로 클래스 애트리뷰트를 표시하라

- 메타클래스를 사용하면 어떤 클래스가 완전히 정의되기 전에 클래스의 애트리뷰트를 변경할 수 있다.

  ~~~python
  class Meta(type):
      def __new__(meta, name, bases, class_dict):
          for key, value in class_dict.items():
              if isinstance(value, Field):
                  value.name = key
                  value.internal_name = '_' + key
          cls = type.__new__(meta, name, bases, class_dict)
          return cls
  
  class DatabaseRow(metaclass=Meta):
      pass
  
  class Field:
      def __init__(self):
          # 이 두 정보를 메타클래스가 채워 준다
          self.name = None
          self.internal_name = None
  
      def __get__(self, instance, instance_type):
          if instance is None:
              return self
          return getattr(instance, self.internal_name, '')
  
      def __set__(self, instance, value):
          setattr(instance, self.internal_name, value)
  
  class BetterCustomer(DatabaseRow):
      first_name = Field()
      last_name = Field()
      prefix = Field()
      suffix = Field()
  
  cust = BetterCustomer()
  print(f'이전: {cust.first_name!r} {cust.__dict__}')
  cust.first_name = '오일러'
  print(f'이후: {cust.first_name!r} {cust.__dict__}')
  
  #이전: '' {}
  #이후: '오일러' {'_first_name': '오일러'}
  ~~~

- `__set_name__` 틀별 메서드를 디스크립터 클래스에 정의하면 디스크립터가 포함된 클래스 프로퍼티 이름을 처리할 수 있다.

  ~~~python
  class Field:
      def __init__(self):
          self.name = None
          self.internal_name = None
  
      def __set_name__(self, owner, name):
          # 클래스가 생성될 때 모든 스크립터에 대해 이 메서드가 호출된다
          self.name = name
          self.internal_name = '_' + name
  
      def __get__(self, instance, instance_type):
          if instance is None:
              return self
          return getattr(instance, self.internal_name, '')
  
      def __set__(self, instance, value):
          setattr(instance, self.internal_name, value)
  
  class FixedCustomer:
      first_name = Field()
      last_name = Field()
      prefix = Field()
      suffix = Field()
  
  cust = FixedCustomer()
  print(f'이전: {cust.first_name!r} {cust.__dict__}')
  cust.first_name = '메르센'
  print(f'이후: {cust.first_name!r} {cust.__dict__}')
  
  #이전: '' {}
  #이후: '메르센' {'_first_name': '메르센'}
  ~~~

### 51. 합성 가능한 클래스 확장이 필요하면 메타클래스보다는 클래스 데코레이터를 사용하라

- 클래스 데코레이터는 class 인스턴스를 파라미터로 받아서 변경한 클래스나 새로운 클래스를 반환해주는 간단한 함수이다.

  ~~~python
  def my_class_decorator(klass):
      klass.extra_param = '안녕'
      return klass
  
  @my_class_decorator
  class MyClass:
      pass
  
  print(MyClass)
  print(MyClass.extra_param)
  
  #<class '__main__.MyClass'>
  #안녕
  ~~~

  ~~~python
  def trace_func(func):
      if hasattr(func, 'tracing'):  # 단 한번만 데코레이터를 적용한다
          return func
  
      @wraps(func)
      def wrapper(*args, **kwargs):
          result = None
          try:
              result = func(*args, **kwargs)
              return result
          except Exception as e:
              result = e
              raise
          finally:
              print(f'{func.__name__}({args!r}, {kwargs!r}) -> '
                    f'{result!r}')
  
      wrapper.tracing = True
      return wrapper
  
  def trace(klass):
      for key in dir(klass):
          value = getattr(klass, key)
          if isinstance(value, trace_types):
              wrapped = trace_func(value)
              setattr(klass, key, wrapped)
      return klass
  
  @trace
  class TraceDict(dict):
      pass
  
  trace_dict = TraceDict([('안녕', 1)])
  trace_dict['거기'] = 2
  trace_dict['안녕']
  try:
      trace_dict['존재하지 않음']
  except KeyError:
      pass # 키 오류가 발생할 것으로 예상함
  
  #__new__((<class '__main__.TraceDict'>, [('안녕', 1)]), {}) -> {}
  #__getitem__(({'안녕': 1, '거기': 2}, '안녕'), {}) -> 1
  #__getitem__(({'안녕': 1, '거기': 2}, '존재하지 않음'), {}) -> KeyError('존재하지 않음')
  ~~~

  

