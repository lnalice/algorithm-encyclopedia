# 그리디 (탐욕법)

## 분류
- **카테고리**: 탐욕법
- **관련 알고리즘**: [다익스트라](../shortest-path/dijkstra.md), [크루스칼](../minimum-spanning-tree/kruskal.md), [DP](../dynamic-programming/dp.md)

> **참고**: 다익스트라 알고리즘(최단 경로)과 크루스칼 알고리즘(최소 신장 트리)은 그리디 전략을 핵심 원리로 사용하는 대표적인 하위 알고리즘이다. 다익스트라는 "현재까지 거리가 가장 짧은 정점을 우선 선택"하는 그리디 선택을, 크루스칼은 "가중치가 가장 작은 간선부터 선택"하는 그리디 선택을 수행한다.

## 개요
그리디 알고리즘은 각 단계에서 **지역적으로 최적인 선택(Local Optimum)**을 반복하여 **전역 최적해(Global Optimum)**를 구하는 알고리즘 설계 기법이다. 이전 선택을 번복하지 않으며, 한 번 결정한 것은 그대로 유지한다. 그리디가 최적해를 보장하려면 **탐욕적 선택 속성(Greedy Choice Property)**과 **최적 부분 구조(Optimal Substructure)**를 만족해야 한다.

## 핵심 로직
1. **정렬 또는 우선순위 설정**: 탐욕적 기준에 따라 후보를 정렬하거나 우선순위 큐를 사용한다.
2. **탐욕적 선택 (Greedy Choice)**: 현재 상황에서 가장 좋아 보이는 선택을 한다.
3. **유효성 검사**: 선택이 제약 조건을 만족하는지 확인한다.
4. **반복**: 더 이상 선택할 것이 없을 때까지 2-3을 반복한다.

> **그리디가 정당한지 증명하는 방법**:
> - **탐욕적 선택 속성**: 지역 최적 선택이 전역 최적해에 포함됨을 보인다.
> - **교환 논증 (Exchange Argument)**: 그리디가 아닌 해를 그리디 해로 교환해도 결과가 나빠지지 않음을 보인다.

## 그리디 기반 주요 알고리즘
| 알고리즘 | 그리디 전략 | 문제 |
|---|---|---|
| **다익스트라** | 현재 최단 거리 정점을 우선 선택 | 단일 출발점 최단 경로 |
| **크루스칼** | 최소 가중치 간선부터 선택 (사이클 방지) | 최소 신장 트리 |
| **프림** | 현재 트리에서 가장 가까운 정점 선택 | 최소 신장 트리 |
| **허프만 코딩** | 빈도가 가장 낮은 두 노드를 병합 | 데이터 압축 |
| **활동 선택** | 종료 시간이 가장 빠른 활동 선택 | 구간 스케줄링 |

## 언제 사용하는가?
- **활동 선택 / 회의실 배정**: 겹치지 않는 최대 활동 수 선택
- **거스름돈 문제**: 최소 동전 개수로 거스름돈 만들기 (단, 동전 단위가 배수 관계일 때)
- **분할 가능 배낭 (Fractional Knapsack)**: 물건을 쪼갤 수 있는 배낭 문제
- **허프만 코딩**: 최적 접두사 코드 생성
- **최단 경로 (다익스트라)**: 가중치가 양수인 그래프에서 최단 경로
- **최소 신장 트리 (크루스칼/프림)**: 모든 정점을 최소 비용으로 연결
- **구간 문제**: 구간 합치기, 구간 커버링 등

