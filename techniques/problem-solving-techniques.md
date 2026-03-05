# 코딩테스트 핵심 테크닉 모음

코딩테스트에서 자주 등장하는 소규모 테크닉들을 정리한다.
각 테크닉은 독립된 알고리즘이라기보다 다른 알고리즘과 결합하여 사용하는 기법이다.

---

## 1. 좌표 압축 (Coordinate Compression)

값의 범위가 매우 클 때(\(10^9\) 이상) **상대적 순서만 보존**하면서 값을 0 ~ N-1로 압축하는 기법이다.

### 핵심 아이디어
- 실제 값이 아니라 **정렬했을 때의 순위(rank)**만 중요한 경우가 많다.
- 중복을 제거하고 정렬한 뒤, 원래 값을 정렬된 배열에서의 인덱스로 치환한다.
- 세그먼트 트리, BIT, 스위핑 등에서 배열 크기를 값의 범위가 아닌 **원소 개수**로 줄일 수 있다.

### 언제 쓰는가
- 값의 범위가 \(10^9\) 이상이라 배열 인덱스로 직접 사용할 수 없을 때
- 세그먼트 트리·BIT에 값을 인덱스로 넣어야 하는데 범위가 너무 클 때
- 스위핑·이벤트 처리에서 좌표를 정규화해야 할 때

### Java 구현

#### 방법 1: 정렬 + 이진 탐색

```java
import java.util.*;

public class CoordinateCompression {

    // 좌표 압축: 원본 배열의 각 값을 0-based 순위로 변환
    static int[] compress(int[] arr) {
        int n = arr.length;

        // 중복 제거 후 정렬
        int[] sorted = Arrays.stream(arr).distinct().sorted().toArray();

        // 이진 탐색으로 각 원소의 압축된 순위를 구함
        int[] result = new int[n];
        for (int i = 0; i < n; i++) {
            result[i] = Arrays.binarySearch(sorted, arr[i]);
        }
        return result;
    }

    public static void main(String[] args) {
        int[] arr = {1000000000, 3, 3, 500, 999999999};
        int[] compressed = compress(arr);
        // 원본: [1000000000, 3, 3, 500, 999999999]
        // 압축: [3, 0, 0, 1, 2]
        System.out.println(Arrays.toString(compressed));
    }
}
```

#### 방법 2: HashMap 활용

```java
import java.util.*;

public class CoordinateCompressionMap {

    // HashMap으로 좌표 압축: 값 → 순위 매핑을 O(1)로 조회
    static int[] compress(int[] arr) {
        int n = arr.length;

        // TreeSet으로 중복 제거 + 자동 정렬
        TreeSet<Integer> set = new TreeSet<>();
        for (int v : arr) set.add(v);

        // 순위 매핑 테이블 생성
        Map<Integer, Integer> rank = new HashMap<>();
        int idx = 0;
        for (int v : set) {
            rank.put(v, idx++);
        }

        // 원본 배열을 압축된 순위로 변환
        int[] result = new int[n];
        for (int i = 0; i < n; i++) {
            result[i] = rank.get(arr[i]);
        }
        return result;
    }

    public static void main(String[] args) {
        int[] arr = {1000000000, 3, 3, 500, 999999999};
        int[] compressed = compress(arr);
        System.out.println(Arrays.toString(compressed));
        // [3, 0, 0, 1, 2]
    }
}
```

---

## 2. 라인 스위핑 (Line Sweep / Sweeping)

이벤트를 **시작점과 끝점**으로 분리하고, 정렬 후 한 방향으로 쓸면서 처리하는 기법이다.

### 핵심 아이디어
- 구간 `[s, e]`을 **+1 이벤트(시작)**과 **-1 이벤트(끝)** 두 개로 분해한다.
- 모든 이벤트를 좌표 기준으로 정렬한 뒤, 왼쪽에서 오른쪽으로 쓸며 상태를 갱신한다.
- 겹치는 구간 수, 구간 합치기, 최대 겹침 등을 \(O(N \log N)\)에 해결할 수 있다.

