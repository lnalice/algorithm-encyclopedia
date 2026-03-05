# 동적 프로그래밍 (Dynamic Programming, DP)

## 분류
- **카테고리**: 동적 프로그래밍
- **관련 알고리즘**: [분할 정복](../divide-and-conquer/divide-and-conquer.md), [그리디](../greedy/greedy.md), [플로이드-워셜](../shortest-path/floyd-warshall.md), [벨만-포드](../shortest-path/bellman-ford.md)

## 개요
동적 프로그래밍(DP)은 복잡한 문제를 더 작은 하위 문제로 나누어 풀되, **이미 계산한 결과를 저장해 두고 재활용**함으로써 중복 계산을 제거하는 알고리즘 설계 기법이다. DP가 적용되려면 **최적 부분 구조(Optimal Substructure)**와 **중복 부분 문제(Overlapping Subproblems)** 두 가지 성질을 만족해야 한다. 구현 방식은 재귀 + 캐싱 기반의 **메모이제이션(Top-Down)**과 반복문 기반의 **타뷸레이션(Bottom-Up)** 두 가지가 있다.

## 핵심 로직

### 1단계: 상태(State) 정의
- 문제를 하위 문제로 쪼갤 수 있도록 `dp[i]`, `dp[i][j]` 등의 상태를 정의한다.
- 예: 피보나치 → `dp[i] = i번째 피보나치 수`, 배낭 → `dp[i][w] = i번째 물건까지 고려했을 때 용량 w에서의 최대 가치`

### 2단계: 점화식(Recurrence Relation) 도출
- 현재 상태를 이전 상태들의 조합으로 표현하는 식을 세운다.
- 예: `dp[i] = dp[i-1] + dp[i-2]` (피보나치)

### 3단계: 초기값(Base Case) 설정
- 점화식의 시작점이 되는 초기값을 설정한다.
- 예: `dp[0] = 0`, `dp[1] = 1`

### 4단계: 계산 순서 결정 및 구현
- **Top-Down**: 큰 문제에서 시작해 재귀적으로 작은 문제를 호출하며, 메모이제이션으로 중복 방지
- **Bottom-Up**: 작은 문제부터 순서대로 채워 나가며 최종 답을 구함

## DP의 하위 알고리즘
DP는 특정 문제 영역에 점화식을 적용한 다양한 알고리즘의 상위 개념이다:
- **[플로이드-워셜](../shortest-path/floyd-warshall.md)**: 모든 쌍 최단 경로 문제에 DP를 적용. `dp[k][i][j] = 정점 1~k를 경유할 때 i→j 최단 거리`
- **[벨만-포드](../shortest-path/bellman-ford.md)**: 단일 출발점 최단 경로에 DP를 적용. `dp[i][v] = 최대 i개 간선을 사용한 출발점→v 최단 거리`

## 그리디와의 차이점
| 구분 | 동적 프로그래밍 | 그리디 |
|------|----------------|--------|
| **선택 방식** | 모든 선택지를 고려하여 최적해 보장 | 현재 시점에서 최선의 선택만 수행 |
| **최적해 보장** | 항상 보장 | 그리디 선택 속성이 증명된 경우만 보장 |
| **시간 복잡도** | 상대적으로 높음 | 상대적으로 낮음 |
| **적용 조건** | 최적 부분 구조 + 중복 부분 문제 | 최적 부분 구조 + 탐욕 선택 속성 |

> **판단 기준**: 그리디로 풀 수 있으면 그리디가 효율적이다. 하지만 그리디 선택이 전체 최적해를 보장하지 못하면 DP를 사용해야 한다.

## 언제 사용하는가?
- **최적 부분 구조 + 중복 부분 문제**를 가지는 최적화 문제
- **배낭 문제** (0/1 Knapsack, Unbounded Knapsack)
- **최장 공통 부분 수열** (LCS, Longest Common Subsequence)
- **최장 증가 부분 수열** (LIS, Longest Increasing Subsequence)
- **격자 위 경로 수 계산** (좌→우, 상→하 이동)
- **문자열 편집 거리** (Edit Distance)
- **동전 교환 문제** (Coin Change)
- **구간 DP** (행렬 곱셈 순서, 팰린드롬 분할 등)

## Java 구현

