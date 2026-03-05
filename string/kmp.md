# KMP 알고리즘 (Knuth-Morris-Pratt)

## 분류
- **카테고리**: 문자열
- **관련 알고리즘**: [트라이](../data-structures/trie.md)

## 개요
KMP 알고리즘은 텍스트에서 패턴 문자열을 효율적으로 찾는 문자열 매칭 알고리즘이다. 단순 매칭(brute-force)은 불일치가 발생하면 텍스트의 다음 위치부터 처음부터 다시 비교하지만, KMP는 **실패 함수(pi 배열)**를 이용하여 이미 일치한 부분을 건너뛰므로 불필요한 비교를 제거한다. 이를 통해 시간 복잡도를 \(O(NM)\)에서 \(O(N+M)\)으로 개선한다.

## 핵심 로직

### 단순 매칭 vs KMP 비교

**단순 매칭 (Brute-Force)**:
- 텍스트의 모든 위치에서 패턴을 처음부터 하나씩 비교한다.
- 불일치 시 텍스트 위치를 한 칸만 이동하고 패턴을 처음부터 다시 비교한다.
- 최악의 경우 \(O(NM)\) — 예: 텍스트 `AAAAAB`, 패턴 `AAAB`

**KMP 알고리즘**:
- 불일치 시 **이미 일치한 접두사/접미사 정보**를 활용하여 패턴의 비교 위치를 효율적으로 이동한다.
- 텍스트 포인터는 절대 뒤로 돌아가지 않는다.
- 항상 \(O(N+M)\)

### 실패 함수 (Pi 배열) — 핵심 아이디어

실패 함수 `pi[i]`는 패턴의 `0~i` 부분 문자열에서 **접두사(prefix)와 접미사(suffix)가 같은 최대 길이**를 저장한다.

> **직관적 이해**: 패턴 비교 중 `j`번째 위치에서 불일치가 발생하면, `pi[j-1]`을 참조하여 패턴의 어디부터 다시 비교할지 결정한다. 이미 일치한 부분 중 "재활용 가능한 접두사"의 길이를 알려주는 것이다.

**예시**: 패턴 `ABACABAD`

| i | 부분 문자열 | 일치하는 접두사=접미사 | pi[i] |
|---|-----------|---------------------|-------|
| 0 | A         | (없음)               | 0     |
| 1 | AB        | (없음)               | 0     |
| 2 | ABA       | A                   | 1     |
| 3 | ABAC      | (없음)               | 0     |
| 4 | ABACA     | A                   | 1     |
| 5 | ABACAB    | AB                  | 2     |
| 6 | ABACABA   | ABA                 | 3     |
| 7 | ABACABAD  | (없음)               | 0     |

### 알고리즘 단계

**1단계 — 실패 함수(Pi 배열) 구축**:
1. `pi[0] = 0`으로 초기화한다. 길이가 1인 문자열은 자기 자신을 제외하면 접두사=접미사가 없다.
2. 두 포인터 `i`(현재 위치, 1부터 시작)와 `j`(매칭 길이, 0부터 시작)를 사용한다.
3. `pattern[i] == pattern[j]`이면 `j`를 증가시키고 `pi[i] = j`로 기록한다.
4. 불일치 시 `j > 0`이면 `j = pi[j-1]`로 되돌린다 (이전 접두사=접미사 정보 활용).
5. `j == 0`이고 불일치이면 `pi[i] = 0`으로 기록한다.

**2단계 — KMP 매칭**:
1. 텍스트 포인터 `i`와 패턴 포인터 `j`를 각각 0으로 초기화한다.
2. `text[i] == pattern[j]`이면 둘 다 증가시킨다.
3. `j`가 패턴 길이에 도달하면 매칭 성공 — 매칭 시작 위치를 기록하고 `j = pi[j-1]`로 업데이트한다.
4. 불일치 시 `j > 0`이면 `j = pi[j-1]`로 되돌린다.
5. `j == 0`이고 불일치이면 `i`만 증가시킨다.

## 언제 사용하는가?
- **문자열에서 패턴 찾기**: 긴 텍스트에서 특정 패턴이 등장하는 모든 위치를 효율적으로 찾아야 할 때
- **반복되는 부분 문자열 검출**: 문자열 내에서 반복 구조를 찾는 문제 (예: 문자열 `S`의 최소 반복 단위)
- **문자열 주기 찾기**: `pi[n-1]`을 이용하면 문자열의 주기를 구할 수 있다 — 주기 = `n - pi[n-1]`
- **여러 패턴 동시 검색의 기초**: 아호-코라식 알고리즘의 기반이 되는 개념

