# 슬라이딩 윈도우 (Sliding Window)

## 분류
- **카테고리**: 배열 기법
- **관련 알고리즘**: [투 포인터](./two-pointer.md), [구간합](./prefix-sum.md)

## 개요
슬라이딩 윈도우는 배열이나 문자열에서 일정한 범위(윈도우)를 유지하면서 한 칸씩 이동하며 문제를 해결하는 기법이다. 전체 구간을 매번 다시 계산하는 대신, 윈도우가 이동할 때 빠지는 원소와 새로 들어오는 원소만 처리하여 중복 계산을 제거한다. 연속 부분 배열이나 부분 문자열에서 특정 조건을 만족하는 최적값을 구할 때 매우 효율적이다.

## 핵심 로직

### 고정 크기 윈도우 (Fixed-size Window)
1. **초기 윈도우 설정**: 처음 k개의 원소로 초기 윈도우의 값을 계산한다.
2. **윈도우 슬라이드**: 윈도우를 한 칸 오른쪽으로 이동할 때마다:
   - 왼쪽 끝 원소를 제거한다.
   - 오른쪽 끝에 새 원소를 추가한다.
   - 현재 윈도우의 값을 갱신한다.
3. **결과 갱신**: 매 이동마다 최대/최소 등 원하는 조건을 확인하고 결과를 갱신한다.

### 가변 크기 윈도우 (Variable-size Window)
1. **두 포인터 유지**: `left`와 `right` 포인터로 윈도우의 양 끝을 관리한다.
2. **윈도우 확장**: `right`를 오른쪽으로 이동하며 윈도우에 원소를 추가한다.
3. **윈도우 축소**: 조건을 위반하면 `left`를 오른쪽으로 이동하며 윈도우를 축소한다.
4. **결과 갱신**: 조건을 만족하는 시점에 결과(최소/최대 길이 등)를 갱신한다.

> 고정 크기 윈도우는 크기 k가 주어질 때, 가변 크기 윈도우는 "조건을 만족하는 최소/최대 구간"을 찾을 때 사용한다.

## 언제 사용하는가?
- **연속 부분 배열의 최대/최소 합**: 크기 k인 연속 부분 배열의 최대 합 구하기
- **특정 조건을 만족하는 최소/최대 구간**: 합이 S 이상인 가장 짧은 부분 배열
- **부분 문자열 문제**: 중복 없는 가장 긴 부분 문자열, 애너그램 찾기
- **연속 구간 내 특정 원소 개수**: 윈도우 내 특정 문자의 빈도 관리
- **데이터 스트림 처리**: 최근 k개 데이터의 평균, 최대값 등 실시간 계산

## Java 구현
```java
import java.util.*;

public class SlidingWindow {

    // === 고정 크기 윈도우 ===
    // 크기 k인 연속 부분 배열의 최대 합을 반환한다.
    static int maxSumFixedWindow(int[] arr, int k) {
        int n = arr.length;
        if (n < k) return -1;

        // 첫 번째 윈도우의 합 계산
        int windowSum = 0;
        for (int i = 0; i < k; i++) {
            windowSum += arr[i];
        }

        int maxSum = windowSum;

        // 윈도우를 한 칸씩 오른쪽으로 이동
        for (int i = k; i < n; i++) {
            windowSum += arr[i] - arr[i - k]; // 새 원소 추가, 오래된 원소 제거
            maxSum = Math.max(maxSum, windowSum);
        }

        return maxSum;
    }

    // === 가변 크기 윈도우 ===
    // 합이 target 이상인 가장 짧은 연속 부분 배열의 길이를 반환한다.
    // 존재하지 않으면 0을 반환한다.
    static int minLengthSubarraySum(int[] arr, int target) {
        int n = arr.length;
        int left = 0;
        int currentSum = 0;
        int minLength = Integer.MAX_VALUE;

        for (int right = 0; right < n; right++) {
            currentSum += arr[right]; // 윈도우 확장

            // 조건을 만족하면 윈도우를 축소하며 최소 길이 갱신
            while (currentSum >= target) {
                minLength = Math.min(minLength, right - left + 1);
                currentSum -= arr[left]; // 왼쪽 원소 제거
                left++;
            }
        }

        return minLength == Integer.MAX_VALUE ? 0 : minLength;
    }

    // === 가변 크기 윈도우 (문자열) ===
    // 중복 문자가 없는 가장 긴 부분 문자열의 길이를 반환한다.
    static int longestUniqueSubstring(String s) {
        int n = s.length();
        Set<Character> window = new HashSet<>();
        int left = 0;
        int maxLength = 0;

        for (int right = 0; right < n; right++) {
            char c = s.charAt(right);

            // 중복 문자가 있으면 윈도우 축소
            while (window.contains(c)) {
                window.remove(s.charAt(left));
                left++;
            }

            window.add(c);
            maxLength = Math.max(maxLength, right - left + 1);
        }

        return maxLength;
    }

    public static void main(String[] args) {
        // === 고정 크기 윈도우 예시 ===
        int[] arr1 = {2, 1, 5, 1, 3, 2};
        int k = 3;
        System.out.println("크기 " + k + "인 부분 배열의 최대 합: "
                + maxSumFixedWindow(arr1, k)); // 출력: 9 (5+1+3)

        // === 가변 크기 윈도우 예시 (배열) ===
        int[] arr2 = {2, 3, 1, 2, 4, 3};
        int target = 7;
        System.out.println("합이 " + target + " 이상인 최소 길이: "
                + minLengthSubarraySum(arr2, target)); // 출력: 2 (4+3)

        // === 가변 크기 윈도우 예시 (문자열) ===
        String s = "abcabcbb";
        System.out.println("중복 없는 가장 긴 부분 문자열 길이: "
                + longestUniqueSubstring(s)); // 출력: 3 ("abc")
    }
}
```

## 시간 복잡도 / 공간 복잡도
| | 복잡도 |
|---|---|
| **시간 복잡도 (고정 윈도우)** | \(O(N)\) — 배열을 한 번 순회 |
| **시간 복잡도 (가변 윈도우)** | \(O(N)\) — `left`와 `right` 각각 최대 N번 이동 |
| **공간 복잡도** | \(O(1)\) — 고정 윈도우 기준. 가변 윈도우에서 Set 사용 시 \(O(K)\) (K: 윈도우 내 고유 원소 수) |

> 브루트 포스 \(O(N \times K)\) 대비 슬라이딩 윈도우는 \(O(N)\)으로 크게 효율적이다.

## 관련 문제 유형
- **최대 부분 배열 합 (크기 k)**: 고정 크기 윈도우로 최대 합 계산
- **합이 S 이상인 최소 길이 부분 배열**: 가변 윈도우로 최소 구간 탐색
- **중복 없는 가장 긴 부분 문자열**: 가변 윈도우 + Set으로 중복 관리
- **문자열 내 애너그램 찾기**: 고정 윈도우 + 빈도 배열 비교
- **최대 연속 1의 개수 (k번 뒤집기 허용)**: 가변 윈도우로 0의 개수 관리
- **과일 바구니 (최대 2종류)**: 가변 윈도우 + Map으로 종류 수 관리
