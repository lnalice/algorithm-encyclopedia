# 투 포인터 (Two Pointer)

## 분류
- **카테고리**: 배열 기법
- **관련 알고리즘**: [슬라이딩 윈도우](./sliding-window.md), [이진 탐색](../search/binary-search.md), [구간합](./prefix-sum.md)

## 개요
투 포인터는 배열이나 리스트에서 두 개의 포인터를 사용하여 원하는 조건을 만족하는 쌍이나 구간을 효율적으로 탐색하는 기법이다. 정렬된 배열에서 양 끝에서 좁혀오는 방식(반대 방향)과, 같은 방향으로 이동하며 구간을 관리하는 방식(같은 방향)이 있다. 브루트 포스로 \(O(N^2)\)인 문제를 \(O(N)\)으로 해결할 수 있어 매우 효율적이다.

## 핵심 로직

### 반대 방향 투 포인터 (양 끝에서 좁혀오기)
1. **초기화**: `left = 0`, `right = n - 1`로 배열의 양 끝에 포인터를 배치한다.
2. **비교 및 이동**:
   - 두 포인터가 가리키는 값의 합이 목표보다 작으면 `left++` (합을 키운다).
   - 합이 목표보다 크면 `right--` (합을 줄인다).
   - 합이 목표와 같으면 정답을 기록한다.
3. **종료**: `left >= right`이면 탐색을 종료한다.

### 같은 방향 투 포인터 (구간 관리)
1. **초기화**: `left = 0`, `right = 0`으로 시작한다.
2. **윈도우 확장**: 조건이 만족될 때까지 `right`를 이동하며 구간을 넓힌다.
3. **윈도우 축소**: 조건을 초과하면 `left`를 이동하며 구간을 줄인다.
4. **결과 갱신**: 매 단계마다 결과를 갱신한다.

> 반대 방향 투 포인터는 **정렬된 배열**에서 쌍을 찾을 때, 같은 방향 투 포인터는 **연속 구간** 문제에 주로 사용한다.

## 언제 사용하는가?
- **정렬된 배열에서 두 수의 합**: 합이 특정 값이 되는 쌍 찾기 (Two Sum)
- **세 수의 합 (3Sum)**: 정렬 후 한 수를 고정하고 나머지에 투 포인터 적용
- **연속 부분 배열의 합**: 합이 특정 값 이상/이하인 구간 찾기
- **물 담기 (Container With Most Water)**: 양 끝에서 좁혀오며 최대 넓이 탐색
- **팰린드롬 검사**: 양 끝에서 중앙으로 비교
- **정렬된 두 배열 병합**: 각 배열에 포인터를 두고 병합

