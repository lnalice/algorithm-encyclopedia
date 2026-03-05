# 구간합 (Prefix Sum, 누적합)

## 분류
- **카테고리**: 배열 기법
- **관련 알고리즘**: [세그먼트 트리](../data-structures/segment-tree.md), [슬라이딩 윈도우](../array-techniques/sliding-window.md), [투 포인터](../array-techniques/two-pointer.md)

## 개요
구간합(Prefix Sum)은 배열의 **누적합을 미리 계산해 두고**, 임의 구간 `[l, r]`의 합을 \(O(1)\)에 구하는 기법이다. 전처리에 \(O(N)\)이 소요되지만, 이후 구간 합 질의를 상수 시간에 답할 수 있어 갱신 없이 질의만 반복되는 상황에서 매우 효율적이다. 2차원으로 확장하면 직사각형 영역의 합도 \(O(1)\)에 구할 수 있다.

## 핵심 로직

### 1차원 누적합
1. **전처리**: 누적합 배열 `prefix[i] = arr[0] + arr[1] + ... + arr[i]`를 계산한다.
   - `prefix[0] = arr[0]`
   - `prefix[i] = prefix[i-1] + arr[i]`
2. **구간 합 질의**: 구간 `[l, r]`의 합을 다음과 같이 \(O(1)\)에 구한다.
   - `l == 0`일 때: `prefix[r]`
   - `l > 0`일 때: `prefix[r] - prefix[l-1]`

### 2차원 누적합
1. **전처리**: `prefix[i][j]` = 왼쪽 위 `(0,0)`에서 `(i,j)`까지의 직사각형 영역 합
   - **포함-배제 원리** 적용:
   - `prefix[i][j] = arr[i][j] + prefix[i-1][j] + prefix[i][j-1] - prefix[i-1][j-1]`
2. **영역 합 질의**: `(r1,c1)` ~ `(r2,c2)` 직사각형 영역의 합:
   - `prefix[r2][c2] - prefix[r1-1][c2] - prefix[r2][c1-1] + prefix[r1-1][c1-1]`
   - 마찬가지로 **포함-배제 원리**로 네 영역을 조합한다.

## 언제 사용하는가?
- **구간 합 쿼리가 여러 번 주어지고, 배열 값의 갱신이 없는 경우** (핵심 조건)
- **부분 배열의 합**: 연속된 구간의 합을 빠르게 구해야 할 때
- **2차원 영역의 합**: 격자에서 직사각형 영역의 합을 구할 때
- **특정 합을 가지는 부분 배열 개수 세기**: `prefix[r] - prefix[l-1] = target`을 변형하여 활용
- **차이 배열(Difference Array)**: 구간에 값을 일괄 추가하고 최종 결과를 구할 때 (누적합의 역 연산)

> **주의**: 배열의 값이 중간에 변경되면 누적합 배열 전체를 다시 계산해야 하므로 \(O(N)\)이 소요된다. 갱신이 빈번한 경우에는 [세그먼트 트리](../data-structures/segment-tree.md)를 사용해야 한다.

## Java 구현

