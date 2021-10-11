---
layout: post
title: Effective Python 2nd - 리스트와 딕셔너리
date: 2021-10-11 23:02:00 +09:00
category: Python
---

Chapter 2. 리스트와 딕셔너리를 읽으면서 정리합니다.

새로 알게 되었거나 중요하다고 생각되는 부분 위주로 정리하고자 합니다.

# Effective Python 2nd

## Chapter 2. 리스트와 딕셔너리

### 13. 슬라이싱보다는 나머지를 모두 잡아내는 언패킹을 사용하라

- 언패킹 대입에 별표 식을 사용하면 언패킹 패턴에 대입되지 않는 모든 부분을 리스트에 잡아낼 수 있다.

  ~~~python
  car_ages = [0, 9, 4, 8, 7, 20, 19, 1, 6, 15]
  car_ages_descending = sorted(car_ages, reverse=True)
  oldest, second_oldest, *others = car_ages_descending
  print(oldest, second_oldest, others)
  ~~~

  

- 리스트를 서로 겹치지 않게 여러 조각으로 나눌 경우, 슬라이싱과 인덱싱을 사용하기보다는 나머지를 모드 잡아내는 언패킹을 사용해야 실수할 여지가 훨씬 줄어든다.

  ~~~python
  # 아래처럼이 아니라
  all_csv_rows = list(generate_csv())
  header = all_csv_rows[0]
  rows = all_csv_rows[1:]
  print('CSV 헤더:', header)
  print('행 수: ', len(rows))
  
  # 이렇게 구현하자
  it = generate_csv()
  header, *rows = it
  print('CSV 헤더:', header)
  print('행 수: ', len(rows))
  ~~~

### 14. 복잡한 기준을 사용해 정렬할 때는 key 파라미터를 사용하라

- sort 메서드의 key 파라미터를 사용하면 리스트의 각 원소 대신 비교에 사용할 객체를 반환하는 도우미 함수를 제공할 수 있다.

  ~~~python
  class Tool:
      def __init__(self, name, weight):
          self.name = name
          self.weight = weight
  
      def __repr__(self):
          return f'Tool({self.name!r}, {self.weight})'
  
  tools = [
      Tool('수준계', 3.5),
      Tool('해머', 1.25),
      Tool('스크류드라이버', 0.5),
      Tool('끌', 0.25),
  ]
  
  print('미정렬:', repr(tools))
  tools.sort(key=lambda x: x.name)
  print('\n정렬: ', tools)
  ~~~

- key 함수에서 튜플을 반환하면 여러 정렬 기준을 하나로 엮을 수 있다.

  단항 부호 반전 연산자를 사요하면 정렬 순서를 반대로 바꿀 수 있다.

  ~~~python
  power_tools.sort(key=lambda x: (-x.weight, x.name))
  print(power_tools)
  ~~~

- 부호를 바꿀 수 없는 타입의 경우 reverse 값으로 정렬 순서를 지정하면서 sort 메서드를 여러 번 사용해야 한다.

  ~~~python
  power_tools.sort(key=lambda x: x.name)   # name 기준 오름차순
  power_tools.sort(key=lambda x: x.weight, # weight 기준 내림차순
                   reverse=True)
  ~~~

### 15. 딕셔너리 삽입 순서에 의존할 때는 조심하라

- python 3.7부터는 키를 삽입한 순서대로 돌려받는다.

- 딕셔너리와 비슷한 객체를 쉽게 만들 수 있다.

  ~~~python
  from collections.abc import MutableMapping
  
  # 알파벳 순으로 표시해야 할 때
  class SortedDict(MutableMapping):
      def __init__(self):
          self.data = {}
  
      def __getitem__(self, key):
          return self.data[key]
  
      def __setitem__(self, key, value):
          self.data[key] = value
  
      def __delitem__(self, key):
          del self.data[key]
  
      def __iter__(self):
          keys = list(self.data.keys())
          keys.sort()
          for key in keys:
              yield key
  
      def __len__(self):
          return len(self.data)
  ~~~

