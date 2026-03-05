# 벨만-포드 알고리즘 (Bellman-Ford Algorithm)

## 분류
- **카테고리**: 최단 경로 (DP 기반)
- **관련 알고리즘**: [다익스트라](./dijkstra.md), [플로이드-워셜](./floyd-warshall.md), [DP](../dynamic-programming/dp.md)

## 개요

벨만-포드 알고리즘은 **하나의 출발점**에서 다른 모든 정점까지의 최단 경로를 구하는 알고리즘이다. **동적 프로그래밍(DP)** 기반으로, 다익스트라와 달리 **음수 가중치가 있는 간선도 처리**할 수 있다. 또한 **음수 사이클(Negative Cycle)의 존재 여부를 검출**할 수 있어, 음수 가중치가 등장하는 문제에서 핵심적으로 사용된다.

## 핵심 로직

1. **초기화**: 출발 정점의 거리를 0, 나머지 모든 정점의 거리를 ∞로 설정한다.
2. **간선 완화 반복 (V-1회)**: 모든 간선 `(u, v, w)`에 대해 다음을 `V-1`번 반복한다:
   - `dist[u] + w < dist[v]`이면 `dist[v] = dist[u] + w`로 갱신한다.
   - 최단 경로는 최대 `V-1`개의 간선을 가지므로, `V-1`번 반복하면 모든 최단 경로가 확정된다.
3. **음수 사이클 검출 (V번째 반복)**: 한 번 더 모든 간선을 확인한다:
   - 여전히 갱신이 가능한 간선이 존재하면, 그래프에 **음수 사이클**이 존재한다는 의미이다.

> **DP 원리**: `dist[v]`는 "최대 k개의 간선을 사용하여 도달할 수 있는 최단 거리"를 나타낸다. k를 1부터 V-1까지 증가시키며 모든 간선을 완화하면 최종 최단 거리가 구해진다.

## 언제 사용하는가?

- **음수 가중치가 있는 그래프**에서의 단일 출발점 최단 경로
- **음수 사이클 검출**: 환율 차익 거래(Arbitrage), 무한 이득 경로 판별
- 다익스트라를 사용할 수 없는 경우 (음수 가중치 존재)
- SPFA(Shortest Path Faster Algorithm)의 기반이 되는 알고리즘

## Java 구현

```java
import java.util.*;

public class BellmanFord {

    static final long INF = Long.MAX_VALUE;

    // 간선 정보: 출발, 도착, 가중치
    static int[] from, to, weight;
    static long[] dist;

    /**
     * 벨만-포드 알고리즘
     * @param n     정점의 수 (1번 ~ n번)
     * @param m     간선의 수
     * @param start 출발 정점
     * @return 음수 사이클 존재 여부 (true: 음수 사이클 있음)
     */
    static boolean bellmanFord(int n, int m, int start) {
        dist = new long[n + 1];
        Arrays.fill(dist, INF);
        dist[start] = 0;

        // (V-1)번 반복: 최단 경로 확정
        for (int i = 1; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (dist[from[j]] != INF && dist[from[j]] + weight[j] < dist[to[j]]) {
                    dist[to[j]] = dist[from[j]] + weight[j];
                }
            }
        }

        // V번째 반복: 음수 사이클 검출
        for (int j = 0; j < m; j++) {
            if (dist[from[j]] != INF && dist[from[j]] + weight[j] < dist[to[j]]) {
                return true; // 음수 사이클 존재
            }
        }

        return false; // 음수 사이클 없음
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt(); // 정점 수
        int m = sc.nextInt(); // 간선 수
        int start = sc.nextInt(); // 출발 정점

        from = new int[m];
        to = new int[m];
        weight = new int[m];

        // 간선 입력: 출발, 도착, 가중치
        for (int i = 0; i < m; i++) {
            from[i] = sc.nextInt();
            to[i] = sc.nextInt();
            weight[i] = sc.nextInt();
        }

        boolean hasNegativeCycle = bellmanFord(n, m, start);

        if (hasNegativeCycle) {
            System.out.println("음수 사이클이 존재합니다.");
        } else {
            StringBuilder sb = new StringBuilder();
            for (int i = 1; i <= n; i++) {
                sb.append(dist[i] == INF ? "INF" : dist[i]).append('\n');
            }
            System.out.print(sb);
        }
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 구분 | 복잡도 |
|------|--------|
| **시간 복잡도** | \(O(V \times E)\) — V-1번 반복 × 모든 간선 확인 |
| **공간 복잡도** | \(O(V + E)\) — 거리 배열 + 간선 배열 |

- `V`: 정점의 수, `E`: 간선의 수
- 다익스트라 \(O((V+E) \log V)\)보다 느리지만, 음수 가중치를 처리할 수 있다는 장점이 있다.

## 관련 문제 유형

- **음수 가중치 최단 경로**: 가중치에 음수가 포함된 그래프에서의 최단 경로
- **음수 사이클 검출**: 무한히 비용을 줄일 수 있는 사이클 존재 여부 판별
- **환율 차익 거래 (Arbitrage)**: 환율 그래프에서 이득을 볼 수 있는 순환 경로 탐지
- **시간 제약 최단 경로**: 특정 횟수 이내의 간선을 사용하는 최단 경로 (반복 횟수 제한)
- **타임 트래블 문제**: 시간이 되돌려지는(음수 가중치) 경로가 포함된 그래프
