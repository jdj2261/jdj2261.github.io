---
layout: post
title: Effective Python 2nd - 함수
date: 2021-10-13 11:00:00 +09:00
category: Python
---

Chapter 3. 함수를 읽으면서 정리합니다.

새로 알게 되었거나 중요하다고 생각되는 부분 위주로 정리하고자 합니다.

# Effective Python 2nd

## Chapter 3. 함수

### 19. 함수가 여러 값을 반환하는 경우 절대로 네 값 이상을 언패킹하지 말라

- 함수가 여러 값을 반환하기 위해 값들을 튜플에 넣어서 반환하고, 호출하는 쪽에서는 파이썬 언패킹 구문을 쓸 수 있다.

  ~~~python
  def get_stats(numbers):
      minimum = min(numbers)
      maximum = max(numbers)
      return minimum, maximum
  
  lengths = [63, 73, 72, 60, 67, 66, 71, 61, 72, 70]
  
  minimum, maximum = get_stats(lengths)  # 반환 값이 두 개
  ~~~

- 함수가 반환한 여러 값, 모든 값을 처리하는 별표 식을 사용해 언패킹할 수도 있다.

  ~~~python
  def get_avg_ratio(numbers):
      average = sum(numbers) / len(numbers)
      scaled = [x / average for x in numbers]
      scaled.sort(reverse=True)
      return scaled
  
  longest, *middle, shortest = get_avg_ratio(lengths)
  print(f'최대 길이: {longest:>4.0%}')
  print(f'최소 길이: {shortest:>4.0%}')
  ~~~

- 언패킹 구문에 변수가 네 개 이상 나오면 실수하기 쉬우므로 변수를 네 개 이상 사용하면 안된다.

  대신 작은 클래스를 반환하거나 namedtuple 인스턴스를 반환하라.

### 20. None을 반환하기보다는 예외를 발생시켜라

- 특별한 상황을 표현하기 위해 None을 반환하는 대신 예외를 발생시켜라. 

  문서에 예외 정보를 기록해 호출자가 예외를 제대로 처리하도록 하라.

  ~~~python
  def careful_divide(a, b):
      try:
          return a / b
      except ZeroDivisionError as e:
          raise ValueError('잘못된 입력')
  ~~~

- 그 어떤 경우에도 절대로 None을 반환하지 않는다는 사실을 타입 애너테이션으로 명시할 수 있다.

  ~~~python
  def careful_divide(a: float, b: float) -> float:
      """a를 b로 나눈다.
      Raises:
          ValueError: b가 0이어서 나눗셈을 할 수 없을 때
      """
      try:
          return a / b
      except ZeroDivisionError as e:
          raise ValueError('잘못된 입력')
  ~~~

### 21. 변수 영역과 클로저의 상호작용 방식을 이해하라

- 클로저 함수는 자신이 정의된 영역 외부에서 정의된 변수도 참조할 수 있다.

  ~~~python
  def sort_priority(values, group):
      def helper(x):
          if x in group:
              return (0, x)
          return (1, x)
      values.sort(key=helper)
  numbers = [8, 3, 1, 2, 5, 4, 7, 6]
  group = {2, 3, 5, 7}
  
  # Answer
  # [2, 3, 5, 7, 1, 4, 6, 8]
  ~~~

  - 클로저란?
    - 자신이 정의된 영역 밖의 변수를 참조하는 함수

- 기본적으로 클로저 내부에 사용한 대입문은 클로저를 감싸는 영역에 영향을 끼칠 수 없다.

  영역 지정 버그(scoping bug)라고 부르기도 한다.

  ~~~python
  def sort_priority2(numbers, group):
      found = False  # 영역: 'sort_priority2'
      def helper(x):
          if x in group:
              found = True  # 영역: 'helper' -- 좋지 않음!
              return (0, x)
          return (1, x)
      numbers.sort(key=helper)
      return found
    
  # found가 바뀌지 않는다
  ~~~

- 클로저가 자신을 감싸는 영역의 변수를 변경한다는 사실을 표시할 때 nonlocal 문을 사용할 수 있지만 간단한 함수가 아닌 경우 nonlocal 문을 사용하지 말라.

  ~~~python
  def sort_priority2(numbers, group):
      found = False
      def helper(x):
          nonlocal found       # 추가함
          if x in group:
              found = True
              return (0, x)
          return (1, x)
      numbers.sort(key=helper)
      return found
  ~~~

