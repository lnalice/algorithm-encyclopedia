# 정렬 알고리즘 (Sorting Algorithms)

## 분류
- **카테고리**: 정렬
- **관련 알고리즘**: [분할 정복](../divide-and-conquer/divide-and-conquer.md), [이진 탐색](../search/binary-search.md), [그리디](../greedy/greedy.md)

## 개요

정렬은 데이터를 특정 기준에 따라 순서대로 나열하는 알고리즘이다. 코딩테스트에서 정렬은 그 자체가 목적이 아니라, **이진 탐색·투 포인터·그리디 등 후속 알고리즘의 전처리 단계**로 활용되는 경우가 대부분이다. 따라서 직접 정렬 알고리즘을 구현하는 것보다 **`Arrays.sort`와 커스텀 `Comparator`를 능숙하게 다루는 것이 실전에서 훨씬 중요하다.**

> **코딩테스트 핵심**: 직접 정렬을 구현할 일은 거의 없다. `Arrays.sort`, `Collections.sort`, `Comparator`를 자유자재로 쓸 수 있어야 한다. 정렬 알고리즘의 원리는 면접 대비와 카운팅 정렬 같은 특수 케이스를 위해 알아두자.

## 핵심 로직

### 정렬 알고리즘 비교표

| 알고리즘 | 평균 | 최악 | 공간 | 안정 | 특징 |
|---------|------|------|------|------|------|
| **버블 정렬** | \(O(N^2)\) | \(O(N^2)\) | \(O(1)\) | ✅ | 인접 원소 반복 교환. 구현 쉬우나 느림 |
| **선택 정렬** | \(O(N^2)\) | \(O(N^2)\) | \(O(1)\) | ❌ | 최솟값을 찾아 앞으로 이동. 교환 횟수 최소 |
| **삽입 정렬** | \(O(N^2)\) | \(O(N^2)\) | \(O(1)\) | ✅ | 거의 정렬된 데이터에 매우 빠름 \(O(N)\) |
| **병합 정렬** | \(O(N \log N)\) | \(O(N \log N)\) | \(O(N)\) | ✅ | 분할 정복 기반. 항상 일정한 성능 보장 |
| **퀵 정렬** | \(O(N \log N)\) | \(O(N^2)\) | \(O(\log N)\) | ❌ | 실제로 가장 빠름. Java `Arrays.sort(int[])` 내부 사용 |
| **힙 정렬** | \(O(N \log N)\) | \(O(N \log N)\) | \(O(1)\) | ❌ | 추가 메모리 없이 \(O(N \log N)\) 보장 |
| **카운팅 정렬** | \(O(N + K)\) | \(O(N + K)\) | \(O(K)\) | ✅ | 값의 범위 K가 작을 때 초고속. 비교 기반 아님 |
| **기수 정렬** | \(O(d \cdot (N + K))\) | \(O(d \cdot (N + K))\) | \(O(N + K)\) | ✅ | 자릿수 d 기준 정렬. 정수 특화 |

> - **안정 정렬**: 같은 값의 원소끼리 원래 순서가 유지되는 정렬
> - Java의 `Arrays.sort(Object[])`, `Collections.sort()`는 **TimSort** (병합+삽입 하이브리드)로 안정 정렬이 보장된다.
> - Java의 `Arrays.sort(int[])`는 **Dual-Pivot QuickSort**로 불안정 정렬이다.

### Java 내장 정렬의 내부 동작
- **`Arrays.sort(int[])` (primitive)**: Dual-Pivot QuickSort → 평균 \(O(N \log N)\), 불안정
- **`Arrays.sort(Object[])` (참조 타입)**: TimSort → 최악 \(O(N \log N)\), 안정
- **`Collections.sort()`**: 내부적으로 `List.sort()` → TimSort 사용, 안정

## 언제 사용하는가?

- **정렬 후 이진 탐색**: 정렬된 배열에서 `Arrays.binarySearch()` 또는 직접 이진 탐색
- **정렬 후 투 포인터/그리디**: 정렬로 탐색 범위를 줄인 뒤 두 포인터 또는 그리디 적용
- **커스텀 정렬이 필요한 문제**: 다중 조건 정렬, 문자열 정렬, 좌표 정렬 등
- **카운팅 정렬이 유리한 경우**: 값의 범위가 작고 데이터가 많을 때 (예: 나이 정렬, 성적 등급 정렬)
- **상대 순서 보존이 필요한 경우**: 안정 정렬이 필요하면 `Arrays.sort(Integer[])` 또는 `Collections.sort()` 사용

## Java 구현

### 1. Java 내장 정렬 사용법 (가장 중요)

코딩테스트에서 실제로 사용하는 것은 대부분 이 내용이다.

