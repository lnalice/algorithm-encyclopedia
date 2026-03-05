# MST — 최소 신장 트리 (Minimum Spanning Tree)

## 분류
- **카테고리**: 최소 신장 트리
- **관련 알고리즘**: [크루스칼](./kruskal.md), [Union-Find](../data-structures/union-find.md), [그리디](../greedy/greedy.md)

## 개요

최소 신장 트리(MST)란, 가중치가 있는 무방향 연결 그래프에서 **모든 정점을 포함**하면서 **간선 가중치의 합이 최소**인 트리(사이클 없는 연결 부분 그래프)를 말한다. MST를 구하는 대표적인 알고리즘으로 **크루스칼(Kruskal)**과 **프림(Prim)**이 있으며, 둘 다 **그리디 알고리즘**에 기반한다.

## 핵심 로직

### MST의 성질

1. **간선 수**: 정점이 V개인 MST는 정확히 **V-1개의 간선**을 가진다.
2. **유일성**: 모든 간선의 가중치가 서로 다르면 MST는 유일하다.
3. **Cut Property**: 그래프의 임의의 컷(cut)에서 가중치가 가장 작은 간선은 반드시 MST에 포함된다.
4. **Cycle Property**: 그래프의 임의의 사이클에서 가중치가 가장 큰 간선은 MST에 포함되지 않는다.
5. **사이클 없음**: MST는 트리이므로 사이클이 존재하지 않는다.

### 크루스칼 vs 프림 비교

| 비교 항목 | 크루스칼 (Kruskal) | 프림 (Prim) |
|-----------|-------------------|-------------|
| **접근 방식** | 간선 중심 (간선 정렬 후 선택) | 정점 중심 (인접 정점 확장) |
| **자료구조** | Union-Find | 우선순위 큐 (최소 힙) |
| **시간 복잡도** | \(O(E \log E)\) | \(O(E \log V)\) |
| **적합한 그래프** | 희소 그래프 (간선 적음) | 밀집 그래프 (간선 많음) |
| **구현 난이도** | Union-Find 구현 필요 | 우선순위 큐 활용 |
| **그리디 전략** | 가장 가벼운 간선부터 선택 | 현재 트리에서 가장 가까운 정점 선택 |

### 프림 알고리즘 동작 원리

1. **초기화**: 임의의 시작 정점을 선택하여 MST 집합에 넣는다.
2. **확장**: MST 집합에 속한 정점과 연결된 간선 중 **가중치가 최소인 간선**을 선택한다.
   - 해당 간선으로 연결되는 정점이 MST에 아직 포함되지 않았으면 추가한다.
3. **반복**: 모든 정점이 MST에 포함될 때까지 2번을 반복한다.

## 언제 사용하는가?

- **최소 비용 신장 트리** 문제 전반
- **모든 노드를 최소 비용으로 연결**: 네트워크, 도로, 파이프라인 설계
- **클러스터링**: MST를 구한 뒤 가장 비용이 큰 간선 K-1개를 제거하면 K개의 클러스터가 생성됨
- **근사 알고리즘의 기반**: TSP(외판원 문제) 등의 근사 해법에 MST 활용
- 희소 그래프에서는 크루스칼, 밀집 그래프에서는 프림이 유리하다

## Java 구현

아래는 **프림 알고리즘**의 구현이다. (크루스칼 구현은 [크루스칼 문서](./kruskal.md)를 참고)

```java
import java.util.*;

public class Prim {

    static final int INF = Integer.MAX_VALUE;

    // 인접 리스트: {도착 정점, 가중치}
    static List<int[]>[] graph;

    /**
     * 프림 알고리즘: 우선순위 큐를 사용한 MST 구현
     * @param n     정점의 수 (1번 ~ n번)
     * @param start 시작 정점
     * @return MST의 총 가중치
     */
    static long prim(int n, int start) {
        boolean[] visited = new boolean[n + 1];
        // 최소 힙: {가중치, 도착 정점}
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);

        pq.offer(new int[]{0, start});
        long totalWeight = 0;
        int edgeCount = 0;

        while (!pq.isEmpty() && edgeCount < n) {
            int[] cur = pq.poll();
            int w = cur[0];
            int u = cur[1];

            // 이미 MST에 포함된 정점이면 무시
            if (visited[u]) continue;

            visited[u] = true;
            totalWeight += w;
            edgeCount++;

            // 인접한 정점들을 큐에 추가
            for (int[] edge : graph[u]) {
                int v = edge[0];
                int weight = edge[1];
                if (!visited[v]) {
                    pq.offer(new int[]{weight, v});
                }
            }
        }

        return totalWeight;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt(); // 정점 수
        int m = sc.nextInt(); // 간선 수

        graph = new ArrayList[n + 1];
        for (int i = 0; i <= n; i++) {
            graph[i] = new ArrayList<>();
        }

        // 간선 입력: 출발, 도착, 가중치 (무방향)
        for (int i = 0; i < m; i++) {
            int u = sc.nextInt();
            int v = sc.nextInt();
            int w = sc.nextInt();
            graph[u].add(new int[]{v, w});
            graph[v].add(new int[]{u, w});
        }

        long result = prim(n, 1);
        System.out.println(result);
    }
}
```

## 시간 복잡도 / 공간 복잡도

### 프림 알고리즘

| 구분 | 복잡도 |
|------|--------|
| **시간 복잡도** | \(O(E \log V)\) — 우선순위 큐 사용 시 |
| **공간 복잡도** | \(O(V + E)\) — 인접 리스트 + 방문 배열 + 우선순위 큐 |

### 크루스칼 알고리즘

| 구분 | 복잡도 |
|------|--------|
| **시간 복잡도** | \(O(E \log E)\) — 간선 정렬이 지배적 |
| **공간 복잡도** | \(O(V + E)\) — Union-Find 배열 + 간선 배열 |

- `V`: 정점의 수, `E`: 간선의 수

## 관련 문제 유형

- **최소 비용 네트워크 연결**: 모든 도시/컴퓨터를 최소 비용으로 연결
- **도로/통신망 설계**: 인프라 건설 비용 최소화
- **클러스터링 문제**: MST에서 가장 비싼 간선을 제거하여 그룹 분리
- **최소 비용 간선 선택 + 전체 연결 보장**: 예산 내 최대 연결
- **섬 연결하기**: 여러 섬(연결 요소)을 다리(간선)로 연결하는 최소 비용