## Java 구현
```java
import java.util.*;

public class Greedy {

    // ========================================
    // 활동 선택 문제 (Activity Selection Problem)
    // 겹치지 않는 활동을 최대한 많이 선택한다.
    // 그리디 전략: 종료 시간이 빠른 활동부터 선택
    // ========================================
    static List<int[]> activitySelection(int[][] activities) {
        // 종료 시간 기준으로 정렬 (종료 시간이 같으면 시작 시간 기준)
        Arrays.sort(activities, (a, b) -> {
            if (a[1] != b[1]) return a[1] - b[1];
            return a[0] - b[0];
        });

        List<int[]> selected = new ArrayList<>();
        int lastEndTime = 0; // 마지막으로 선택한 활동의 종료 시간

        for (int[] activity : activities) {
            int start = activity[0];
            int end = activity[1];

            // 현재 활동의 시작 시간이 마지막 선택 활동의 종료 시간 이후이면 선택
            if (start >= lastEndTime) {
                selected.add(activity);
                lastEndTime = end;
            }
        }

        return selected;
    }

    // ========================================
    // 거스름돈 문제
    // 그리디 전략: 가장 큰 단위의 동전부터 사용
    // 주의: 동전 단위가 배수 관계가 아니면 그리디가 최적이 아닐 수 있음
    // ========================================
    static Map<Integer, Integer> makeChange(int amount, int[] coins) {
        // 동전을 내림차순 정렬
        Arrays.sort(coins);
        Map<Integer, Integer> result = new LinkedHashMap<>();

        for (int i = coins.length - 1; i >= 0; i--) {
            if (amount <= 0) break;

            int count = amount / coins[i]; // 해당 동전으로 낼 수 있는 최대 개수
            if (count > 0) {
                result.put(coins[i], count);
                amount -= coins[i] * count;
            }
        }

        return result;
    }

    // ========================================
    // 분할 가능 배낭 문제 (Fractional Knapsack)
    // 그리디 전략: 단위 무게당 가치가 높은 물건부터 담기
    // ========================================
    static double fractionalKnapsack(int capacity, int[][] items) {
        // items[i] = {무게, 가치}
        // 단위 무게당 가치 기준 내림차순 정렬
        Arrays.sort(items, (a, b) ->
            Double.compare((double) b[1] / b[0], (double) a[1] / a[0])
        );

        double totalValue = 0;
        int remainingCapacity = capacity;

        for (int[] item : items) {
            int weight = item[0];
            int value = item[1];

            if (remainingCapacity >= weight) {
                // 물건 전체를 담을 수 있는 경우
                totalValue += value;
                remainingCapacity -= weight;
            } else {
                // 일부만 담는 경우 (분할)
                totalValue += (double) value * remainingCapacity / weight;
                break; // 배낭이 가득 참
            }
        }

        return totalValue;
    }

    public static void main(String[] args) {
        // === 활동 선택 문제 ===
        System.out.println("=== 활동 선택 문제 ===");
        int[][] activities = {
            {1, 4}, {3, 5}, {0, 6}, {5, 7}, {3, 9}, {5, 9},
            {6, 10}, {8, 11}, {8, 12}, {2, 14}, {12, 16}
        };

        List<int[]> selected = activitySelection(activities);
        System.out.println("선택된 활동 수: " + selected.size());
        for (int[] act : selected) {
            System.out.println("  시작: " + act[0] + ", 종료: " + act[1]);
        }

        // === 거스름돈 문제 ===
        System.out.println("\n=== 거스름돈 문제 ===");
        int[] coins = {500, 100, 50, 10};
        int amount = 1260;
        Map<Integer, Integer> change = makeChange(amount, coins);
        System.out.println(amount + "원 거스름돈:");
        int totalCoins = 0;
        for (Map.Entry<Integer, Integer> entry : change.entrySet()) {
            System.out.println("  " + entry.getKey() + "원: " + entry.getValue() + "개");
            totalCoins += entry.getValue();
        }
        System.out.println("총 동전 수: " + totalCoins);

        // === 분할 가능 배낭 문제 ===
        System.out.println("\n=== 분할 가능 배낭 문제 ===");
        int[][] items = {{10, 60}, {20, 100}, {30, 120}}; // {무게, 가치}
        int capacity = 50;
        double maxValue = fractionalKnapsack(capacity, items);
        System.out.println("배낭 용량: " + capacity);
        System.out.printf("최대 가치: %.2f%n", maxValue);
    }
}
```

## 시간 복잡도 / 공간 복잡도
| 문제 | 시간 복잡도 | 공간 복잡도 |
|---|---|---|
| **활동 선택** | \(O(N \log N)\) — 정렬 지배적 | \(O(N)\) |
| **거스름돈** | \(O(K)\) — K는 동전 종류 수 | \(O(K)\) |
| **분할 가능 배낭** | \(O(N \log N)\) — 정렬 지배적 | \(O(N)\) |
| **다익스트라** | \(O((V + E) \log V)\) — 우선순위 큐 사용 | \(O(V)\) |
| **크루스칼** | \(O(E \log E)\) — 간선 정렬 | \(O(V)\) |

## 관련 문제 유형
- **회의실 배정 / 활동 선택**: 겹치지 않는 구간 최대 선택
- **거스름돈 문제**: 최소 동전 개수
- **분할 가능 배낭**: 물건을 쪼갤 수 있는 최대 가치 배낭
- **구간 스케줄링**: 작업 시작/종료 시간 기반 최적 배정
- **최단 경로 / MST**: 다익스트라, 크루스칼, 프림 알고리즘
- **문자열 압축 (허프만)**: 빈도 기반 최적 코드 생성
