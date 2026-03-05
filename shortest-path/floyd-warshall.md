# 플로이드-워셜 알고리즘 (Floyd-Warshall Algorithm)

## 분류
- **카테고리**: 최단 경로 (DP 기반)
- **관련 알고리즘**: [다익스트라](./dijkstra.md), [벨만-포드](./bellman-ford.md), [DP](../dynamic-programming/dp.md)

## 개요

플로이드-워셜 알고리즘은 **모든 정점 쌍 사이의 최단 경로**를 구하는 알고리즘이다. **동적 프로그래밍(DP)** 기반으로, 각 정점을 경유지로 고려하면서 최단 거리를 점진적으로 갱신한다. 음수 가중치를 가진 간선도 처리할 수 있지만, 음수 사이클이 존재하면 올바른 결과를 보장하지 않는다.

## 핵심 로직

1. **초기화**: 2차원 거리 배열 `dist[i][j]`를 설정한다.
   - `i == j`이면 `dist[i][j] = 0`
   - 간선 `(i, j)`가 존재하면 `dist[i][j] = weight(i, j)`
   - 그 외에는 `dist[i][j] = ∞`
2. **경유지 반복 (DP 전이)**: 모든 정점 `k`를 경유지로 순회한다 (k = 1 ~ N):
   - 모든 정점 쌍 `(i, j)`에 대해: `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`
3. **종료**: `dist[i][j]`에 정점 `i`에서 `j`까지의 최단 거리가 저장된다.

> **DP 점화식**: `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`
> 이는 "정점 k를 경유하는 것이 더 짧은가?"를 매번 판단하는 것이다.

## 언제 사용하는가?

- **모든 쌍 최단 경로**: 모든 정점 쌍 간의 최단 거리를 한 번에 구할 때
- **경유지 문제**: 특정 정점을 경유하는 최단 경로를 구할 때
- **도달 가능성 판별**: 정점 간 연결 여부를 확인할 때 (Transitive Closure)
- 정점의 수가 적을 때 (일반적으로 \(V \leq 500\) 정도)
- 그래프의 구조가 자주 변경되지 않고, 여러 쿼리에 대해 응답해야 할 때

## Java 구현

```java
import java.util.*;

public class FloydWarshall {

    static final int INF = 100_000_000; // 오버플로우 방지를 위해 적절한 큰 값 사용

    /**
     * 플로이드-워셜 알고리즘: 모든 쌍 최단 경로
     * @param n    정점의 수 (1번 ~ n번)
     * @param dist 초기 거리 행렬 (직접 호출 시 미리 초기화 필요)
     */
    static void floydWarshall(int n, int[][] dist) {
        // k: 경유 정점
        for (int k = 1; k <= n; k++) {
            // i: 출발 정점
            for (int i = 1; i <= n; i++) {
                // j: 도착 정점
                for (int j = 1; j <= n; j++) {
                    if (dist[i][k] + dist[k][j] < dist[i][j]) {
                        dist[i][j] = dist[i][k] + dist[k][j];
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt(); // 정점 수
        int m = sc.nextInt(); // 간선 수

        // 거리 배열 초기화
        int[][] dist = new int[n + 1][n + 1];
        for (int i = 0; i <= n; i++) {
            Arrays.fill(dist[i], INF);
            dist[i][i] = 0; // 자기 자신으로의 거리는 0
        }

        // 간선 입력: 출발, 도착, 가중치
        for (int i = 0; i < m; i++) {
            int u = sc.nextInt();
            int v = sc.nextInt();
            int w = sc.nextInt();
            // 동일 경로에 여러 간선이 있을 수 있으므로 최솟값 유지
            dist[u][v] = Math.min(dist[u][v], w);
        }

        floydWarshall(n, dist);

        // 결과 출력
        StringBuilder sb = new StringBuilder();
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                if (j > 1) sb.append(' ');
                sb.append(dist[i][j] >= INF ? "INF" : dist[i][j]);
            }
            sb.append('\n');
        }
        System.out.print(sb);
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 구분 | 복잡도 |
|------|--------|
| **시간 복잡도** | \(O(V^3)\) — 3중 반복문 |
| **공간 복잡도** | \(O(V^2)\) — 2차원 거리 배열 |

- `V`: 정점의 수
- 정점 수가 많아지면 매우 느려지므로, 일반적으로 \(V \leq 500\) 이하에서 사용한다.

## 관련 문제 유형

- **모든 쌍 최단 경로**: 모든 도시 간의 최소 이동 비용
- **경유지 문제**: 특정 도시를 반드시 거쳐야 하는 최단 경로
- **도달 가능성**: 정점 A에서 정점 B로 갈 수 있는지 여부 판별
- **최소 사이클 탐지**: 자기 자신으로 돌아오는 최단 사이클 찾기
- **중간 경유 최적화**: 여러 경유지 중 최적의 경유지를 선택하는 문제
