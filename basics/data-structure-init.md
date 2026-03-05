# 자료구조 초기화 패턴 (Java)

코딩테스트에서 문제를 풀기 전, 입력을 받아 자료구조를 세팅하는 단계는 매번 반복된다.
이 문서는 **그래프, 트리, 그리드, 간선 리스트** 등 자주 쓰는 자료구조의 초기화 패턴을 정리한다.

---

## 1. 그래프 (Graph)

### 어떤 표현을 쓸까?

| 표현 방식 | 장점 | 단점 | 추천 상황 |
|----------|------|------|----------|
| **인접 리스트 (배열)** | 빠름, 메모리 효율적 | 정점 번호가 정수여야 함 | BFS, DFS, 다익스트라 등 대부분 |
| **인접 리스트 (Map)** | 정점이 문자열 등 비정수 가능 | 약간의 오버헤드 | 정점이 문자열이거나 번호가 불규칙할 때 |
| **인접 행렬** | 두 정점 간 연결 여부 O(1) | 메모리 O(V²), 희소 그래프에 낭비 | 플로이드-워셜, V가 작을 때 (≤ 500) |
| **간선 리스트** | 간선 자체를 정렬/순회 | 특정 정점의 인접 노드 찾기 어려움 | 크루스칼, 벨만-포드 |

---

### 1-1. 인접 리스트 (배열 기반) — 가장 많이 사용

```java
import java.util.*;

// === 가중치 없는 그래프 ===
int n = 6; // 정점 수 (1번 ~ n번)
List<Integer>[] graph = new ArrayList[n + 1];
for (int i = 0; i <= n; i++) {
    graph[i] = new ArrayList<>();
}

// 간선 추가 (양방향)
graph[u].add(v);
graph[v].add(u);

// 간선 추가 (단방향)
graph[u].add(v);
```

```java
// === 가중치 있는 그래프 ===
// int[] = {도착 정점, 가중치}
List<int[]>[] graph = new ArrayList[n + 1];
for (int i = 0; i <= n; i++) {
    graph[i] = new ArrayList<>();
}

// 간선 추가 (양방향, 가중치 w)
graph[u].add(new int[]{v, w});
graph[v].add(new int[]{u, w});
```

```java
// === 전체 입력 패턴 (Scanner) ===
Scanner sc = new Scanner(System.in);
int n = sc.nextInt(); // 정점 수
int m = sc.nextInt(); // 간선 수

List<int[]>[] graph = new ArrayList[n + 1];
for (int i = 0; i <= n; i++) {
    graph[i] = new ArrayList<>();
}

for (int i = 0; i < m; i++) {
    int u = sc.nextInt();
    int v = sc.nextInt();
    int w = sc.nextInt(); // 가중치 (없으면 생략)
    graph[u].add(new int[]{v, w});
    graph[v].add(new int[]{u, w}); // 양방향이면
}
```

```java
// === 전체 입력 패턴 (BufferedReader, 대량 입력 시 권장) ===
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
StringTokenizer st = new StringTokenizer(br.readLine());
int n = Integer.parseInt(st.nextToken());
int m = Integer.parseInt(st.nextToken());

List<int[]>[] graph = new ArrayList[n + 1];
for (int i = 0; i <= n; i++) {
    graph[i] = new ArrayList<>();
}

for (int i = 0; i < m; i++) {
    st = new StringTokenizer(br.readLine());
    int u = Integer.parseInt(st.nextToken());
    int v = Integer.parseInt(st.nextToken());
    int w = Integer.parseInt(st.nextToken());
    graph[u].add(new int[]{v, w});
    graph[v].add(new int[]{u, w});
}
```

---

### 1-2. 인접 리스트 (Map 기반)

정점이 문자열이거나 번호가 0부터 시작하지 않는 불규칙한 경우에 사용한다.

```java
// 가중치 없는 그래프
Map<String, List<String>> graph = new HashMap<>();

// 정점 추가 + 간선 추가
graph.computeIfAbsent("A", k -> new ArrayList<>()).add("B");
graph.computeIfAbsent("B", k -> new ArrayList<>()).add("A");
```

```java
// 가중치 있는 그래프
Map<Integer, List<int[]>> graph = new HashMap<>();

for (int i = 0; i < m; i++) {
    int u = sc.nextInt(), v = sc.nextInt(), w = sc.nextInt();
    graph.computeIfAbsent(u, k -> new ArrayList<>()).add(new int[]{v, w});
    graph.computeIfAbsent(v, k -> new ArrayList<>()).add(new int[]{u, w});
}
```

---

### 1-3. 인접 행렬