- 간단한 인터페이스 경우 클래스 대신 함수를 받아라.

  ~~~python
  class Sorter:
      def __init__(self, group):
          self.group = group
          self.found = False
  
      def __call__(self, x):
          if x in self.group:
              self.found = True
              return (0, x)
          return (1, x)
  
  sorter = Sorter(group)
  numbers.sort(key=sorter)
  assert sorter.found is True
  ~~~

### 22. 변수 위치 인자를 사용해 시각적인 잡음을 줄여라

- def 문에서 *args를 사용하면 함수가 가변 위치 기반 인자를 받을 수 있다.

  ~~~python
  def log(message, *values):  # 달라진 유일한 부분
      if not values:
          print(message)
      else:
          values_str = ', '.join(str(x) for x in values)
          print(f'{message}: {values_str}')
  
  log('내 숫자는 ', [1, 2])
  log('안녕 ')  # 훨씬 좋다
  ~~~

- 제너레이터에 * 연산자를 사용하면 프로그램이 메모리를 모두 소진하고 중단될 수 있다.

  인자의 개수가 처리하기 좋을 정도로 충분히 작다는 사실을 알고 있는 경우에 적합하다.

  ~~~python
  def my_generator():
      for i in range(10):
          yield i
  
  def my_func(*args):
      print(args)
  
  it = my_generator()
  my_func(*it)
  ~~~

- *arg를 받는 함수에 새로운 위치 기반 인자를 넣으면 감지하기 힘든 버그가 생길 수 있다.

  키워드 기반 인자를 사용하거나 타입 애너테이션을 사용해도 된다.

  ~~~python
  def log(sequence, message, *values):
      if not values:
          print(f'{sequence} - {message}')
      else:
          values_str = ', '.join(str(x) for x in values)
          print(f'{sequence} - {message}: {values_str}')
  
  log(1, '좋아하는 숫자는', 7, 33)   # 새 코드에서 가변 인자를 사용. 문제 없음
  log(1, '안녕')                   # 새 코드에서 가변 인자 없이 메시지만 사용. 문제 없음
  log('좋아하는 숫자는', 7, 33)      # 예전 방식 코드는 깨짐
  ~~~

### 23. 키워드 인자로 선택적인 기능을 제공하라

- 키워드를 사용하면 위치 인자만 사용할 때 혼동할 수 있는 여러 인자의 목적을 명확히 할 수 있다.

  ~~~python
  def flow_rate(weight_diff, time_diff, period=1):
      return (weight_diff / time_diff) * period
  
  flow_per_second = flow_rate(weight_diff, time_diff)
  flow_per_hour = flow_rate(weight_diff, time_diff, period=3600)
  ~~~

- 키워드 인자와 디폴트 값을 함께 사용하면 기본 호출 코드를 마이그레이션하지 않고도 함수에 새로운 기능을 쉽게 추가할 수 있다.

  ~~~python
  def flow_rate(weight_diff, time_diff,
                period=1, units_per_kg=1):
      return ((weight_diff * units_per_kg) / time_diff) * period
  ~~~

- 선택적 키워드 인자는 항상 위치가 아니라 키워드를 사용하자.

### 24. None과 독스트링을 사용해 동적인 디폴트 인자를 지정하라

- 동적인 값을 가질 수 있는 키워드 인자의 디폴트 값을 표현할 때는 None을 사용하라.

  그리고 함수의 독스트링에 문서화해두라.

  ~~~python
  def log(message, when=None):
      """메시지와 타임스탬프를 로그에 남긴다.
      Args:
          message: 출력할 메시지.
          when: 메시지가 발생한 시각(datetime).
              디폴트 값은 현재 시간이다.
      """
      if when is None:
          when = datetime.now()
      print(f'{when}: {message}')
  
  log('안녕!')
  sleep(0.1)
  log('다시 안녕!')
  ~~~

- 타입 애너테이션을 사용할 때도 None을 사용해 키워드 인자의 디폴트 값을 표현하는 방식을 적용할 수 있다.

  ~~~python
  from typing import Optional
  
  def log_typed(message: str,
                when: Optional[datetime]=None) -> None:
      """메시지와 타임스탬프를 로그에 남긴다.
      Args:
          message: 출력할 메시지.
          when: 메시지가 발생한 시각(datetime).
              디폴트 값은 현재 시간이다.
      """
      if when is None:
          when = datetime.now()
      print(f'{when}: {message}')
  ~~~

