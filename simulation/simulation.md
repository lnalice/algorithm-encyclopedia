# 시뮬레이션 / 구현

## 분류
- **카테고리**: 시뮬레이션 / 구현
- **관련 알고리즘**: [BFS](../graph-traversal/bfs.md), [DFS](../graph-traversal/dfs.md)

## 개요
시뮬레이션(구현) 문제는 문제에서 주어진 조건과 규칙을 **그대로 코드로 옮기는** 유형이다. 특별한 알고리즘 지식보다는 정확한 구현력, 꼼꼼한 예외 처리, 그리고 체계적인 상태 관리가 핵심이다. 네이버 코딩 테스트에서는 **매 시험마다 1~2문제가 반드시 시뮬레이션/구현 문제로 출제**되므로, 이 유형에 익숙해지는 것이 합격의 필수 조건이다.

## 핵심 로직

시뮬레이션 문제는 정형화된 알고리즘이 없지만, 다음과 같은 **공통 패턴과 테크닉**이 반복적으로 등장한다.

### 1. 방향 배열 (dx, dy) 활용법

2D 격자에서 상하좌우 이동은 방향 배열로 깔끔하게 처리한다.

```
// 상, 하, 좌, 우 (북, 남, 서, 동)
int[] dx = {-1, 1, 0, 0};
int[] dy = {0, 0, -1, 1};

// 8방향이 필요한 경우 (상, 하, 좌, 우, 대각선 4방향)
int[] dx8 = {-1, -1, -1, 0, 0, 1, 1, 1};
int[] dy8 = {-1, 0, 1, -1, 1, -1, 0, 1};
```

> **팁**: 방향에 인덱스를 부여하면 `(인덱스 + 1) % 4`로 시계 방향 90도 회전, `(인덱스 + 3) % 4`로 반시계 방향 90도 회전을 구현할 수 있다.

### 2. 좌표계 주의사항 (행/열 vs x/y)

| | 수학 좌표 (x, y) | 배열 좌표 (r, c) |
|---|---|---|
| **가로** | x (오른쪽 증가) | c (열, 오른쪽 증가) |
| **세로** | y (위쪽 증가) | r (행, **아래쪽** 증가) |
| **접근** | `grid[y][x]` | `grid[r][c]` |

> **핵심**: 문제가 `(x, y)`를 줄 때 배열에서는 `grid[y][x]`로 접근해야 한다. 이것을 혼동하면 디버깅이 매우 어려워진다. **일관되게 `(행, 열)` 또는 `(r, c)` 표기를 사용**하는 것을 추천한다.

### 3. 범위 체크 패턴

격자 밖으로 나가는지 확인하는 헬퍼 메서드를 항상 준비한다.

```
static boolean inRange(int r, int c, int N, int M) {
    return r >= 0 && r < N && c >= 0 && c < M;
}
```

### 4. 상태 관리 전략

- **단순 변수**: 위치, 방향, 점수 등 단일 값은 변수로 관리
- **배열/리스트**: 격자 상태, 여러 객체의 위치 등은 배열이나 리스트로 관리
- **클래스/구조체**: 복잡한 상태(위치+방향+체력 등)는 별도 클래스로 묶어서 관리
- **상태 복사**: 동시에 일어나는 변화(예: 게임 한 턴)는 이전 상태를 복사해놓고 새 상태에 반영

## 언제 사용하는가?
- **규칙 기반 문제**: 문제 지문에 "다음 규칙에 따라 동작한다"와 같은 설명이 있는 경우
- **2D 격자 조작**: 로봇 이동, 뱀 게임, 미세먼지 확산, 낚시왕 등 격자 위에서 객체가 움직이는 문제
- **게임 시뮬레이션**: 주사위 굴리기, 사다리 타기, 카드 게임 등 게임 규칙을 구현하는 문제
- **프로세스 시뮬레이션**: 대기열 관리, 작업 스케줄링, 프린터 큐 등
- **시간 순서대로 진행**: "1초마다", "매 턴마다" 등 시간 흐름에 따라 상태가 변하는 문제

## Java 구현

### 예시 1: 2차원 격자 로봇 이동 시뮬레이션

로봇이 N×M 격자 위에서 명령어에 따라 이동한다.
- `G`: 현재 방향으로 한 칸 전진 (벽이면 무시)
- `L`: 왼쪽(반시계 방향)으로 90도 회전
- `R`: 오른쪽(시계 방향)으로 90도 회전

