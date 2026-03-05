# 백트래킹 (Backtracking)

## 분류
- **카테고리**: 완전 탐색 / 백트래킹
- **관련 알고리즘**: [DFS](../graph-traversal/dfs.md), [분할 정복](../divide-and-conquer/divide-and-conquer.md)

## 개요
백트래킹은 해를 찾기 위해 모든 가능성을 탐색하되, 현재 경로가 해가 될 수 없다고 판단되면 즉시 되돌아가(가지치기, Pruning) 불필요한 탐색을 줄이는 기법이다. DFS를 기반으로 해 공간 트리(State Space Tree)를 탐색하며, 유망하지 않은 분기를 조기에 차단하여 순수 완전 탐색보다 효율적으로 동작한다. 조합, 순열, 제약 조건 만족 문제(CSP) 등에서 핵심적으로 활용된다.

## 핵심 로직
1. **선택 (Choose)**: 현재 단계에서 가능한 선택지 중 하나를 고른다.
2. **유망성 검사 (Promising Check)**: 선택한 결과가 제약 조건을 만족하는지 확인한다.
   - **유망하면**: 다음 단계로 재귀 진행한다.
   - **유망하지 않으면**: 해당 선택을 취소하고 다른 선택지를 시도한다 (가지치기).
3. **해 도달 확인**: 모든 단계를 완료하면 해를 기록한다.
4. **되돌아가기 (Backtrack)**: 현재 선택을 취소하고 이전 상태로 복원한다.

> 핵심은 **가지치기(Pruning)**이다. 유망하지 않은 경로를 빠르게 잘라내는 조건을 잘 설계할수록 성능이 향상된다.

## 언제 사용하는가?
- **순열/조합 생성**: N개 중 R개를 뽑는 모든 경우를 탐색
- **N-Queens 문제**: N×N 체스판에 N개의 퀸을 서로 공격하지 않게 배치
- **스도쿠 풀기**: 빈 칸에 1-9 숫자를 규칙에 맞게 배치
- **부분 집합 합 (Subset Sum)**: 원소들의 부분집합 중 합이 특정 값이 되는 경우
- **문자열/경로 탐색**: 가능한 모든 경로를 생성하되 조건에 맞지 않으면 조기 종료
- **제약 조건 만족 문제 (CSP)**: 변수에 값을 할당하며 제약 조건을 모두 만족하는 해 탐색

## Java 구현
```java
import java.util.*;

public class Backtracking {

    static int n;         // 퀸의 수 (보드 크기)
    static int[] queens;  // queens[i] = i번째 행에 놓인 퀸의 열 번호
    static int count;     // 해의 개수

    // N-Queens 문제: N×N 체스판에 N개의 퀸을 서로 공격 불가능하게 배치
    static void solveNQueens(int row) {
        if (row == n) {
            // 모든 행에 퀸을 성공적으로 배치 → 해 발견
            count++;
            printBoard();
            return;
        }

        for (int col = 0; col < n; col++) {
            if (isPromising(row, col)) {
                queens[row] = col;         // 선택: row행 col열에 퀸 배치
                solveNQueens(row + 1);     // 다음 행으로 진행
                queens[row] = -1;          // 되돌아가기: 퀸 제거 (백트래킹)
            }
            // isPromising이 false면 이 열은 건너뜀 (가지치기)
        }
    }

    // 유망성 검사: (row, col)에 퀸을 놓을 수 있는지 확인
    static boolean isPromising(int row, int col) {
        for (int prevRow = 0; prevRow < row; prevRow++) {
            int prevCol = queens[prevRow];

            // 같은 열에 퀸이 있는지 확인
            if (prevCol == col) return false;

            // 대각선에 퀸이 있는지 확인 (행 차이 == 열 차이이면 대각선)
            if (Math.abs(prevRow - row) == Math.abs(prevCol - col)) return false;
        }
        return true;
    }

    // 현재 배치 상태를 체스판 형태로 출력
    static void printBoard() {
        System.out.println("--- 해 #" + count + " ---");
        for (int i = 0; i < n; i++) {
            StringBuilder sb = new StringBuilder();
            for (int j = 0; j < n; j++) {
                sb.append(queens[i] == j ? "Q " : ". ");
            }
            System.out.println(sb.toString().trim());
        }
        System.out.println();
    }

    // ========================================
    // 범용 백트래킹 틀: 부분집합 생성
    // ========================================
    static void generateSubsets(int[] arr, int index, List<Integer> current) {
        // 현재 부분집합 출력
        System.out.println(current);

        for (int i = index; i < arr.length; i++) {
            current.add(arr[i]);              // 선택
            generateSubsets(arr, i + 1, current); // 다음 원소로 진행
            current.remove(current.size() - 1);   // 백트래킹
        }
    }

    // ========================================
    // 범용 백트래킹 틀: 순열 생성
    // ========================================
    static void generatePermutations(int[] arr, boolean[] used, List<Integer> current) {
        if (current.size() == arr.length) {
            System.out.println(current);
            return;
        }

        for (int i = 0; i < arr.length; i++) {
            if (used[i]) continue; // 이미 사용된 원소 건너뜀

            used[i] = true;              // 선택
            current.add(arr[i]);
            generatePermutations(arr, used, current); // 다음 자리 채우기
            current.remove(current.size() - 1);       // 백트래킹
            used[i] = false;
        }
    }

    public static void main(String[] args) {
        // === N-Queens 문제 (N=4) ===
        n = 4;
        queens = new int[n];
        Arrays.fill(queens, -1);
        count = 0;

        System.out.println("=== N-Queens (N=" + n + ") ===\n");
        solveNQueens(0);
        System.out.println("총 해의 수: " + count + "\n");

        // === 부분집합 생성 ===
        System.out.println("=== 부분집합 생성 {1, 2, 3} ===");
        generateSubsets(new int[]{1, 2, 3}, 0, new ArrayList<>());

        // === 순열 생성 ===
        System.out.println("\n=== 순열 생성 {1, 2, 3} ===");
        generatePermutations(new int[]{1, 2, 3}, new boolean[3], new ArrayList<>());
    }
}
```

## 시간 복잡도 / 공간 복잡도
| | 복잡도 |
|---|---|
| **시간 복잡도 (최악)** | 문제에 따라 다름 — N-Queens: \(O(N!)\), 부분집합: \(O(2^N)\), 순열: \(O(N!)\) |
| **공간 복잡도** | \(O(N)\) — 재귀 깊이 (해 저장 시 추가 공간 필요) |

> 가지치기의 효과에 따라 실제 실행 시간은 최악 복잡도보다 훨씬 적을 수 있다. 좋은 가지치기 조건을 설계하는 것이 핵심이다.

## 관련 문제 유형
- **N-Queens**: N×N 보드에 퀸 배치
- **스도쿠 풀기**: 9×9 격자에 숫자 배치
- **순열/조합 생성**: 가능한 모든 경우 나열
- **부분집합 합 (Subset Sum)**: 합이 특정 값이 되는 부분집합 찾기
- **단어 검색 (Word Search)**: 2D 격자에서 문자열 경로 탐색
- **괄호 생성**: 올바른 괄호 조합 생성
- **색칠 문제 (Graph Coloring)**: 인접한 정점이 같은 색이 되지 않도록 색칠