플로이드-워셜이나 정점 수가 적을 때(V ≤ 500) 사용한다.

```java
int INF = (int) 1e9;
int n = sc.nextInt();
int m = sc.nextInt();

int[][] dist = new int[n + 1][n + 1];

// 초기화: 자기 자신 = 0, 나머지 = INF
for (int i = 1; i <= n; i++) {
    Arrays.fill(dist[i], INF);
    dist[i][i] = 0;
}

// 간선 입력
for (int i = 0; i < m; i++) {
    int u = sc.nextInt(), v = sc.nextInt(), w = sc.nextInt();
    dist[u][v] = Math.min(dist[u][v], w); // 중복 간선 처리
    dist[v][u] = Math.min(dist[v][u], w); // 양방향이면
}
```

---

### 1-4. 간선 리스트 (Edge List)

크루스칼(간선 정렬)이나 벨만-포드(모든 간선 순회)에서 사용한다.

```java
// 방법 1: 2차원 배열
int[][] edges = new int[m][3]; // {출발, 도착, 가중치}
for (int i = 0; i < m; i++) {
    edges[i][0] = sc.nextInt(); // u
    edges[i][1] = sc.nextInt(); // v
    edges[i][2] = sc.nextInt(); // w
}
// 가중치 기준 정렬 (크루스칼)
Arrays.sort(edges, (a, b) -> a[2] - b[2]);
```

```java
// 방법 2: Edge 클래스
class Edge implements Comparable<Edge> {
    int from, to, weight;
    Edge(int from, int to, int weight) {
        this.from = from;
        this.to = to;
        this.weight = weight;
    }
    @Override
    public int compareTo(Edge o) {
        return this.weight - o.weight;
    }
}

List<Edge> edges = new ArrayList<>();
for (int i = 0; i < m; i++) {
    int u = sc.nextInt(), v = sc.nextInt(), w = sc.nextInt();
    edges.add(new Edge(u, v, w));
}
Collections.sort(edges); // 가중치 기준 정렬
```

---

## 2. 트리 (Tree)

### 어떤 표현을 쓸까?

| 표현 방식 | 추천 상황 |
|----------|----------|
| **인접 리스트** | 일반 트리 탐색 (DFS/BFS), LCA 등 대부분 |
| **부모 배열** | 부모-자식 관계만 필요할 때, Union-Find |
| **왼쪽/오른쪽 자식 배열** | 이진 트리가 명시적으로 주어질 때 |
| **Node 클래스** | 트리 구조를 직접 만들어야 할 때 |

---

### 2-1. 인접 리스트 (그래프와 동일)

트리도 그래프의 일종이므로 인접 리스트로 저장한 뒤, 루트에서 DFS/BFS로 탐색한다.

```java
int n = sc.nextInt(); // 노드 수
List<Integer>[] tree = new ArrayList[n + 1];
for (int i = 0; i <= n; i++) {
    tree[i] = new ArrayList<>();
}

// 트리 간선은 n-1개
for (int i = 0; i < n - 1; i++) {
    int u = sc.nextInt(), v = sc.nextInt();
    tree[u].add(v);
    tree[v].add(u);
}

// 루트(보통 1번)에서 DFS 탐색 시 visited로 부모 방향 재방문 방지
boolean[] visited = new boolean[n + 1];
```

---

### 2-2. 부모 배열

각 노드의 부모만 저장하는 방식. 입력이 "자식 → 부모" 형태로 주어질 때 유용하다.

```java
int[] parent = new int[n + 1]; // parent[i] = i의 부모 노드

// 루트 노드
parent[root] = 0; // 또는 -1

// 입력으로 부모가 주어지는 경우
for (int i = 2; i <= n; i++) {
    parent[i] = sc.nextInt();
}

// 인접 리스트에서 부모 배열 구축 (BFS)
Queue<Integer> queue = new LinkedList<>();
queue.offer(1); // 루트
visited[1] = true;
while (!queue.isEmpty()) {
    int cur = queue.poll();
    for (int child : tree[cur]) {
        if (!visited[child]) {
            parent[child] = cur;
            visited[child] = true;
            queue.offer(child);
        }
    }
}
```

---

### 2-3. 이진 트리 (왼쪽/오른쪽 자식 배열)

```java
int[] left = new int[n + 1];   // left[i] = i의 왼쪽 자식 (-1이면 없음)
int[] right = new int[n + 1];  // right[i] = i의 오른쪽 자식
Arrays.fill(left, -1);
Arrays.fill(right, -1);

// 입력: 노드번호, 왼쪽자식, 오른쪽자식
for (int i = 0; i < n; i++) {
    int node = sc.nextInt();
    int l = sc.nextInt();   // 없으면 -1
    int r = sc.nextInt();
    left[node] = l;
    right[node] = r;
}
```

