# 모노톤 스택 (Monotone Stack)

## 분류
- **카테고리**: 자료구조
- **관련 알고리즘**: [슬라이딩 윈도우](../array-techniques/sliding-window.md), [투 포인터](../array-techniques/two-pointer.md)

## 개요
모노톤 스택(Monotone Stack)은 스택 내부의 원소가 항상 **단조 증가** 또는 **단조 감소** 순서를 유지하도록 관리하는 스택 활용 기법이다. 배열을 한 번 순회하면서 각 원소에 대해 "다음에 나보다 큰/작은 원소"를 \(O(N)\) 시간에 모두 구할 수 있다는 것이 핵심이다. 브루트포스로 \(O(N^2)\)이 걸리는 문제를 **선형 시간으로 최적화**하는 강력한 테크닉이다.

## 핵심 로직
1. **배열 순회**: 원소를 하나씩 처리하면서 스택을 관리한다.
2. **단조성 유지**: 새 원소를 넣기 전에, 스택의 단조성을 깨는 원소를 모두 pop한다.
   - **단조 감소 스택** (Next Greater Element): 새 원소가 스택 top보다 크면, pop되는 원소의 "다음으로 큰 원소"가 새 원소이다.
   - **단조 증가 스택** (Next Smaller Element): 새 원소가 스택 top보다 작으면, pop되는 원소의 "다음으로 작은 원소"가 새 원소이다.
3. **스택에 push**: 단조성이 유지되면 새 원소(또는 인덱스)를 push한다.
4. **잔여 처리**: 순회 후 스택에 남은 원소는 "다음으로 큰/작은 원소가 없는" 것이다.

> 핵심 원리: 각 원소는 스택에 최대 한 번 push, 한 번 pop되므로 전체 시간 복잡도가 \(O(N)\)이다.

## 언제 사용하는가?
- **Next Greater Element (NGE)**: 각 원소의 오른쪽에서 처음으로 자신보다 큰 원소 찾기
- **Next Smaller Element**: 각 원소의 오른쪽에서 처음으로 자신보다 작은 원소 찾기
- **Previous Greater/Smaller Element**: 각 원소의 왼쪽에서 가장 가까운 큰/작은 원소 찾기
- **히스토그램에서 가장 큰 직사각형**: 각 막대에 대해 양쪽으로 확장 가능한 범위를 O(N)에 계산
- **주식 가격 스팬**: 현재 가격 이하인 연속 일수 계산
- **빗물 트래핑 (Trapping Rain Water)**: 막대 사이에 고이는 빗물의 양 계산
- **문제에서 "나보다 큰/작은 가장 가까운 원소" 키워드가 보이면** 모노톤 스택을 의심

