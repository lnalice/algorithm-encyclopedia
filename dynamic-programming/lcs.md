# LCS (최장 공통 부분 수열, Longest Common Subsequence)

## 분류
- **카테고리**: 동적 프로그래밍
- **관련 알고리즘**: [DP](./dp.md), [LIS](./lis.md)

## 개요
LCS(Longest Common Subsequence)는 두 수열(문자열)에서 **순서를 유지하면서 공통으로 나타나는 부분 수열 중 가장 긴 것의 길이**를 구하는 문제이다. 2차원 DP 테이블을 사용하여 O(NM)에 풀 수 있으며, 역추적을 통해 실제 LCS 문자열도 복원할 수 있다. diff 알고리즘, DNA 서열 비교, 편집 거리 등 다양한 분야의 기반이 되는 핵심 알고리즘이다.

## 핵심 로직

### 상태 정의
`dp[i][j]` = 문자열 A의 처음 i글자와 문자열 B의 처음 j글자 사이의 LCS 길이

### 점화식

```
if A[i-1] == B[j-1]:
    dp[i][j] = dp[i-1][j-1] + 1     ← 두 문자가 같으면 LCS에 포함
else:
    dp[i][j] = max(dp[i-1][j], dp[i][j-1])  ← 다르면 둘 중 하나를 건너뜀
```

### 직관적 이해

- **문자가 같을 때** (`A[i-1] == B[j-1]`): 이 문자는 공통 부분 수열에 포함시킬 수 있다. 양쪽 모두 한 칸 이전 상태 `dp[i-1][j-1]`에서 길이 1을 더한다.
- **문자가 다를 때**: 이 문자를 LCS에 포함시킬 수 없다. 따라서 A에서 한 글자를 빼거나(`dp[i-1][j]`), B에서 한 글자를 빼는(`dp[i][j-1]`) 두 경우 중 더 긴 쪽을 택한다.

### 예시: A = "ABCBDAB", B = "BDCAB"

|   | "" | B | D | C | A | B |
|---|---|---|---|---|---|---|
| "" | 0 | 0 | 0 | 0 | 0 | 0 |
| A | 0 | 0 | 0 | 0 | **1** | 1 |
| B | 0 | **1** | 1 | 1 | 1 | **2** |
| C | 0 | 1 | 1 | **2** | 2 | 2 |
| B | 0 | 1 | 1 | 2 | 2 | **3** |
| D | 0 | 1 | **2** | 2 | 2 | 3 |
| A | 0 | 1 | 2 | 2 | **3** | 3 |
| B | 0 | 1 | 2 | 2 | 3 | **4** |

→ LCS 길이 = 4, LCS = "BCAB"

### 공간 최적화

LCS 길이만 필요한 경우, `dp[i][j]`는 이전 행 `dp[i-1][...]`만 참조하므로 **1차원 배열 2개** (또는 1개 + 임시 변수)로 공간을 O(min(N, M))으로 줄일 수 있다. 단, 역추적이 필요한 경우 2차원 테이블을 유지해야 한다.

## 언제 사용하는가?
- **두 문자열의 공통 부분 수열** 길이 또는 문자열을 구하는 문제
- **diff 알고리즘**: 두 파일의 차이점을 찾을 때 LCS를 기반으로 변경 부분을 파악
- **DNA 서열 비교**: 두 유전자 서열의 유사도 측정
- **편집 거리 관련 문제**: LCS 길이를 알면 편집 거리 계산 가능 (`편집거리 ≥ (N + M) - 2 * LCS`)
- **부분 수열 판별**: 한 문자열이 다른 문자열의 부분 수열인지 확인

## Java 구현

