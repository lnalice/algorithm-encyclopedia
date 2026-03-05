# 유클리드 호제법 (Euclidean Algorithm)

## 분류
- **카테고리**: 정수론
- **관련 알고리즘**: [오일러 피 함수](./euler-totient.md), [확장 유클리드 호제법](#확장-유클리드-호제법-extended-gcd)

## 개요
유클리드 호제법은 두 정수의 최대공약수(GCD)를 효율적으로 구하는 알고리즘으로, 기원전 300년경 유클리드가 제시한 가장 오래된 알고리즘 중 하나이다. "두 수의 GCD는 큰 수에서 작은 수를 뺀 것과 작은 수의 GCD와 같다"는 성질을 이용하며, 나머지 연산으로 빠르게 수렴한다. 이를 확장한 확장 유클리드 호제법은 모듈러 역원, 디오판토스 방정식 등 다양한 정수론 문제의 핵심 도구이다.

## 핵심 로직

### 기본 유클리드 호제법 (GCD)
핵심 성질: \(\gcd(a, b) = \gcd(b, a \mod b)\), \(\gcd(a, 0) = a\)

1. **b가 0이면** a가 GCD이다.
2. **b가 0이 아니면** `gcd(b, a % b)`를 재귀 호출한다.
3. b가 0이 될 때까지 반복하면 GCD를 구할 수 있다.

### 확장 유클리드 호제법 (Extended GCD)
\(ax + by = \gcd(a, b)\)를 만족하는 정수 x, y를 함께 구한다.

1. **기저 조건**: `b = 0`이면 `x = 1, y = 0`이다 (a × 1 + 0 × 0 = a).
2. **재귀 호출**: `extGcd(b, a % b)`를 호출하여 x₁, y₁을 구한다.
3. **역추적**: `x = y₁`, `y = x₁ - (a / b) × y₁`로 현재 단계의 x, y를 계산한다.

### 최소공배수 (LCM)
\[\text{lcm}(a, b) = \frac{a \times b}{\gcd(a, b)}\]

> 오버플로우를 방지하기 위해 `a / gcd(a, b) * b` 순서로 계산한다.

## 언제 사용하는가?
- **최대공약수(GCD) 계산**: 두 수 또는 여러 수의 GCD
- **최소공배수(LCM) 계산**: GCD를 활용한 LCM 계산
- **모듈러 역원**: \(ax \equiv 1 \pmod{m}\)에서 x를 구할 때 (확장 유클리드)
- **디오판토스 방정식**: \(ax + by = c\)의 정수해 존재 여부 및 해 구하기
- **분수 약분**: 분자와 분모의 GCD로 기약분수 만들기
- **배열의 GCD/LCM**: 여러 수의 GCD/LCM을 순차적으로 계산

## Java 구현
```java
import java.util.*;

public class EuclideanAlgorithm {

    // === 기본 GCD (재귀) ===
    static long gcd(long a, long b) {
        if (b == 0) return a;
        return gcd(b, a % b);
    }

    // === 기본 GCD (반복문) ===
    static long gcdIterative(long a, long b) {
        while (b != 0) {
            long temp = b;
            b = a % b;
            a = temp;
        }
        return a;
    }

    // === LCM (최소공배수) ===
    // 오버플로우 방지를 위해 나눗셈을 먼저 수행한다.
    static long lcm(long a, long b) {
        return a / gcd(a, b) * b;
    }

    // === 확장 유클리드 호제법 ===
    // ax + by = gcd(a, b)를 만족하는 {gcd, x, y}를 반환한다.
    static long[] extGcd(long a, long b) {
        if (b == 0) {
            return new long[]{a, 1, 0}; // gcd = a, x = 1, y = 0
        }

        long[] result = extGcd(b, a % b);
        long g = result[0];
        long x1 = result[1];
        long y1 = result[2];

        // 역추적: 현재 단계의 x, y 계산
        long x = y1;
        long y = x1 - (a / b) * y1;

        return new long[]{g, x, y};
    }

    // === 모듈러 역원 ===
    // ax ≡ 1 (mod m)을 만족하는 x를 반환한다.
    // a와 m이 서로소가 아니면 -1을 반환한다.
    static long modInverse(long a, long m) {
        long[] result = extGcd(a, m);
        long g = result[0];
        long x = result[1];

        if (g != 1) return -1; // 역원이 존재하지 않음

        // x가 음수일 수 있으므로 양수로 변환
        return ((x % m) + m) % m;
    }

    // === 여러 수의 GCD ===
    static long gcdArray(long[] arr) {
        long result = arr[0];
        for (int i = 1; i < arr.length; i++) {
            result = gcd(result, arr[i]);
            if (result == 1) return 1; // GCD가 1이면 더 줄어들 수 없음
        }
        return result;
    }

    // === 여러 수의 LCM ===
    static long lcmArray(long[] arr) {
        long result = arr[0];
        for (int i = 1; i < arr.length; i++) {
            result = lcm(result, arr[i]);
        }
        return result;
    }

    public static void main(String[] args) {
        // === GCD / LCM 예시 ===
        long a = 48, b = 18;
        System.out.println("gcd(" + a + ", " + b + ") = " + gcd(a, b));       // 6
        System.out.println("lcm(" + a + ", " + b + ") = " + lcm(a, b));       // 144

        // === 확장 유클리드 호제법 예시 ===
        long[] ext = extGcd(a, b);
        System.out.println("\n확장 유클리드 호제법:");
        System.out.println(a + " × (" + ext[1] + ") + " + b + " × (" + ext[2]
                + ") = " + ext[0]);
        // 48 × (-1) + 18 × (3) = 6 → 검증: -48 + 54 = 6

        // === 모듈러 역원 예시 ===
        long num = 3, mod = 11;
        long inv = modInverse(num, mod);
        System.out.println("\n" + num + "의 mod " + mod + " 역원: " + inv);
        System.out.println("검증: " + num + " × " + inv + " mod " + mod
                + " = " + (num * inv % mod)); // 1

        // === 여러 수의 GCD / LCM 예시 ===
        long[] arr = {12, 18, 24};
        System.out.println("\n배열 " + Arrays.toString(arr) + "의 GCD: "
                + gcdArray(arr)); // 6
        System.out.println("배열 " + Arrays.toString(arr) + "의 LCM: "
                + lcmArray(arr)); // 72

        // === 디오판토스 방정식 예시 ===
        // 48x + 18y = 12 의 정수해
        long c = 12;
        long g = ext[0]; // gcd(48, 18) = 6
        if (c % g == 0) {
            long scale = c / g;
            long x0 = ext[1] * scale; // 특수해
            long y0 = ext[2] * scale;
            System.out.println("\n" + a + "x + " + b + "y = " + c + " 의 특수해:");
            System.out.println("x = " + x0 + ", y = " + y0);
            System.out.println("검증: " + a + "×" + x0 + " + " + b + "×" + y0
                    + " = " + (a * x0 + b * y0));
        } else {
            System.out.println("정수해가 존재하지 않음");
        }
    }
}
```

## 시간 복잡도 / 공간 복잡도
| | 복잡도 |
|---|---|
| **시간 복잡도 (GCD)** | \(O(\log(\min(a, b)))\) — 매 단계 나머지가 절반 이하로 감소 |
| **시간 복잡도 (확장 GCD)** | \(O(\log(\min(a, b)))\) — 기본 GCD와 동일 |
| **시간 복잡도 (LCM)** | \(O(\log(\min(a, b)))\) — GCD 계산 후 상수 연산 |
| **공간 복잡도 (재귀)** | \(O(\log(\min(a, b)))\) — 재귀 호출 스택 |
| **공간 복잡도 (반복문)** | \(O(1)\) — 변수만 사용 |

> 유클리드 호제법은 최악의 경우에도 \(O(\log N)\)으로 매우 효율적이다. 최악의 경우는 두 수가 연속 피보나치 수일 때이다.

## 관련 문제 유형
- **최대공약수 / 최소공배수**: 기본 GCD/LCM 계산
- **분수 약분**: 분자와 분모를 GCD로 나누어 기약분수 생성
- **모듈러 역원**: \(ax \equiv 1 \pmod{m}\) 풀기 (확장 유클리드)
- **디오판토스 방정식**: \(ax + by = c\)의 정수해 구하기
- **배열의 GCD**: N개의 수의 최대공약수 구하기
- **톱니바퀴 / 주기 문제**: 여러 주기의 LCM으로 동시 발생 시점 계산
- **중국인의 나머지 정리 (CRT)**: 확장 유클리드로 연립 합동식 풀기
