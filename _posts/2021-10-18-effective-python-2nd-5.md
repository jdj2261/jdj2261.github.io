---
layout: post
title: Effective Python 2nd - 클래스와 인터페이스
date: 2021-10-18 12:20:00 +09:00
category: Python
---

Chapter 5. 클래스와 인터페이스를 읽으면서 정리합니다.

새로 알게 되었거나 중요하다고 생각되는 부분 위주로 정리하고자 합니다.

# Effective Python 2nd

## Chapter 5. 클래스와 인터페이스



### 37. 내장 타입을 여러 단계로 내포시키기보다는 클래스를 합성하라

- 딕셔너리, 긴 튜플, 다른 내장 타입이 복잡하게 내포된 데이터를 값으로 사용하는 딕셔너리를 만들지 말라.

- 완전한 클래스가 제공하는 유연성이 필요하지 않고 가벼운 불변 데이터 컨테이너가 필요하다면 nametuple을 사용하라.

  ~~~python
  from collections import namedtuple
  Grade = namedtuple('Grade', ('score', 'weight'))
  ~~~

  

- 내부 상태를 표현하는 딕셔너리가 복잡해지면 이 데이터를 관리하는 코드를 여러 클래스로 나눠서 작성하라.

  ~~~python
  class Subject:
      def __init__(self):
          self._grades = []
  
      def report_grade(self, score, weight):
          self._grades.append(Grade(score, weight))
  
      def average_grade(self):
          total, total_weight = 0, 0
          for grade in self._grades:
              total += grade.score * grade.weight
              total_weight += grade.weight
          return total / total_weight
        
  class Student:
      def __init__(self):
          self._subjects = defaultdict(Subject)
  
      def get_subject(self, name):
          return self._subjects[name]
  
      def average_grade(self):
          total, count = 0, 0
          for subject in self._subjects.values():
              total += subject.average_grade()
              count += 1
          return total / count
        
  class Gradebook:
      def __init__(self):
          self._students = defaultdict(Student)
  
      def get_student(self, name):
          return self._students[name]
        
  book = Gradebook()
  albert = book.get_student('알버트 아인슈타인')
  math = albert.get_subject('수학')
  math.report_grade(75, 0.05)
  math.report_grade(65, 0.15)
  math.report_grade(70, 0.80)
  gym = albert.get_subject('체육')
  gym.report_grade(100, 0.40)
  gym.report_grade(85, 0.60)
  print(albert.average_grade())
  ~~~

### 38. 간단한 인터페이스의 경우 클래스 대신 함수를 받아라

- 파이썬의 여러 컴포넌트 사이에 간단한 인터페이스가 필요할 때는 클래스를 정의하고 인스턴스화하는 대신 간단히 함수를 사용할 수 있다.

- `__call__` 특별 메서드를 사용하면 클래스의 인스턴스인 객체를 일반 파이썬 함수처럼 호출할 수 있다.

- 상태를 유지하기 위한 함수가 필요한 경우에는 상태가 있는 클로저를 정의하는 대신 `__call__` 메서드가 있는 클래스를 정의할지 고려하라.

  ```python
  class BetterCountMissing:
      def __init__(self):
          self.added = 0
  
      def __call__(self):
          self.added += 1
          return 0
  
  counter = BetterCountMissing()
  assert counter() == 0
  assert callable(counter)
  
  counter = BetterCountMissing()
  result = defaultdict(counter, current) # __call__에 의존함
  for key, amount in increments:
      result[key] += amount
  assert counter.added == 2
  ```

### 39. 객체를 제너릭하게 구성하려면 @classmethod를 통한 다형성을 활용하라

- 파이썬의 클래스에는 생성자 `__init__` 메서드뿐이다.

- @classmethod를 사용하면 클래스에 다른 생성자를 정의할 수 있다.

- 클래스 메서드 다형성을 활용하면 여러 구체적인 하위 클래스의 객체를 만들고 연결하는 제너릭한 방법을 제공할 수 있다.

  ```python
  ### GenericInputData와 PathInputData를 사용한 방법
  class GenericInputData:
      def read(self):
          raise NotImplementedError
  
      @classmethod
      def generate_inputs(cls, config):
          raise NotImplementedError
  
  class PathInputData(GenericInputData):
      def __init__(self, path):
          super().__init__()
          self.path = path
  
      def read(self):
          with open(self.path) as f:
              return f.read()
  
      @classmethod
      def generate_inputs(cls, config):
          data_dir = config['data_dir']
          for name in os.listdir(data_dir):
              yield cls(os.path.join(data_dir, name))
  
  class GenericWorker:
      def __init__(self, input_data):
          self.input_data = input_data
          self.result = None
  
      def map(self):
          raise NotImplementedError
  
      def reduce(self, other):
          raise NotImplementedError
  
      @classmethod
      def create_workers(cls, input_class, config):
          workers = []
          for input_data in input_class.generate_inputs(config):
              workers.append(cls(input_data))
          return workers
  
  class LineCountWorker(GenericWorker):
      def map(self):
          data = self.input_data.read()
          self.result = data.count('\n')
  
      def reduce(self, other):
          self.result += other.result
  
  def mapreduce(worker_class, input_class, config):
      workers = worker_class.create_workers(input_class, config)
      return execute(workers)
    
  config = {'data_dir': tmpdir}
  result = mapreduce(LineCountWorker, PathInputData, config)
  print(f'총 {result} 줄이 있습니다.')
  ```