```java
import java.util.*;

public class JavaBuiltInSort {

    public static void main(String[] args) {
        // ============================================================
        // 1) Arrays.sort — 기본형 배열 정렬
        // ============================================================
        int[] arr = {5, 3, 8, 1, 2};
        Arrays.sort(arr); // 오름차순: [1, 2, 3, 5, 8]

        // 구간 정렬: 인덱스 1~3만 정렬
        int[] arr2 = {5, 3, 8, 1, 2};
        Arrays.sort(arr2, 1, 4); // [5, 1, 3, 8, 2]

        // 기본형 배열 내림차순 — Integer[]로 변환 필요
        Integer[] boxed = {5, 3, 8, 1, 2};
        Arrays.sort(boxed, Collections.reverseOrder()); // [8, 5, 3, 2, 1]

        System.out.println("기본 정렬: " + Arrays.toString(arr));
        System.out.println("구간 정렬: " + Arrays.toString(arr2));
        System.out.println("내림차순: " + Arrays.toString(boxed));

        // ============================================================
        // 2) Arrays.sort — 객체 배열 + Comparator
        // ============================================================
        String[] words = {"banana", "apple", "cherry", "date"};

        // 길이 기준 오름차순
        Arrays.sort(words, (a, b) -> a.length() - b.length());
        System.out.println("길이순: " + Arrays.toString(words));

        // 사전순 내림차순
        Arrays.sort(words, (a, b) -> b.compareTo(a));
        System.out.println("사전역순: " + Arrays.toString(words));

        // ============================================================
        // 3) Collections.sort — 리스트 정렬
        // ============================================================
        List<Integer> list = new ArrayList<>(Arrays.asList(5, 3, 8, 1, 2));
        Collections.sort(list); // 오름차순
        System.out.println("리스트 정렬: " + list);

        list.sort(Collections.reverseOrder()); // 내림차순
        System.out.println("리스트 내림차순: " + list);

        // ============================================================
        // 4) 다중 조건 정렬 — Comparator 체이닝
        // ============================================================
        // 학생: {이름, 나이, 점수}
        String[][] students = {
            {"Kim", "25", "90"},
            {"Lee", "22", "95"},
            {"Park", "25", "85"},
            {"Choi", "22", "95"},
        };

        // 1순위: 나이 오름차순 → 2순위: 점수 내림차순 → 3순위: 이름 사전순
        Arrays.sort(students, Comparator
            .comparingInt((String[] s) -> Integer.parseInt(s[1]))       // 나이 오름차순
            .thenComparingInt((String[] s) -> -Integer.parseInt(s[2]))  // 점수 내림차순
            .thenComparing((String[] s) -> s[0])                        // 이름 사전순
        );

        System.out.println("\n=== 다중 조건 정렬 ===");
        for (String[] s : students) {
            System.out.println(s[0] + " (나이:" + s[1] + ", 점수:" + s[2] + ")");
        }

        // ============================================================
        // 5) 2차원 배열 정렬
        // ============================================================
        int[][] intervals = {{3, 5}, {1, 3}, {2, 8}, {1, 2}};

        // 시작점 오름차순 → 같으면 끝점 오름차순
        Arrays.sort(intervals, (a, b) -> {
            if (a[0] != b[0]) return a[0] - b[0];
            return a[1] - b[1];
        });

        // 위와 동일한 Comparator.comparing 방식
        // Arrays.sort(intervals, Comparator.comparingInt((int[] a) -> a[0])
        //                                  .thenComparingInt(a -> a[1]));

        System.out.println("\n=== 2차원 배열 정렬 ===");
        for (int[] interval : intervals) {
            System.out.println(Arrays.toString(interval));
        }
    }
}
```

### 2. 카운팅 정렬 (Counting Sort)

값의 범위가 작을 때 \(O(N)\)에 정렬하는 비교 기반이 아닌 정렬이다. 예를 들어 0~100 사이 점수 정렬, 알파벳 빈도 정렬 등에 활용한다.

