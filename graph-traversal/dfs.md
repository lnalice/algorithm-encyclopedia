# DFS (깊이 우선 탐색)

## 분류
- **카테고리**: 그래프 탐색
- **관련 알고리즘**: [BFS](./bfs.md), [백트래킹](../backtracking/backtracking.md)

## 개요
DFS(Depth-First Search)는 그래프에서 한 방향으로 최대한 깊이 탐색한 뒤, 더 이상 진행할 수 없으면 이전 갈림길로 되돌아가서 다른 방향을 탐색하는 알고리즘이다. 재귀 호출 또는 명시적 스택을 사용하여 구현한다. 경로 탐색, 사이클 검출, 연결 요소 파악, 위상 정렬 등 다양한 그래프 문제의 기반이 된다.

## 핵심 로직
1. **시작 정점 방문**: 시작 정점을 방문 처리한다.
2. **인접 정점 탐색**: 현재 정점의 인접 정점 중 미방문 정점을 하나 선택하여 재귀적으로 방문한다.
3. **되돌아가기 (Backtrack)**: 더 이상 방문할 인접 정점이 없으면 이전 정점으로 돌아간다.
4. **반복**: 모든 정점을 방문할 때까지 2-3을 반복한다.

> 재귀 구현은 시스템 콜 스택을 사용하고, 반복 구현은 명시적 스택(Stack)을 사용한다. 깊은 그래프에서는 재귀 시 스택 오버플로우에 주의해야 한다.

## 언제 사용하는가?
- **경로 탐색**: 시작점에서 도착점까지의 경로 존재 여부 또는 모든 경로 탐색
- **사이클 검출**: 방향/무방향 그래프에서 사이클이 존재하는지 판별
- **연결 요소 (Connected Component)**: 그래프에서 연결된 컴포넌트의 수와 구성 파악
- **위상 정렬**: DAG(방향 비순환 그래프)에서 선후 관계에 따른 정렬
- **트리 탐색**: 트리 구조에서 전위/중위/후위 순회
- **백트래킹의 기반**: 조합, 순열 등 해 공간을 탐색할 때 DFS를 기반으로 가지치기

## Java 구현
```java
import java.util.*;

public class DFS {

    // ========================================
    // 방법 1: 재귀 기반 DFS
    // ========================================
    static void dfsRecursive(List<List<Integer>> graph, int current, boolean[] visited) {
        visited[current] = true;
        System.out.print(current + " ");

        for (int next : graph.get(current)) {
            if (!visited[next]) {
                dfsRecursive(graph, next, visited);
            }
        }
    }

    // ========================================
    // 방법 2: 스택 기반 DFS (반복)
    // ========================================
    static void dfsIterative(List<List<Integer>> graph, int start, int n) {
        boolean[] visited = new boolean[n];
        Deque<Integer> stack = new ArrayDeque<>();

        stack.push(start);

        while (!stack.isEmpty()) {
            int current = stack.pop();

            if (visited[current]) continue; // 이미 방문한 정점은 건너뜀
            visited[current] = true;
            System.out.print(current + " ");

            // 인접 정점을 역순으로 스택에 넣으면 작은 번호부터 방문
            List<Integer> neighbors = graph.get(current);
            for (int i = neighbors.size() - 1; i >= 0; i--) {
                int next = neighbors.get(i);
                if (!visited[next]) {
                    stack.push(next);
                }
            }
        }
    }

    // ========================================
    // 응용: 연결 요소 개수 세기
    // ========================================
    static int countConnectedComponents(List<List<Integer>> graph, int n) {
        boolean[] visited = new boolean[n];
        int count = 0;

        for (int i = 0; i < n; i++) {
            if (!visited[i]) {
                dfsRecursive(graph, i, visited);
                System.out.println(); // 각 컴포넌트 출력 후 줄바꿈
                count++;
            }
        }

        return count;
    }

    // ========================================
    // 응용: 사이클 검출 (방향 그래프)
    // 0: 미방문, 1: 탐색 중(현재 경로), 2: 탐색 완료
    // ========================================
    static boolean hasCycle(List<List<Integer>> graph, int n) {
        int[] state = new int[n];

        for (int i = 0; i < n; i++) {
            if (state[i] == 0 && dfsCycleCheck(graph, i, state)) {
                return true;
            }
        }
        return false;
    }

    static boolean dfsCycleCheck(List<List<Integer>> graph, int current, int[] state) {
        state[current] = 1; // 탐색 중 표시

        for (int next : graph.get(current)) {
            if (state[next] == 1) return true;  // 현재 경로에서 다시 만남 → 사이클
            if (state[next] == 0 && dfsCycleCheck(graph, next, state)) return true;
        }

        state[current] = 2; // 탐색 완료 표시
        return false;
    }

    public static void main(String[] args) {
        int n = 7; // 정점 수
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            graph.add(new ArrayList<>());
        }

        // 양방향 간선 추가 (정점 0-4는 연결, 5-6은 별도 연결)
        int[][] edges = {{0, 1}, {0, 2}, {1, 3}, {2, 4}, {5, 6}};
        for (int[] edge : edges) {
            graph.get(edge[0]).add(edge[1]);
            graph.get(edge[1]).add(edge[0]);
        }

        // 재귀 DFS
        System.out.println("=== 재귀 DFS (정점 0부터) ===");
        boolean[] visited = new boolean[n];
        dfsRecursive(graph, 0, visited);
        System.out.println();

        // 스택 DFS
        System.out.println("\n=== 스택 DFS (정점 0부터) ===");
        dfsIterative(graph, 0, n);
        System.out.println();

        // 연결 요소 개수
        System.out.println("\n=== 연결 요소 개수 ===");
        int components = countConnectedComponents(graph, n);
        System.out.println("연결 요소 수: " + components);
    }
}
```

## 시간 복잡도 / 공간 복잡도
| | 복잡도 |
|---|---|
| **시간 복잡도** | \(O(V + E)\) — 모든 정점(V)과 간선(E)을 한 번씩 탐색 |
| **공간 복잡도** | \(O(V)\) — 방문 배열 + 재귀 호출 스택 또는 명시적 스택 |

> 재귀 DFS의 경우 그래프 깊이가 매우 깊으면 스택 오버플로우가 발생할 수 있다. 이때는 스택 기반 반복 DFS를 사용한다.

## 관련 문제 유형
- **연결 요소 개수**: 그래프에서 서로 연결된 그룹의 수
- **경로 존재 여부**: 두 정점 사이에 경로가 있는지 판별
- **사이클 검출**: 그래프에 순환이 존재하는지 확인
- **위상 정렬**: 선후 관계가 있는 작업의 수행 순서 결정
- **섬의 개수 (2D 격자)**: 상하좌우 연결된 영역 세기
- **미로에서 모든 경로 찾기**: 가능한 모든 경로를 탐색
