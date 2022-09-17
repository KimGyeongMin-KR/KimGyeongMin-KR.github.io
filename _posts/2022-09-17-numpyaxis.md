---
title : numpy axis 쉽게 이해하기!
# use_math : true
tags : python numpy ai machine-learning
category : [machine-learning, basic]
---

numpy axis 축 쉽게 이해하기
====
## 이것만 기억하면 끝!
- <span style = "color:yellow; font-size: 2em">괄호 바깥에서 안으로!</span>

또는
- <span style = "color:yellow">열(col)은 결국 맨 뒤다</span>

## 2차원 배열 예시
<div class='col-12 text-center' style='font-size:1.5em;'>------여기부터 꼭 보세요-------</div>
2차원 배열을 예로 들겠다.

```
[
[1,2,3],
[4,5,6],
[7,8,9]
]
```

껍데기를 바깥에서부터 빼면


``` 
[1,2,3] 
[4,5,6] 
[7,8,9]
```
행 단위로 나뉜 것을 볼 수 있다.

**axis 0축의 이동 방향**은 `[1,2,3] -> [4,5,6] -> [7,8,9]`

여기서 또 바깥 괄호를 빼면
```
1 2 3
4 5 6
7 8 9
```
열 단위로 나뉜 것을 볼 수 있다.

**axis 1축의 이동 방향**은 `1->2->3 or 4->5->6 or 7->8->9`

이 배열의 모양은 2차원으로

`(axis0, axis1)`으로 나타낼 수 있고

`(0축= 행, 1축= 열)` 이므로

`(3,3)`이다.
<div class='col-12 text-center' style='font-size:1.5em;'>---------이해하셨으면 스킵!---------</div>



<!-- ![image.png](attachment:image.png) -->
![2차원 예시](/images/axis-2d-ex-1.png)

## 3차원 예시
[
    [
        [1,2],
        [3,4]
    ],
    [
        [5,6],
        [7,8]
    ],
    [
        [9,10],
        [11,12]
    ]
]


그렇다면 이 배열의 모양은 어떻게 될까

axis 0 축의 이동방향은 `[[1, 2], [3, 4]] -> [[5, 6], [7, 8]] -> [[9, 10], [11, 12]]`

axis 0 축의 수는 3

axis 1 축의 이동방향은 `[1, 2] -> [3, 4] or [5, 6] -> [7, 8] or [9, 10] -> [11, 12]`

axis 1 축의 수는 2

axis 2 축의 이동방향은 `1 -> 2 or 3-> 4 ,,, or 11-> 12`

axis 2 축의 수는 2

그러므로 모양은 `(3,2,2)`이다.

(0축, 1축, 2축) -> (높이, 행, 열) -> (3,2,2)

## 발그림
<!-- ![image-3.png](attachment:image-3.png) -->
![3차원 예시](/images/axis-3d-ex-1.png)


