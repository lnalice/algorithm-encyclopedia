# 이진 탐색 (Binary Search)

## 분류
- **카테고리**: 탐색
- **관련 알고리즘**: [분할 정복](../divide-and-conquer/divide-and-conquer.md), [투 포인터](../array-techniques/two-pointer.md)

## 개요
이진 탐색은 **정렬된 배열**에서 탐색 범위를 절반씩 줄여가며 원하는 값을 찾는 알고리즘이다. 매 단계마다 중간값과 비교하여 탐색 범위를 반으로 좁히므로, 선형 탐색 \(O(N)\)보다 훨씬 빠른 \(O(\log N)\)의 시간 복잡도를 보장한다. 단순 값 탐색 외에도 Lower Bound/Upper Bound, 매개변수 탐색(Parametric Search) 등 다양한 변형이 존재한다.

## 핵심 로직
1. **초기화**: `left = 0`, `right = 배열 길이 - 1`로 탐색 범위를 설정한다.
2. **중간값 비교**: `mid = (left + right) / 2` 위치의 값을 목표값과 비교한다.
   - `arr[mid] == target`: 값을 찾았으므로 인덱스 반환
   - `arr[mid] < target`: 목표값이 오른쪽에 있으므로 `left = mid + 1`
   - `arr[mid] > target`: 목표값이 왼쪽에 있으므로 `right = mid - 1`
3. **종료 조건**: `left > right`이면 값이 존재하지 않는다.

> **Lower Bound**: target 이상인 첫 번째 위치를 찾는다.
> **Upper Bound**: target 초과인 첫 번째 위치를 찾는다.
> **매개변수 탐색**: "조건을 만족하는 최솟값/최댓값"을 이진 탐색으로 결정한다.

## 언제 사용하는가?
- **정렬된 배열에서 값 탐색**: 특정 값의 존재 여부 또는 위치 탐색
- **Lower Bound / Upper Bound**: 특정 값 이상/초과인 첫 번째 위치 탐색
- **매개변수 탐색 (Parametric Search)**: "X 이상으로 만들 수 있는가?"를 판별하며 최적값을 탐색
  - 예: 나무 자르기(최대 높이), 랜선 자르기(최대 길이), 예산 배분(최대 상한)
- **최적화 문제**: 답의 범위가 정해져 있고, 특정 값 이상이면 가능/불가능 판별이 되는 문제
- **정렬된 데이터에서의 개수 세기**: Lower Bound와 Upper Bound의 차이로 특정 값의 개수 계산

