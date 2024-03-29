# 5일차 학습일지 (알고리즘 데이!)

## 백준 알고리즘 Class1, Class2 풀이

## 알고리즘 유형 정리

### 슬라이딩 윈도우 알고리즘

### 두 포인터 알고리즘

### DP(다이나믹 프로그래밍) 알고리즘
> 동적 계획법은 "어떤 문제를 풀기 위해 그 문제를 더 작은 문제의 연장선으로 생각하고, 과거에 구한 해를 활용하는" 방식의 알고리즘을 총칭한다.
  - 백준 10870번 피보나치 수 5
      - 피보나치 수 란?
      > 첫째 및 둘째 항이 1이며 그 뒤의 모든 항은 바로 앞 두 항의 합인 수열이다. 처음 여섯 항은 각각 1, 1, 2, 3, 5, 8이다. 편의상 0번째 항을 0으로 두기도 한다.
      - n이 주어졌을 때, n번째 피보나치 수를 구하는 프로그램을 작성하시오.
      ```java
      // 핵심 로직
      int[] dp = new int[N+1];
        dp[0] = 0;
        dp[1] = 1;
        for (int i = 2; i < dp.length; i ++) {
            dp[i] = dp[i-2] + dp[i-1];
        }
      ```
      - 이 문제에서는 첫번째 항이 0으로 주어진다.
      - 0번쨰와 첫번째를 0과 1로 고정을 시킨 후 두번째 항 부터 과거에 구한 값을 활용하여, 해당 항 의 값을 구한 후 배열에 저장하고, n번째 수를 출력한다.

### 디그리 알고리즘

### 에라토스테네스의 체
> 수학에서 에라토스테네스의 체는 소수를 찾는 방법이다. 고대 그리스 수학자 에라토스테네스가 발견하였다.
  - 백준 1929번 소수 구하기 문제
      - M이상 N이하의 소수를 모두 출력하는 프로그램을 작성하시오.
      - 첫째줄에 M과 N이 주어지고 그 사이의 소수를 출력
      ```java
      // 핵심 로직
      boolean[] arr = new boolean[max + 1];
              arr[0] = true;
              arr[1] = true;
              for (int i = 2; i <= Math.sqrt(max) ; i++) {
                  if (arr[i]) continue;
                  for (int j = i * i; j < arr.length; j += i) {
                      arr[j] = true;
                  }
              }
      ```
      - N값의 제곱근을 구하여 그 수보다 작거나 같을떄까지 i값을 증가시켜 모든 i값의배수 를 ture값으로 boolean리스트에 저장 시킨 후
    false값을 출력하면 배수가 없는수 만 출력됨
