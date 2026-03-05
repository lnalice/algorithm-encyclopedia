# 오일러 피 함수 (Euler's Totient Function)

## 분류
- **카테고리**: 정수론
- **관련 알고리즘**: [유클리드 호제법](./euclidean-algorithm.md), [소수 판별](./primality-test.md), [에라토스테네스의 체](./sieve-of-eratosthenes.md)

## 개요
오일러 피 함수 \(\phi(n)\)은 1부터 n까지의 정수 중에서 n과 서로소(GCD가 1)인 수의 개수를 구하는 함수이다. 예를 들어 \(\phi(12) = 4\)인데, 12와 서로소인 수는 1, 5, 7, 11로 총 4개이다. 오일러 피 함수는 RSA 암호, 모듈러 역원 계산, 오일러 정리 등 정수론의 핵심 도구로 활용된다.

## 핵심 로직

### 단일 값 계산
오일러 피 함수의 공식은 소인수분해를 기반으로 한다:

\[\phi(n) = n \times \prod_{p | n} \left(1 - \frac{1}{p}\right)\]

여기서 p는 n의 서로 다른 소인수이다.

1. **결과값을 n으로 초기화**한다.
2. **2부터 √n까지** 순회하며 소인수 p를 찾는다.
3. **p가 n의 소인수이면**: `result = result / p * (p - 1)`을 적용한다.
4. **n에서 p를 모두 나눈다** (같은 소인수 중복 제거).
5. **순회 후 n > 1이면**: n 자체가 소인수이므로 한 번 더 적용한다.

### 에라토스테네스 체 방식 (1~N 배열 계산)
1. **배열 초기화**: `phi[i] = i`로 초기화한다.
2. **체 순회**: 2부터 N까지 순회하며, `phi[i] == i`이면 i는 소수이다.
3. **소수 배수 갱신**: 소수 i의 모든 배수 j에 대해 `phi[j] = phi[j] / i * (i - 1)`을 적용한다.

> 단일 값 계산은 \(O(\sqrt{N})\), 1~N 전체 계산은 에라토스테네스 체 방식으로 \(O(N \log \log N)\)에 가능하다.

## 언제 사용하는가?
- **n과 서로소인 수의 개수**: 1~n에서 n과 GCD가 1인 수의 개수
- **RSA 암호**: 공개키/비밀키 생성에서 \(\phi(n)\) 계산이 핵심
- **오일러 정리 활용**: \(a^{\phi(n)} \equiv 1 \pmod{n}\) (a와 n이 서로소)
- **모듈러 역원 계산**: 페르마 소정리의 일반화로 모듈러 역원을 구할 때
- **기약 분수 개수**: 분모가 n인 기약 분수의 개수 = \(\phi(n)\)
- **순환군의 생성원 개수**: 위수가 n인 순환군의 생성원 개수 = \(\phi(n)\)

## Java 구현
```java
import java.util.*;

public class EulerTotient {

    // === 단일 값 오일러 피 계산 ===
    // n과 서로소인 1~n 사이 정수의 개수를 반환한다.
    static long eulerPhi(long n) {
        long result = n;

        // 소인수분해하며 오일러 피 공식 적용
        for (long p = 2; p * p <= n; p++) {
            if (n % p == 0) {
                // p는 n의 소인수 → result에서 p의 기여분 반영
                result = result / p * (p - 1);

                // n에서 소인수 p를 모두 제거
                while (n % p == 0) {
                    n /= p;
                }
            }
        }

        // n이 1보다 크면 남은 n 자체가 소인수
        if (n > 1) {
            result = result / n * (n - 1);
        }

        return result;
    }

    // === 에라토스테네스 체 방식으로 1~N까지 오일러 피 배열 계산 ===
    // phi[i] = i와 서로소인 1~i 사이 정수의 개수
    static long[] eulerPhiSieve(int maxN) {
        long[] phi = new long[maxN + 1];

        // phi[i] = i로 초기화
        for (int i = 0; i <= maxN; i++) {
            phi[i] = i;
        }

        // 에라토스테네스 체와 유사한 방식으로 소수를 찾고 배수에 적용
        for (int i = 2; i <= maxN; i++) {
            if (phi[i] == i) { // i가 소수인 경우
                // i의 모든 배수에 대해 오일러 피 공식 적용
                for (int j = i; j <= maxN; j += i) {
                    phi[j] = phi[j] / i * (i - 1);
                }
            }
        }

        return phi;
    }

    public static void main(String[] args) {
        // === 단일 값 계산 예시 ===
        long[] testValues = {1, 6, 12, 36, 97};
        System.out.println("=== 단일 값 오일러 피 ===");
        for (long n : testValues) {
            System.out.println("φ(" + n + ") = " + eulerPhi(n));
        }
        // φ(1)=1, φ(6)=2, φ(12)=4, φ(36)=12, φ(97)=96

        // === 체 방식 배열 계산 예시 ===
        int maxN = 20;
        long[] phi = eulerPhiSieve(maxN);
        System.out.println("\n=== 1~" + maxN + " 오일러 피 (체 방식) ===");
        for (int i = 1; i <= maxN; i++) {
            System.out.printf("φ(%2d) = %d%n", i, phi[i]);
        }

        // === 검증: φ(n)의 합 성질 ===
        // n의 모든 약수 d에 대해 φ(d)의 합 = n
        int n = 12;
        long sum = 0;
        for (int d = 1; d <= n; d++) {
            if (n % d == 0) {
                sum += phi[d];
            }
        }
        System.out.println("\n" + n + "의 약수들의 φ 합: " + sum
                + " (= " + n + " 이어야 함)");
    }
}
```

## 시간 복잡도 / 공간 복잡도
| | 복잡도 |
|---|---|
| **시간 복잡도 (단일 값)** | \(O(\sqrt{N})\) — 소인수분해와 동일 |
| **시간 복잡도 (체 방식)** | \(O(N \log \log N)\) — 에라토스테네스의 체와 동일 |
| **공간 복잡도 (단일 값)** | \(O(1)\) — 변수만 사용 |
| **공간 복잡도 (체 방식)** | \(O(N)\) — phi 배열 저장 |

> 단일 값만 필요하면 \(O(\sqrt{N})\), 1~N 전체가 필요하면 체 방식 \(O(N \log \log N)\)을 사용한다.

## 관련 문제 유형
- **n과 서로소인 수의 개수**: 기본 오일러 피 함수 적용
- **기약 분수 세기**: 분모가 n인 기약 분수의 수 = \(\phi(n)\)
- **오일러 정리 응용**: \(a^b \mod m\) 계산 시 지수를 \(\phi(m)\)으로 줄이기
- **모듈러 거듭제곱의 주기**: 거듭제곱의 모듈러 주기가 \(\phi(m)\)의 약수
- **RSA 관련 문제**: 큰 수의 \(\phi(n)\) 계산이 핵심
- **약수의 오일러 피 합**: \(\sum_{d|n} \phi(d) = n\) 성질 활용
