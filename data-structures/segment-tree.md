# 세그먼트 트리 (Segment Tree)

## 분류
- **카테고리**: 자료구조 (분할 정복 기반)
- **관련 알고리즘**: [구간합 (Prefix Sum)](../array-techniques/prefix-sum.md), [분할 정복](../divide-and-conquer/divide-and-conquer.md), [펜윅 트리 (BIT)](../data-structures/fenwick-tree.md)

## 개요
세그먼트 트리는 **분할 정복의 원리를 자료구조에 적용**한 것으로, 배열의 구간에 대한 질의(Query)와 갱신(Update)을 효율적으로 처리하는 이진 트리 구조이다. 배열을 절반씩 나누어 트리의 각 노드가 특정 구간의 대표값(합, 최솟값, 최댓값 등)을 저장하며, 구간 질의와 점 갱신 모두 \(O(\log N)\)에 수행할 수 있다.

## 핵심 로직

### 분할 정복 원리의 적용
세그먼트 트리는 분할 정복을 **전처리 단계에서 적용**하여 트리를 구축하고, 이후 질의와 갱신에서도 같은 원리를 반복 활용한다:
- **Divide**: 구간 `[left, right]`를 `[left, mid]`와 `[mid+1, right]`로 분할
- **Conquer**: 각 부분 구간에 대해 재귀적으로 처리
- **Combine**: 두 자식 노드의 결과를 합쳐 부모 노드의 값을 결정

### Build (트리 구축) — \(O(N)\)
1. 리프 노드에 배열의 각 원소를 저장한다.
2. 내부 노드는 두 자식 노드의 값을 합산(또는 min/max)하여 저장한다.
3. 재귀적으로 아래에서 위로 구축한다.

### Query (구간 질의) — \(O(\log N)\)
1. 현재 노드의 구간이 질의 구간에 완전히 포함되면 해당 노드의 값을 반환한다.
2. 전혀 겹치지 않으면 항등원(합이면 0, 최솟값이면 ∞)을 반환한다.
3. 일부만 겹치면 왼쪽/오른쪽 자식으로 재귀하여 결과를 합친다.

### Update (점 갱신) — \(O(\log N)\)
1. 갱신할 위치가 속한 리프 노드까지 내려간다.
2. 리프 노드의 값을 변경한다.
3. 올라오면서 부모 노드들의 값을 갱신한다.

## 구간합(Prefix Sum)과의 비교
| 구분 | Prefix Sum | 세그먼트 트리 |
|------|-----------|-------------|
| 구간 합 질의 | \(O(1)\) | \(O(\log N)\) |
| 값 갱신 | \(O(N)\) — 누적합 재계산 필요 | \(O(\log N)\) |
| **적합한 상황** | 갱신 없이 질의만 반복 | **질의 + 갱신이 모두 빈번** |

> 갱신이 없다면 [구간합(Prefix Sum)](../array-techniques/prefix-sum.md)이 더 효율적이다. 갱신이 필요하면 세그먼트 트리를 사용한다.

## 언제 사용하는가?
- **구간 쿼리 + 갱신이 빈번한 경우** (핵심 사용 조건)
- **구간 합(Range Sum)**: 특정 구간 내 원소의 합
- **구간 최솟값/최댓값(Range Min/Max)**: 특정 구간 내 최소/최대 원소
- **구간 GCD**: 특정 구간의 최대공약수
- **Lazy Propagation을 활용한 구간 갱신**: 구간 전체에 값을 더하거나 변경
- **좌표 압축 + 세그먼트 트리**: 값의 범위가 큰 경우

## Java 구현

