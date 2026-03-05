# 조합론 (Combinatorics) — 순열, 조합, 이항계수

## 분류
- **카테고리**: 조합론 / 수학
- **관련 알고리즘**: [DP](../dynamic-programming/dp.md), [유클리드 호제법](../number-theory/euclidean-algorithm.md), [오일러 피](../number-theory/euler-totient.md), [백트래킹](../backtracking/backtracking.md)

## 개요
조합론은 유한한 대상을 **세고, 나열하고, 선택하는** 방법을 다루는 수학 분야이다. 알고리즘 문제에서는 순열(Permutation), 조합(Combination), 이항계수(Binomial Coefficient)가 핵심이며, "경우의 수를 구하시오" 류의 문제에서 빠지지 않고 등장한다. 큰 수에 대한 모듈러 연산(\(\mod p\))과 결합되는 경우가 많아, 팩토리얼 전처리와 모듈러 역원 기법을 함께 익혀야 한다.

## 핵심 로직

### 1. 순열 (Permutation)
n개의 원소 중 r개를 **순서를 고려하여** 나열하는 경우의 수이다.

\[nPr = \frac{n!}{(n-r)!}\]

- 예: 5명 중 3명을 뽑아 줄 세우기 → \(5P3 = 5 \times 4 \times 3 = 60\)
- **중복 순열**: 같은 원소를 반복 선택 가능하면 \(n^r\)가지이다.

### 2. 조합 (Combination)
n개의 원소 중 r개를 **순서 없이** 선택하는 경우의 수이다.

\[\binom{n}{r} = nCr = \frac{n!}{r! \times (n-r)!}\]

- 예: 5명 중 3명을 뽑기 → \(\binom{5}{3} = 10\)
- **중복 조합**: 같은 원소를 반복 선택 가능하면 \(\binom{n+r-1}{r} = \binom{n+r-1}{n-1}\)가지이다. 이는 "n종류에서 r개를 중복 허용하여 고르기"에 해당한다.

### 3. 이항계수 계산 방법

#### 방법 1: 파스칼 삼각형 (DP)
이항계수의 점화 관계를 이용한다:

\[\binom{n}{r} = \binom{n-1}{r-1} + \binom{n-1}{r}\]

- **기저 조건**: \(\binom{n}{0} = \binom{n}{n} = 1\)
- 2차원 DP 배열 `dp[n][r]`에 값을 채워나간다.
- **장점**: 구현이 간단하고, 모듈러 연산이 자연스럽게 적용된다.
- **한계**: n이 크면 \(O(n \times r)\) 테이블이 필요하므로, n ≤ 수천 정도에서 적합하다.

#### 방법 2: 팩토리얼 + 모듈러 역원 (페르마 소정리)
n이 매우 클 때(최대 \(10^6\) 이상) 사용하는 방법이다.

\[\binom{n}{r} \equiv n! \times (r!)^{-1} \times ((n-r)!)^{-1} \pmod{p}\]

**모듈러 역원**은 **페르마 소정리**로 구한다. p가 소수일 때:

\[a^{-1} \equiv a^{p-2} \pmod{p}\]

1. **팩토리얼 배열 전처리**: `fact[i] = i! mod p`를 \(O(n)\)에 구한다.
2. **역팩토리얼 배열 전처리**: `inv_fact[n]`을 거듭제곱으로 구한 뒤, 역순으로 `inv_fact[i] = inv_fact[i+1] * (i+1) mod p`를 채운다.
3. **쿼리 응답**: \(\binom{n}{r} = \text{fact}[n] \times \text{inv\_fact}[r] \times \text{inv\_fact}[n-r] \mod p\)로 \(O(1)\)에 답한다.

> 전처리 \(O(n + \log p)\), 쿼리 \(O(1)\). **p가 소수**여야만 페르마 소정리를 적용할 수 있다.

### 4. 순열/조합 나열
경우의 수를 세는 것이 아니라, 실제로 모든 순열이나 조합을 **하나씩 나열**해야 하는 문제도 있다. C++의 `next_permutation`과 동일한 로직을 직접 구현할 수 있다:

1. **뒤에서부터** `arr[i] < arr[i+1]`인 가장 큰 i를 찾는다.
2. **뒤에서부터** `arr[i] < arr[j]`인 가장 큰 j를 찾는다.
3. `arr[i]`와 `arr[j]`를 **교환**한다.
4. i+1 이후 구간을 **뒤집는다**.

## 언제 사용하는가?
- **경우의 수 계산 문제**: "~를 선택/배치하는 경우의 수를 구하시오"
- **"~가지 방법" 문제**: 조합 공식이나 이항계수로 직접 계산
- **조합론 + DP 결합 문제**: 조합의 점화식을 DP 테이블로 구성하는 문제
- **확률 문제**: 전체 경우의 수 대비 특정 경우의 수 비율 계산
- **모듈러 연산이 포함된 큰 수 문제**: nCr mod p 형태로 출제되는 문제
- **순열 나열 / 사전순 k번째 순열 문제**: next_permutation 로직 활용