## Java 구현
```java
import java.util.*;

public class MonotoneStack {

    // ========================================
    // 다음으로 큰 원소 (Next Greater Element)
    // 각 원소에 대해 오른쪽에서 처음으로 자신보다 큰 원소를 찾는다
    // 없으면 -1
    // ========================================
    static int[] nextGreaterElement(int[] arr) {
        int n = arr.length;
        int[] result = new int[n];
        Arrays.fill(result, -1);

        // 단조 감소 스택 (인덱스 저장)
        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = 0; i < n; i++) {
            // 스택 top의 원소보다 현재 원소가 크면 → pop되는 원소의 NGE는 현재 원소
            while (!stack.isEmpty() && arr[stack.peek()] < arr[i]) {
                result[stack.pop()] = arr[i];
            }
            stack.push(i);
        }

        return result;
    }

    // ========================================
    // 이전에 나보다 작은 원소 (Previous Smaller Element)
    // 각 원소에 대해 왼쪽에서 가장 가까운 자신보다 작은 원소를 찾는다
    // 없으면 -1
    // ========================================
    static int[] previousSmallerElement(int[] arr) {
        int n = arr.length;
        int[] result = new int[n];
        Arrays.fill(result, -1);

        // 단조 증가 스택 (인덱스 저장)
        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = 0; i < n; i++) {
            // 스택 top의 원소가 현재 원소 이상이면 pop (더 작은 원소를 찾기 위해)
            while (!stack.isEmpty() && arr[stack.peek()] >= arr[i]) {
                stack.pop();
            }
            if (!stack.isEmpty()) {
                result[i] = arr[stack.peek()];
            }
            stack.push(i);
        }

        return result;
    }

    // ========================================
    // 히스토그램에서 가장 큰 직사각형
    // 각 막대를 높이로 하는 최대 직사각형의 넓이를 구한다
    // ========================================
    static long largestRectangleInHistogram(int[] heights) {
        int n = heights.length;
        long maxArea = 0;

        // 단조 증가 스택 (인덱스 저장)
        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = 0; i <= n; i++) {
            // 배열 끝에 높이 0인 가상의 막대를 추가하여 스택을 비운다
            int currentHeight = (i == n) ? 0 : heights[i];

            while (!stack.isEmpty() && heights[stack.peek()] > currentHeight) {
                int height = heights[stack.pop()];
                // 왼쪽 경계: 스택이 비었으면 0, 아니면 스택 top + 1
                int width = stack.isEmpty() ? i : i - stack.peek() - 1;
                maxArea = Math.max(maxArea, (long) height * width);
            }

            stack.push(i);
        }

        return maxArea;
    }

    public static void main(String[] args) {
        // === Next Greater Element 테스트 ===
        System.out.println("=== Next Greater Element ===");
        int[] arr1 = {2, 1, 2, 4, 3};
        int[] nge = nextGreaterElement(arr1);
        System.out.println("배열:   " + Arrays.toString(arr1));
        System.out.println("NGE:    " + Arrays.toString(nge));
        // 기대 결과: [4, 2, 4, -1, -1]
        // 2→4, 1→2, 2→4, 4→없음, 3→없음

        System.out.println();

        int[] arr2 = {4, 5, 2, 10, 8};
        int[] nge2 = nextGreaterElement(arr2);
        System.out.println("배열:   " + Arrays.toString(arr2));
        System.out.println("NGE:    " + Arrays.toString(nge2));
        // 기대 결과: [5, 10, 10, -1, -1]

        // === Previous Smaller Element 테스트 ===
        System.out.println("\n=== Previous Smaller Element ===");
        int[] arr3 = {4, 5, 2, 10, 8};
        int[] pse = previousSmallerElement(arr3);
        System.out.println("배열:   " + Arrays.toString(arr3));
        System.out.println("PSE:    " + Arrays.toString(pse));
        // 기대 결과: [-1, 4, -1, 2, 2]

        // === 히스토그램 최대 직사각형 테스트 ===
        System.out.println("\n=== 히스토그램에서 가장 큰 직사각형 ===");
        int[] histogram = {2, 1, 5, 6, 2, 3};
        long maxArea = largestRectangleInHistogram(histogram);
        System.out.println("히스토그램: " + Arrays.toString(histogram));
        System.out.println("최대 직사각형 넓이: " + maxArea);
        // 기대 결과: 10 (높이 5, 너비 2)

        int[] histogram2 = {6, 2, 5, 4, 5, 1, 6};
        long maxArea2 = largestRectangleInHistogram(histogram2);
        System.out.println("\n히스토그램: " + Arrays.toString(histogram2));
        System.out.println("최대 직사각형 넓이: " + maxArea2);
        // 기대 결과: 12 (높이 4, 너비 3: 인덱스 2~4)
    }
}
```

## 시간 복잡도 / 공간 복잡도
| | 복잡도 |
|---|---|
| **시간 복잡도** | \(O(N)\) — 각 원소가 스택에 최대 한 번 push, 한 번 pop |
| **공간 복잡도** | \(O(N)\) — 스택에 최대 N개의 원소 저장 |

> 브루트포스로 모든 쌍을 비교하면 \(O(N^2)\)이지만, 모노톤 스택을 사용하면 **\(O(N)\)**으로 해결할 수 있다. "다음에 나보다 큰/작은 원소"를 찾아야 하는 문제에서 가장 먼저 떠올려야 할 테크닉이다.

## 관련 문제 유형
- **Next Greater Element**: 배열에서 각 원소의 다음으로 큰 원소 찾기
- **히스토그램 최대 직사각형**: 막대 차트에서 가장 큰 직사각형 넓이
- **빗물 트래핑**: 높이 배열에서 고이는 빗물의 총량 계산
- **주식 가격 스팬**: 현재 주가 이하인 연속 일수
- **탑 문제**: 오른쪽에서 쏜 레이저를 받는 탑 찾기
- **오큰수 / 오등큰수**: 각 원소의 오른쪽에서 처음으로 큰 수 (수열 문제)
- **최대 직사각형 (2D)**: 2D 격자에서 최대 직사각형을 히스토그램으로 변환하여 풀기