```java
public class SegmentTree {

    static int[] tree; // 세그먼트 트리 배열 (1-based 인덱싱)
    static int n;      // 원본 배열의 크기

    /**
     * 세그먼트 트리 구축 (Build)
     * 배열의 구간을 반으로 나누며 재귀적으로 트리를 만든다.
     *
     * @param arr   원본 배열
     * @param node  현재 트리 노드 번호
     * @param start 현재 노드가 담당하는 구간의 시작
     * @param end   현재 노드가 담당하는 구간의 끝
     */
    static void build(int[] arr, int node, int start, int end) {
        if (start == end) {
            tree[node] = arr[start]; // 리프 노드: 배열 원소 저장
            return;
        }
        int mid = (start + end) / 2;
        build(arr, node * 2, start, mid);       // 왼쪽 자식 구축
        build(arr, node * 2 + 1, mid + 1, end); // 오른쪽 자식 구축
        tree[node] = tree[node * 2] + tree[node * 2 + 1]; // 두 자식의 합
    }

    /**
     * 구간 합 질의 (Query)
     * [l, r] 구간에 해당하는 원소들의 합을 반환한다.
     *
     * @param node  현재 트리 노드 번호
     * @param start 현재 노드가 담당하는 구간의 시작
     * @param end   현재 노드가 담당하는 구간의 끝
     * @param l     질의 구간의 시작
     * @param r     질의 구간의 끝
     */
    static int query(int node, int start, int end, int l, int r) {
        if (r < start || end < l) {
            return 0; // 구간이 전혀 겹치지 않음 → 합의 항등원 반환
        }
        if (l <= start && end <= r) {
            return tree[node]; // 현재 구간이 질의 구간에 완전히 포함됨
        }
        int mid = (start + end) / 2;
        int leftResult = query(node * 2, start, mid, l, r);
        int rightResult = query(node * 2 + 1, mid + 1, end, l, r);
        return leftResult + rightResult;
    }

    /**
     * 점 갱신 (Update)
     * idx 위치의 값을 val로 변경하고, 관련 노드들을 갱신한다.
     *
     * @param node  현재 트리 노드 번호
     * @param start 현재 노드가 담당하는 구간의 시작
     * @param end   현재 노드가 담당하는 구간의 끝
     * @param idx   갱신할 배열 인덱스
     * @param val   새로운 값
     */
    static void update(int node, int start, int end, int idx, int val) {
        if (start == end) {
            tree[node] = val; // 리프 노드의 값 갱신
            return;
        }
        int mid = (start + end) / 2;
        if (idx <= mid) {
            update(node * 2, start, mid, idx, val);       // 왼쪽 자식으로 이동
        } else {
            update(node * 2 + 1, mid + 1, end, idx, val); // 오른쪽 자식으로 이동
        }
        tree[node] = tree[node * 2] + tree[node * 2 + 1]; // 부모 값 재계산
    }

    public static void main(String[] args) {
        int[] arr = {1, 3, 5, 7, 9, 11};
        n = arr.length;
        tree = new int[4 * n]; // 트리 배열 크기: 최대 4N

        // 트리 구축
        build(arr, 1, 0, n - 1);

        // 구간 합 질의: arr[1] + arr[2] + arr[3] = 3 + 5 + 7 = 15
        System.out.println("구간 [1, 3]의 합: " + query(1, 0, n - 1, 1, 3));

        // 점 갱신: arr[2]를 5 → 10으로 변경
        update(1, 0, n - 1, 2, 10);

        // 갱신 후 구간 합: arr[1] + arr[2] + arr[3] = 3 + 10 + 7 = 20
        System.out.println("갱신 후 구간 [1, 3]의 합: " + query(1, 0, n - 1, 1, 3));

        // 전체 구간 합
        System.out.println("전체 구간 [0, 5]의 합: " + query(1, 0, n - 1, 0, n - 1));
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 연산 | 시간 복잡도 | 비고 |
|------|-----------|------|
| Build (구축) | \(O(N)\) | 각 노드를 한 번씩 방문 |
| Query (구간 질의) | \(O(\log N)\) | 트리의 높이만큼 탐색 |
| Update (점 갱신) | \(O(\log N)\) | 루트까지 경로 갱신 |

- **공간 복잡도**: \(O(N)\) — 트리 배열 크기 약 `4N`

> Lazy Propagation을 추가하면 **구간 갱신(Range Update)**도 \(O(\log N)\)에 처리 가능하다.

## 관련 문제 유형
- **구간 합 쿼리**: 특정 구간의 합을 구하고 값을 갱신하는 문제
- **구간 최솟값/최댓값 쿼리**: RMQ (Range Minimum/Maximum Query)
- **구간 갱신**: Lazy Propagation을 활용한 구간 일괄 갱신
- **역순 쌍 세기**: 세그먼트 트리를 이용한 Inversion Count
- **좌표 압축 + 세그먼트 트리**: 값의 범위가 매우 큰 경우
- **머지 소트 트리**: 세그먼트 트리의 각 노드에 정렬된 리스트를 저장