```java
import java.util.*;

public class LCS {

    // ========================================
    // 방법 1: LCS 길이 구하기 (2차원 DP)
    // dp[i][j] = A의 처음 i글자와 B의 처음 j글자의 LCS 길이
    // ========================================
    static int lcsLength(String a, String b) {
        int n = a.length();
        int m = b.length();
        int[][] dp = new int[n + 1][m + 1];

        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                if (a.charAt(i - 1) == b.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1] + 1; // 같은 문자 → LCS에 포함
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]); // 다른 문자 → 둘 중 최대
                }
            }
        }

        return dp[n][m];
    }

    // ========================================
    // 방법 2: LCS 문자열 역추적 (Traceback)
    // DP 테이블을 거꾸로 추적하여 실제 LCS 문자열을 복원
    // ========================================
    static String lcsTraceback(String a, String b) {
        int n = a.length();
        int m = b.length();
        int[][] dp = new int[n + 1][m + 1];

        // DP 테이블 채우기
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                if (a.charAt(i - 1) == b.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }

        // 역추적: dp[n][m]에서 출발하여 dp[0][0]까지 이동
        StringBuilder sb = new StringBuilder();
        int i = n, j = m;

        while (i > 0 && j > 0) {
            if (a.charAt(i - 1) == b.charAt(j - 1)) {
                sb.append(a.charAt(i - 1)); // 공통 문자를 결과에 추가
                i--;
                j--;
            } else if (dp[i - 1][j] > dp[i][j - 1]) {
                i--; // 위쪽 셀이 더 크면 A 방향으로 이동
            } else {
                j--; // 왼쪽 셀이 더 크면 B 방향으로 이동
            }
        }

        return sb.reverse().toString(); // 역순으로 추가했으므로 뒤집기
    }

    // ========================================
    // 공간 최적화 버전: 1차원 DP (길이만 구하기)
    // 이전 행만 참조하므로 O(min(N, M)) 공간으로 가능
    // ========================================
    static int lcsLengthOptimized(String a, String b) {
        // 짧은 문자열을 열(column)로 사용하여 공간 절약
        if (a.length() < b.length()) {
            String temp = a;
            a = b;
            b = temp;
        }

        int m = b.length();
        int[] prev = new int[m + 1];
        int[] curr = new int[m + 1];

        for (int i = 1; i <= a.length(); i++) {
            for (int j = 1; j <= m; j++) {
                if (a.charAt(i - 1) == b.charAt(j - 1)) {
                    curr[j] = prev[j - 1] + 1;
                } else {
                    curr[j] = Math.max(prev[j], curr[j - 1]);
                }
            }
            // prev와 curr 교체
            int[] temp = prev;
            prev = curr;
            curr = temp;
            Arrays.fill(curr, 0);
        }

        return prev[m];
    }

    public static void main(String[] args) {
        String a = "ABCBDAB";
        String b = "BDCAB";

        // LCS 길이
        System.out.println("=== LCS 길이 ===");
        System.out.println("A = \"" + a + "\"");
        System.out.println("B = \"" + b + "\"");
        System.out.println("LCS 길이: " + lcsLength(a, b));

        // LCS 문자열 역추적
        System.out.println("\n=== LCS 역추적 ===");
        String lcs = lcsTraceback(a, b);
        System.out.println("LCS 문자열: \"" + lcs + "\"");
        System.out.println("LCS 길이: " + lcs.length());

        // 공간 최적화 버전
        System.out.println("\n=== 공간 최적화 LCS ===");
        System.out.println("LCS 길이: " + lcsLengthOptimized(a, b));

        // 추가 예제
        String c = "ACAYKP";
        String d = "CAPCAK";
        System.out.println("\n=== 추가 예제 ===");
        System.out.println("A = \"" + c + "\", B = \"" + d + "\"");
        System.out.println("LCS 길이: " + lcsLength(c, d));
        System.out.println("LCS 문자열: \"" + lcsTraceback(c, d) + "\"");
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 방법 | 시간 복잡도 | 공간 복잡도 |
|------|-----------|-----------|
| 2차원 DP (기본) | \(O(N \times M)\) | \(O(N \times M)\) — dp 테이블 |
| 2차원 DP + 역추적 | \(O(N \times M)\) | \(O(N \times M)\) — dp 테이블 |
| 1차원 DP (공간 최적화) | \(O(N \times M)\) | \(O(\min(N, M))\) — 1차원 배열 |

- `N`: 문자열 A의 길이, `M`: 문자열 B의 길이
- 역추적은 dp 테이블 순회이므로 O(N + M) 추가 시간

## 관련 문제 유형
- **최장 공통 부분 수열 길이**: 기본 LCS 문제
- **LCS 문자열 복원**: 역추적을 통해 실제 공통 부분 수열 출력
- **최단 공통 슈퍼시퀀스**: 두 문자열을 합치되 LCS를 공유 → 길이 = N + M - LCS
- **편집 거리 (Edit Distance)**: LCS와 밀접한 관계 — 삽입/삭제만 허용 시 편집 거리 = N + M - 2×LCS
- **diff 도구**: 두 텍스트 파일의 차이를 LCS 기반으로 계산
- **팰린드롬 부분 수열**: 문자열과 그 역순의 LCS = 최장 팰린드롬 부분 수열