## Java 구현
```java
import java.util.*;

public class TwoPointer {

    // === 반대 방향 투 포인터 ===
    // 정렬된 배열에서 합이 target인 두 원소의 인덱스를 반환한다.
    // 존재하지 않으면 {-1, -1}을 반환한다.
    static int[] twoSum(int[] arr, int target) {
        int left = 0;
        int right = arr.length - 1;

        while (left < right) {
            int sum = arr[left] + arr[right];

            if (sum == target) {
                return new int[]{left, right}; // 정답 발견
            } else if (sum < target) {
                left++;  // 합이 부족하므로 왼쪽 포인터를 오른쪽으로
            } else {
                right--; // 합이 초과하므로 오른쪽 포인터를 왼쪽으로
            }
        }

        return new int[]{-1, -1}; // 조건을 만족하는 쌍 없음
    }

    // === 같은 방향 투 포인터 ===
    // 양수로 이루어진 배열에서 합이 정확히 target인 연속 부분 배열의
    // 시작과 끝 인덱스를 반환한다. 존재하지 않으면 {-1, -1}을 반환한다.
    static int[] subarraySum(int[] arr, int target) {
        int n = arr.length;
        int left = 0;
        int currentSum = 0;

        for (int right = 0; right < n; right++) {
            currentSum += arr[right]; // 윈도우 확장

            // 합이 target을 초과하면 왼쪽 포인터 이동
            while (currentSum > target && left <= right) {
                currentSum -= arr[left];
                left++;
            }

            if (currentSum == target) {
                return new int[]{left, right}; // 정답 발견
            }
        }

        return new int[]{-1, -1}; // 조건을 만족하는 구간 없음
    }

    // === 3Sum (세 수의 합) ===
    // 배열에서 합이 0인 서로 다른 세 수의 조합을 모두 반환한다.
    static List<List<Integer>> threeSum(int[] arr) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(arr); // 정렬 필수

        for (int i = 0; i < arr.length - 2; i++) {
            // 중복된 첫 번째 수 건너뛰기
            if (i > 0 && arr[i] == arr[i - 1]) continue;

            int left = i + 1;
            int right = arr.length - 1;

            while (left < right) {
                int sum = arr[i] + arr[left] + arr[right];

                if (sum == 0) {
                    result.add(Arrays.asList(arr[i], arr[left], arr[right]));
                    // 중복된 값 건너뛰기
                    while (left < right && arr[left] == arr[left + 1]) left++;
                    while (left < right && arr[right] == arr[right - 1]) right--;
                    left++;
                    right--;
                } else if (sum < 0) {
                    left++;
                } else {
                    right--;
                }
            }
        }

        return result;
    }

    public static void main(String[] args) {
        // === Two Sum 예시 ===
        int[] sorted = {1, 2, 3, 4, 6, 8, 11};
        int target = 10;
        int[] result = twoSum(sorted, target);
        System.out.println("합이 " + target + "인 쌍의 인덱스: "
                + Arrays.toString(result));
        // 출력: [1, 5] → arr[1]=2, arr[5]=8, 합=10

        // === 연속 부분 배열 합 예시 ===
        int[] arr = {1, 4, 20, 3, 10, 5};
        int subTarget = 33;
        int[] subResult = subarraySum(arr, subTarget);
        System.out.println("합이 " + subTarget + "인 부분 배열: 인덱스 "
                + Arrays.toString(subResult));
        // 출력: [2, 4] → 20+3+10=33

        // === 3Sum 예시 ===
        int[] nums = {-1, 0, 1, 2, -1, -4};
        List<List<Integer>> triplets = threeSum(nums);
        System.out.println("합이 0인 세 수의 조합: " + triplets);
        // 출력: [[-1, -1, 2], [-1, 0, 1]]
    }
}
```

## 시간 복잡도 / 공간 복잡도
| | 복잡도 |
|---|---|
| **시간 복잡도 (Two Sum)** | \(O(N)\) — 양 끝에서 한 번 순회 (정렬 필요 시 \(O(N \log N)\)) |
| **시간 복잡도 (부분 배열 합)** | \(O(N)\) — 각 포인터가 최대 N번 이동 |
| **시간 복잡도 (3Sum)** | \(O(N^2)\) — 정렬 \(O(N \log N)\) + 고정 루프 × 투 포인터 |
| **공간 복잡도** | \(O(1)\) — 포인터 변수만 사용 (결과 저장 공간 제외) |

> 브루트 포스 \(O(N^2)\) 또는 \(O(N^3)\) 대비 투 포인터로 차수를 하나 줄일 수 있다.

## 관련 문제 유형
- **Two Sum (정렬된 배열)**: 합이 목표인 두 수 찾기
- **3Sum / 4Sum**: 합이 목표인 세/네 수의 조합 찾기
- **연속 부분 배열 합**: 합이 특정 값인/이상인 구간 찾기
- **물 담기 (Container With Most Water)**: 양 끝 투 포인터로 최대 넓이
- **중복 제거 (정렬된 배열)**: 같은 방향 투 포인터로 제자리 중복 제거
- **두 정렬 배열 병합**: 각 배열에 포인터 배치 후 병합
- **팰린드롬 검사**: 양 끝에서 중앙으로 문자 비교