```java
public class PrefixSum {

    // ========================================
    // 1. 1차원 누적합
    // ========================================

    /**
     * 1차원 누적합 배열을 생성한다.
     * prefix[i] = arr[0] + arr[1] + ... + arr[i]
     */
    static long[] buildPrefix1D(int[] arr) {
        int n = arr.length;
        long[] prefix = new long[n];
        prefix[0] = arr[0];
        for (int i = 1; i < n; i++) {
            prefix[i] = prefix[i - 1] + arr[i];
        }
        return prefix;
    }

    /**
     * 1차원 구간 합 질의: arr[l] + arr[l+1] + ... + arr[r]
     * @param prefix 누적합 배열
     * @param l 시작 인덱스 (포함)
     * @param r 끝 인덱스 (포함)
     * @return 구간 [l, r]의 합
     */
    static long rangeSum1D(long[] prefix, int l, int r) {
        if (l == 0) return prefix[r];
        return prefix[r] - prefix[l - 1];
    }

    // ========================================
    // 2. 2차원 누적합
    // ========================================

    /**
     * 2차원 누적합 배열을 생성한다. (1-based 인덱싱 사용)
     * prefix[i][j] = (1,1)에서 (i,j)까지의 직사각형 영역 합
     *
     * 편의를 위해 0번 행/열은 0으로 패딩하여 경계 처리를 단순화한다.
     */
    static long[][] buildPrefix2D(int[][] arr) {
        int rows = arr.length;
        int cols = arr[0].length;
        // 1-based 인덱싱: 0번 행/열은 0으로 패딩
        long[][] prefix = new long[rows + 1][cols + 1];

        for (int i = 1; i <= rows; i++) {
            for (int j = 1; j <= cols; j++) {
                // 포함-배제 원리로 누적합 계산
                prefix[i][j] = arr[i - 1][j - 1]
                        + prefix[i - 1][j]
                        + prefix[i][j - 1]
                        - prefix[i - 1][j - 1];
            }
        }
        return prefix;
    }

    /**
     * 2차원 구간 합 질의: (r1,c1)에서 (r2,c2)까지의 직사각형 영역 합
     * 입력 좌표는 0-based이며, 내부적으로 1-based로 변환하여 처리한다.
     */
    static long rangeSum2D(long[][] prefix, int r1, int c1, int r2, int c2) {
        // 0-based → 1-based 변환
        r1++; c1++; r2++; c2++;
        // 포함-배제 원리로 영역 합 계산
        return prefix[r2][c2]
                - prefix[r1 - 1][c2]
                - prefix[r2][c1 - 1]
                + prefix[r1 - 1][c1 - 1];
    }

    public static void main(String[] args) {
        // === 1차원 누적합 예시 ===
        int[] arr = {2, 4, 6, 8, 10};
        long[] prefix1D = buildPrefix1D(arr);

        // 구간 [1, 3]의 합: 4 + 6 + 8 = 18
        System.out.println("1차원 구간 [1, 3]의 합: " + rangeSum1D(prefix1D, 1, 3));

        // 구간 [0, 4]의 합: 2 + 4 + 6 + 8 + 10 = 30
        System.out.println("1차원 구간 [0, 4]의 합: " + rangeSum1D(prefix1D, 0, 4));

        // === 2차원 누적합 예시 ===
        int[][] grid = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };
        long[][] prefix2D = buildPrefix2D(grid);

        // (0,0) ~ (1,1) 영역의 합: 1 + 2 + 4 + 5 = 12
        System.out.println("2차원 영역 (0,0)~(1,1)의 합: " + rangeSum2D(prefix2D, 0, 0, 1, 1));

        // (1,1) ~ (2,2) 영역의 합: 5 + 6 + 8 + 9 = 28
        System.out.println("2차원 영역 (1,1)~(2,2)의 합: " + rangeSum2D(prefix2D, 1, 1, 2, 2));

        // 전체 영역의 합: 1+2+3+4+5+6+7+8+9 = 45
        System.out.println("2차원 전체 영역의 합: " + rangeSum2D(prefix2D, 0, 0, 2, 2));
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 연산 | 시간 복잡도 | 비고 |
|------|-----------|------|
| 1차원 전처리 | \(O(N)\) | 누적합 배열 생성 |
| 1차원 구간 합 질의 | \(O(1)\) | 뺄셈 한 번 |
| 2차원 전처리 | \(O(N \times M)\) | N×M 격자의 누적합 생성 |
| 2차원 영역 합 질의 | \(O(1)\) | 포함-배제로 상수 연산 |

- **공간 복잡도**:
  - 1차원: \(O(N)\) — 누적합 배열
  - 2차원: \(O(N \times M)\) — 누적합 2차원 배열

## 관련 문제 유형
- **구간 합 쿼리**: 주어진 배열에서 여러 구간의 합을 빠르게 구하는 문제
- **부분 배열 합**: 합이 K인 부분 배열의 개수, 최대 부분 배열 합 등
- **2차원 영역 합**: 격자에서 직사각형 영역의 합을 구하는 문제
- **차이 배열**: 구간에 값을 더하는 쿼리를 효율적으로 처리 (누적합의 역 연산)
- **균형점 찾기**: 왼쪽 합과 오른쪽 합이 같은 인덱스 찾기