```java
import java.util.*;

public class RobotSimulation {

    // 방향: 0=북, 1=동, 2=남, 3=서
    static int[] dr = {-1, 0, 1, 0};
    static int[] dc = {0, 1, 0, -1};
    static String[] dirName = {"북", "동", "남", "서"};

    static boolean inRange(int r, int c, int N, int M) {
        return r >= 0 && r < N && c >= 0 && c < M;
    }

    static int[][] simulate(int N, int M, int[][] grid, int startR, int startC,
                            int startDir, String commands) {
        int r = startR;
        int c = startC;
        int dir = startDir;

        // 방문 경로 기록 (0=미방문, 방문 순서로 기록)
        int[][] visited = new int[N][M];
        int step = 1;
        visited[r][c] = step++;

        for (char cmd : commands.toCharArray()) {
            switch (cmd) {
                case 'G': // 전진
                    int nr = r + dr[dir];
                    int nc = c + dc[dir];
                    // 범위 안이고 벽(1)이 아닌 경우에만 이동
                    if (inRange(nr, nc, N, M) && grid[nr][nc] != 1) {
                        r = nr;
                        c = nc;
                    }
                    break;
                case 'L': // 반시계 90도 회전
                    dir = (dir + 3) % 4;
                    break;
                case 'R': // 시계 90도 회전
                    dir = (dir + 1) % 4;
                    break;
            }
            if (cmd == 'G') {
                visited[r][c] = step++;
            }
        }

        System.out.println("최종 위치: (" + r + ", " + c + "), 방향: " + dirName[dir]);
        return visited;
    }

    public static void main(String[] args) {
        int N = 5, M = 5;

        // 격자: 0=빈칸, 1=벽
        int[][] grid = {
            {0, 0, 0, 0, 0},
            {0, 1, 1, 0, 0},
            {0, 0, 0, 0, 0},
            {0, 0, 1, 1, 0},
            {0, 0, 0, 0, 0}
        };

        // 시작: (0,0) 위치, 동쪽(1) 방향
        String commands = "GGGRGGRGLGG";

        System.out.println("=== 로봇 이동 시뮬레이션 ===");
        System.out.println("격자 크기: " + N + "×" + M);
        System.out.println("시작 위치: (0, 0), 방향: 동");
        System.out.println("명령어: " + commands);
        System.out.println();

        int[][] visited = simulate(N, M, grid, 0, 0, 1, commands);

        System.out.println("\n방문 순서:");
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < M; j++) {
                if (grid[i][j] == 1) {
                    System.out.printf(" ## ");
                } else if (visited[i][j] > 0) {
                    System.out.printf("%3d ", visited[i][j]);
                } else {
                    System.out.printf("  . ");
                }
            }
            System.out.println();
        }
    }
}
```

### 예시 2: 큐를 활용한 프로세스 스케줄링 시뮬레이션

라운드 로빈(Round Robin) 방식의 프로세스 스케줄링을 시뮬레이션한다.
- 각 프로세스는 (이름, 도착 시간, 실행 시간)을 가진다.
- 타임 퀀텀만큼 실행 후 남은 시간이 있으면 큐 뒤로 이동한다.

