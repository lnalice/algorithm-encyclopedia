# 위상 정렬 (Topological Sort)

## 분류
- **카테고리**: 그래프 탐색
- **관련 알고리즘**: [BFS](./bfs.md), [DFS](./dfs.md)

## 개요
위상 정렬(Topological Sort)은 **DAG(방향 비순환 그래프, Directed Acyclic Graph)**에서 정점들의 선후 관계를 유지하면서 모든 정점을 일렬로 나열하는 알고리즘이다. 간선 (u → v)가 있으면 정렬 결과에서 u가 반드시 v보다 앞에 위치한다. BFS 기반(Kahn's Algorithm, 진입차수 이용)과 DFS 기반 두 가지 방법으로 구현할 수 있으며, **사이클 검출**에도 활용된다.

## 핵심 로직

### 방법 1: BFS 기반 (Kahn's Algorithm — 진입차수 이용)
1. **진입차수 계산**: 모든 정점의 진입차수(들어오는 간선 수)를 계산한다.
2. **큐 초기화**: 진입차수가 0인 정점을 모두 큐에 넣는다.
3. **반복 처리**: 큐에서 정점을 꺼내 결과에 추가하고, 해당 정점에서 나가는 간선을 제거(인접 정점의 진입차수 감소)한다.
4. **새로운 진입차수 0**: 진입차수가 0이 된 정점을 큐에 추가한다.
5. **종료**: 큐가 빌 때까지 반복한다.

> **사이클 검출**: 모든 정점이 결과에 포함되지 않으면(결과 크기 < V) 그래프에 사이클이 존재한다.

### 방법 2: DFS 기반 (후위 순회의 역순)
1. **DFS 수행**: 모든 미방문 정점에 대해 DFS를 수행한다.
2. **후위 처리**: DFS에서 정점의 모든 인접 정점 탐색이 끝나면 해당 정점을 스택에 넣는다.
3. **역순 출력**: 스택에서 하나씩 꺼내면 위상 정렬 결과가 된다.

> DFS 기반 방법에서는 **탐색 중(visiting)** 상태인 정점을 다시 만나면 사이클이 존재하는 것이다.

## 언제 사용하는가?
- **작업 순서 결정**: 선행 작업이 있는 일련의 작업들을 올바른 순서로 나열
- **선수과목 문제**: 과목 간 선수 관계가 주어졌을 때 수강 순서 결정
- **의존성 해결**: 패키지 설치 순서, 빌드 의존성 해결
- **DAG에서 순서 정하기**: 방향 비순환 그래프의 모든 정점에 대해 선후 관계를 만족하는 정렬
- **사이클 검출**: 방향 그래프에서 사이클 존재 여부 판별 (위상 정렬 불가능 = 사이클 존재)
- **최장 경로 (DAG)**: 위상 정렬 순서로 DP를 적용하여 DAG의 최장 경로를 구함

## Java 구현
```java
import java.util.*;

public class TopologicalSort {

    // ========================================
    // 방법 1: BFS 기반 (Kahn's Algorithm)
    // 진입차수를 이용한 위상 정렬
    // ========================================
    static List<Integer> topologicalSortBFS(List<List<Integer>> graph, int n) {
        int[] inDegree = new int[n];

        // 모든 정점의 진입차수 계산
        for (int u = 0; u < n; u++) {
            for (int v : graph.get(u)) {
                inDegree[v]++;
            }
        }

        // 진입차수가 0인 정점을 큐에 삽입
        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            if (inDegree[i] == 0) {
                queue.offer(i);
            }
        }

        List<Integer> result = new ArrayList<>();

        while (!queue.isEmpty()) {
            int current = queue.poll();
            result.add(current);

            // 현재 정점에서 나가는 간선 제거 (인접 정점의 진입차수 감소)
            for (int next : graph.get(current)) {
                inDegree[next]--;
                if (inDegree[next] == 0) {
                    queue.offer(next);
                }
            }
        }

        // 모든 정점이 포함되지 않으면 사이클 존재
        if (result.size() != n) {
            System.out.println("사이클이 존재합니다! 위상 정렬 불가능.");
            return Collections.emptyList();
        }

        return result;
    }

    // ========================================
    // 방법 2: DFS 기반 위상 정렬
    // 후위 순회의 역순을 이용
    // ========================================
    static List<Integer> topologicalSortDFS(List<List<Integer>> graph, int n) {
        // 0: 미방문, 1: 탐색 중, 2: 탐색 완료
        int[] state = new int[n];
        Deque<Integer> stack = new ArrayDeque<>();
        boolean hasCycle = false;

        for (int i = 0; i < n; i++) {
            if (state[i] == 0) {
                if (!dfs(graph, i, state, stack)) {
                    hasCycle = true;
                    break;
                }
            }
        }

        if (hasCycle) {
            System.out.println("사이클이 존재합니다! 위상 정렬 불가능.");
            return Collections.emptyList();
        }

        // 스택에서 꺼내면 위상 정렬 결과
        List<Integer> result = new ArrayList<>();
        while (!stack.isEmpty()) {
            result.add(stack.pop());
        }
        return result;
    }

    // DFS 탐색: 사이클이 없으면 true, 있으면 false 반환
    static boolean dfs(List<List<Integer>> graph, int current, int[] state, Deque<Integer> stack) {
        state[current] = 1; // 탐색 중

        for (int next : graph.get(current)) {
            if (state[next] == 1) return false; // 탐색 중인 정점 재방문 → 사이클
            if (state[next] == 0) {
                if (!dfs(graph, next, state, stack)) return false;
            }
        }

        state[current] = 2; // 탐색 완료
        stack.push(current); // 후위 처리: 모든 후속 정점 처리 후 스택에 삽입
        return true;
    }

    public static void main(String[] args) {
        // 그래프 구성 (6개의 정점, DAG)
        // 0 → 1, 0 → 2
        // 1 → 3
        // 2 → 3, 2 → 4
        // 3 → 5
        // 4 → 5
        int n = 6;
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            graph.add(new ArrayList<>());
        }

        graph.get(0).add(1);
        graph.get(0).add(2);
        graph.get(1).add(3);
        graph.get(2).add(3);
        graph.get(2).add(4);
        graph.get(3).add(5);
        graph.get(4).add(5);

        // BFS 기반 위상 정렬
        System.out.println("=== BFS 기반 위상 정렬 (Kahn's Algorithm) ===");
        List<Integer> bfsResult = topologicalSortBFS(graph, n);
        System.out.println("결과: " + bfsResult);

        // DFS 기반 위상 정렬
        System.out.println("\n=== DFS 기반 위상 정렬 ===");
        List<Integer> dfsResult = topologicalSortDFS(graph, n);
        System.out.println("결과: " + dfsResult);

        // 사이클이 있는 그래프 테스트
        System.out.println("\n=== 사이클 있는 그래프 테스트 ===");
        List<List<Integer>> cyclicGraph = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            cyclicGraph.add(new ArrayList<>());
        }
        cyclicGraph.get(0).add(1);
        cyclicGraph.get(1).add(2);
        cyclicGraph.get(2).add(0); // 사이클: 0 → 1 → 2 → 0

        List<Integer> cyclicResult = topologicalSortBFS(cyclicGraph, 3);
        System.out.println("결과: " + cyclicResult);
    }
}
```

## 시간 복잡도 / 공간 복잡도
| | 복잡도 |
|---|---|
| **시간 복잡도** | \(O(V + E)\) — 모든 정점(V)과 간선(E)을 한 번씩 처리 |
| **공간 복잡도** | \(O(V + E)\) — 인접 리스트, 진입차수 배열(BFS) 또는 상태 배열 + 스택(DFS) |

> BFS 기반과 DFS 기반 모두 동일한 시간/공간 복잡도를 가진다. 코딩 테스트에서는 구현이 직관적인 **BFS 기반(Kahn's Algorithm)**을 더 많이 사용한다.

## 관련 문제 유형
- **선수과목 순서**: 과목 간 선수 관계를 만족하는 수강 순서 결정
- **작업 스케줄링**: 선행 작업 조건이 있는 작업의 실행 순서
- **사이클 검출**: 방향 그래프에서 사이클 존재 여부 판별
- **줄 세우기**: 키 비교 결과를 바탕으로 학생들을 줄 세우기
- **DAG 최장 경로**: 위상 정렬 + DP로 방향 비순환 그래프의 최장 경로
- **게임 맵 의존성**: 스킬 트리, 퀘스트 선행 조건 등 순서 결정
