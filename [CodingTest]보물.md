# 보물

[문제 바로가기](https://www.acmicpc.net/problem/1026)

`그리디`

## 문제접근

식: `S = A[0] × B[0] + ... + A[N-1] × B[N-1]`

두 배열 중 A 배열의 원소를 재배치하여 두 식을 이용해 최소값을 구하는 것이다.   
A의 최댓값과 B의 최솟값을 서로 곱하면 결국 두 배열 곱의 합에 대한 최솟값을 구할 수 있다고 판단했다.   

따라서, 아래와 같이 문제를 풀었다.

1. A배열을 오름차순으로 정렬한다.
2. B배열을 내림차순으로 정렬한다.
3. 두 배열의 각 인덱스에 해당하는 원소를 곱한다.
4. 3번의 결과를 모두 합한다.

## 문제풀이

```javascript
    let fs = require('fs');
    let input = fs.readFileSync('/dev/stdin/').toString().split('\n');

    let count = input[0];
    let numbers = [];
    let sum = 0;
    let [A, B] = [[], []]

    for (let i = 1; i < input.length; i++) {
    if (input[i] !== '') {
        numbers.push(input[i].split(' '));
    }
    }

    for (let i = 0; i < numbers.length; i++){
        A = numbers[0].sort((a, b) => a-b)
        B = numbers[1].sort((a, b) => b-a)
    }

    for(let i=0; i<count; i++){
        sum += (A[i] * B[i])
    }
    console.log(sum)
```