---

### 2-4. Node 클래스

트라이처럼 트리를 직접 구축해야 하는 경우에 사용한다.

```java
class TreeNode {
    int val;
    TreeNode left, right;

    TreeNode(int val) {
        this.val = val;
    }
}

// N-ary 트리 (자식이 여러 개)
class NaryNode {
    int val;
    List<NaryNode> children = new ArrayList<>();

    NaryNode(int val) {
        this.val = val;
    }
}
```

---

## 3. 그리드 (2D 격자)

### 3-1. 기본 그리드 입력

```java
int n = sc.nextInt(); // 행
int m = sc.nextInt(); // 열

// 정수 그리드
int[][] grid = new int[n][m];
for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
        grid[i][j] = sc.nextInt();
    }
}

// 문자 그리드
char[][] grid = new char[n][m];
for (int i = 0; i < n; i++) {
    grid[i] = sc.next().toCharArray();
}

// BufferedReader로 문자 그리드 (빠른 입력)
char[][] grid = new char[n][m];
for (int i = 0; i < n; i++) {
    grid[i] = br.readLine().toCharArray();
}
```

---

### 3-2. 방향 배열 (dx, dy) — 4방향 / 8방향

그리드 탐색의 핵심 패턴. 반드시 외워두자.

```java
// 4방향: 상, 하, 좌, 우
int[] dx = {-1, 1, 0, 0};
int[] dy = {0, 0, -1, 1};

// 8방향: 상, 하, 좌, 우 + 대각선 4방향
int[] dx = {-1, -1, -1, 0, 0, 1, 1, 1};
int[] dy = {-1, 0, 1, -1, 1, -1, 0, 1};

// 범위 체크 + 이동 패턴
for (int d = 0; d < 4; d++) {
    int nx = x + dx[d];
    int ny = y + dy[d];
    if (nx >= 0 && nx < n && ny >= 0 && ny < m) {
        // 유효한 좌표
    }
}
```

> **주의**: `grid[행][열]` → `grid[x][y]`에서 `x`는 행(세로), `y`는 열(가로)이다.
> `dx`는 행 방향 이동, `dy`는 열 방향 이동이다. 수학의 (x, y) 좌표계와 혼동하지 말 것.

---

### 3-3. 그리드 BFS 템플릿

```java
boolean[][] visited = new boolean[n][m];
Queue<int[]> queue = new LinkedList<>();

// 시작점
queue.offer(new int[]{startX, startY});
visited[startX][startY] = true;

int[] dx = {-1, 1, 0, 0};
int[] dy = {0, 0, -1, 1};

while (!queue.isEmpty()) {
    int[] cur = queue.poll();
    int x = cur[0], y = cur[1];

    for (int d = 0; d < 4; d++) {
        int nx = x + dx[d];
        int ny = y + dy[d];

        if (nx < 0 || nx >= n || ny < 0 || ny >= m) continue; // 범위 밖
        if (visited[nx][ny]) continue;                          // 이미 방문
        if (grid[nx][ny] == 0) continue;                        // 벽 등 조건

        visited[nx][ny] = true;
        queue.offer(new int[]{nx, ny});
    }
}
```

---

### 3-4. 그리드에서 거리 구하기 (BFS)

```java
int[][] dist = new int[n][m];
for (int[] row : dist) Arrays.fill(row, -1); // -1 = 미방문

dist[startX][startY] = 0;
Queue<int[]> queue = new LinkedList<>();
queue.offer(new int[]{startX, startY});

while (!queue.isEmpty()) {
    int[] cur = queue.poll();
    int x = cur[0], y = cur[1];

    for (int d = 0; d < 4; d++) {
        int nx = x + dx[d];
        int ny = y + dy[d];
        if (nx < 0 || nx >= n || ny < 0 || ny >= m) continue;
        if (dist[nx][ny] != -1) continue;
        if (grid[nx][ny] == 0) continue;

        dist[nx][ny] = dist[x][y] + 1;
        queue.offer(new int[]{nx, ny});
    }
}
```

---

## 4. 자주 쓰는 Java 자료구조 초기화

### 4-1. Stack / Queue / Deque

