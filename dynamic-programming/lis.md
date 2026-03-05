# LIS (최장 증가 부분 수열, Longest Increasing Subsequence)

## 분류
- **카테고리**: 동적 프로그래밍
- **관련 알고리즘**: [DP](./dp.md), [이진 탐색](../search/binary-search.md), [LCS](./lcs.md)

## 개요
LIS(Longest Increasing Subsequence)는 주어진 수열에서 **순서를 유지하면서 원소가 순증가하는 부분 수열 중 가장 긴 것의 길이**를 구하는 문제이다. DP를 사용하면 O(N²), 이진 탐색을 결합하면 O(N log N)에 풀 수 있다. 코딩 테스트에서 매우 빈출되는 핵심 알고리즘 중 하나이다.

## 핵심 로직

### 방법 1: O(N²) DP 풀이

1. **상태 정의**: `dp[i]` = `arr[i]`를 **마지막 원소로 하는** LIS의 길이
2. **점화식**: 모든 `j < i`에 대해, `arr[j] < arr[i]`이면 `dp[i] = max(dp[i], dp[j] + 1)`
3. **초기값**: 모든 `dp[i] = 1` (자기 자신만으로도 길이 1의 LIS)
4. **정답**: `dp` 배열 전체에서 최댓값

> **직관**: 각 원소에 대해 "나보다 앞에 있고 나보다 작은 원소들 중, LIS가 가장 긴 것 뒤에 나를 붙인다"는 발상이다.

### 방법 2: O(N log N) 이진 탐색 풀이 (Patience Sorting)

1. **`tails` 배열 유지**: `tails[k]` = 길이가 `k+1`인 증가 부분 수열을 만들 때, 가능한 마지막 원소의 **최솟값**
2. **각 원소에 대해**:
   - `arr[i]`가 `tails`의 마지막 원소보다 크면 → `tails` 끝에 추가 (LIS 길이 확장)
   - 그렇지 않으면 → `tails`에서 `arr[i]` 이상인 첫 번째 위치를 **이진 탐색(lower bound)**으로 찾아 교체
3. **정답**: `tails` 배열의 길이가 LIS의 길이

> **`tails` 배열의 의미**: `tails`는 항상 정렬된 상태를 유지한다. `tails[k]`는 "길이 k+1인 증가 수열을 만들 때, 끝 값이 최대한 작아야 뒤에 더 많은 원소를 붙일 수 있다"는 그리디 아이디어를 반영한다. 예를 들어, 수열 `[3, 1, 2, 8, 5, 6]`에서:
> - `3` → tails = `[3]`
> - `1` → tails = `[1]` (3을 1로 교체 — 길이 1인 수열의 끝을 더 작게)
> - `2` → tails = `[1, 2]` (확장)
> - `8` → tails = `[1, 2, 8]` (확장)
> - `5` → tails = `[1, 2, 5]` (8을 5로 교체 — 길이 3인 수열의 끝을 더 작게)
> - `6` → tails = `[1, 2, 5, 6]` (확장) → **LIS 길이 = 4**
>
> **주의**: `tails` 배열 자체가 실제 LIS는 아니다. LIS의 **길이**만 정확하게 구할 수 있다.

### 두 방법의 비교

| 구분 | O(N²) DP | O(N log N) 이진 탐색 |
|------|----------|---------------------|
| **시간 복잡도** | O(N²) | O(N log N) |
| **공간 복잡도** | O(N) | O(N) |
| **구현 난이도** | 쉬움 | 보통 |
| **실제 LIS 복원** | 가능 (dp 역추적) | 추가 처리 필요 |
| **적용 기준** | N ≤ 5,000 정도 | N ≤ 100,000 이상 |

## 언제 사용하는가?
- **가장 긴 증가하는 부분 수열의 길이**를 구하는 문제
- **가장 적은 수의 감소하지 않는 부분 수열로 분할** (딜워스 정리: 최소 체인 분할 수 = 최장 반체인 길이)
- 전깃줄 문제 등 **교차하지 않는 최대 쌍** 구하기
- 가장 긴 감소 부분 수열 (배열을 뒤집어서 LIS 적용)
- 상자 쌓기, 러시아 인형 등 **정렬 후 LIS** 패턴

## Java 구현

