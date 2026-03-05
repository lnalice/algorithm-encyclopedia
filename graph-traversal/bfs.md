# BFS (너비 우선 탐색)

## 분류
- **카테고리**: 그래프 탐색
- **관련 알고리즘**: [DFS](./dfs.md), [다익스트라](../shortest-path/dijkstra.md)

## 개요
BFS(Breadth-First Search)는 그래프에서 시작 정점으로부터 가까운 정점부터 우선적으로 탐색하는 알고리즘이다. 큐(Queue)를 사용하여 현재 정점과 인접한 모든 정점을 먼저 방문한 뒤, 다음 레벨의 정점들을 탐색한다. 가중치가 없는 그래프에서 최단 경로를 보장하므로, 최소 이동 횟수나 최단 거리 문제에 널리 사용된다.

## 핵심 로직
1. **초기화**: 시작 정점을 큐에 넣고 방문 처리한다.
2. **탐색 반복**: 큐가 빌 때까지 다음을 반복한다.
   - 큐에서 정점 하나를 꺼낸다 (`poll`).
   - 해당 정점과 인접한 모든 정점에 대해:
     - 아직 방문하지 않았다면 방문 처리 후 큐에 넣는다.
3. **종료**: 큐가 비면 탐색이 완료된다.

> BFS는 **레벨 단위**로 탐색이 진행되기 때문에, 시작점에서 각 정점까지의 최단 거리(간선 수 기준)를 자연스럽게 구할 수 있다.

## 언제 사용하는가?
- **최단 경로 (가중치 없는 그래프)**: 모든 간선의 가중치가 동일할 때 최단 거리를 구하는 문제
- **레벨별 탐색**: 트리나 그래프를 레벨(깊이) 단위로 처리해야 하는 문제
- **최소 이동 횟수**: 미로 탈출, 체스 나이트 이동 등 최소 몇 번 만에 도달하는지 구하는 문제
- **연결 요소 탐색**: 그래프의 연결된 컴포넌트를 찾는 문제
- **2차원 격자 탐색**: 상하좌우 이동으로 영역을 탐색하는 문제 (예: 섬의 개수, 토마토 익히기)

## Java 구현
```java
import java.util.*;

public class BFS {

    // 인접 리스트 기반 BFS
    // 시작 정점에서 모든 정점까지의 최단 거리를 반환한다.
    static int[] bfs(List<List<Integer>> graph, int start, int n) {
        int[] distance = new int[n]; // 시작점으로부터의 거리
        Arrays.fill(distance, -1);   // -1은 미방문 상태

        Queue<Integer> queue = new LinkedList<>();
        queue.offer(start);
        distance[start] = 0;

        while (!queue.isEmpty()) {
            int current = queue.poll();

            for (int next : graph.get(current)) {
                if (distance[next] == -1) { // 아직 방문하지 않은 정점
                    distance[next] = distance[current] + 1;
                    queue.offer(next);
                }
            }
        }

        return distance;
    }

    // 2차원 격자에서의 BFS (최소 이동 횟수)
    static int bfsGrid(int[][] grid, int startRow, int startCol, int endRow, int endCol) {
        int rows = grid.length;
        int cols = grid[0].length;
        int[] dr = {-1, 1, 0, 0}; // 상, 하, 좌, 우
        int[] dc = {0, 0, -1, 1};

        boolean[][] visited = new boolean[rows][cols];
        Queue<int[]> queue = new LinkedList<>();

        queue.offer(new int[]{startRow, startCol, 0}); // {행, 열, 거리}
        visited[startRow][startCol] = true;

        while (!queue.isEmpty()) {
            int[] cur = queue.poll();
            int row = cur[0], col = cur[1], dist = cur[2];

            if (row == endRow && col == endCol) {
                return dist; // 도착지에 도달하면 최단 거리 반환
            }

            for (int d = 0; d < 4; d++) {
                int nr = row + dr[d];
                int nc = col + dc[d];

                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols
                        && !visited[nr][nc] && grid[nr][nc] != 1) { // 1은 벽
                    visited[nr][nc] = true;
                    queue.offer(new int[]{nr, nc, dist + 1});
                }
            }
        }

        return -1; // 도달 불가능
    }

    public static void main(String[] args) {
        // === 그래프 BFS 예시 ===
        int n = 6; // 정점 수
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            graph.add(new ArrayList<>());
        }

        // 양방향 간선 추가
        int[][] edges = {{0, 1}, {0, 2}, {1, 3}, {2, 4}, {3, 5}, {4, 5}};
        for (int[] edge : edges) {
            graph.get(edge[0]).add(edge[1]);
            graph.get(edge[1]).add(edge[0]);
        }

        int[] distance = bfs(graph, 0, n);
        System.out.println("정점 0에서 각 정점까지의 최단 거리:");
        for (int i = 0; i < n; i++) {
            System.out.println("  정점 " + i + ": " + distance[i]);
        }

        // === 격자 BFS 예시 ===
        int[][] grid = {
            {0, 0, 0, 0},
            {1, 1, 0, 1},
            {0, 0, 0, 0},
            {0, 1, 1, 0}
        };

        int result = bfsGrid(grid, 0, 0, 3, 3);
        System.out.println("\n격자 (0,0)에서 (3,3)까지 최단 거리: " + result);
    }
}
```

## 시간 복잡도 / 공간 복잡도
| | 복잡도 |
|---|---|
| **시간 복잡도** | \(O(V + E)\) — 모든 정점(V)과 간선(E)을 한 번씩 탐색 |
| **공간 복잡도** | \(O(V)\) — 방문 배열과 큐에 최대 V개의 정점 저장 |

> 2차원 격자(R×C)의 경우: 시간 \(O(R \times C)\), 공간 \(O(R \times C)\)

## 관련 문제 유형
- **미로 최단 경로**: 격자 위에서 시작점→도착점 최단 거리
- **토마토 익히기 (다중 시작점 BFS)**: 여러 시작점에서 동시에 BFS
- **섬의 개수**: 연결된 영역의 개수 세기
- **트리 레벨 순회**: 트리를 레벨별로 순회하며 처리
- **단어 변환 (최소 변환 횟수)**: 문자열 그래프에서 BFS
- **나이트 최소 이동**: 체스판에서 나이트의 최소 이동 횟수