```java
public class DynamicProgramming {

    // ========================================
    // 1. 피보나치 수열 - 메모이제이션 (Top-Down)
    // ========================================
    static long[] memo;

    /**
     * 재귀 + 메모이제이션 방식의 피보나치
     * 이미 계산한 값은 memo 배열에 저장하여 재활용한다.
     */
    static long fibMemo(int n) {
        if (n <= 1) return n;
        if (memo[n] != -1) return memo[n]; // 이미 계산된 값이면 바로 반환
        memo[n] = fibMemo(n - 1) + fibMemo(n - 2);
        return memo[n];
    }

    // ========================================
    // 2. 피보나치 수열 - 타뷸레이션 (Bottom-Up)
    // ========================================

    /**
     * 반복문 기반의 피보나치
     * 작은 문제부터 순서대로 테이블을 채워 나간다.
     */
    static long fibTab(int n) {
        if (n <= 1) return n;
        long[] dp = new long[n + 1];
        dp[0] = 0;
        dp[1] = 1;
        for (int i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        return dp[n];
    }

    // ========================================
    // 3. 0/1 배낭 문제 (0/1 Knapsack)
    // ========================================

    /**
     * 각 물건을 넣거나(1) 넣지 않거나(0)를 결정하여 최대 가치를 구한다.
     * dp[i][w] = i번째 물건까지 고려했을 때, 용량 w에서의 최대 가치
     *
     * 점화식:
     *   weight[i] > w  → dp[i][w] = dp[i-1][w]  (현재 물건을 넣을 수 없음)
     *   그 외          → dp[i][w] = max(dp[i-1][w], dp[i-1][w-weight[i]] + value[i])
     */
    static int knapsack(int[] weight, int[] value, int capacity) {
        int n = weight.length;
        int[][] dp = new int[n + 1][capacity + 1];

        for (int i = 1; i <= n; i++) {
            for (int w = 0; w <= capacity; w++) {
                dp[i][w] = dp[i - 1][w]; // 현재 물건을 넣지 않는 경우
                if (weight[i - 1] <= w) {
                    // 현재 물건을 넣는 경우와 비교하여 더 큰 값 선택
                    dp[i][w] = Math.max(dp[i][w],
                            dp[i - 1][w - weight[i - 1]] + value[i - 1]);
                }
            }
        }
        return dp[n][capacity];
    }

    public static void main(String[] args) {
        // 피보나치 - 메모이제이션
        int n = 10;
        memo = new long[n + 1];
        java.util.Arrays.fill(memo, -1);
        System.out.println("피보나치(메모이제이션) F(" + n + ") = " + fibMemo(n));

        // 피보나치 - 타뷸레이션
        System.out.println("피보나치(타뷸레이션) F(" + n + ") = " + fibTab(n));

        // 0/1 배낭 문제
        int[] weight = {2, 3, 4, 5};   // 각 물건의 무게
        int[] value = {3, 4, 5, 6};    // 각 물건의 가치
        int capacity = 5;              // 배낭의 최대 용량
        System.out.println("0/1 배낭 최대 가치 = " + knapsack(weight, value, capacity));
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 알고리즘 | 시간 복잡도 | 공간 복잡도 |
|---------|-----------|-----------|
| 피보나치 (메모이제이션) | \(O(N)\) | \(O(N)\) — 메모 배열 + 재귀 스택 |
| 피보나치 (타뷸레이션) | \(O(N)\) | \(O(N)\) — dp 배열 (최적화 시 \(O(1)\)) |
| 0/1 배낭 | \(O(N \times W)\) | \(O(N \times W)\) — dp 테이블 (1차원 최적화 시 \(O(W)\)) |

> 일반적으로 DP의 복잡도 = **(상태의 수) × (각 상태에서의 전이 비용)**

## 관련 문제 유형
- **피보나치형**: 계단 오르기, 타일링 문제
- **배낭형**: 0/1 배낭, 완전 배낭, 부분합 문제
- **LCS/LIS**: 최장 공통 부분 수열, 최장 증가 부분 수열
- **격자 경로**: 격자 위 최소 비용 경로, 경로 수 세기
- **구간 DP**: 행렬 곱셈 순서, 팰린드롬 분할
- **트리 DP**: 트리 위에서의 최적화 문제
- **비트마스크 DP**: 외판원 문제(TSP), 집합 상태를 비트로 표현하는 문제
