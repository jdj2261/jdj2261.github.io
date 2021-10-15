---
layout: post
title: Effective Python 2nd - 컴프리헨션과 제너레이터
date: 2021-10-15 12:20:00 +09:00
category: Python
---

Chapter 4. 컴프리헨션과 제너레이터를 읽으면서 정리합니다.

새로 알게 되었거나 중요하다고 생각되는 부분 위주로 정리하고자 합니다.

# Effective Python 2nd

## Chapter 4. 컴프리헨션과 제너레이터



### 27. map과 filter 대신 컴프리헨션을 사용하라

- 리스트 컴프리헨션은 lambda 식을 사용하지 않기 때문에 같은 일을 하는 map과 filter 내장 함수를 사용하는 것보다 명확하다

  ~~~python
  squares = [x**2 for x in a] # 리스트 컴프리핸션
  ~~~

- 리스트 컴프리헨션을 사용하면 쉽게 입력 리스트의 원소를 건너뛸 수 있다.

  ~~~python
  even_squares = [x**2 for x in a if x % 2 == 0]
  ~~~

- 딕셔너리와 집합도 컴프리헨션으로 생성할 수 있다.

  ~~~python
  even_squares_dict = {x: x**2 for x in a if x % 2 == 0}
  threes_cubed_set = {x**3 for x in a if x % 3 == 0}
  ~~~

### 28. 컴프리헨션 내부에 제어 하위 식을 세 개 이상 사용하지 말라

- 제어 하위 식이 세 개 이상인 컴프리헨션은 이해하기 매우 어려우므로 가능하면 피하자.

  ~~~python
  # 아래보다는
  flat = [x for sublist1 in my_lists
          for sublist2 in sublist1
          for x in sublist2]
  
  # 아래가 더 읽기 편하다.
  flat = []
  for sublist1 in my_lists:
      for sublist2 in sublist1:
          flat.extend(sublist2)
  ~~~

### 29. 대입식을 사용해 컴프리헨션 안에서 반복 작업을 피하라

- 대입식을 통해 조건 부분에서 사용한 값을 같은 컴프리헨션이나 제너레이터의 다른 위치에서 재사용할 수 있다.

  이를 통해 가독성과 성능을 향상시킬 수 있다.

  ~~~python
  found = {name: batches for name in order
           if (batches := get_batches(stock.get(name, 0), 8))}
  ~~~