### 언제 쓰는가
- 여러 구간이 겹치는 최대 개수를 구할 때
- 구간들을 합쳐서 하나로 만들 때 (Merge Intervals)
- 직사각형 넓이 합, 선분 교차 판정 등 기하 문제

### Java 구현: 구간 합치기 (Merge Intervals)

```java
import java.util.*;

public class LineSweep {

    // 겹치는 구간을 합쳐서 반환
    static int[][] mergeIntervals(int[][] intervals) {
        if (intervals.length == 0) return new int[0][];

        // 시작점 기준 정렬
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);

        List<int[]> merged = new ArrayList<>();
        int[] current = intervals[0];
        merged.add(current);

        for (int i = 1; i < intervals.length; i++) {
            if (intervals[i][0] <= current[1]) {
                // 겹침 → 끝점을 더 큰 값으로 확장
                current[1] = Math.max(current[1], intervals[i][1]);
            } else {
                // 겹치지 않음 → 새 구간 시작
                current = intervals[i];
                merged.add(current);
            }
        }

        return merged.toArray(new int[0][]);
    }

    // 최대 겹침 개수: 이벤트 정렬 방식
    static int maxOverlap(int[][] intervals) {
        // 이벤트 리스트: (좌표, 타입) — 시작이면 +1, 끝이면 -1
        int[][] events = new int[intervals.length * 2][2];
        for (int i = 0; i < intervals.length; i++) {
            events[i * 2] = new int[]{intervals[i][0], +1};     // 시작
            events[i * 2 + 1] = new int[]{intervals[i][1], -1}; // 끝
        }

        // 좌표 기준 정렬, 같으면 끝(-1)을 먼저 처리
        Arrays.sort(events, (a, b) -> a[0] != b[0] ? a[0] - b[0] : a[1] - b[1]);

        int max = 0, count = 0;
        for (int[] event : events) {
            count += event[1];
            max = Math.max(max, count);
        }
        return max;
    }

    public static void main(String[] args) {
        // 구간 합치기
        int[][] intervals = {{1, 3}, {2, 6}, {8, 10}, {15, 18}};
        int[][] merged = mergeIntervals(intervals);
        System.out.println("=== 구간 합치기 ===");
        for (int[] m : merged) {
            System.out.println(Arrays.toString(m));
        }
        // [1, 6], [8, 10], [15, 18]

        // 최대 겹침
        int[][] rooms = {{0, 30}, {5, 10}, {15, 20}};
        System.out.println("\n최대 겹침 수: " + maxOverlap(rooms));
        // 2
    }
}
```

---

## 3. 매개변수 탐색 (Parametric Search)

**"최솟값의 최댓값"**, **"최댓값의 최솟값"** 등 최적화 문제를 **결정 문제(Yes/No)**로 바꿔 이진 탐색으로 해결하는 기법이다.

### 핵심 아이디어
- 정답이 될 수 있는 범위 `[lo, hi]`를 잡고 이진 탐색한다.
- `mid` 값이 **조건을 만족하는지(feasible)** 판정하는 함수를 작성한다.
- 조건 만족 여부에 따라 탐색 범위를 절반으로 줄인다.
- 단조성(monotonicity)이 보장되어야 한다: 조건이 한 지점을 기준으로 항상 True→False 또는 False→True로 바뀌어야 한다.

### 언제 쓰는가
- "X를 만족하는 최대/최소 값을 구하라" 형태의 문제
- 직접 최적값을 구하기 어렵지만, 특정 값이 가능한지 판정하기는 쉬운 경우
- 관련: [이진 탐색](../search/binary-search.md)

### Java 구현: 나무 자르기 (백준 2805번 스타일)

> 높이 H로 잘랐을 때 나무 윗부분의 합이 M미터 이상이 되는 최대 H를 구하라.

