# 비트마스크 DP (Bitmask DP)

## 분류
- **카테고리**: 동적 프로그래밍
- **관련 알고리즘**: [DP](./dp.md), [백트래킹](../backtracking/backtracking.md)

## 개요
비트마스크 DP는 **집합의 상태를 비트(bit)로 표현**하여 DP의 상태로 활용하는 기법이다. N개의 원소로 이루어진 집합의 부분집합을 정수 하나(0 ~ 2^N - 1)로 표현할 수 있어, 모든 부분집합 상태를 효율적으로 관리할 수 있다. N이 작은(보통 N ≤ 20) 완전 탐색 문제에서 **같은 상태를 중복 방문하는 것을 방지**하여 백트래킹보다 효율적으로 풀 수 있다.

## 핵심 로직

### 비트 연산 기초

| 연산 | 코드 | 의미 |
|------|------|------|
| i번째 비트 켜기 | `mask \| (1 << i)` | 집합에 원소 i를 추가 |
| i번째 비트 끄기 | `mask & ~(1 << i)` | 집합에서 원소 i를 제거 |
| i번째 비트 확인 | `mask & (1 << i)` | 원소 i가 집합에 있는지 확인 (0이면 없음, 양수면 있음) |
| 전체 집합 | `(1 << N) - 1` | N개 원소가 모두 포함된 집합 |
| 공집합 | `0` | 아무 원소도 없는 상태 |
| 원소 개수 | `Integer.bitCount(mask)` | 집합에 포함된 원소의 수 |

### 비트마스크 DP의 동작 원리

1. **상태 정의**: `dp[mask][i]` = `mask`에 해당하는 원소들을 방문한 상태에서, 현재 위치가 `i`일 때의 최적값
2. **전이**: `mask`에서 아직 방문하지 않은 원소 `j`를 방문하여 새로운 상태 `mask | (1 << j)`로 전이
3. **기저 조건**: 모든 원소를 방문한 상태(`mask == (1 << N) - 1`)에서 값을 반환
4. **결과**: 모든 가능한 시작/종료 상태에서의 최적값

### 백트래킹 vs 비트마스크 DP

| 구분 | 백트래킹 | 비트마스크 DP |
|------|---------|-------------|
| **중복 상태** | 같은 상태를 여러 번 계산 | 한 번 계산한 상태를 저장하여 재활용 |
| **시간 복잡도** | O(N!) (최악) | O(2^N × N) 또는 O(2^N × N²) |
| **적용 조건** | N이 매우 작을 때 (N ≤ 12) | N이 작을 때 (N ≤ 20) |
| **핵심 차이** | 순서가 다르면 다른 탐색 경로 | 방문 **집합**이 같으면 동일 상태로 처리 |

> **예시**: 4개 도시 A→B→C→D를 방문한 상태와 A→C→B→D를 방문한 상태는, 둘 다 "A, B, C, D를 모두 방문하고 현재 D에 있음"이라는 동일한 상태다. 백트래킹은 이 두 경로를 각각 탐색하지만, 비트마스크 DP는 한 번만 계산한다.

## 언제 사용하는가?
- **N이 작은(N ≤ 20) 완전 탐색 + DP 문제**
- **외판원 문제 (TSP)**: N개 도시를 모두 방문하고 출발점으로 돌아오는 최소 비용
- **작업 배정 문제**: N개의 작업을 N명에게 배정할 때 최소 비용
- **집합 분할 문제**: 집합을 특정 조건에 맞게 분할
- 방문한 원소들의 **집합** 상태에 따라 결과가 달라지는 모든 문제
- 순열 탐색에서 **이미 사용한 원소의 조합**만 중요하고 순서는 무관한 경우

## Java 구현