```java
import java.util.*;

public class LIS {

    // ========================================
    // 방법 1: O(N²) DP 풀이
    // dp[i] = arr[i]를 마지막 원소로 하는 LIS 길이
    // ========================================
    static int lisDp(int[] arr) {
        int n = arr.length;
        int[] dp = new int[n];
        Arrays.fill(dp, 1); // 자기 자신만으로 길이 1

        int maxLen = 1;
        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (arr[j] < arr[i]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            maxLen = Math.max(maxLen, dp[i]);
        }

        return maxLen;
    }

    // ========================================
    // 방법 1 응용: O(N²) DP + 실제 LIS 역추적
    // ========================================
    static List<Integer> lisWithTraceback(int[] arr) {
        int n = arr.length;
        int[] dp = new int[n];
        int[] prev = new int[n]; // 역추적을 위한 이전 인덱스 저장
        Arrays.fill(dp, 1);
        Arrays.fill(prev, -1);

        int maxLen = 1;
        int maxIdx = 0;

        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (arr[j] < arr[i] && dp[j] + 1 > dp[i]) {
                    dp[i] = dp[j] + 1;
                    prev[i] = j; // i 이전에 j가 온다는 것을 기록
                }
            }
            if (dp[i] > maxLen) {
                maxLen = dp[i];
                maxIdx = i;
            }
        }

        // 역추적으로 실제 LIS 복원
        List<Integer> lis = new ArrayList<>();
        for (int idx = maxIdx; idx != -1; idx = prev[idx]) {
            lis.add(arr[idx]);
        }
        Collections.reverse(lis);

        return lis;
    }

    // ========================================
    // 방법 2: O(N log N) 이진 탐색 풀이
    // tails[k] = 길이 k+1인 증가 수열의 마지막 원소 최솟값
    // ========================================
    static int lisBinarySearch(int[] arr) {
        int n = arr.length;
        int[] tails = new int[n]; // tails 배열
        int size = 0;             // 현재 tails의 유효 길이 = LIS 길이

        for (int num : arr) {
            // tails[0..size-1]에서 num 이상인 첫 번째 위치를 이진 탐색
            int pos = lowerBound(tails, 0, size, num);

            tails[pos] = num; // 해당 위치에 값 교체 (또는 끝에 추가)

            if (pos == size) {
                size++; // tails 끝에 추가된 경우 LIS 길이 증가
            }
        }

        return size;
    }

    /**
     * lower bound: arr[left..right) 구간에서 target 이상인 첫 번째 인덱스를 반환
     * Arrays.binarySearch 대신 직접 구현하면 경계 조건을 명확하게 제어 가능
     */
    static int lowerBound(int[] arr, int left, int right, int target) {
        while (left < right) {
            int mid = (left + right) / 2;
            if (arr[mid] < target) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return left;
    }

    public static void main(String[] args) {
        int[] arr = {10, 20, 10, 30, 20, 50};

        // O(N²) DP
        System.out.println("=== O(N²) DP 풀이 ===");
        System.out.println("LIS 길이: " + lisDp(arr));

        // O(N²) DP + 역추적
        System.out.println("\n=== O(N²) DP + 역추적 ===");
        List<Integer> lis = lisWithTraceback(arr);
        System.out.println("LIS 길이: " + lis.size());
        System.out.println("LIS: " + lis);

        // O(N log N) 이진 탐색
        System.out.println("\n=== O(N log N) 이진 탐색 풀이 ===");
        System.out.println("LIS 길이: " + lisBinarySearch(arr));

        // 추가 예제
        int[] arr2 = {3, 1, 2, 8, 5, 6};
        System.out.println("\n=== 배열 [3, 1, 2, 8, 5, 6] ===");
        System.out.println("LIS 길이 (DP): " + lisDp(arr2));
        System.out.println("LIS 길이 (이진탐색): " + lisBinarySearch(arr2));
        System.out.println("LIS: " + lisWithTraceback(arr2));
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 방법 | 시간 복잡도 | 공간 복잡도 |
|------|-----------|-----------|
| O(N²) DP | \(O(N^2)\) | \(O(N)\) — dp 배열 |
| O(N log N) 이진 탐색 | \(O(N \log N)\) | \(O(N)\) — tails 배열 |

> N이 10,000 이상이면 O(N²)은 시간 초과가 날 수 있으므로 이진 탐색 풀이를 사용해야 한다.

## 관련 문제 유형
- **가장 긴 증가하는 부분 수열**: 기본 LIS 문제
- **전깃줄 문제**: 교차하지 않는 최대 전깃줄 수 = LIS
- **가장 긴 감소하는 부분 수열**: 배열을 뒤집어서 LIS
- **최소 체인 분할**: 딜워스 정리를 활용 (최장 반체인 = LIS)
- **상자 쌓기 / 러시아 인형**: 정렬 후 LIS 적용
- **줄 세우기**: 이미 정렬된 부분을 최대한 유지 → N - LIS 길이