```java
import java.util.*;

public class ParametricSearch {

    // 높이 h로 잘랐을 때 얻는 나무의 총 길이
    static long getWood(int[] trees, long h) {
        long total = 0;
        for (int tree : trees) {
            if (tree > h) {
                total += tree - h;
            }
        }
        return total;
    }

    // M미터 이상을 얻을 수 있는 최대 높이 H를 이진 탐색으로 구함
    static long solve(int[] trees, long M) {
        long lo = 0;
        long hi = 0;
        for (int t : trees) hi = Math.max(hi, t);

        long answer = 0;
        while (lo <= hi) {
            long mid = (lo + hi) / 2;

            if (getWood(trees, mid) >= M) {
                // 조건 충족 → 더 높이 잘라도 될 수 있음
                answer = mid;
                lo = mid + 1;
            } else {
                // 조건 미충족 → 더 낮게 잘라야 함
                hi = mid - 1;
            }
        }
        return answer;
    }

    public static void main(String[] args) {
        int[] trees = {20, 15, 10, 17};
        long M = 7;
        // 높이 15로 자르면: (20-15) + 0 + 0 + (17-15) = 7 → 조건 충족
        System.out.println("최대 높이: " + solve(trees, M)); // 15
    }
}
```

---

## 4. 누적합 응용 (Prefix Sum Tricks)

기본 누적합을 확장한 **차이 배열(Difference Array)**과 **IMOS법**으로 구간 업데이트를 \(O(1)\)에 처리하는 기법이다.

### 핵심 아이디어
- **차이 배열**: 원본 배열 대신 **인접 원소 간의 차이**를 저장하는 배열을 관리한다.
  - 구간 `[l, r]`에 값 `v`를 더하려면 `diff[l] += v`, `diff[r+1] -= v`만 하면 된다.
  - 차이 배열의 누적합을 구하면 원본 배열이 복원된다.
- **IMOS법**: 차이 배열의 일본식 명칭. 시간/좌표 축에서 이벤트를 시작/끝으로 누적하고 마지막에 한 번 스윕한다.

### 언제 쓰는가
- 다수의 구간에 동일한 값을 더하는 쿼리가 반복될 때
- 모든 업데이트를 마친 후 최종 배열 상태를 구할 때
- 관련: [구간합](../array-techniques/prefix-sum.md)

### Java 구현: 차이 배열로 구간 덧셈

```java
import java.util.*;

public class DifferenceArray {

    // 차이 배열을 이용한 구간 업데이트
    // queries[i] = {l, r, v} → arr[l..r]에 v를 더함
    static int[] applyRangeUpdates(int n, int[][] queries) {
        int[] diff = new int[n + 1]; // 차이 배열 (인덱스 오버플로우 방지를 위해 +1)

        // 각 쿼리를 O(1)로 처리
        for (int[] q : queries) {
            int l = q[0], r = q[1], v = q[2];
            diff[l] += v;
            if (r + 1 < n) {
                diff[r + 1] -= v;
            }
        }

        // 누적합으로 원본 배열 복원
        int[] result = new int[n];
        result[0] = diff[0];
        for (int i = 1; i < n; i++) {
            result[i] = result[i - 1] + diff[i];
        }
        return result;
    }

    public static void main(String[] args) {
        int n = 6;
        int[][] queries = {
            {1, 3, 2},  // arr[1..3] += 2
            {2, 5, 3},  // arr[2..5] += 3
            {0, 2, 1},  // arr[0..2] += 1
        };

        int[] result = applyRangeUpdates(n, queries);
        System.out.println(Arrays.toString(result));
        // [1, 3, 6, 5, 3, 3]
        // 인덱스: 0  1  2  3  4  5
        //   q1:   0  2  2  2  0  0
        //   q2:   0  0  3  3  3  3
        //   q3:   1  1  1  0  0  0
        //   합:   1  3  6  5  3  3
    }
}
```

---

## 5. 비트 연산 트릭 (Bit Manipulation Tricks)

비트 단위 연산으로 **부분집합 열거, 비트 카운트, LSB 추출** 등을 효율적으로 처리하는 기법이다.

