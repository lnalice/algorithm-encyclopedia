# Union-Find (서로소 집합, Disjoint Set Union)

## 분류
- **카테고리**: 자료구조
- **관련 알고리즘**: [크루스칼](../minimum-spanning-tree/kruskal.md), [MST](../minimum-spanning-tree/kruskal.md), [그래프 탐색](../graph-traversal/bfs.md)

## 개요
Union-Find(서로소 집합, DSU)는 **서로 겹치지 않는 집합들**을 효율적으로 관리하는 자료구조로, 두 가지 핵심 연산을 제공한다: 두 원소가 같은 집합에 속하는지 확인하는 **Find**와 두 집합을 하나로 합치는 **Union**. 경로 압축(Path Compression)과 랭크 기반 합치기(Union by Rank)를 적용하면 거의 \(O(1)\)에 가까운 연산 속도를 달성할 수 있다.

## 핵심 로직

### 자료구조 초기화
- 각 원소는 자기 자신을 부모(대표)로 가지는 독립된 집합으로 시작한다.
- `parent[i] = i`, `rank[i] = 0`

### Find 연산 (경로 압축)
1. 원소 `x`의 루트(대표 원소)를 재귀적으로 찾는다.
2. **경로 압축**: 찾는 과정에서 만나는 모든 노드의 부모를 루트로 직접 연결한다.
3. 이후 같은 원소에 대한 Find가 \(O(1)\)에 가까워진다.

### Union 연산 (랭크 기반 합치기)
1. 두 원소의 루트를 각각 찾는다.
2. 이미 같은 집합이면 합치지 않는다 (사이클 검출에 활용).
3. **랭크 기반 합치기**: 랭크(트리 높이)가 낮은 쪽을 높은 쪽에 붙여 트리가 편향되는 것을 방지한다.

## 크루스칼 알고리즘과의 관계
Union-Find는 **[크루스칼 알고리즘](../minimum-spanning-tree/kruskal.md)**에서 핵심적으로 사용되는 자료구조이다:
1. 간선을 가중치 오름차순으로 정렬한다.
2. 각 간선에 대해 **Find**로 두 정점이 같은 집합인지 확인한다.
3. 다른 집합이면 **Union**으로 합치고 MST에 포함시킨다.
4. 같은 집합이면 사이클이 형성되므로 건너뛴다.

> Union-Find 없이는 크루스칼 알고리즘을 효율적으로 구현할 수 없다. 사이클 검출을 위해 매번 그래프 탐색을 하면 \(O(V \times E)\)로 비효율적이지만, Union-Find를 사용하면 \(O(E \log E)\) (정렬 시간이 지배적)로 해결된다.

## 언제 사용하는가?
- **집합의 합치기(Union) / 찾기(Find) 연산**이 빈번한 경우
- **연결 요소(Connected Component) 판별**: 그래프에서 서로 연결된 그룹 수 세기
- **크루스칼 MST**: 최소 신장 트리 구성 시 사이클 검출
- **사이클 검출**: 무방향 그래프에서 간선 추가 시 사이클 여부 확인
- **동적 연결성**: 온라인으로 간선이 추가되면서 연결 여부를 판단하는 문제
- **동치 관계 처리**: "A와 B는 같은 그룹이다" 류의 관계를 처리하는 문제

## Java 구현

```java
public class UnionFind {

    static int[] parent; // 각 원소의 부모 (대표 원소 추적)
    static int[] rank;   // 각 루트의 트리 높이 (랭크)

    /**
     * 초기화: 각 원소는 자기 자신이 대표인 독립 집합
     * @param n 원소의 개수 (0 ~ n-1)
     */
    static void init(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i; // 자기 자신이 부모
            rank[i] = 0;   // 초기 랭크는 0
        }
    }

    /**
     * Find 연산 (경로 압축 적용)
     * x가 속한 집합의 루트(대표 원소)를 반환한다.
     * 재귀 과정에서 경로 상의 모든 노드를 루트에 직접 연결한다.
     */
    static int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]); // 경로 압축: 부모를 루트로 갱신
        }
        return parent[x];
    }

    /**
     * Union 연산 (랭크 기반 합치기)
     * x가 속한 집합과 y가 속한 집합을 합친다.
     * @return 합쳐졌으면 true, 이미 같은 집합이면 false (사이클 검출 용도)
     */
    static boolean union(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);

        if (rootX == rootY) return false; // 이미 같은 집합 → 사이클

        // 랭크가 낮은 트리를 높은 트리 아래에 붙임
        if (rank[rootX] < rank[rootY]) {
            parent[rootX] = rootY;
        } else if (rank[rootX] > rank[rootY]) {
            parent[rootY] = rootX;
        } else {
            parent[rootY] = rootX;
            rank[rootX]++; // 랭크가 같으면 합친 후 랭크 증가
        }
        return true;
    }

    /**
     * 두 원소가 같은 집합에 속하는지 확인
     */
    static boolean isConnected(int x, int y) {
        return find(x) == find(y);
    }

    public static void main(String[] args) {
        int n = 7; // 원소 0 ~ 6
        init(n);

        // 집합 합치기
        union(0, 1);
        union(1, 2);
        union(3, 4);
        union(5, 6);

        // 연결 여부 확인
        System.out.println("0과 2가 같은 집합? " + isConnected(0, 2)); // true
        System.out.println("0과 3이 같은 집합? " + isConnected(0, 3)); // false

        // 두 집합을 합침
        union(2, 4);
        System.out.println("0과 3이 같은 집합? " + isConnected(0, 3)); // true

        // 사이클 검출 예시 (간선 0-2를 추가하면 사이클)
        boolean hasCycle = !union(0, 2);
        System.out.println("0-2 간선 추가 시 사이클 발생? " + hasCycle); // true
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 연산 | 시간 복잡도 | 비고 |
|------|-----------|------|
| Find (경로 압축) | \(O(\alpha(N))\) ≈ \(O(1)\) | \(\alpha\)는 아커만 함수의 역함수 (사실상 상수) |
| Union (랭크 기반) | \(O(\alpha(N))\) ≈ \(O(1)\) | Find를 호출하므로 동일 |
| 초기화 | \(O(N)\) | 모든 원소를 초기화 |

- **공간 복잡도**: \(O(N)\) — parent 배열 + rank 배열

> M번의 Union/Find 연산에 대해 전체 시간 복잡도는 \(O(M \cdot \alpha(N))\)으로, 사실상 \(O(M)\)이다.

## 관련 문제 유형
- **MST 구성**: 크루스칼 알고리즘으로 최소 신장 트리 구하기
- **연결 요소**: 그래프의 연결 요소 개수 세기
- **사이클 검출**: 무방향 그래프에서 사이클 존재 여부 판단
- **동치 관계**: "A와 B는 친구이다", "X와 Y는 같은 네트워크이다" 등
- **집합 크기 관리**: Union-Find에 size 배열을 추가하여 각 집합의 크기 추적
- **온라인 연결성**: 간선이 순차적으로 추가될 때 연결 여부 판단