### 40. super로 부모 클래스를 초기화하라

- 표준 메서드 결정 순서(MRO)를 활용해 상위 클래스 초기화 순서와 다이아몬드 상속 문제를 해결한다.

  - 다이아몬드 상속 문제란?

    어떤 클래스가 두 가지 서로 다른 클래스를 상속하는데, 두 상위 클래스의 상위 계층을 거슬러 올라가면 같은 조상 클래스가 존재하는 경우 조상 클래스의 `__init__` 메서드가 여러 번 호출 될 수 있다.

- 부모 클래스를 초기화 할때는 super 내장 함수를 아무 인자 없이 호출하라.

  super를 아무 인자 없이 호출하면 파이썬 컴파일러가 자동으로 올바른 파라미터를 넣어준다.

  ```python
  class ExplicitTrisect(MyBaseClass):
      def __init__(self, value):
          super(ExplicitTrisect, self).__init__(value)
          self.value /= 3
  
  class AutomaticTrisect(MyBaseClass):
      def __init__(self, value):
          super(__class__, self).__init__(value)
          self.value /= 3
  
  class ImplicitTrisect(MyBaseClass):
      def __init__(self, value):
          super().__init__(value)
          self.value /= 3
  
  assert ExplicitTrisect(9).value == 3
  assert AutomaticTrisect(9).value == 3
  assert ImplicitTrisect(9).value == 3
  ```

### 41. 기능을 합성할 때는 믹스인 클래스를 사용하라

- 믹스인을 사용해 구현할 수 있는 기능을 인스턴스 애트리뷰트와  `__init__` 을 사용하는 다중 상속을 통해 구현하지 말라.

- 믹스인에는 필요에 따라 인스턴스 메서드는 물론 클래스 메서드도 포함될 수 있다.

- 믹스인을 합성하면 단순한 동작으로부터 더 복잡한 기능을 만들어낼 수 있다.

  ```python
  class ToDictMixin:
      def to_dict(self):
          return self._traverse_dict(self.__dict__)
  
      def _traverse_dict(self, instance_dict):
          output = {}
          for key, value in instance_dict.items():
              output[key] = self._traverse(key, value)
          return output
  
      def _traverse(self, key, value):
          if isinstance(value, ToDictMixin):
              return value.to_dict()
          elif isinstance(value, dict):
              return self._traverse_dict(value)
          elif isinstance(value, list):
              return [self._traverse(key, i) for i in value]
          elif hasattr(value, '__dict__'):
              return self._traverse_dict(value.__dict__)
          else:
              return value
  
  class BinaryTree(ToDictMixin):
      def __init__(self, value, left=None, right=None):
          self.value = value
          self.left = left
          self.right = right
  
  tree = BinaryTree(10,
                    left=BinaryTree(7, right=BinaryTree(9)),
                    right=BinaryTree(13, left=BinaryTree(11)))
  print(tree.to_dict())
  
  #{'value': 10, 'left': {'value': 7, 'left': None, 'right': {'value': 9, 'left': None, 'right': None}}, 'right': {'value': 13, 'left': {'value': 11, 'left': None, 'right': None}, 'right': None}}
  ```

  ```python
  class BinaryTreeWithParent(BinaryTree):
      def __init__(self, value, left=None,
                   right=None, parent=None):
          super().__init__(value, left=left, right=right)
          self.parent = parent
  
      def _traverse(self, key, value):
          if (isinstance(value, BinaryTreeWithParent) and
                  key == 'parent'):
              return value.value  # Prevent cycles
          else:
              return super()._traverse(key, value)
  
  root = BinaryTreeWithParent(10)
  root.left = BinaryTreeWithParent(7, parent=root)
  root.left.right = BinaryTreeWithParent(9, parent=root.left)
  print(root.to_dict())
  
  #{'value': 10, 'left': {'value': 7, 'left': None, 'right': {'value': 9, 'left': None, 'right': None, 'parent': 7}, 'parent': 10}, 'right': None, 'parent': None}
  
  class NamedSubTree(ToDictMixin):
      def __init__(self, name, tree_with_parent):
          self.name = name
          self.tree_with_parent = tree_with_parent
  
  my_tree = NamedSubTree('foobar', root.left.right)
  print(my_tree.to_dict()) # 무한 루프없음
  # {'name': 'foobar', 'tree_with_parent': {'value': 9, 'left': None, 'right': None, 'parent': 7}}
  ```

  ```python
  import json
  
  class JsonMixin:
      @classmethod
      def from_json(cls, data):
          kwargs = json.loads(data)
          return cls(**kwargs)
  
      def to_json(self):
          return json.dumps(self.to_dict())
  
  class DatacenterRack(ToDictMixin, JsonMixin):
      def __init__(self, switch=None, machines=None):
          self.switch = Switch(**switch)
          self.machines = [
              Machine(**kwargs) for kwargs in machines]
  
  class Switch(ToDictMixin, JsonMixin):
      def __init__(self, ports=None, speed=None):
          self.ports = ports
          self.speed = speed
  
  class Machine(ToDictMixin, JsonMixin):
      def __init__(self, cores=None, ram=None, disk=None):
          self.cores = cores
          self.ram = ram
          self.disk = disk
  
  serialized = """{
      "switch": {"ports": 5, "speed": 1e9},
      "machines": [
          {"cores": 8, "ram": 32e9, "disk": 5e12},
          {"cores": 4, "ram": 16e9, "disk": 1e12},
          {"cores": 2, "ram": 4e9, "disk": 500e9}
      ]
  }"""
  
  deserialized = DatacenterRack.from_json(serialized)
  roundtrip = deserialized.to_json()
  assert json.loads(serialized) == json.loads(roundtrip)
  ```