### 핵심 아이디어
| 연산 | 코드 | 설명 |
|---|---|---|
| **최하위 비트(LSB)** | `x & (-x)` | 켜진 비트 중 가장 낮은 비트를 추출. BIT(펜윅 트리)의 핵심 연산 |
| **비트 카운트** | `Integer.bitCount(x)` | 켜진 비트의 수를 반환 |
| **특정 비트 확인** | `(x >> k) & 1` | k번째 비트가 켜져 있는지 확인 |
| **특정 비트 켜기** | `x \| (1 << k)` | k번째 비트를 1로 설정 |
| **특정 비트 끄기** | `x & ~(1 << k)` | k번째 비트를 0으로 설정 |
| **부분집합 열거** | 아래 코드 참조 | 비트마스크의 모든 부분집합을 순회 |

### 언제 쓰는가
- 비트마스크 DP에서 부분집합을 열거할 때
- 펜윅 트리(BIT)에서 LSB를 이용한 인덱스 이동
- 원소 수 20개 이하에서 모든 부분집합을 탐색할 때
- 비트 단위로 상태를 압축해야 할 때

### Java 구현

```java
public class BitTricks {

    public static void main(String[] args) {
        // === 1. 부분집합 열거 ===
        // 비트마스크 mask의 모든 부분집합(자기 자신 포함, 공집합 제외)을 순회
        int mask = 0b1011; // {0, 1, 3}
        System.out.println("=== 부분집합 열거 (mask=" + Integer.toBinaryString(mask) + ") ===");
        for (int sub = mask; sub > 0; sub = (sub - 1) & mask) {
            System.out.println(Integer.toBinaryString(sub));
        }
        // 1011, 1010, 1001, 1000, 0011, 0010, 0001

        // === 2. 비트 카운트 ===
        int x = 0b110101; // 53
        System.out.println("\n비트 카운트(" + x + "): " + Integer.bitCount(x)); // 4

        // === 3. 최하위 비트(LSB) 추출 ===
        // x = 12 = 1100 → LSB = 0100 = 4
        x = 12;
        int lsb = x & (-x);
        System.out.println("LSB(" + x + "): " + lsb); // 4

        // === 4. 모든 부분집합 탐색 (원소 N개) ===
        // N개의 원소에서 가능한 모든 부분집합: 2^N개
        int N = 3;
        String[] items = {"A", "B", "C"};
        System.out.println("\n=== 전체 부분집합 (N=" + N + ") ===");
        for (int bit = 0; bit < (1 << N); bit++) {
            StringBuilder sb = new StringBuilder("{");
            for (int i = 0; i < N; i++) {
                if ((bit & (1 << i)) != 0) {
                    if (sb.length() > 1) sb.append(", ");
                    sb.append(items[i]);
                }
            }
            sb.append("}");
            System.out.println(sb);
        }
        // {}, {A}, {B}, {A, B}, {C}, {A, C}, {B, C}, {A, B, C}

        // === 5. 특정 비트 조작 ===
        int val = 0b1010;
        System.out.println("\n원본:           " + Integer.toBinaryString(val));
        System.out.println("2번 비트 켜기:  " + Integer.toBinaryString(val | (1 << 2)));
        System.out.println("3번 비트 끄기:  " + Integer.toBinaryString(val & ~(1 << 3)));
        System.out.println("1번 비트 확인:  " + ((val >> 1) & 1));
    }
}
```

---

## 요약 비교

| 테크닉 | 핵심 | 시간 복잡도 | 대표 결합 |
|---|---|---|---|
| **좌표 압축** | 값 → 순위 변환 | \(O(N \log N)\) | 세그먼트 트리, BIT, 스위핑 |
| **라인 스위핑** | 이벤트 정렬 후 순회 | \(O(N \log N)\) | 정렬, 우선순위 큐 |
| **매개변수 탐색** | 최적화 → 결정 문제 변환 | \(O(N \log X)\) | 이진 탐색, 그리디 |
| **차이 배열 / IMOS** | 구간 업데이트 O(1) | \(O(N + Q)\) | 누적합 |
| **비트 연산 트릭** | 비트 단위 상태 압축 | 연산 \(O(1)\) | 비트마스크 DP, 펜윅 트리 |
