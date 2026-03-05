# 분할 정복 (Divide and Conquer)

## 분류
- **카테고리**: 분할 정복
- **관련 알고리즘**: [이진 탐색](../search/binary-search.md), [세그먼트 트리](../data-structures/segment-tree.md), [동적 프로그래밍](../dynamic-programming/dp.md), [병합 정렬](#java-구현)

## 개요
분할 정복은 큰 문제를 **동일한 구조의 더 작은 부분 문제로 나누고(Divide)**, 각 부분 문제를 **재귀적으로 해결한 뒤(Conquer)**, 그 결과를 **합쳐서(Combine)** 원래 문제의 답을 구하는 알고리즘 설계 기법이다. DP와 달리 부분 문제들이 서로 겹치지 않는(독립적인) 경우에 주로 사용된다.

## 핵심 로직

### 1단계: Divide (분할)
- 현재 문제를 같은 유형의 더 작은 부분 문제 2개 이상으로 나눈다.
- 예: 배열을 절반으로 나누기, 탐색 범위를 반으로 줄이기

### 2단계: Conquer (정복)
- 나뉜 부분 문제가 충분히 작으면(Base Case) 직접 해결한다.
- 그렇지 않으면 재귀적으로 다시 분할한다.

### 3단계: Combine (결합)
- 부분 문제의 결과를 합쳐 원래 문제의 답을 만든다.
- 예: 정렬된 두 배열을 병합(Merge)하기

## 분할 정복 기반 알고리즘과 자료구조
분할 정복 원리는 다양한 알고리즘과 자료구조의 기반이 된다:

### 알고리즘
- **병합 정렬(Merge Sort)**: 배열을 반으로 나누어 정렬 후 병합 → \(O(N \log N)\)
- **퀵 정렬(Quick Sort)**: 피벗 기준으로 분할 후 각각 정렬 → 평균 \(O(N \log N)\)
- **거듭제곱 분할(Fast Exponentiation)**: 지수를 반씩 나누어 \(O(\log N)\)에 거듭제곱 계산
- **카라츠바 곱셈**: 큰 수의 곱셈을 3번의 작은 곱셈으로 분할

### 자료구조
- **[이진 탐색](../search/binary-search.md)**: 정렬된 배열을 반씩 나누어 탐색 → 분할 정복의 가장 기본적인 적용
- **[세그먼트 트리](../data-structures/segment-tree.md)**: 구간을 반씩 나누어 트리로 구성 → 분할 정복 원리를 **자료구조**에 적용한 대표적 예시

## 언제 사용하는가?
- **문제를 동일한 구조의 부분 문제로 나눌 수 있을 때** (재귀적 구조)
- **정렬**: 병합 정렬, 퀵 정렬
- **거듭제곱**: 빠른 거듭제곱, 행렬 거듭제곱
- **행렬 곱셈**: 슈트라센 알고리즘
- **최근접 점 쌍 문제**: 2차원 평면에서 가장 가까운 두 점 찾기
- **큰 수 곱셈**: 카라츠바 곱셈
- **구간 쿼리**: 세그먼트 트리 기반 문제

## Java 구현

```java
public class DivideAndConquer {

    // ========================================
    // 병합 정렬 (Merge Sort)
    // ========================================

    /**
     * 배열을 반으로 분할하고 재귀적으로 정렬한 뒤 병합한다.
     * @param arr  정렬할 배열
     * @param left 시작 인덱스 (포함)
     * @param right 끝 인덱스 (포함)
     */
    static void mergeSort(int[] arr, int left, int right) {
        if (left >= right) return; // 원소가 1개 이하이면 이미 정렬됨 (Base Case)

        int mid = (left + right) / 2;

        // [Divide] 배열을 절반으로 나누어 각각 정렬 (Conquer)
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);

        // [Combine] 정렬된 두 부분 배열을 하나로 병합
        merge(arr, left, mid, right);
    }

    /**
     * 두 정렬된 부분 배열 [left..mid]와 [mid+1..right]를 병합한다.
     * 임시 배열을 사용하여 두 구간의 원소를 순서대로 합친다.
     */
    static void merge(int[] arr, int left, int mid, int right) {
        int[] temp = new int[right - left + 1];
        int i = left;      // 왼쪽 부분 배열의 포인터
        int j = mid + 1;   // 오른쪽 부분 배열의 포인터
        int k = 0;         // 임시 배열의 포인터

        // 두 부분 배열에서 더 작은 원소를 선택하여 임시 배열에 채움
        while (i <= mid && j <= right) {
            if (arr[i] <= arr[j]) {
                temp[k++] = arr[i++];
            } else {
                temp[k++] = arr[j++];
            }
        }

        // 왼쪽 부분 배열에 남은 원소 복사
        while (i <= mid) {
            temp[k++] = arr[i++];
        }

        // 오른쪽 부분 배열에 남은 원소 복사
        while (j <= right) {
            temp[k++] = arr[j++];
        }

        // 임시 배열의 결과를 원본 배열에 반영
        for (int t = 0; t < temp.length; t++) {
            arr[left + t] = temp[t];
        }
    }

    public static void main(String[] args) {
        int[] arr = {38, 27, 43, 3, 9, 82, 10};

        System.out.println("정렬 전: " + java.util.Arrays.toString(arr));
        mergeSort(arr, 0, arr.length - 1);
        System.out.println("정렬 후: " + java.util.Arrays.toString(arr));
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 알고리즘 | 시간 복잡도 | 공간 복잡도 |
|---------|-----------|-----------|
| 병합 정렬 | \(O(N \log N)\) — 항상 | \(O(N)\) — 임시 배열 |
| 퀵 정렬 | 평균 \(O(N \log N)\), 최악 \(O(N^2)\) | \(O(\log N)\) — 재귀 스택 |
| 이진 탐색 | \(O(\log N)\) | \(O(1)\) (반복) / \(O(\log N)\) (재귀) |
| 빠른 거듭제곱 | \(O(\log N)\) | \(O(\log N)\) — 재귀 스택 |

> 분할 정복의 시간 복잡도는 **마스터 정리(Master Theorem)** `T(n) = aT(n/b) + O(n^d)`로 분석할 수 있다.

## 관련 문제 유형
- **정렬**: 병합 정렬, 퀵 정렬, 역순 쌍(Inversion Count) 세기
- **탐색**: 이진 탐색, 삼분 탐색
- **수학**: 빠른 거듭제곱, 행렬 거듭제곱, 카라츠바 곱셈
- **기하**: 최근접 점 쌍 (Closest Pair of Points)
- **구간 쿼리**: 세그먼트 트리 기반 문제
- **분할 정복 최적화**: DP에 분할 정복 기법을 적용하는 고급 테크닉