## Java 구현
```java
import java.util.*;

public class BinarySearch {

    // ========================================
    // 기본 이진 탐색: 정확한 값의 인덱스를 반환 (없으면 -1)
    // ========================================
    static int binarySearch(int[] arr, int target) {
        int left = 0;
        int right = arr.length - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2; // 오버플로우 방지

            if (arr[mid] == target) {
                return mid;
            } else if (arr[mid] < target) {
                left = mid + 1; // 오른쪽 절반 탐색
            } else {
                right = mid - 1; // 왼쪽 절반 탐색
            }
        }

        return -1; // 값이 존재하지 않음
    }

    // ========================================
    // Lower Bound: target 이상인 첫 번째 인덱스 반환
    // 모든 원소가 target 미만이면 arr.length 반환
    // ========================================
    static int lowerBound(int[] arr, int target) {
        int left = 0;
        int right = arr.length;

        while (left < right) {
            int mid = left + (right - left) / 2;

            if (arr[mid] < target) {
                left = mid + 1; // target 이상인 구간은 오른쪽
            } else {
                right = mid;    // arr[mid] >= target이면 후보에 포함
            }
        }

        return left;
    }

    // ========================================
    // Upper Bound: target 초과인 첫 번째 인덱스 반환
    // 모든 원소가 target 이하이면 arr.length 반환
    // ========================================
    static int upperBound(int[] arr, int target) {
        int left = 0;
        int right = arr.length;

        while (left < right) {
            int mid = left + (right - left) / 2;

            if (arr[mid] <= target) {
                left = mid + 1; // target 초과인 구간은 오른쪽
            } else {
                right = mid;    // arr[mid] > target이면 후보에 포함
            }
        }

        return left;
    }

    // ========================================
    // 특정 값의 개수 세기 (Lower Bound + Upper Bound 활용)
    // ========================================
    static int countOccurrences(int[] arr, int target) {
        return upperBound(arr, target) - lowerBound(arr, target);
    }

    // ========================================
    // 매개변수 탐색 (Parametric Search) 예시
    // 문제: 나무 N개의 높이가 주어질 때, 절단기 높이를 H로 설정하면
    //       각 나무에서 max(0, height-H)만큼 가져간다.
    //       최소 M미터의 나무를 가져가기 위한 절단기의 최대 높이를 구하라.
    // ========================================
    static int parametricSearch(int[] trees, int target) {
        int left = 0;
        int right = Arrays.stream(trees).max().getAsInt();
        int answer = 0;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            // mid 높이로 잘랐을 때 얻을 수 있는 나무 총량 계산
            long total = 0;
            for (int height : trees) {
                if (height > mid) {
                    total += height - mid;
                }
            }

            if (total >= target) {
                answer = mid;      // 조건 만족 → 더 높은 높이 시도
                left = mid + 1;
            } else {
                right = mid - 1;   // 조건 불만족 → 높이 낮추기
            }
        }

        return answer;
    }

    public static void main(String[] args) {
        // === 기본 이진 탐색 ===
        System.out.println("=== 기본 이진 탐색 ===");
        int[] arr = {1, 3, 5, 7, 9, 11, 13, 15};
        int target = 7;
        int idx = binarySearch(arr, target);
        System.out.println("배열: " + Arrays.toString(arr));
        System.out.println(target + "의 인덱스: " + idx);
        System.out.println(6 + "의 인덱스: " + binarySearch(arr, 6));

        // === Lower Bound / Upper Bound ===
        System.out.println("\n=== Lower Bound / Upper Bound ===");
        int[] arr2 = {1, 2, 2, 2, 3, 4, 5};
        System.out.println("배열: " + Arrays.toString(arr2));
        System.out.println("lowerBound(2) = " + lowerBound(arr2, 2)); // 1
        System.out.println("upperBound(2) = " + upperBound(arr2, 2)); // 4
        System.out.println("2의 개수: " + countOccurrences(arr2, 2));  // 3

        // === 매개변수 탐색 (나무 자르기) ===
        System.out.println("\n=== 매개변수 탐색 (나무 자르기) ===");
        int[] trees = {20, 15, 10, 17};
        int need = 7;
        int maxHeight = parametricSearch(trees, need);
        System.out.println("나무 높이: " + Arrays.toString(trees));
        System.out.println("필요한 나무: " + need + "m");
        System.out.println("절단기 최대 높이: " + maxHeight);
    }
}
```

## 시간 복잡도 / 공간 복잡도
| | 복잡도 |
|---|---|
| **시간 복잡도** | \(O(\log N)\) — 매 단계마다 탐색 범위가 절반으로 감소 |
| **공간 복잡도** | \(O(1)\) — 반복 구현 기준, 추가 공간 불필요 |

> 매개변수 탐색의 경우: 이진 탐색 \(O(\log X)\) × 판별 함수 \(O(N)\) = \(O(N \log X)\) (X는 탐색 범위)

## 관련 문제 유형
- **정렬된 배열에서 값 찾기**: 기본 이진 탐색
- **Lower Bound / Upper Bound**: 특정 값의 첫 등장 위치, 개수 세기
- **매개변수 탐색 (Parametric Search)**:
  - 나무 자르기 — 절단기 최대 높이
  - 랜선 자르기 — 최대 랜선 길이
  - 예산 배분 — 상한액의 최댓값
  - 공유기 설치 — 최소 거리의 최댓값
- **회전 정렬 배열 탐색**: 회전된 정렬 배열에서 원소 찾기
- **제곱근 / 거듭제곱 계산**: 정수 범위에서 이진 탐색