### 42. 비공개 애트리뷰트보다는 공개 애트리뷰트를 사용하라

- 파이썬 컴파일러는 비공개 애트리뷰트를 자식 클래스나 클래스 외부에서 사용하지 못하도록 엄격히 금지하지 않는다.

- 비공개 애트리뷰트로 접근을 막으려고 시도하기보다 보호된 필드를 사용하면서 문서에 적절한 가이드를 남겨라.

- 코드 작성을 제어할 수 없는 하위 클래스에서 이름 충돌이 일어나는 경우를 막고 싶을 때만 비공개 애트리뷰트를 사용하라.

  ```python
  class ApiClass:
      def __init__(self):
          self._value = 5
  
      def get(self):
          return self._value
  
  class Child(ApiClass):
      def __init__(self):
          super().__init__()
          self._value = 'hello'  # 충돌
  a = Child()
  print(f'{a.get()} 와 {a._value} 는 달라야 합니다.')
  # hello 와 hello 는 달라야 합니다.
  
  class ApiClass:
      def __init__(self):
          self.__value = 5    # 밑줄 2개!
  
      def get(self):
          return self.__value # 밑줄 2개!
  
  class Child(ApiClass):
      def __init__(self):
          super().__init__()
          self._value = 'hello' # OK!
  
  a = Child()
  print(f'{a.get()} 와 {a._value} 는 다릅니다.')
  # 5 와 hello 는 다릅니다.
  ```

### 43. 커스텀 컨테이너 타입은 collections.abc를 상속하라

- 간편하게 사용할 경우에는 파이썬 컨테이너 타입(리스트, 딕셔너리)을 직접 상속하라

  ```python
  class FrequencyList(list):
      def __init__(self, members):
          super().__init__(members)
  
      def frequency(self):
          counts = {}
          for item in self:
              counts[item] = counts.get(item, 0) + 1
          return counts
  
  foo = FrequencyList(['a', 'b', 'a', 'c', 'b', 'a', 'd'])
  print('길이: ', len(foo))
  
  foo.pop()
  print('pop한 다음:', repr(foo))
  print('빈도:', foo.frequency())
  
  #길이:  7
  #pop한 다음: ['a', 'b', 'a', 'c', 'b', 'a']
  #빈도: {'a': 3, 'b': 2, 'c': 1}
  ```

- 커스텀 턴테이너 타입이 collection.abc에 정의된 인터페이스를 상속하면 커스텀 컨테이너 타입이 정상적으로 작동하기 위해 피요한 인터페이스와 기능을 제대로 구현하도록 보장할 수 있다.

  ~~~python
  from collections.abc import Sequence
  
  class SequenceNode(IndexableNode):
      def __len__(self):
          for count, _ in enumerate(self._traverse(), 1):
              pass
          return count
  
  class BadType(Sequence):
      pass
  
  # 오류가 나는 부분. 오류를 보고 싶으면 커멘트를 해제할것
  #foo = BadType()
  
  class BetterNode(SequenceNode, Sequence):
      pass
  
  tree = BetterNode(
      10,
      left=BetterNode(
          5,
          left=BetterNode(2),
          right=BetterNode(
              6,
              right=BetterNode(7))),
      right=BetterNode(
          15,
          left=BetterNode(11))
  )
  
  print('7의 인덱스:', tree.index(7))
  print('10의 개수:', tree.count(10))
  # 7의 인덱스: 3
  # 10의 개수: 1
  ~~~

  