## Java 구현

### 1. 파스칼 삼각형으로 이항계수 구하기 (DP)
```java
import java.util.*;

public class BinomialDP {

    static final int MOD = 1_000_000_007;

    // 파스칼 삼각형 DP로 이항계수 테이블을 구축한다.
    // dp[i][j] = iCj mod MOD
    static long[][] buildPascal(int maxN) {
        long[][] dp = new long[maxN + 1][maxN + 1];

        for (int i = 0; i <= maxN; i++) {
            dp[i][0] = 1; // iC0 = 1
            for (int j = 1; j <= i; j++) {
                dp[i][j] = (dp[i - 1][j - 1] + dp[i - 1][j]) % MOD;
            }
        }

        return dp;
    }

    public static void main(String[] args) {
        int maxN = 10;
        long[][] C = buildPascal(maxN);

        System.out.println("=== 파스칼 삼각형 (0 ~ " + maxN + ") ===");
        for (int i = 0; i <= maxN; i++) {
            StringBuilder sb = new StringBuilder();
            for (int j = 0; j <= i; j++) {
                if (j > 0) sb.append(' ');
                sb.append(C[i][j]);
            }
            System.out.println(sb);
        }

        // 이항계수 쿼리 예시
        System.out.println("\n=== 이항계수 쿼리 ===");
        int[][] queries = {{5, 2}, {10, 3}, {10, 5}, {7, 0}, {7, 7}};
        for (int[] q : queries) {
            System.out.printf("C(%d, %d) = %d%n", q[0], q[1], C[q[0]][q[1]]);
        }
        // C(5,2)=10, C(10,3)=120, C(10,5)=252, C(7,0)=1, C(7,7)=1
    }
}
```

### 2. 팩토리얼 전처리 + 모듈러 역원으로 nCr 구하기 (페르마 소정리)
```java
import java.util.*;

public class BinomialFermat {

    static final int MOD = 1_000_000_007;
    static long[] fact;    // fact[i] = i! mod MOD
    static long[] invFact; // invFact[i] = (i!)^(-1) mod MOD

    // 모듈러 거듭제곱: base^exp mod MOD
    static long power(long base, long exp, long mod) {
        long result = 1;
        base %= mod;
        while (exp > 0) {
            if ((exp & 1) == 1) {
                result = result * base % mod;
            }
            base = base * base % mod;
            exp >>= 1;
        }
        return result;
    }

    // 팩토리얼 및 역팩토리얼 배열을 전처리한다.
    static void precompute(int maxN) {
        fact = new long[maxN + 1];
        invFact = new long[maxN + 1];

        fact[0] = 1;
        for (int i = 1; i <= maxN; i++) {
            fact[i] = fact[i - 1] * i % MOD;
        }

        // 페르마 소정리: a^(-1) ≡ a^(p-2) (mod p)
        invFact[maxN] = power(fact[maxN], MOD - 2, MOD);

        // 역순으로 역팩토리얼 계산: invFact[i] = invFact[i+1] * (i+1)
        for (int i = maxN - 1; i >= 0; i--) {
            invFact[i] = invFact[i + 1] * (i + 1) % MOD;
        }
    }

    // nCr mod MOD를 O(1)에 반환한다.
    static long nCr(int n, int r) {
        if (r < 0 || r > n) return 0;
        return fact[n] % MOD * invFact[r] % MOD * invFact[n - r] % MOD;
    }

    // nPr mod MOD를 O(1)에 반환한다.
    static long nPr(int n, int r) {
        if (r < 0 || r > n) return 0;
        return fact[n] % MOD * invFact[n - r] % MOD;
    }

    public static void main(String[] args) {
        int maxN = 1_000_000;
        precompute(maxN);

        System.out.println("=== 페르마 소정리 기반 이항계수 ===");
        int[][] queries = {{5, 2}, {10, 3}, {100, 50}, {1000000, 500000}};
        for (int[] q : queries) {
            System.out.printf("C(%d, %d) mod %d = %d%n",
                    q[0], q[1], MOD, nCr(q[0], q[1]));
        }

        System.out.println("\n=== 순열 수 ===");
        System.out.printf("P(%d, %d) mod %d = %d%n", 5, 3, MOD, nPr(5, 3));
        System.out.printf("P(%d, %d) mod %d = %d%n", 10, 4, MOD, nPr(10, 4));

        // 검증: C(5,2) = 10, P(5,3) = 60
        System.out.println("\n=== 검증 ===");
        System.out.println("C(5,2) = " + nCr(5, 2) + " (기대값: 10)");
        System.out.println("P(5,3) = " + nPr(5, 3) + " (기대값: 60)");
    }
}
```

