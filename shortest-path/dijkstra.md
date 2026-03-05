# 다익스트라 알고리즘 (Dijkstra's Algorithm)

## 분류
- **카테고리**: 최단 경로 (그리디 기반)
- **관련 알고리즘**: [벨만-포드](./bellman-ford.md), [플로이드-워셜](./floyd-warshall.md), [BFS](../graph-traversal/bfs.md), [그리디](../greedy/greedy.md)

## 개요

다익스트라 알고리즘은 **하나의 출발점**에서 다른 모든 정점까지의 최단 경로를 구하는 알고리즘이다. **그리디(탐욕) 알고리즘의 대표적인 예시**로, 매 단계에서 아직 방문하지 않은 정점 중 거리가 가장 짧은 정점을 선택하여 확장한다. 단, **음수 가중치가 있는 그래프에서는 사용할 수 없다.**

## 핵심 로직

1. **초기화**: 출발 정점의 거리를 0, 나머지 모든 정점의 거리를 ∞(무한대)로 설정한다.
2. **우선순위 큐에 출발 정점 삽입**: (거리 0, 출발 정점)을 큐에 넣는다.
3. **반복**: 우선순위 큐가 빌 때까지 다음을 수행한다:
   - 큐에서 거리가 가장 짧은 정점 `u`를 꺼낸다.
   - 이미 확정된 거리보다 큰 값이면 무시한다 (중복 방문 방지).
   - `u`의 모든 인접 정점 `v`에 대해, `dist[u] + weight(u, v) < dist[v]`이면 `dist[v]`를 갱신하고 큐에 삽입한다.
4. **종료**: 모든 정점의 최단 거리가 확정된다.

> **그리디 원리**: 매번 "현재까지 발견된 최단 거리 정점"을 선택하는 것이 최적해를 보장한다. 이는 간선 가중치가 음수가 아닐 때만 성립한다.

## 언제 사용하는가?

- **단일 출발점 최단 경로** 문제 (음수 가중치 없는 경우)
- 가중치가 있는 그래프에서 특정 정점으로부터 다른 모든 정점까지의 최단 거리를 구할 때
- 네비게이션, 네트워크 라우팅 등 실세계 최단 경로 문제
- 가중치가 모두 1인 경우에는 BFS가 더 효율적이므로 BFS를 사용

## Java 구현

```java
import java.util.*;

public class Dijkstra {

    static final int INF = Integer.MAX_VALUE;

    // 인접 리스트의 간선 정보: (도착 정점, 가중치)
    static List<int[]>[] graph;
    static int[] dist;

    /**
     * 우선순위 큐(최소 힙)를 사용한 다익스트라 알고리즘
     * @param n     정점의 수 (1번 ~ n번)
     * @param start 출발 정점
     */
    static void dijkstra(int n, int start) {
        dist = new int[n + 1];
        Arrays.fill(dist, INF);
        dist[start] = 0;

        // 최소 힙: {거리, 정점}
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        pq.offer(new int[]{0, start});

        while (!pq.isEmpty()) {
            int[] cur = pq.poll();
            int curDist = cur[0];
            int u = cur[1];

            // 이미 더 짧은 경로가 확정된 경우 무시
            if (curDist > dist[u]) continue;

            for (int[] edge : graph[u]) {
                int v = edge[0];
                int weight = edge[1];
                int newDist = dist[u] + weight;

                if (newDist < dist[v]) {
                    dist[v] = newDist;
                    pq.offer(new int[]{newDist, v});
                }
            }
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt(); // 정점 수
        int m = sc.nextInt(); // 간선 수
        int start = sc.nextInt(); // 출발 정점

        graph = new ArrayList[n + 1];
        for (int i = 0; i <= n; i++) {
            graph[i] = new ArrayList<>();
        }

        // 간선 입력: 출발, 도착, 가중치
        for (int i = 0; i < m; i++) {
            int u = sc.nextInt();
            int v = sc.nextInt();
            int w = sc.nextInt();
            graph[u].add(new int[]{v, w});
        }

        dijkstra(n, start);

        // 결과 출력
        StringBuilder sb = new StringBuilder();
        for (int i = 1; i <= n; i++) {
            sb.append(dist[i] == INF ? "INF" : dist[i]).append('\n');
        }
        System.out.print(sb);
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 구분 | 복잡도 |
|------|--------|
| **시간 복잡도** | \(O((V + E) \log V)\) — 우선순위 큐 사용 시 |
| **공간 복잡도** | \(O(V + E)\) — 인접 리스트 + 거리 배열 + 우선순위 큐 |

- `V`: 정점의 수, `E`: 간선의 수
- 배열 기반(우선순위 큐 미사용) 구현 시 시간 복잡도는 \(O(V^2)\)이며, 간선이 적은 희소 그래프에서는 우선순위 큐 방식이 유리하다.

## 관련 문제 유형

- **단일 출발점 최단 경로**: 한 정점에서 다른 모든 정점까지의 최단 거리
- **특정 정점 간 최단 경로**: 출발점과 도착점이 정해진 최단 경로
- **경유지가 있는 최단 경로**: 특정 정점을 반드시 거쳐야 하는 경우 (다익스트라 여러 번 수행)
- **네트워크 지연 시간**: 모든 노드에 신호가 도달하는 최소 시간
- **최소 비용 경로 탐색**: 가중치가 있는 그래프에서의 이동 비용 최소화
