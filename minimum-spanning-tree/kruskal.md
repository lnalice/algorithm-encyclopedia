# 크루스칼 알고리즘 (Kruskal's Algorithm)

## 분류
- **카테고리**: 최소 신장 트리 (그리디 기반)
- **관련 알고리즘**: [MST 개요](./mst.md), [Union-Find](../data-structures/union-find.md), [그리디](../greedy/greedy.md), [프림](./mst.md#프림-알고리즘-prim)

## 개요

크루스칼 알고리즘은 **최소 신장 트리(MST)** 를 구하는 대표적인 알고리즘으로, **그리디(탐욕) 알고리즘의 대표적인 예시**이다. 모든 간선을 가중치 순으로 정렬한 뒤, 사이클을 형성하지 않는 간선을 하나씩 선택하여 MST를 구축한다. 사이클 판별에는 **Union-Find(서로소 집합) 자료구조**를 활용한다.

## 핵심 로직

1. **간선 정렬**: 모든 간선을 가중치 기준 오름차순으로 정렬한다.
2. **Union-Find 초기화**: 각 정점을 독립적인 집합으로 초기화한다.
3. **간선 선택 (그리디)**: 가중치가 작은 간선부터 순서대로 확인한다:
   - 간선의 두 정점이 **서로 다른 집합**에 속하면 (사이클 미형성):
     - 해당 간선을 MST에 포함시키고, 두 집합을 `Union`한다.
   - 간선의 두 정점이 **같은 집합**에 속하면 (사이클 형성):
     - 해당 간선을 건너뛴다.
4. **종료**: MST의 간선이 `V-1`개가 되면 완성이다.

> **그리디 원리**: 매번 가장 가중치가 작은 간선을 선택하되, 사이클이 생기지 않도록 한다. 이 탐욕적 선택이 전체 최적해를 보장한다 (Cut Property에 의해 증명).

## 언제 사용하는가?

- **최소 비용으로 모든 노드를 연결**해야 하는 문제
- **네트워크 설계**: 도시 간 도로, 통신망, 전력망 등의 최소 비용 연결
- **간선 중심의 MST 문제**: 간선 리스트가 주어지는 경우 크루스칼이 자연스럽다
- **희소 그래프**: 간선이 적은 그래프에서 특히 효율적
- Union-Find를 함께 활용하는 연결 요소 관련 문제

## Java 구현

```java
import java.util.*;

public class Kruskal {

    static int[] parent, rank;

    // Union-Find: Find 연산 (경로 압축)
    static int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    // Union-Find: Union 연산 (랭크 기반 합치기)
    static boolean union(int a, int b) {
        int rootA = find(a);
        int rootB = find(b);

        if (rootA == rootB) return false; // 이미 같은 집합 (사이클 발생)

        // 랭크가 낮은 트리를 높은 트리 아래에 붙임
        if (rank[rootA] < rank[rootB]) {
            parent[rootA] = rootB;
        } else if (rank[rootA] > rank[rootB]) {
            parent[rootB] = rootA;
        } else {
            parent[rootB] = rootA;
            rank[rootA]++;
        }
        return true;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt(); // 정점 수
        int m = sc.nextInt(); // 간선 수

        // 간선 배열: {가중치, 출발 정점, 도착 정점}
        int[][] edges = new int[m][3];
        for (int i = 0; i < m; i++) {
            edges[i][1] = sc.nextInt(); // 출발
            edges[i][2] = sc.nextInt(); // 도착
            edges[i][0] = sc.nextInt(); // 가중치
        }

        // 가중치 기준 오름차순 정렬 (그리디)
        Arrays.sort(edges, (a, b) -> a[0] - b[0]);

        // Union-Find 초기화
        parent = new int[n + 1];
        rank = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            parent[i] = i;
        }

        int mstWeight = 0; // MST 총 가중치
        int edgeCount = 0; // 선택된 간선 수

        for (int[] edge : edges) {
            int w = edge[0];
            int u = edge[1];
            int v = edge[2];

            // 사이클이 생기지 않으면 간선 선택
            if (union(u, v)) {
                mstWeight += w;
                edgeCount++;

                // MST 완성 (간선 V-1개)
                if (edgeCount == n - 1) break;
            }
        }

        System.out.println(mstWeight);
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 구분 | 복잡도 |
|------|--------|
| **시간 복잡도** | \(O(E \log E)\) — 간선 정렬이 지배적 |
| **공간 복잡도** | \(O(V + E)\) — Union-Find 배열 + 간선 배열 |

- `V`: 정점의 수, `E`: 간선의 수
- Union-Find 연산은 경로 압축 + 랭크 기반 합치기 시 거의 \(O(\alpha(V))\) ≈ \(O(1)\)
- 간선 정렬 \(O(E \log E)\)이 전체 시간 복잡도를 결정한다.

## 관련 문제 유형

- **최소 비용 네트워크 연결**: 모든 도시/노드를 최소 비용으로 연결
- **도로 건설 문제**: N개의 도시를 잇는 최소 비용 도로 네트워크
- **통신망/전력망 설계**: 최소 케이블 비용으로 모든 건물 연결
- **최소 비용 간선 선택**: K개의 간선을 선택하여 비용 최소화
- **연결 요소 + 비용 최적화**: Union-Find와 함께 사용하는 복합 문제