```java
import java.util.*;

public class ProcessScheduling {

    static class Process {
        String name;
        int arrivalTime;    // 도착 시간
        int burstTime;      // 총 실행 시간
        int remainingTime;  // 남은 실행 시간
        int completionTime; // 완료 시간
        int waitingTime;    // 대기 시간

        Process(String name, int arrivalTime, int burstTime) {
            this.name = name;
            this.arrivalTime = arrivalTime;
            this.burstTime = burstTime;
            this.remainingTime = burstTime;
        }
    }

    static void roundRobinScheduling(List<Process> processes, int timeQuantum) {
        Queue<Process> readyQueue = new LinkedList<>();
        List<String> timeline = new ArrayList<>(); // 간트 차트용

        // 도착 시간 기준 정렬
        processes.sort(Comparator.comparingInt(p -> p.arrivalTime));

        int currentTime = 0;
        int idx = 0;            // 아직 도착하지 않은 프로세스 인덱스
        int completed = 0;
        int totalProcesses = processes.size();

        // 첫 번째 프로세스 도착 시간으로 초기화
        if (processes.get(0).arrivalTime > 0) {
            currentTime = processes.get(0).arrivalTime;
        }

        // 처음 도착한 프로세스들을 큐에 추가
        while (idx < totalProcesses && processes.get(idx).arrivalTime <= currentTime) {
            readyQueue.offer(processes.get(idx));
            idx++;
        }

        while (completed < totalProcesses) {
            if (readyQueue.isEmpty()) {
                // 대기 중인 프로세스가 없으면 다음 프로세스 도착까지 시간 점프
                if (idx < totalProcesses) {
                    currentTime = processes.get(idx).arrivalTime;
                    while (idx < totalProcesses && processes.get(idx).arrivalTime <= currentTime) {
                        readyQueue.offer(processes.get(idx));
                        idx++;
                    }
                }
                continue;
            }

            Process current = readyQueue.poll();

            // 타임 퀀텀 또는 남은 시간 중 작은 값만큼 실행
            int execTime = Math.min(timeQuantum, current.remainingTime);
            current.remainingTime -= execTime;
            currentTime += execTime;
            timeline.add(current.name + "(" + execTime + "초)");

            // 실행 중 새로 도착한 프로세스를 큐에 추가
            while (idx < totalProcesses && processes.get(idx).arrivalTime <= currentTime) {
                readyQueue.offer(processes.get(idx));
                idx++;
            }

            if (current.remainingTime == 0) {
                // 프로세스 완료
                completed++;
                current.completionTime = currentTime;
                current.waitingTime = current.completionTime - current.arrivalTime - current.burstTime;
            } else {
                // 아직 남았으면 큐 뒤로 이동
                readyQueue.offer(current);
            }
        }

        // 결과 출력
        System.out.println("실행 순서: " + String.join(" → ", timeline));
        System.out.println();
        System.out.printf("%-6s %-8s %-8s %-8s %-8s%n",
                "이름", "도착시간", "실행시간", "완료시간", "대기시간");
        System.out.println("─".repeat(42));

        double totalWaiting = 0;
        for (Process p : processes) {
            System.out.printf("%-6s %-8d %-8d %-8d %-8d%n",
                    p.name, p.arrivalTime, p.burstTime, p.completionTime, p.waitingTime);
            totalWaiting += p.waitingTime;
        }
        System.out.printf("%n평균 대기 시간: %.2f초%n", totalWaiting / totalProcesses);
    }

    public static void main(String[] args) {
        List<Process> processes = new ArrayList<>();
        processes.add(new Process("P1", 0, 8));
        processes.add(new Process("P2", 1, 4));
        processes.add(new Process("P3", 2, 9));
        processes.add(new Process("P4", 3, 5));

        int timeQuantum = 3;

        System.out.println("=== 라운드 로빈 스케줄링 시뮬레이션 ===");
        System.out.println("타임 퀀텀: " + timeQuantum + "초");
        System.out.println();

        roundRobinScheduling(processes, timeQuantum);
    }
}
```

## 시간 복잡도 / 공간 복잡도
| | 복잡도 |
|---|---|
| **시간 복잡도** | 문제마다 다름 — 일반적으로 \(O(N \times M)\) 또는 \(O(T \times N)\) (T=시뮬레이션 턴 수) |
| **공간 복잡도** | 문제마다 다름 — 일반적으로 격자 크기 \(O(N \times M)\) 또는 객체 수 \(O(N)\) |

> 시뮬레이션 문제는 시간/공간 복잡도보다 **구현의 정확성**이 핵심이다. 제한 시간 내에 정확하게 동작하는 코드를 작성하는 것이 가장 중요하다.

## 시뮬레이션 문제를 잘 풀기 위한 팁

### 1. 문제를 읽기 전에 구현 틀을 머릿속에 준비하라
- 방향 배열, 범위 체크, 격자 입출력 등의 **보일러플레이트 코드**를 미리 외워두자.
- 시험 시작 직후 이 코드들을 바로 작성할 수 있어야 한다.

### 2. 문제를 단계별로 분해하라
- 한 번에 전체를 구현하지 말고, **한 턴(한 스텝)의 동작**을 먼저 완벽히 구현한다.
- 그 후 반복 구조를 씌운다.

### 3. 디버깅을 위한 출력을 미리 작성하라
- 매 턴마다 격자 상태를 출력하는 `printGrid()` 메서드를 항상 준비한다.
- 제출 전에 주석 처리하면 된다.

### 4. 동시성 처리에 주의하라
- "모든 객체가 동시에 이동한다"는 표현이 있으면, **이전 상태를 복사**해놓고 새 상태에 반영해야 한다.
- 순서대로 처리하면 이미 이동한 객체가 다른 객체의 이동에 영향을 줄 수 있다.

### 5. 인덱스 실수를 방지하라
- 0-indexed인지 1-indexed인지 문제를 꼼꼼히 확인한다.
- `(행, 열)` 순서와 `(x, y)` 순서를 혼동하지 않는다.
- 범위 체크는 반드시 헬퍼 메서드로 분리한다.

## 관련 문제 유형
- **로봇 이동 / 뱀 게임**: 격자 위에서 방향 전환 + 이동 + 충돌 처리
- **미세먼지 확산**: 동시 확산 후 공기청정기 작동 (동시성 처리 필요)
- **낚시왕 / 상어 문제**: 여러 객체가 동시 이동 + 충돌 규칙
- **주사위 굴리기**: 주사위 상태를 배열로 관리하며 이동
- **톱니바퀴 회전**: 연쇄적인 상태 변화 처리
- **프린터 큐 / 작업 스케줄링**: 큐를 활용한 순서 시뮬레이션
- **2048 게임**: 격자 내 타일 이동 + 합치기 규칙