## Java 구현
```java
import java.util.*;

public class KMP {

    // 실패 함수 (Pi 배열) 구축
    // pi[i] = pattern[0..i]에서 접두사와 접미사가 같은 최대 길이
    static int[] buildFailureFunction(String pattern) {
        int m = pattern.length();
        int[] pi = new int[m];
        // pi[0]은 항상 0 (길이 1인 문자열의 진접두사/진접미사는 없음)

        int j = 0; // 현재까지 일치한 접두사 길이
        for (int i = 1; i < m; i++) {
            // 불일치 시 이전 접두사=접미사 정보로 되돌림
            while (j > 0 && pattern.charAt(i) != pattern.charAt(j)) {
                j = pi[j - 1];
            }
            // 일치하면 매칭 길이 증가
            if (pattern.charAt(i) == pattern.charAt(j)) {
                j++;
            }
            pi[i] = j;
        }

        return pi;
    }

    // KMP 문자열 매칭 — 모든 매칭 위치를 반환
    static List<Integer> kmpSearch(String text, String pattern) {
        List<Integer> result = new ArrayList<>();
        int n = text.length();
        int m = pattern.length();

        if (m == 0 || n < m) return result;

        int[] pi = buildFailureFunction(pattern);

        int j = 0; // 패턴에서 현재 비교 위치
        for (int i = 0; i < n; i++) {
            // 불일치 시 실패 함수를 따라 이동
            while (j > 0 && text.charAt(i) != pattern.charAt(j)) {
                j = pi[j - 1];
            }
            // 현재 문자가 일치하면 패턴 포인터 전진
            if (text.charAt(i) == pattern.charAt(j)) {
                j++;
            }
            // 패턴 전체가 일치한 경우
            if (j == m) {
                result.add(i - m + 1); // 매칭 시작 인덱스 저장
                j = pi[j - 1];         // 다음 매칭을 위해 실패 함수 참조
            }
        }

        return result;
    }

    // 문자열의 최소 반복 단위(주기) 구하기
    static int findMinimumPeriod(String s) {
        int[] pi = buildFailureFunction(s);
        int n = s.length();
        int period = n - pi[n - 1];
        // n이 period로 나누어 떨어지면 완전한 반복 구조
        if (n % period == 0) {
            return period;
        }
        return n; // 반복 구조가 없으면 전체 문자열이 최소 단위
    }

    public static void main(String[] args) {
        // === 예시 1: 기본 패턴 매칭 ===
        String text = "ABABDABACDABABCABAB";
        String pattern = "ABABCABAB";

        System.out.println("=== KMP 문자열 매칭 ===");
        System.out.println("텍스트 : " + text);
        System.out.println("패턴   : " + pattern);

        int[] pi = buildFailureFunction(pattern);
        System.out.println("실패 함수(pi): " + Arrays.toString(pi));

        List<Integer> matches = kmpSearch(text, pattern);
        if (matches.isEmpty()) {
            System.out.println("매칭 결과: 패턴을 찾을 수 없음");
        } else {
            System.out.println("매칭 위치: " + matches);
            for (int pos : matches) {
                System.out.println("  인덱스 " + pos + ": \""
                        + text.substring(pos, pos + pattern.length()) + "\"");
            }
        }

        // === 예시 2: 중복 매칭 (오버래핑) ===
        System.out.println("\n=== 중복 매칭 예시 ===");
        String text2 = "AAAAAA";
        String pattern2 = "AAA";
        List<Integer> matches2 = kmpSearch(text2, pattern2);
        System.out.println("텍스트: " + text2 + ", 패턴: " + pattern2);
        System.out.println("매칭 위치: " + matches2);

        // === 예시 3: 문자열 최소 반복 단위 ===
        System.out.println("\n=== 문자열 최소 반복 단위 ===");
        String[] testStrings = {"ABCABCABC", "ABABABAB", "ABCDEF", "AAAA"};
        for (String s : testStrings) {
            int period = findMinimumPeriod(s);
            System.out.println("\"" + s + "\" → 최소 반복 단위 길이: " + period
                    + " (\"" + s.substring(0, period) + "\")");
        }
    }
}
```

## 시간 복잡도 / 공간 복잡도
| | 복잡도 |
|---|---|
| **시간 복잡도 (실패 함수 구축)** | \(O(M)\) — 패턴 길이에 비례 |
| **시간 복잡도 (KMP 매칭)** | \(O(N)\) — 텍스트 길이에 비례 |
| **시간 복잡도 (전체)** | \(O(N + M)\) — 텍스트 길이 N, 패턴 길이 M |
| **공간 복잡도** | \(O(M)\) — pi 배열 저장 |

> **단순 매칭과의 비교**: 단순 매칭은 최악 \(O(NM)\)이지만, KMP는 텍스트 포인터가 절대 뒤로 돌아가지 않으므로 항상 \(O(N+M)\)을 보장한다.

## 관련 문제 유형
- **문자열 검색**: 텍스트에서 패턴이 등장하는 모든 위치 찾기
- **부분 문자열 판별**: 문자열 A가 문자열 B에 포함되는지 확인
- **문자열 반복 주기**: 문자열의 최소 반복 단위 구하기 (pi 배열 활용)
- **문자열 접두사-접미사 매칭**: 접두사와 접미사가 같은 모든 길이 구하기
- **순환 문자열 검색**: 문자열을 두 번 이어 붙여서 순환 포함 여부 판별
