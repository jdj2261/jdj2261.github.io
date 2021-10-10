---
layout: post
title: Effective Python 2nd - 파이썬답게 생각하기 
date: 2021-10-10 15:39:00 +09:00
category: Python
---

Effective Python 2nd 책을 읽으면서 정리하려고 합니다.

[길벗 Github](https://github.com/gilbutITbook/080235)와 [저자 Github](https://github.com/bslatkin/effectivepython)를 통해 새로 알게된 내용, 중요하다고 생각되는 것 위주로 정리하고자 합니다.

# Effective Python 2nd

## Chapter 1. 파이썬답게 생각하기

### 2. PEP 8 스타일 가이드를 따르라

- 공백

  - 탭 대신 스페이스를 사용해 들여쓰기
  - 문법적으로 중요한 들여쓰기에는 4칸 스페이스를 사용하기
  - 라인 길이는 79개 문자 이하
  - 파일 안 각 함수와 클래스 사이에는 빈 줄을 두 줄 넣기
  - 클래스 안 메서드와 메서드 사이에는 빈 줄을 한 줄 넣기
  - 딕셔너리에서 키와 콜론 사이에 공백을 넣지 않고 한 줄 안에 키와 값을 넣는 경우 콜론 다음에 스페이스를 하나 넣기
  - 변수 대입에서 = 전후에는 스페이스를 하나씩 넣기

- 명명 규약

  - 함수, 변수, 애트리뷰트는 lowercase_underscore처럼 소문자와 밑줄 사용하기
    - snake case
  - 보호돼야 하는 애트리뷰트는 _leading_underscore처럼 밑줄로 시작하기
  - 비공개(private) 인스턴스 애트리뷰트는 __leading_underscore처럼 밑줄 두개로 시작하기
  - 클래스는 CapitalizedWord처럼 여러 단어를 이어 붙이되, 각 단어의 첫 글자를 대문자로 만들기
    - camel case
  - 모듈 수준의 상수는 ALL_CAPS처럼 모든 글자를 대문자로 하고 단어와 단어 사이를 밑줄로 연결하기

- 식과 문

  - 긍정인 식을 부정하지 말고(if not a is b) 부정을 내부에 넣기(if a is not b)
  - []나 ' '등을 검사할 때 길이를 0과 비교하지 말고 'if not 컨테이너'라는 조건문 쓰기
  - 한 줄짜리 if문, for, while, except 복합문을 사용하지 말기
  - 여러 줄에 걸쳐 식을 쓸 때 \ 문자보다는 괄호 사용하기

- 임포트

  - import 문은 항상 파일 맨 앞에 위치시키기

  - 절대적인 이름을 사용하고 현 모듈의 상대적인 이름은 사용하지 말기

    - from bar import foo 라고 해야 하며, import foo 라고 하면 안된다.

  - 반드시 상대적인 경로로 임포트해야 하는 경우에는 from . import foo 처럼 명시적인 구문 사용하기

  - 표준 라이브러리 모듈, 서드 파티 모듈, 내가 만든 모듈 순으로 섹션을 나누기

    각 섹션은 알파벳 순서로 임포트하기

### 3. bytes와 str의 차이를 알아두라

- bytes에는 8비트 값의 시퀀스가 들어있고, str에는 유니코드 코드 포인트의 시퀀스가 들어 있다.
- bytes와 str 인스턴스를 연산자에 섞어서 사용할 수 없다.
- 이진 데이트를 파일에서 읽거나 쓰고 싶으면 항상 이진 모드('rb'나 'wb')로 열자.

### 4. C 스타일의 형식 문자열을 str.format과 쓰기보다는 f-문자열을 통한 인터폴레이션을 사용하라.

~~~python
places = 3
number = 1.23456
print(f'내가 고른 숫자는 {number:.{places}f}')
~~~

### 5. 복잡한 식을 쓰는 대신 도우미 함수를 작성하라

- 아래의 코드로 표현하는 것은 파이썬의 장점이지만 이해하기가 힘들다.

  ~~~python
  # 질의 문자열이 `빨강=5&파랑=0&초록='인 경우
  red = my_values.get('빨강', [''])[0] or 0
  green = my_values.get('파랑', [''])[0] or 0
  opacity = my_values.get('투명도', [''])[0] or 0
  print(f'빨강: {red!r}')
  print(f'초록: {green!r}')
  print(f'투명도: {opacity!r}')
  ~~~

- 아래의 코드와 같이 이해하기 쉬운 도우미 함수를 작성하자

  ~~~python
  def get_first_int(values, key, default=0):
      found = values.get(key, [''])
      if found[0]:
          return int(found[0])
      return default
  
  green = get_first_int(my_values, '초록')
  ~~~

### 6. 인덱스를 사용하는 대신 대입을 사용해 데이터를 언패킹하라

- 아래처럼 작성하는 것보다

  ~~~python
  snacks = [('베이컨', 350), ('도넛', 240), ('머핀', 190)]
  for i in range(len(snacks)):
      item = snacks[i]
      name = item[0]
      calories = item[1]
      print(f'#{i+1}: {name} 은 {calories} 칼로리입니다.')
  ~~~

- 이렇게 작성하는게 훨씬 더 파이썬스럽다.

  ~~~python
  snacks = [('베이컨', 350), ('도넛', 240), ('머핀', 190)]
  for rank, (name, calories) in enumerate(snacks, 1):
      print(f'#{rank}: {name} 은 {calories} 칼로리입니다.')
  ~~~

### 7. range보다는 enumerate를 사용하라

- 아래처럼 작성하는 것보다

  ~~~python
  flavor_list = ['바닐라', '초콜릿', '피칸', '딸기']
  
  for i in range(len(flavor_list)):
      flavor = flavor_list[i]
      print(f'{i + 1}: {flavor}')
  ~~~

- 이렇게 작성하는게 훨씬 더 파이썬스럽다.

  ~~~python
  flavor_list = ['바닐라', '초콜릿', '피칸', '딸기']
  
  for i in range(len(flavor_list)):
      flavor = flavor_list[i]
      print(f'{i + 1}: {flavor}')
  
  for i, flavor in enumerate(flavor_list):
      print(f'{i + 1}: {flavor}')
  
  # 어디서부터 원소를 가져오기 시작할지 지정할 수 있다.
  # 디폴트 값은 0
  for i, flavor in enumerate(flavor_list, 1):
      print(f'{i}: {flavor}')
  ~~~

### 8. 여러 이터레이터에 대해 나란히 루프를 수행하려면 zip을 사용하라

- 아래처럼 작성하는 것보다

  ~~~python
  longest_name = None
  max_count = 0
  
  for i, name in enumerate(names):
      count = counts[i]
      if count > max_count:
          longest_name = name
          max_count = count
  ~~~

- 이렇게 작성하는게 훨씬 더 파이썬스럽다.

  zip에 전달한 리스트의 길이가 같지 않을 것으로 예상한다면 itertools의 zip_longest를 사용하자.

  ~~~python
  longest_name = None
  max_count = 0
  
  for name, count in zip(names, counts):
      if count > max_count:
          longest_name = name
          max_count = count
  print(longest_name)
  
  names.append('Rosalind')
  import itertools
  for name, count in itertools.zip_longest(names, counts):
      print(f'{name}: {count}')
  ~~~

### 9. for나 while 루프 뒤에 else 블록을 사용하지 말라

- 이렇게 작성하지 말라는 것이다

  ~~~python
  for i in range(3):
      print('Loop', i)
  else:
      print('Else block!')
  ~~~

- 예를 들어,서로소를 비교하는 함수를 만들 때 아래처럼 이해하기 쉽게 작성하자

  ~~~python
  a = 4
  b = 9
  def coprime(a, b):
      for i in range(2, min(a, b) + 1):
          if a % i == 0 and b % i == 0:
              return False
      return True
  assert coprime(4, 9)
  assert not coprime(3, 6)
  
  def coprime_alternate(a, b):
      is_coprime = True
      for i in range(2, min(a, b) + 1):
          if a % i == 0 and b % i == 0:
              is_coprime = False
              break
      return is_coprime
  
  assert coprime_alternate(4, 9)
  assert not coprime_alternate(3, 6)
  ~~~

### 10. 대입식을 사용해 반복을 피하라

- 왈러스 연산자를 사용하자

  ~~~python
  fresh_fruit['레몬'] = 5  # 테스트를 위해 갯수 리셋
  if count := fresh_fruit.get('lemon', 0):
      make_lemonade(count)
  else:
      out_of_stock()
  ~~~

- switch/case 문 같은 우아한 해법을 만들 수 있다.

  ~~~python
  if (count := fresh_fruit.get('바나나', 0)) >= 2:
      pieces = slice_bananas(count)
      to_enjoy = make_smoothies(pieces)
  elif (count := fresh_fruit.get('사과', 0)) >= 4:
      to_enjoy = make_cider(count)
  elif count := fresh_fruit.get('레몬', 0):
      to_enjoy = make_lemonade(count)
  else:
      to_enjoy = '아무것도 없음'
  ~~~

- 무한 루프 중간에서 끝내기 관용어의 필요성이 줄어든다.

  이 관용어를 사용하면 코드 반복을 없앨 수 있지만, while 루프의 유용성이 줄어든다.

  ~~~python
  bottles = []
  while True: # 무한루프
      fresh_fruit = pick_fruit()
      if not fresh_fruit: # 중간에서 끝내기
          break
  
      for fruit, count in fresh_fruit.items():
          batch = make_juice(fruit, count)
          bottles.extend(batch)
  print(bottles)
  
  bottles = []
  while fresh_fruit := pick_fruit():
      for fruit, count in fresh_fruit.items():
          batch = make_juice(fruit, count)
          bottles.extend(batch)
  print(bottles)
  ~~~

  