```java
import java.util.*;

public class CountingSort {

    /**
     * 카운팅 정렬: 0 이상 maxVal 이하의 정수 배열을 정렬
     * @param arr    정렬할 배열
     * @param maxVal 배열 원소의 최댓값
     * @return 정렬된 새 배열
     */
    static int[] countingSort(int[] arr, int maxVal) {
        int[] count = new int[maxVal + 1];

        // 각 값의 등장 횟수 세기
        for (int val : arr) {
            count[val]++;
        }

        // 누적합으로 변환 → 각 값이 결과 배열에서 끝나는 위치
        for (int i = 1; i <= maxVal; i++) {
            count[i] += count[i - 1];
        }

        // 안정 정렬을 위해 뒤에서부터 순회하며 결과 배열에 배치
        int[] result = new int[arr.length];
        for (int i = arr.length - 1; i >= 0; i--) {
            result[--count[arr[i]]] = arr[i];
        }

        return result;
    }

    /**
     * 간단 버전: 누적합 없이 바로 출력 (안정 정렬이 필요 없을 때)
     */
    static int[] countingSortSimple(int[] arr, int maxVal) {
        int[] count = new int[maxVal + 1];
        for (int val : arr) {
            count[val]++;
        }

        int[] result = new int[arr.length];
        int idx = 0;
        for (int i = 0; i <= maxVal; i++) {
            while (count[i]-- > 0) {
                result[idx++] = i;
            }
        }
        return result;
    }

    public static void main(String[] args) {
        int[] scores = {85, 92, 78, 95, 88, 72, 95, 85, 90, 78};
        int maxScore = 100;

        System.out.println("원본: " + Arrays.toString(scores));

        // 안정 정렬 버전
        int[] sorted1 = countingSort(scores, maxScore);
        System.out.println("카운팅 정렬 (안정): " + Arrays.toString(sorted1));

        // 간단 버전
        int[] sorted2 = countingSortSimple(scores, maxScore);
        System.out.println("카운팅 정렬 (간단): " + Arrays.toString(sorted2));

        // 응용: 알파벳 빈도순 정렬
        String str = "programming";
        int[] freq = new int[26];
        for (char c : str.toCharArray()) {
            freq[c - 'a']++;
        }
        System.out.println("\n'" + str + "' 알파벳 빈도:");
        for (int i = 0; i < 26; i++) {
            if (freq[i] > 0) {
                System.out.println("  " + (char)('a' + i) + ": " + freq[i]);
            }
        }
    }
}
```

### 3. 병합 정렬 핵심 코드 (분할 정복 기반)

분할 정복의 대표 예시로, 상세 설명은 [분할 정복](../divide-and-conquer/divide-and-conquer.md)을 참고한다.

```java
import java.util.*;

public class MergeSort {

    static int[] temp; // 병합 시 사용할 임시 배열

    static void mergeSort(int[] arr, int left, int right) {
        if (left >= right) return;

        int mid = (left + right) / 2;
        mergeSort(arr, left, mid);      // 왼쪽 반 정렬
        mergeSort(arr, mid + 1, right); // 오른쪽 반 정렬
        merge(arr, left, mid, right);   // 병합
    }

    static void merge(int[] arr, int left, int mid, int right) {
        // 임시 배열에 복사
        for (int i = left; i <= right; i++) {
            temp[i] = arr[i];
        }

        int i = left;       // 왼쪽 포인터
        int j = mid + 1;    // 오른쪽 포인터
        int k = left;       // 결과 포인터

        while (i <= mid && j <= right) {
            if (temp[i] <= temp[j]) {
                arr[k++] = temp[i++];
            } else {
                arr[k++] = temp[j++];
            }
        }

        // 왼쪽 잔여 원소 복사 (오른쪽 잔여는 이미 제자리)
        while (i <= mid) {
            arr[k++] = temp[i++];
        }
    }

    public static void main(String[] args) {
        int[] arr = {38, 27, 43, 3, 9, 82, 10};
        temp = new int[arr.length];

        System.out.println("원본: " + Arrays.toString(arr));
        mergeSort(arr, 0, arr.length - 1);
        System.out.println("병합 정렬: " + Arrays.toString(arr));
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 구분 | 복잡도 |
|------|--------|
| **Java 내장 정렬 (primitive)** | 시간 \(O(N \log N)\), 공간 \(O(\log N)\) — Dual-Pivot QuickSort |
| **Java 내장 정렬 (Object)** | 시간 \(O(N \log N)\), 공간 \(O(N)\) — TimSort |
| **카운팅 정렬** | 시간 \(O(N + K)\), 공간 \(O(K)\) — K는 값의 범위 |
| **병합 정렬** | 시간 \(O(N \log N)\), 공간 \(O(N)\) — 항상 일정 |

> 대부분의 코딩테스트에서는 Java 내장 정렬만으로 충분하다. 값의 범위가 작고 초고속 정렬이 필요한 경우에만 카운팅 정렬을 고려한다.

## 관련 문제 유형

- **단순 정렬 후 출력**: 기본 `Arrays.sort` 활용
- **커스텀 정렬**: 좌표 정렬, 문자열 특수 정렬, 다중 조건 정렬
- **정렬 후 이진 탐색**: 정렬 전처리 → `Arrays.binarySearch()` 또는 직접 구현
- **정렬 후 투 포인터**: 두 수의 합, 세 수의 합 등
- **정렬 후 그리디**: 회의실 배정, 구간 스케줄링 등
- **빈도 정렬**: 카운팅 배열로 빈도 계산 후 빈도 기준 정렬
- **K번째 원소 찾기**: 정렬 후 인덱스 접근, 또는 퀵 셀렉트 활용
- **안정 정렬 필요**: 1차 정렬 결과를 유지하면서 2차 정렬 수행