```java
import java.util.*;

public class BitmaskDP {

    static final int INF = Integer.MAX_VALUE / 2;
    static int N;           // 도시의 수
    static int[][] cost;    // cost[i][j] = 도시 i에서 j로 가는 비용
    static int[][] dp;      // dp[mask][i] = mask 상태에서 현재 i에 있을 때의 최소 비용

    // ========================================
    // 외판원 문제 (TSP) - 비트마스크 DP
    // 모든 도시를 한 번씩 방문하고 출발점으로 돌아오는 최소 비용
    //
    // 상태: dp[visited][current]
    //   visited = 방문한 도시들의 비트마스크
    //   current = 현재 위치한 도시
    //   값 = 이 상태에서 남은 도시를 모두 방문하고 출발점으로 돌아가는 최소 비용
    // ========================================
    static int tsp(int mask, int current) {
        // 모든 도시를 방문한 경우 → 출발점(0번)으로 돌아가는 비용
        if (mask == (1 << N) - 1) {
            return cost[current][0] != 0 ? cost[current][0] : INF;
        }

        // 이미 계산한 상태면 저장된 값 반환 (메모이제이션)
        if (dp[mask][current] != -1) {
            return dp[mask][current];
        }

        dp[mask][current] = INF;

        // 아직 방문하지 않은 도시를 하나씩 시도
        for (int next = 0; next < N; next++) {
            // next 도시를 아직 방문하지 않았고, 갈 수 있는 경우
            if ((mask & (1 << next)) == 0 && cost[current][next] != 0) {
                int newMask = mask | (1 << next); // next 도시를 방문 처리
                int result = tsp(newMask, next);

                if (result != INF) {
                    dp[mask][current] = Math.min(
                        dp[mask][current],
                        cost[current][next] + result
                    );
                }
            }
        }

        return dp[mask][current];
    }

    // ========================================
    // TSP 경로 역추적
    // 최적 경로를 실제로 복원
    // ========================================
    static List<Integer> traceback() {
        List<Integer> path = new ArrayList<>();
        int mask = 1; // 0번 도시 방문한 상태로 시작
        int current = 0;
        path.add(0);

        for (int step = 1; step < N; step++) {
            int bestNext = -1;
            int bestCost = INF;

            for (int next = 0; next < N; next++) {
                if ((mask & (1 << next)) == 0 && cost[current][next] != 0) {
                    int newMask = mask | (1 << next);
                    int totalCost = cost[current][next] + dp[newMask][next];
                    if (totalCost < bestCost) {
                        bestCost = totalCost;
                        bestNext = next;
                    }
                }
            }

            path.add(bestNext);
            mask |= (1 << bestNext);
            current = bestNext;
        }

        path.add(0); // 출발점으로 복귀
        return path;
    }

    // ========================================
    // Bottom-Up 방식 TSP (반복문 기반)
    // ========================================
    static int tspBottomUp() {
        int fullMask = (1 << N) - 1;
        int[][] table = new int[1 << N][N];

        for (int[] row : table) Arrays.fill(row, INF);
        table[1][0] = 0; // 0번 도시에서 출발, 0번만 방문한 상태

        for (int mask = 1; mask < (1 << N); mask++) {
            for (int u = 0; u < N; u++) {
                if (table[mask][u] == INF) continue;
                if ((mask & (1 << u)) == 0) continue; // u가 mask에 포함되지 않으면 스킵

                for (int v = 0; v < N; v++) {
                    if ((mask & (1 << v)) != 0) continue; // 이미 방문한 도시는 스킵
                    if (cost[u][v] == 0) continue;         // 갈 수 없는 경우

                    int newMask = mask | (1 << v);
                    table[newMask][v] = Math.min(
                        table[newMask][v],
                        table[mask][u] + cost[u][v]
                    );
                }
            }
        }

        // 모든 도시를 방문한 후 0번으로 돌아오는 최소 비용
        int answer = INF;
        for (int u = 1; u < N; u++) {
            if (table[fullMask][u] != INF && cost[u][0] != 0) {
                answer = Math.min(answer, table[fullMask][u] + cost[u][0]);
            }
        }
        return answer;
    }

    public static void main(String[] args) {
        // 4개 도시의 비용 행렬 (0이면 직접 경로 없음)
        cost = new int[][] {
            {0, 10, 15, 20},
            {10,  0, 35, 25},
            {15, 35,  0, 30},
            {20, 25, 30,  0}
        };
        N = cost.length;

        // Top-Down 방식
        dp = new int[1 << N][N];
        for (int[] row : dp) Arrays.fill(row, -1);

        int startMask = 1; // 0번 도시 방문 상태
        int minCost = tsp(startMask, 0);

        System.out.println("=== TSP (Top-Down 비트마스크 DP) ===");
        System.out.println("최소 비용: " + minCost);

        // 경로 역추적
        List<Integer> path = traceback();
        System.out.print("최적 경로: ");
        for (int i = 0; i < path.size(); i++) {
            System.out.print(path.get(i));
            if (i < path.size() - 1) System.out.print(" → ");
        }
        System.out.println();

        // Bottom-Up 방식
        System.out.println("\n=== TSP (Bottom-Up 비트마스크 DP) ===");
        System.out.println("최소 비용: " + tspBottomUp());

        // 비트 연산 기초 데모
        System.out.println("\n=== 비트 연산 기초 ===");
        int mask = 0;
        System.out.println("공집합: " + Integer.toBinaryString(mask) + " (10진수: " + mask + ")");

        mask |= (1 << 0); // 도시 0 추가
        mask |= (1 << 2); // 도시 2 추가
        mask |= (1 << 3); // 도시 3 추가
        System.out.println("{0,2,3}: " + Integer.toBinaryString(mask) + " (10진수: " + mask + ")");
        System.out.println("도시 2 포함 여부: " + ((mask & (1 << 2)) != 0));
        System.out.println("도시 1 포함 여부: " + ((mask & (1 << 1)) != 0));
        System.out.println("포함된 도시 수: " + Integer.bitCount(mask));
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 구분 | 복잡도 |
|------|--------|
| **시간 복잡도** | \(O(2^N \times N^2)\) — 상태 수 \(2^N \times N\)에 각 상태에서 전이 O(N) |
| **공간 복잡도** | \(O(2^N \times N)\) — dp 테이블 |

- N = 15일 때: 상태 수 ≈ 2^15 × 15 ≈ 491,520 → 충분히 빠름
- N = 20일 때: 상태 수 ≈ 2^20 × 20 ≈ 20,971,520 → 시간 내에 가능하나 메모리 주의
- N > 20이면 상태 수가 폭발적으로 증가하여 일반적으로 사용 불가

## 관련 문제 유형
- **외판원 문제 (TSP)**: 모든 도시를 방문하고 돌아오는 최소 비용
- **작업 배정 문제 (Assignment Problem)**: N명에게 N개의 작업을 배정하는 최소 비용
- **해밀턴 경로/순환**: 모든 정점을 한 번씩 방문하는 경로 존재 여부/최소 비용
- **집합 분할**: 집합을 조건에 따라 여러 그룹으로 분할
- **게임 이론**: 선택 가능한 원소들의 상태를 비트로 관리하는 게임 문제
- **최소 비용 매칭**: 이분 그래프에서의 최소/최대 가중치 매칭