### 3. 순열/조합 나열 (next_permutation 스타일)
```java
import java.util.*;

public class PermutationEnumeration {

    // 배열의 다음 사전순 순열을 구한다.
    // 마지막 순열이면 false를 반환한다.
    static boolean nextPermutation(int[] arr) {
        int n = arr.length;

        // 1단계: 뒤에서부터 arr[i] < arr[i+1]인 가장 큰 i를 찾는다
        int i = n - 2;
        while (i >= 0 && arr[i] >= arr[i + 1]) {
            i--;
        }

        if (i < 0) return false; // 이미 내림차순 → 마지막 순열

        // 2단계: 뒤에서부터 arr[i] < arr[j]인 가장 큰 j를 찾는다
        int j = n - 1;
        while (arr[j] <= arr[i]) {
            j--;
        }

        // 3단계: arr[i]와 arr[j]를 교환
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;

        // 4단계: i+1 이후 구간을 뒤집는다
        reverse(arr, i + 1, n - 1);

        return true;
    }

    static void reverse(int[] arr, int left, int right) {
        while (left < right) {
            int temp = arr[left];
            arr[left] = arr[right];
            arr[right] = temp;
            left++;
            right--;
        }
    }

    // nCr의 모든 조합을 백트래킹으로 나열한다.
    static void combinations(int n, int r, int start, List<Integer> current,
                             List<List<Integer>> result) {
        if (current.size() == r) {
            result.add(new ArrayList<>(current));
            return;
        }

        for (int i = start; i <= n; i++) {
            current.add(i);
            combinations(n, r, i + 1, current, result);
            current.remove(current.size() - 1);
        }
    }

    public static void main(String[] args) {
        // === 순열 나열 (next_permutation) ===
        System.out.println("=== {1, 2, 3}의 모든 순열 ===");
        int[] perm = {1, 2, 3};
        int permCount = 0;
        do {
            System.out.println(Arrays.toString(perm));
            permCount++;
        } while (nextPermutation(perm));
        System.out.println("총 " + permCount + "개\n");

        // === 조합 나열 (백트래킹) ===
        System.out.println("=== 5C3: {1..5}에서 3개 선택 ===");
        List<List<Integer>> combResult = new ArrayList<>();
        combinations(5, 3, 1, new ArrayList<>(), combResult);
        for (List<Integer> comb : combResult) {
            System.out.println(comb);
        }
        System.out.println("총 " + combResult.size() + "개\n");

        // === next_permutation으로 사전순 k번째 순열 구하기 ===
        int k = 5;
        int[] arr = {1, 2, 3, 4};
        for (int step = 1; step < k; step++) {
            nextPermutation(arr);
        }
        System.out.println("=== {1,2,3,4}의 사전순 " + k + "번째 순열 ===");
        System.out.println(Arrays.toString(arr));
        // {1,2,3,4}의 5번째 순열 = [1, 3, 4, 2]
    }
}
```

## 시간 복잡도 / 공간 복잡도

| 방법 | 시간 복잡도 | 공간 복잡도 |
|------|-------------|-------------|
| **파스칼 삼각형 (DP)** | \(O(n^2)\) 전처리, \(O(1)\) 쿼리 | \(O(n^2)\) |
| **팩토리얼 + 모듈러 역원** | \(O(n + \log p)\) 전처리, \(O(1)\) 쿼리 | \(O(n)\) |
| **순열 나열 (next_permutation)** | \(O(n)\) per step | \(O(1)\) 추가 공간 |
| **조합 나열 (백트래킹)** | \(O(\binom{n}{r})\) | \(O(r)\) 재귀 스택 |

> - n ≤ 수천이면 **파스칼 삼각형**, n ≤ \(10^6\)이면 **팩토리얼 + 모듈러 역원**을 선택한다.
> - 모든 순열/조합을 나열하는 문제는 경우의 수 자체가 지수적이므로, n이 작을 때만 가능하다.

## 관련 문제 유형
- **이항계수 직접 계산**: nCr mod p를 구하는 기본 문제
- **경우의 수 세기**: 조건에 맞는 선택/배치 방법의 수
- **확률 계산**: (유리한 경우의 수) / (전체 경우의 수) 형태
- **조합론 + DP 결합**: 점화식에 이항계수가 등장하는 문제
- **카탈란 수**: \(C_n = \binom{2n}{n} / (n+1)\)로 이항계수를 활용하는 대표 수열
- **포함-배제 원리**: 합집합의 크기를 조합으로 계산
- **격자 경로 세기**: (0,0)에서 (m,n)까지의 최단 경로 수 = \(\binom{m+n}{m}\)
- **중복 조합 문제**: 부정 방정식의 음이 아닌 정수 해의 개수
- **사전순 k번째 순열/조합**: next_permutation 또는 팩토리얼 수 체계 활용