```java
// 스택
Stack<Integer> stack = new Stack<>();
// 또는 (Deque 권장, Stack 클래스는 레거시)
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);    // 삽입
stack.pop();      // 꺼내기
stack.peek();     // 확인만

// 큐
Queue<Integer> queue = new LinkedList<>();
queue.offer(1);   // 삽입
queue.poll();     // 꺼내기
queue.peek();     // 확인만

// 덱 (양쪽에서 삽입/삭제)
Deque<Integer> deque = new ArrayDeque<>();
deque.offerFirst(1);  // 앞에 삽입
deque.offerLast(2);   // 뒤에 삽입
deque.pollFirst();    // 앞에서 꺼내기
deque.pollLast();     // 뒤에서 꺼내기
```

---

### 4-2. PriorityQueue (최소 힙 / 최대 힙)

```java
// 최소 힙 (기본)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// 최대 힙
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());

// 다익스트라용: {거리, 정점} 쌍, 거리 기준 최소 힙
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
pq.offer(new int[]{0, start}); // {거리, 정점}

// 다중 기준 정렬: 1차 오름차순, 2차 내림차순
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> {
    if (a[0] != b[0]) return a[0] - b[0];
    return b[1] - a[1];
});
```

---

### 4-3. HashMap / HashSet

```java
// HashMap: 키-값 쌍 저장
Map<Integer, Integer> map = new HashMap<>();
map.put(key, value);
map.getOrDefault(key, 0);            // 키 없으면 기본값 반환
map.merge(key, 1, Integer::sum);     // 빈도 카운팅에 유용
map.computeIfAbsent(key, k -> new ArrayList<>()).add(value);

// HashSet: 중복 제거, 존재 여부 O(1) 확인
Set<Integer> set = new HashSet<>();
set.add(value);
set.contains(value);

// TreeMap: 키 기준 정렬 (자동)
TreeMap<Integer, Integer> treeMap = new TreeMap<>();
treeMap.firstKey();   // 최솟값 키
treeMap.lastKey();    // 최댓값 키

// TreeSet: 정렬된 집합
TreeSet<Integer> treeSet = new TreeSet<>();
treeSet.lower(x);    // x 미만 최대
treeSet.higher(x);   // x 초과 최소
treeSet.floor(x);    // x 이하 최대
treeSet.ceiling(x);  // x 이상 최소
```

---

### 4-4. StringBuilder (출력 최적화)

대량 출력 시 `System.out.println`을 반복하면 시간 초과가 날 수 있다.

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < n; i++) {
    sb.append(result[i]).append('\n');
}
System.out.print(sb);
```

---

### 4-5. 배열 초기화 패턴

```java
// 1차원 배열 특정 값으로 초기화
int[] arr = new int[n];
Arrays.fill(arr, -1);           // 모두 -1로

// 2차원 배열 특정 값으로 초기화
int[][] arr = new int[n][m];
for (int[] row : arr) {
    Arrays.fill(row, Integer.MAX_VALUE);
}

// boolean 배열 (기본값 false)
boolean[] visited = new boolean[n + 1];

// 큰 수 초기화 (INF)
final int INF = (int) 1e9;      // 10억 (int 범위 내)
final long INF = (long) 1e18;   // long 범위
```

---

## 5. 입력 속도 비교

| 방식 | 속도 | 추천 상황 |
|------|------|----------|
| `Scanner` | 느림 | 입력이 적을 때 (≤ 10만 줄), 간편함 |
| `BufferedReader` + `StringTokenizer` | 빠름 | 입력이 많을 때 (10만+ 줄), 코테 권장 |

```java
// === Scanner (간편) ===
Scanner sc = new Scanner(System.in);
int n = sc.nextInt();
String s = sc.next();

// === BufferedReader (빠름, 코테 권장) ===
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
int n = Integer.parseInt(br.readLine().trim());

// 한 줄에 여러 값
StringTokenizer st = new StringTokenizer(br.readLine());
int a = Integer.parseInt(st.nextToken());
int b = Integer.parseInt(st.nextToken());
```

---

## 한눈에 보는 선택 가이드

```
그래프가 주어졌다
├─ 정점 수 적고 (≤500), 모든 쌍 필요 → 인접 행렬
├─ 간선 정렬이 필요 (크루스칼)       → 간선 리스트
├─ 정점이 문자열이나 불규칙           → 인접 리스트 (Map)
└─ 그 외 대부분                      → 인접 리스트 (배열) ✅ 기본

트리가 주어졌다
├─ 부모만 필요                       → 부모 배열
├─ 이진 트리 명시                    → left/right 배열
└─ 일반 트리                         → 인접 리스트 (배열)

격자가 주어졌다
├─ 숫자                             → int[][]
├─ 문자                             → char[][] + toCharArray()
└─ BFS/DFS 탐색                     → dx/dy 배열 + visited
```