### 25. 위치로만 인자를 지정하게 하거나 키워드로만 인자를 지정하게 해서 함수 호출을 명확하게 만들라

- 키워드로만 지정해야 하는 인자를 사용하면 호출하는 쪽에서 특정 인자를 반드시 키워드를 사용해 호출하도록 강제할 수 있다.

  이로 인해 함수 호출의 의도를 명확히 할 수 있다.

  키워드로만 지정해야 하는 인자는 인자 목록에서 * 다음에 위치한다.

  ~~~python
  def safe_division_c(number, divisor, *,         # 변경
                      ignore_overflow=False,
                      ignore_zero_division=False):
      try:
          return number / divisor
      except OverflowError:
          if ignore_overflow:
              return 0
          else:
              raise
      except ZeroDivisionError:
          if ignore_zero_division:
              return float('inf')
          else:
              raise
  ~~~

- 위치로만 지정해야 하는 인자를 사용하면 호출하는 쪽에서 키워드를 사용해 인자를 지정지 못하게 만들 수 있고,

  이에 따라 함수 구현과 함수 호출 지점 사이의 결합을 줄일 수 있다.

  위치로만 지정해야 하는 인자는 인자 목록에서 / 앞에 위치한다.

  ~~~python
  def safe_division_d(numerator, denominator, /, *, # 변경
                      ignore_overflow=False,
                      ignore_zero_division=False):
      try:
          return numerator / denominator
      except OverflowError:
          if ignore_overflow:
              return 0
          else:
              raise
      except ZeroDivisionError:
          if ignore_zero_division:
              return float('inf')
          else:
              raise
  ~~~

- 인자 목록에서 /와 * 사이에 있는 파라미터는 키워드를 사용해 전달해도 되고 위치를 기반으로 전달해도 된다.
  이런 동작은 파이썬 함수 파라미터의 기본 동작이다.

  ~~~python
  def safe_division_e(numerator, denominator, /,
                      ndigits=10, *,                 # 변경
                      ignore_overflow=False,
                      ignore_zero_division=False):
      try:
          fraction = numerator / denominator         # 변경
          return round(fraction, ndigits)            # 변경
      except OverflowError:
          if ignore_overflow:
              return 0
          else:
              raise
      except ZeroDivisionError:
          if ignore_zero_division:
              return float('inf')
          else:
              raise
  
  result = safe_division_e(22, 7)
  print(result)
  
  result = safe_division_e(22, 7, 5)
  print(result)
  
  result = safe_division_e(22, 7, ndigits=2)
  print(result)
  ~~~

### 26. functools.wrap을 사용해 함수 데코레이터를 정의하라

- 파이썬 데코레이터는 실행 시점에 함수가 다른 함수를 변경할 수 있게 해주는 구문이다.

  ~~~python
  def trace(func):
      def wrapper(*args, **kwargs):
          result = func(*args, **kwargs)
          print(f'{func.__name__}({args!r}, {kwargs!r}) '
                f'-> {result!r}')
          return result
      return wrapper
  
  @trace
  def fibonacci(n):
      """Return n 번째 피보나치 수"""
      if n in (0, 1):
          return n
      return (fibonacci(n - 2) + fibonacci(n - 1))
  
  fibonacci(4)
  ~~~

- 데코레이터를 구현할 때 인트로스펙션에서 문제가 생기지 않길 바란다면 functools 내장 모듈의 wraps 데코레이터를 사용하라.

  wraps를 wrapper 함수에 적용하면 wraps가 데코레이터 내부에 들어가는 함수에서 중요한 메타데이터를 복사해 wrapper 함수에 적용해준다.

  ~~~python
  from functools import wraps
  
  def trace(func):
      @wraps(func)
      def wrapper(*args, **kwargs):
          result = func(*args, **kwargs)
          print(f'{func.__name__}({args!r}, {kwargs!r}) '
                f'-> {result!r}')
          return result
      return wrapper
  
  
  @trace
  def fibonacci(n):
      """Return n 번째 피보나치 수"""
      if n in (0, 1):
          return n
      return (fibonacci(n - 2) + fibonacci(n - 1))
  
  help(fibonacci)
  
  import pickle
  print(pickle.dumps(fibonacci))
  ~~~

  - 인트로스펙션
    - 실행 시점에 프로그램이 어떻게 실행되는지 관찰하는 것 (ex. pdb)
    - 이와 반대되는 용어로 리플렉션이 있음 (실행 시점에 프로그램 조작)
