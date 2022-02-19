---
layout: post
title:  "Data Preprocessing (1) - Numpy ndarray (1)"
date:   2022-02-19 15:43:01 +0900
---
Numpy ndarray 살펴보기 (1) : 만들기, 데이터타입, 형변환, reshape


---
<b>Numpy ndarray</b><span style="color:blue"><sup> N차원(Dimenstion) Array</sup></span>

>내용
>​    ndarry 만들기
>
>​    특정타입 ndarry 만들기
>
>​    형 변환
>
>​    reshape

<br>
<b>데이터 차원</b>

    scalar : 상수값 - 예) 10, 3.14, 'home'
    vector : 1차원 - 예) [10, 20, 30]
    matrix : 2차원 - 예) [  [10, 20, 30], [100, 200, 300] ]   2x3(2행3열)
    3차원 - 예) [ [ [10, 20, 30], [100, 200, 300] ], [ [ 40, 50, 60], [400, 500, 600] ] ]  2x2x3

<br>
<b>Numpy ndarray 만들기</b>

```python
import numpy as np
```

아래와 같이 numpy로 array를 만든다.

```python
arr = np.array( [10,20,30] )
print(arr)
print(type(arr))
arr
```

```markdown
[10 20 30]
<class 'numpy.ndarray'>
array([10, 20, 30])
```

np.array로 만든 변수의 type을 확인해 보면 <class 'numpy.ndarray'> 로 확인된다.

numpy ndarray를 만들었다.



비교를 위해서 numpy array가 아닌 파이썬의 일반 리스트로 변수를 만들어서 type을 확인하면 아래와 같다.

```python
my = [10,20,30]
print(my)
print(type(my))
```

```bash
[10, 20, 30]
<class 'list'>
```

type이 <class 'list'>로 확인된다. 일반 리스트 타입이다.

numpy arrary는 데이터 타입이 ndarray로 설정됨을 기억하자.



2차원 ndarray를 만들어 보자. 아래와 같이 하면 된다.

```python
arr2 = np.array([ [10,20,30], [100,200,300] ])
print(type(arr2))
arr2
```

```bash
<class 'numpy.ndarray'>
array([[ 10,  20,  30],
       [100, 200, 300]])
```

반환하는 값은 ndarray 라는 type의 object 임을 기억하자.


<br>
<b>특정 타입 ndarray</b>

아래와 같이 특정 type의 ndarray를 만들 수 있다.

```python
# 데이터타입(dtype) int32인 ndarray 만들기
arr3 = np.int32([10,20,30])
arr3
```

```bash
array([10, 20, 30], dtype=int32)
```



```python
# dtype float32인 ndarray 만들기

arr4 = np.float32([10,20,30])
arr4
```

```bash
array([10., 20., 30.], dtype=float32)
```



데이터 타입을 지정하지 않고, ndarray를 만들면 dtype int64로 만들어진다.

```python
arr = np.array([11,22,33,44,55])
arr.type
```

```bash
dtype('int64')
```


<br>
<b>형 변환</b>

astype() 멤버 함수로 데이터의 type을 변경 할 수 있다.

```python
arr1 = arr.astype(np.float32)
arr1
```

```bash
array([11., 22., 33., 44., 55.], dtype=float32)
```


<br>
<b>Reshape</b>

reshape() 함수로 데이터의 행,열을 재배치 할 수 있다.

```python
arr2 = np.array([1,2,3,4,5,6])
arr2
```

```bash
array([1, 2, 3, 4, 5, 6])
```



위에 보이는 arr2는 1차원 데이터인데, 이를 아래와 같이 reshape() 해서 2차원 데이터로 만들 수 있다.

```python
arr2.reshape(2,3)
```

```bash
array([[1, 2, 3],
       [4, 5, 6]])
```



reshape()에 -1값을 넣으면 자동으로 계산하라는 의미가 된다.

아래 예제에서 reshape(-1,3)은 열을 3개로 하고, 행은 자동으로 맞추라는 의미가 된다.

```python
arr2.reshape(-1,3)
```

```bash
array([[1, 2, 3],
       [4, 5, 6]])
```

```python
arr2.reshape(3,-1)
```

```bash
array([[1, 2],
       [3, 4],
       [5, 6]])
```



3차원 데이터로의 변환도 가능한다.

```python
arr2.reshape(-1,2,3)
```

```bash
array([[[1, 2, 3],
        [4, 5, 6]]])
```



reshape(-1)은 현재 데이터의 차수와 상관없이 모든 데이터를 1차원으로 풀어 놓으라는 의미가 된다.

```python
arr3 = np.array([[10,20,30],[100,200,300]])
arr3
```

```bash
array([[ 10,  20,  30],
       [100, 200, 300]])
```

```python
arr3.shape
```

```bash
(2, 3)
```

```python
arr3.reshape(-1)
```

```bash
array([ 10,  20,  30, 100, 200, 300])
```