- 조건이 아닌 부분에도 대입식을 사용할 수 있지만, 그런 형태의 사용은 피하자

  ~~~python
  # for 문안의 count가 누출됨
  half = [(last := count // 2) for count in stock.values()]
  
  # 아래의 예시처럼 조건식에만 사용하자
  order = ['나사못', '나비너트', '클립']
  found = ((name, batches) for name in order
           if (batches := get_batches(stock.get(name, 0), 8)))
  print(next(found))
  print(next(found))
  ~~~

### 30. 리스트를 반환하기 보다는 제너레이터를 사용하라

- 제너레이터를 사용하면 결과를 리스트에 합쳐서 반환하는 것보다 깔끔하다.

  ~~~python
  # 결과를 리스트에 합쳐서 반환
  def index_words(text):
      result = []
      if text:
          result.append(0)
      for index, letter in enumerate(text):
          if letter == ' ':
              result.append(index + 1)
      return result
  
  # 제너레이터 사용
  def index_words_iter(text):
      if text:
          yield 0
      for index, letter in enumerate(text):
          if letter == ' ':
              yield index + 1
  ~~~

- 제너레이터를 사용하면 작업 메모리에 모든 입력과 출력을 저장할 필요가 없으므로 입력이 아주 커도 출력 시퀀스를 만들 수 있다.

  ~~~python
  def index_file(handle):
      offset = 0
      for line in handle:
          if line:
              yield offset
          for letter in line:
              offset += 1
              if letter == ' ':
                  yield offset
  
  import itertools
  with open('address.txt', 'r', encoding='utf-8') as f:
      it = index_file(f)
      results = itertools.islice(it, 0, 10)
      print(list(results))
  ~~~

### 31. 인자에 대해 이터레이션할 때는 방어적이 돼라

- 입력 인자를 여러 번 이터레이션 함수나 메서드를 조심하라.

  입력받은 인자가 이터레이터면 함수가 이상하게 작동하거나 결과가 없을 수 있다.

  ~~~python
  def read_visits(data_path):
      with open(data_path) as f:
          for line in f:
              yield int(line)
  
  it = read_visits('my_numbers.txt')
  percentages = normalize(it)
  print(percentages)
  
  # 결과
  # []
  ~~~

- 파이썬 이터레이터 프로토콜은 컨테이너와 이터레이터가 iter, next 내장 함수나 for 루프 등의 관련 식과 상호작용하는 절차를 정의한다.

  ~~~python
  class ReadVisits:
      def __init__(self, data_path):
          self.data_path = data_path
  
      def __iter__(self):
          with open(self.data_path) as f:
              for line in f:
                  yield int(line)
  ~~~

- 어떤 값이 이터레이터인지 감지하려면 이 값을 iter 내장 함수에 넘겨서 반환되는 값이 확인하면 된다.

  다른 방법으로 collections.Iterator 클래스를 isinstance와 함께 사용할수 있다.

  ~~~python
  def normalize_defensive(numbers):
      if iter(numbers) is numbers: # 이터레이터 -- 나쁨!
          raise TypeError('컨테이너를 제공해야 합니다')
      total = sum(numbers)
      result = []
      for value in numbers:
          percent = 100 * value / total
          result.append(percent)
      return result
      
  # 아래의 방법을 추천한다.
  from collections.abc import Iterator
  def normalize_defensive(numbers):
      # 반복 가능한 이터레이터인지 검사하는 다른 방법
      if isinstance(numbers, Iterator): 
          raise TypeError('컨테이너를 제공해야 합니다')
      total = sum(numbers)
      result = []
      for value in numbers:
          percent = 100 * value / total
          result.append(percent)
      return result
    
  visits = [15, 35, 80]
  it = iter(visits)
  normalize_defensive(it)
  # TypeError: 컨테이너를 제공해야 합니다 
  ~~~

### 32. 긴 리스트 컴프리헨션보다는 제너레이터 식을 사용하라

- 입력이 크면 메모리를 너무 많이 사용하기 때문에 리스트 컴프리헨션은 문제를 일으킬 수 있다.

- 제너레이터 식은 이터레이처럼 한 번에 원소를 하나씩 출력하기 때문에 메모리 문제를 피할 수 있다.

  ~~~python
  value = [len(x) for x in open('my_file.txt')]
  print(value)
  
  it = (len(x) for x in open('my_file.txt'))
  print(next(it))
  print(next(it))
  ~~~

- 제너레이터 식이 반환한 이터레이터를 다른 제너레이터 식의 하위 식으로 사용함으로써 제너레이터 식을 서로 합성할 수 있다.

  ~~~python
  roots = ((x, x**0.5) for x in it)
  ~~~

### 33. yield from을 사용해 여러 제너레이터를 합성하라

- yield from 식을 사용하면 여러 내장 제너레이터를 모아서 제너레이터 하나로 합성할 수 있다.

  ~~~python
  def move(period, speed):
      for _ in range(period):
          yield speed
  
  def pause(delay):
      for _ in range(delay):
          yield 0
          
  def animate_composed():
      yield from move(4, 5.0)
      yield from pause(3)
      yield from move(2, 3.0)
  ~~~

- 직접 내포된 제너레이터를 이터레이션하면서 각 제너레이터를 출력을 내보내는 것보다 yield from을 사용하는 것이 성능 면에서 더 좋다.

  ~~~python
  import timeit
  
  def child():
      for i in range(1_000_000):
          yield i
  def slow():
      for i in child():
          yield i
  def fast():
      yield from child()
  
  baseline = timeit.timeit(
      stmt='for _ in slow(): pass',
      globals=globals(),
      number=50)
  print(f'수동 내포: {baseline:.2f}s')
  
  comparison = timeit.timeit(
      stmt='for _ in fast(): pass',
      globals=globals(),
      number=50)
  print(f'합성 사용: {comparison:.2f}s')
  
  reduction = -(comparison - baseline) / baseline
  print(f'{reduction:.1%} 시간이 적게 듦')
  
  # 수동 내포: 4.01s
  # 합성 사용: 3.49s
  # 13.0% 시간이 적게 듦
  ~~~

### 34. send로 제너레이터에 데이터를 주입하지 말라

- 합성할 제너레이터들의 입력으로 이터레이터를 전달하는 방식이 send를 사용하는 방식보다 더 낫다.

  send는 가급적 사용하지 말라.

  ~~~python
  def wave_cascading(amplitude_it, steps):
      step_size = 2 * math.pi / steps
      for step in range(steps):
          radians = step * step_size
          fraction = math.sin(radians)
          amplitude = next(amplitude_it) # 다음 입력 받기
          output = amplitude * fraction
          yield output
  
  def complex_wave_cascading(amplitude_it):
      yield from wave_cascading(amplitude_it, 3)
      yield from wave_cascading(amplitude_it, 4)
      yield from wave_cascading(amplitude_it, 5)
  
  def run_cascading():
      amplitudes = [7, 7, 7, 2, 2, 2, 2, 10, 10, 10, 10, 10]
      it = complex_wave_cascading(iter(amplitudes))
      for amplitude in amplitudes:
          output = next(it)
          transmit(output)
  
  run_cascading()
  ~~~

### 35. 제너레이터 안에서 throw 상태로 변화시키지 말라

- throw 메서드를 사용하면 제너레이터가 마지막으로 실행한 yield 식의 위치에서 예외를 다시 발생시킬 수 있다.

  그러나 가독성이 나빠진다.

  ~~~python
  def run():
      it = timer(4)
      while True:
          try:
              if check_for_reset():
                  current = it.throw(Reset())
              else:
                  current = next(it)
          except StopIteration:
              break
          else:
              announce(current)
  ~~~

- 더 나은 방법으로  `__iter__` 메서드를 구현하는 클래스를 사용하면서 예외적인 경우에 상태를 전이시키는 것이다.

  ~~~python
  class Timer:
      def __init__(self, period):
          self.current = period
          self.period = period
  
      def reset(self):
          self.current = self.period
  
      def __iter__(self):
          while self.current:
              self.current -= 1
              yield self.current
              
  def run():
      timer = Timer(4)
      for current in timer:
          if check_for_reset():
              timer.reset()
          announce(current)
  run()
  ~~~

### 36. 이터레이터나 제너레이터를 다룰 때는 itertools를 사용하라

- 여러 데이터 연결하기 

  - chain

    ~~~python
    import itertools
    
    it = itertools.chain([1, 2, 3], [4, 5, 6])
    # [1, 2, 3, 4, 5, 6]
    ~~~

  - repeat

    ~~~python
    it = itertools.repeat('안녕', 3)
    # ['안녕', '안녕', '안녕']
    ~~~

  - cycle

    ~~~python
    it = itertools.cycle([1, 2])
    result = [next(it) for _ in range (10)]
    # [1, 2, 1, 2, 1, 2, 1, 2, 1, 2]
    ~~~

  - tee

    한 이터레이터를 병렬적으로 두 번째 인자로 지정된 개수의 이터레이터를 만들 때 사용한다.

    이 함수로 만들어진 각 이터레이터를 소비하는 속도가 같지 않으면, 처리가 덜 된 이터레이터의 원소를 큐에 담아둬야 하므로 메모리 사용랑이 늘어난다.

    ~~~python
    it1, it2, it3 = itertools.tee(['하나', '둘'], 3)
    # ['하나', '둘']
    # ['하나', '둘']
    # ['하나', '둘']
    ~~~

  - zip_longest

    ~~~python
    keys = ['하나', '둘', '셋']
    values = [1, 2]
    
    normal = list(zip(keys, values))
    # zip: [('하나', 1), ('둘', 2)]
    
    it = itertools.zip_longest(keys, values, fillvalue='없음')
    longest = list(it)
    #zip_longest: [('하나', 1), ('둘', 2), ('셋', '없음')]
    ~~~

- 이터레이터에서 원소 거르기

  - islice

    끝만 지정, 시작과 끝 지정

    시작, 끝, 증가값 지정

    ~~~python
    values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    
    first_five = itertools.islice(values, 5)
    print('앞에서 다섯 개:', list(first_five))
    # 앞에서 다섯 개: [1, 2, 3, 4, 5]
      
    middle_odds = itertools.islice(values, 2, 8, 2)
    print('중간의 홀수들:', list(middle_odds))
    # 중간의 홀수들: [3, 5, 7]
    ~~~

  - takewhile

    주어진 술어가 False를 반환하는 첫 원소가 나타날 때까지 원소를 돌려준다.

    ~~~python
    values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    less_than_seven = lambda x: x < 7
    it = itertools.takewhile(less_than_seven, values)
    print(list(it))
    # [1, 2, 3, 4, 5, 6]
    ~~~

  - dropwhile

    takewhile의 반대

    ~~~python
    values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    less_than_seven = lambda x: x < 7
    it = itertools.dropwhile(less_than_seven, values)
    print(list(it))
    # [7, 8, 9, 10]
    ~~~

  - filterfalse

    filter 내장 함수의 반대

    ~~~python
    values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    evens = lambda x: x % 2 == 0
    
    filter_result = filter(evens, values)
    print('Filter:', list(filter_result))
    # Filter: [2, 4, 6, 8, 10]
      
    filter_false_result = itertools.filterfalse(evens, values)
    print('Filter false:', list(filter_false_result))
    # Filter false: [1, 3, 5, 7, 9]
    ~~~

- 이터레이터에서 원소의 조합 만들어내기

  - accumulate

    ~~~python
    values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    sum_reduce = itertools.accumulate(values)
    print('합계:', list(sum_reduce))
    # 합계: [1, 3, 6, 10, 15, 21, 28, 36, 45, 55]
    
    def sum_modulo_20(first, second):
        output = first + second
        return output % 20
    
    modulo_reduce = itertools.accumulate(values, sum_modulo_20)
    print('20으로 나눈 나머지의 합계:', list(modulo_reduce))
    # 20으로 나눈 나머지의 합계: [1, 3, 6, 10, 15, 1, 8, 16, 5, 15]
    ~~~

  - product

    데카르트의 곱 반환

    ~~~python
    single = itertools.product([1, 2], repeat=2)
    print('리스트 한 개:', list(single))
    # 리스트 한 개: [(1, 1), (1, 2), (2, 1), (2, 2)]
    
    multiple = itertools.product([1, 2], ['a', 'b'])
    print('리스트 두 개:', list(multiple))
    # 리스트 두 개: [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]
    ~~~

  - permutations (순열)

    ~~~python
    it = itertools.permutations([1, 2, 3, 4], 2)
    print(list(it))
    # [(1, 2), (1, 3), (1, 4), (2, 1), (2, 3), (2, 4), (3, 1), (3, 2), (3, 4), (4, 1), (4, 2), (4, 3)]
    ~~~

  - combinations (조합)

    ~~~python
    it = itertools.combinations([1, 2, 3, 4], 2)
    print(list(it))
    # [(1, 2), (1, 3), (1, 4), (2, 3), (2, 4), (3, 4)]
    ~~~

  - combinations_with_replacement (중복조합)

    ~~~python
    it = itertools.combinations_with_replacement([1, 2, 3, 4], 2)
    print(list(it))
    [(1, 1), (1, 2), (1, 3), (1, 4), (2, 2), (2, 3), (2, 4), (3, 3), (3, 4), (4, 4)]
    ~~~

    