- 딕셔너리와 비슷한 클래스를 조심스럽게 다루는 방법으로

  - dict 인스턴스의 삽입 순서 보존에 의존하지 않고 코드를 작성하는 방법

    ~~~python
    def get_winner(ranks):
        for name, rank in ranks.items():
            if rank == 1:
                return name
    ~~~

    

  - 실행 시점에 명시적으로 dict 타입 검사하는 방법

    ~~~python
    def get_winner(ranks):
        if not isinstance(ranks, dict):
            raise TypeError('dict 인스턴스가 필요합니다')
        return next(iter(ranks))
    ~~~

    

  - 타입 애너테이션과 정적 분석을 사용해 dict 값을 요구하는 방법이 있다.

    ~~~python
    # example.py
    from typing import Dict, MutableMapping
    
    def populate_ranks(votes: Dict[str, int],
                       ranks: Dict[str, int]) -> None:
        names = list(votes.keys())
        names.sort(key=votes.get, reverse=True)
        for i, name in enumerate(names, 1):
            ranks[name] = i
    
    def get_winner(ranks: Dict[str, int]) -> str:
        return next(iter(ranks))
    
    class SortedDict(MutableMapping[str, int]):
        def __init__(self):
            self.data = {}
    
        def __getitem__(self, key):
            return self.data[key]
    
        def __setitem__(self, key, value):
            self.data[key] = value
    
        def __delitem__(self, key):
            del self.data[key]
    
        def __iter__(self):
            keys = list(self.data.keys())
            keys.sort()
            for key in keys:
                yield key
    
        def __len__(self):
            return len(self.data)
    ~~~

    - mypy 도구를 엄격한 모드로 사용한다

      ~~~shell
      $ python -m mypy --strict example.py
      ~~~

### 16. in을 사용하고 딕셔너리 키를 없을 때 KeyError를 처리하기보다는 get을 사용하라

- 기존에는 아래처럼 사용했을 것이다.

  ~~~python
  counters = {
      '품퍼니켈': 2,
      '사워도우': 1,
  }
  key = '밀'
  
  # 첫 번째 방법
  if key in counters:
      count = counters[key]
  else:
      count = 0
  counters[key] = count + 1
  # 두 번째 방법 (더 효율적)
  try:
      count = counters[key]
  except KeyError:
      count = 0
  counters[key] = count + 1
  ~~~

- get 메서드를 이용하자

  ~~~python
  count = counters.get(key, 0)
  counters[key] = count + 1
  ~~~

  ~~~python
  if (names := votes.get(key)) is None:
      votes[key] = names = []
  ~~~

> Note. 카운터로 이뤄진 딕셔너리를 유지해야 하는 경우 collections.Counter 클래스를 고려하자.

### 17. 내부 상태에서 원소가 없는 경우를 처리할 때는 setdefault보다 defaultdict를 사용하라

- 키로 어떤 값이 들어올지 모르는 딕셔너리를 관리해야 할 때 defaultdict를 사용하라

  ~~~python
  class Visits:
      def __init__(self):
          self.data = defaultdict(set)
  
      def add(self, country, city):
          self.data[country].add(city)
  
  visits = Visits()
  visits.add('영국', '바스')
  visits.add('영국', '런던')
  print(visits.data)
  ~~~

- setdefault가 더 짧은 코드를 만들어내는 경우 setdefault를 사용하는 것도 고려해볼 만하다.

  ~~~python
  visits = {
      '미국': {'뉴욕', '로스엔젤레스'},
      '일본': {'하코네'},
  }
  visits.setdefault('프랑스', set()).add('칸') # 짧다
  
  if (japan := visits.get('일본')) is None: # 길다
      visits['일본'] = japan = set()
  japan.add('교토')
  
  print(visits)
  ~~~

### 18. __ missing __을 사용해 키에 따라 다른 디폴트 값을 생성하는 방법을 알아두라

- 디폴트 값을 만드는 계산 비용이 높거나 예외가 발생할 수 있는 상황에서는 setdefault 메서드를 사용하지 말라.

- defaultdict에 전달되는 함수는 인자를 받지 않으므로 접근에 사용한 키 값에 맞는 디폴트 값을 생성하는 것을 불가능하다.

- 디폴트 키를 만들 때 어떤 키를 사용했는지 반드시 알아야 하는 상황이라면 직접 dict의 하위 클래스와 __ missing __ 메서드를 정의하면 된다.

  ~~~python
  class Pictures(dict):
      def __missing__(self, key):
          value = open_picture(key)
          self[key] = value
          return value
  
  pictures = Pictures()
  handle = pictures[path]
  handle.seek(0)
  image_data = handle.read()
  ~~~

  
