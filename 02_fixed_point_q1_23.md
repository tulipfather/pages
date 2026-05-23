# 🔢 Module 2. 24비트 고정소수점(Fixed-Point) Q1.23 수학

본 장에서는 초저전력 임베디드 오디오 코어 및 인공와우(Cochlear Implant) 신호처리 프로세서에서 연산의 기초가 되는 **24비트 Q1.23 고정소수점 표현법**의 산술적 정의, 레지스터 구조상에서의 비트 매핑, 사칙 연산 제어 규칙, 그리고 누적 연산 시의 오버플로우 방지 전략을 학술적으로 분석합니다.

---

## 1. Q1.23 포맷의 구조와 레지스터 비트 매핑

지수부와 가수부를 분리하여 표현하는 부동소수점(Floating-Point) 연산 방식은 하드웨어 연산 장치(FPU)의 물리적 크기 및 연속 구동에 따른 소모 전력이 큽니다. 이를 배제하기 위해 인공와우용 DSP는 일반 **24비트 정수형 레지스터**를 논리적으로 분할하여 소수점 위치를 고정하는 **고정소수점(Fixed-Point)** 방식을 수용합니다.

* **Q1.23 비트 레이아웃의 수학적 성질:**
  * **부호 비트 (Sign Bit, 1비트):** 최상위 비트(MSB, Bit 23)는 신호의 음/양 부호를 결정합니다 (2의 보수 부호화 정수 체계).
  * **소수부 비트 (Fractional Bits, 23비트):** 소수점 이하 자릿수를 23비트로 고정하여 정의합니다.

### 💾 24비트 레지스터 내 Q1.23 비트 레이아웃

<svg viewBox="0 0 800 160" width="100%" height="160" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="800" height="160" rx="8" style="fill: #f8f9fa !important; stroke: #e9ecef !important; stroke-width: 2 !important;" />
  
  <!-- Sign Bit Box -->
  <rect x="40" y="50" width="60" height="50" rx="3" style="fill: #ffe0b2 !important; stroke: #f57c00 !important; stroke-width: 2 !important;" />
  <text x="70" y="75" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #e65100 !important; font-weight: bold !important; text-anchor: middle !important;">부호(S)</text>
  <text x="70" y="93" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #f57c00 !important; text-anchor: middle !important;">Bit 23</text>

  <!-- Binary Point (소수점) -->
  <circle cx="107" cy="85" r="4" style="fill: #d32f2f !important;" />
  <text x="107" y="125" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #d32f2f !important; font-weight: bold !important; text-anchor: middle !important;">소수점 (Binary Point)</text>

  <!-- Fractional Bits Boxes -->
  <rect x="115" y="50" width="80" height="50" rx="3" style="fill: #c8e6c9 !important; stroke: #388e3c !important; stroke-width: 2 !important;" />
  <text x="155" y="75" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #1b5e20 !important; font-weight: bold !important; text-anchor: middle !important;">F_22 (2^-1)</text>
  <text x="155" y="93" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #388e3c !important; text-anchor: middle !important;">Bit 22</text>

  <rect x="200" y="50" width="80" height="50" rx="3" style="fill: #c8e6c9 !important; stroke: #388e3c !important; stroke-width: 2 !important;" />
  <text x="240" y="75" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #1b5e20 !important; font-weight: bold !important; text-anchor: middle !important;">F_21 (2^-2)</text>
  <text x="240" y="93" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #388e3c !important; text-anchor: middle !important;">Bit 21</text>

  <!-- Ellipsis -->
  <text x="320" y="80" style="font-family: Segoe UI, Arial !important; font-size: 24 !important; fill: #777 !important; text-anchor: middle !important;">. . .</text>

  <rect x="360" y="50" width="80" height="50" rx="3" style="fill: #c8e6c9 !important; stroke: #388e3c !important; stroke-width: 2 !important;" />
  <text x="400" y="75" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #1b5e20 !important; font-weight: bold !important; text-anchor: middle !important;">F_1 (2^-22)</text>
  <text x="400" y="93" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #388e3c !important; text-anchor: middle !important;">Bit 1</text>

  <rect x="445" y="50" width="80" height="50" rx="3" style="fill: #c8e6c9 !important; stroke: #388e3c !important; stroke-width: 2 !important;" />
  <text x="485" y="75" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #1b5e20 !important; font-weight: bold !important; text-anchor: middle !important;">F_0 (2^-23)</text>
  <text x="485" y="93" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #388e3c !important; text-anchor: middle !important;">Bit 0 (LSB)</text>

  <!-- Empty bits -->
  <rect x="580" y="50" width="180" height="50" rx="3" style="fill: #e1f5fe !important; stroke: #0288d1 !important; stroke-width: 2 !important;" />
  <text x="670" y="75" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #01579b !important; font-weight: bold !important; text-anchor: middle !important;">상위 8비트는 미사용</text>
  <text x="670" y="93" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #0288d1 !important; text-anchor: middle !important;">Bit 24~31 (32비트 정수 확장 시)</text>
</svg>

### 🔢 주요 실수(Float) 대 Q1.23 대응 테이블

| 십진 실수 (Float) | 정수 매핑 공식 ($x \times 2^{23}$) | 16진수 레지스터 값 (Hex, 24비트 마스크) | 이진수 비트열 (Q1.23 표현) |
| :--- | :--- | :--- | :--- |
| **$+1.0$ (상한 임계)** | $1.0 \times 2^{23} = 8,388,608$ | `0x800000` (오버플로우 상태) | `1000 0000 0000 0000 0000 0000` |
| **$+0.99999988$** | $2^{23} - 1 = 8,388,607$ | `0x7FFFFF` (표현 가능한 양의 최댓값) | `0111 1111 1111 1111 1111 1111` |
| **$+0.5$** | $0.5 \times 2^{23} = 4,194,304$ | `0x400000` | `0100 0000 0000 0000 0000 0000` |
| **$0.0$ (균형점)** | $0.0$ | `0x000000` | `0000 0000 0000 0000 0000 0000` |
| **$-0.25$** | $-0.25 \times 2^{23} = -2,097,152$ | `0xE00000` (2의 보수 표현식) | `1110 0000 0000 0000 0000 0000` |
| **$-1.0$ (음의 최솟값)** | $-1.0 \times 2^{23} = -8,388,608$ | `0x800000` (부호 정의상 대칭축) | `1000 0000 0000 0000 0000 0000` |

*(주의: 32비트 정수형(`int32_t`) 레지스터 구조 상에서 24비트 Q1.23 데이터 형식을 선언하여 사용할 때, 연산 시 상위 8비트는 부호에 비례하여 `0xFF` 또는 `0x00`으로 채워지는 부호 확장(Sign Extension)이 요구됩니다.)*

---

## 2. Q1.23 사칙연산의 수학적 특성과 시프트(Shift) 제어

### A. 덧셈 (Addition)
소수점의 물리적 분할 위치가 동일하므로, 일반적인 2의 보수 기반의 정수 덧셈기(Adder) 회로를 변경 없이 그대로 활용할 수 있습니다.
$$Z_{\text{Q1.23}} = X_{\text{Q1.23}} + Y_{\text{Q1.23}}$$
* **산술적 한계 및 오버플로우:** 두 양수 스칼라 값을 가산할 때 그 최종 합이 $+1.0$ 이상이 되는 시점(예: $0.6 + 0.5 = 1.1$), 최상위 부호 비트(Bit 23) 영역이 논리적으로 침범받아 양수 결과가 음수로 치환되는 치명적인 오버플로우 왜곡 현상이 발생하므로 하드웨어 포화 연산기가 반드시 요구됩니다.

### B. 곱셈 (Multiplication) 및 23비트 우측 시프트(Right Shift)의 필연성
Q1.23 스펙을 가지는 두 데이터를 곱하는 연산의 경우, 원본 데이터 각각에 $2^{23}$의 비트 가중치가 이미 곱해진 상태이므로 피승수와 승수가 곱해진 결과에는 $2^{46}$ 가중치 스케일이 이중으로 누적됩니다.
$$X_{\text{raw}} = x \cdot 2^{23}, \quad Y_{\text{raw}} = y \cdot 2^{23}$$
$$Z_{\text{raw\_mult}} = X_{\text{raw}} \times Y_{\text{raw}} = (x \times y) \cdot 2^{46}$$
따라서 출력 결과 데이터를 기본 포맷인 Q1.23 체계($2^{23}$ 가중치)로 되돌려주기 위해서는 곱셈 직후 반드시 **우측으로 23비트 산술 시프트(Arithmetic Shift Right) 연산**을 수행해 주어야 합니다.
$$Z_{\text{Q1.23}} = Z_{\text{raw\_mult}} \gg 23 = (x \times y) \cdot 2^{23}$$

#### 🎯 양자화 편향 제거를 위한 반올림(Rounding) 공식
단순 우측 시프트 연산($\gg 23$)은 소수점 아래의 값을 무조건 버리는 절삭(Truncation/Floor) 방식으로 작동합니다. 이는 소량의 오차가 음의 방향으로 계속 누적되는 양자화 편향(Quantization Bias)을 유발합니다. 이 편향을 최소화하기 위해 소수 스케일의 $0.5$에 해당하는 $2^{22}$ (16진수 `0x400000`) 값을 더한 후 시프트하는 **반올림(Rounding)** 공식을 적용합니다.
$$\text{Rounded Result} = (\text{temp} + \text{0x400000}) \gg 23$$
이 연산은 임베디드 오디오 필터 연산의 주파수 총조화왜곡(THD)을 유의미하게 개선합니다.

#### ⚠️ 곱셈 연산 시의 예외적 오버플로우와 포화 연산(Saturation)
고정소수점 영역 내에서 두 분수를 곱하면 그 절댓값은 항상 감소하지만, 오직 한 가지 예외인 **$-1.0 \times -1.0$** 상황에서 오버플로우가 발생합니다.
- Q1.23 포맷에서 $-1.0$은 부호가 있는 24비트 2의 보수로 $\text{0x800000}$ ($-8,388,608$)입니다.
- 이 두 값을 곱하면 $+2^{46}$이 유도되며, 이를 $23$비트 우측 시프트하면 $+2^{23}$ (Hex: $\text{0x800000}$, Dec: $8,388,608$)이 됩니다.
- 그러나 24비트 레지스터 상에서 $\text{0x800000}$은 **$-1.0$**으로 재해석됩니다. 즉, 양수가 되어야 할 곱셈 결과가 부호 반전으로 인해 도로 $-1.0$이 되는 심각한 왜곡이 발생합니다.
- 이를 방지하기 위해, 두 입력이 모두 $-1.0$인 예외 조건을 감지하여 표현 가능한 최댓값인 $1.0 - 2^{-23}$ (Hex: $\text{0x7FFFFF}$, Dec: $8,388,607$)로 포화 제한해 주는 **포화 연산(Saturation)** 알고리즘이 곱셈기에 구현되어야 합니다.

> [!TIP]
> **Q1.23 곱셈 산술의 오버플로우 안전성**
> 위 예외 경우인 $-1.0 \times -1.0$을 제외하면, 크기가 $1.0$ 이하인 복수의 분수값을 서로 승산한 결과의 절댓값은 항상 원래 값보다 작거나 같은 특성을 가집니다 ($|x \times y| \le 1.0$).
> 따라서 소수 스칼라 대역 내에서의 순수 곱셈 연산은 산술적 임계를 이탈하지 않으므로 오버플로우 한계로부터 원천적으로 안전합니다.

---

## 3. C언어 기반 고속 Q1.23 산술 구현 예제

아래 C 프로그래밍 코드는 실수형 음원을 Q1.23 고정소수점 규격으로 양자화하여 인코딩하고, 곱셈 연산 과정에서 부호 연산, 23비트 산술 우측 시프트, 반올림(Rounding) 및 포화(Saturation) 연산이 이루어지는 구조를 검증하는 코드입니다.

```c
#include <stdio.h>
#include <stdint.h>

#define Q23_SCALE (1 << 23)
#define FLOAT_TO_Q23(f) ((int32_t)((f) * Q23_SCALE))
#define Q23_TO_FLOAT(q) ((float)(q) / Q23_SCALE)

/**
 * @brief Q1.23 고정소수점 두 데이터에 대한 기본 곱셈(절삭 방식)을 수행합니다.
 * @param a 피승수 (Q1.23 포맷)
 * @param b 승수 (Q1.23 포맷)
 * @return int32_t 곱셈 결과 (Q1.23 포맷)
 */
int32_t q23_multiply(int32_t a, int32_t b) {
    // 두 32비트 정수를 곱하면 중간 결과로 최대 64비트가 유도됩니다 (Q2.46 포맷 발생).
    int64_t temp = (int64_t)a * (int64_t)b;
    // 소수점 스케일을 원래 상태로 축소하기 위해 23비트 산술 시프트 수행
    return (int32_t)(temp >> 23);
}

/**
 * @brief Q1.23 고정소수점 두 데이터에 대해 반올림 및 포화 연산을 반영하여 곱셈을 수행합니다.
 * @param a 피승수 (Q1.23 포맷)
 * @param b 승수 (Q1.23 포맷)
 * @return int32_t 곱셈 및 포화 결과 (Q1.23 포맷)
 */
int32_t q23_multiply_sat(int32_t a, int32_t b) {
    // -1.0 * -1.0 예외 처리 (-1.0은 Q1.23에서 0x800000 즉 -8388608)
    // 이 연산은 +1.0 (Q1.23 범위를 벗어남)을 유도하므로 최댓값인 0x7FFFFF로 포화시킵니다.
    if (a == -8388608 && b == -8388608) {
        return 8388607; // 0x7FFFFF (표현 가능한 양의 최댓값: 1.0 - 2^-23)
    }

    int64_t temp = (int64_t)a * (int64_t)b;
    // 반올림 수행: 절삭되는 하위 23비트의 절반인 2^22 (0x400000)을 더한 후 시프트
    temp += 0x400000;
    
    int32_t result = (int32_t)(temp >> 23);
    return result;
}

/**
 * @brief 24비트 레지스터 구조를 시각적으로 출력하기 위한 헬퍼 함수
 * @param val 출력할 32비트 정수 데이터
 */
void print_bits_24(int32_t val) {
    uint32_t mask = 1 << 23;
    printf("[Sign] ");
    printf("%c . ", (val & mask) ? '1' : '0');
    mask >>= 1;
    for (int i = 22; i >= 0; i--) {
        printf("%c", (val & mask) ? '1' : '0');
        if (i % 4 == 0 && i != 0) printf(" ");
        mask >>= 1;
    }
    printf("\n");
}

int main() {
    float x_f = 0.5f;
    float y_f = -0.25f;

    // 1. 실수 변수값을 24비트 고정소수점으로 양자화 인코딩
    int32_t x_q = FLOAT_TO_Q23(x_f); // 4,194,304 -> 0x400000
    int32_t y_q = FLOAT_TO_Q23(y_f); // -2,097,152 -> 0xFFE00000 (상위 부호 확장 포함)

    printf("1. Q1.23 포맷 변환 검증:\n");
    printf("x = %f -> Q1.23 Hex: 0x%06X (Dec: %d)\n", x_f, x_q & 0xFFFFFF, x_q);
    printf("  비트 레이아웃: "); print_bits_24(x_q);
    
    printf("y = %f -> Q1.23 Hex: 0x%06X (Dec: %d)\n", y_f, y_q & 0xFFFFFF, y_q);
    printf("  비트 레이아웃: "); print_bits_24(y_q);

    // 2. Q1.23 포맷 고속 곱셈 수행 (일반 절삭)
    int32_t result_q = q23_multiply(x_q, y_q); // -1,048,576 -> 0xFFF00000
    float result_f = Q23_TO_FLOAT(result_q);

    printf("\n2. Q1.23 기본 곱셈(절삭) 결과 (x * y):\n");
    printf("결과 Q1.23 Hex: 0x%06X (Dec: %d)\n", result_q & 0xFFFFFF, result_q);
    printf("  비트 레이아웃: "); print_bits_24(result_q);
    printf("실수 역변환 복원값: %f\n", result_f);

    // 3. -1.0 * -1.0 오버플로우 왜곡 및 포화 연산(Saturation) 해결책 검증
    int32_t neg1_q = FLOAT_TO_Q23(-1.0f); // -8,388,608 -> 0x800000

    int32_t unsat_mult = q23_multiply(neg1_q, neg1_q);
    int32_t sat_mult = q23_multiply_sat(neg1_q, neg1_q);

    printf("\n3. -1.0 * -1.0 곱셈 오버플로우 및 포화 연산(Saturation) 검증:\n");
    printf("  입력값 a (-1.0f) Q1.23 Hex: 0x%06X\n", neg1_q & 0xFFFFFF);
    printf("  [일반 곱셈 적용시 결과] Hex: 0x%06X (실수 역변환: %f) -> 부호 반전 왜곡 발생!\n", 
           unsat_mult & 0xFFFFFF, Q23_TO_FLOAT(unsat_mult));
    printf("  [포화 곱셈 적용시 결과] Hex: 0x%06X (실수 역변환: %f) -> 최댓값(1.0 - 2^-23)으로 정상 포화!\n", 
           sat_mult & 0xFFFFFF, Q23_TO_FLOAT(sat_mult));

    return 0;
}
```

---

## 4. 가드 비트 누산기(Guard Bit Accumulator)를 활용한 오버플로우 방제

FIR 필터의 합성곱(Convolution)이나 고속 푸리에 변환(FFT)의 나비 연산은 여러 곱셈 결과를 연속하여 덧셈 누적하는 MAC(Multiply-Accumulate) 연산이 연쇄적으로 발생합니다. 이 과정에서 필연적으로 발생하는 누적 이득 오버플로우를 제어하기 위해, 하드웨어 플랫폼 레벨에서는 상위 비트에 **여분의 가드 비트(Guard Bit)가 설계된 48비트 또는 56비트 전용 누산기 레지스터(Accumulator)**를 적용합니다.

### 🧮 Accumulator 내 Guard Bit 동작 구조

```
  56비트 누산기 레지스터(56-bit Accumulator Register) 구조:
  +--------------------+------------------------+------------------------------------------+
  |  Guard Bits (8비트)|   Integer Part (1비트) |         Fractional Part (47비트)         |
  +--------------------+------------------------+------------------------------------------+
  |  [0000 0000]       |           [S]          | [F_46 . . . . . . . . . . . . . . . LSB] |
  +--------------------+------------------------+------------------------------------------+
  <- 가산 누적 헤드룸 마진 -> <- 기존 Q1.23 최상위 ->
```

* **누적 연산 보호 메커니즘:**
  1. 곱셈기(Multiplier)의 연산 결과가 48비트 또는 56비트 대역폭의 누산기에 직접 합산됩니다.
  2. 합산 결과값이 기준 부호 비트인 $1.0$ 한계를 돌파하더라도, 소수점 이상의 가산 비트가 상위 **Guard Bits (Bit 48~55)** 영역으로 안전하게 넘어가기 때문에 부호 비트 반전 왜곡이나 비트 유실 오염이 일어나지 않습니다.
  3. 총 $2^8 = 256$회에 이르는 연속적인 임계치 가산이 수행되는 동안 비트 무결성이 완벽하게 유지됩니다.
  4. 모든 필터 탭의 가산 누적이 종결된 최종 출력 시점에서, 하드웨어 포화 연산기(Saturation Unit)가 작동하여 56비트 내부 중간값을 원래 크기인 24비트 Q1.23으로 포화 스케일링(Saturation/Clipping)하고 최종 출력 24비트 레지스터로 이송합니다.

---

➡️ **다음 단계:** [Module 3. 나이퀴스트 이론과 주파수 폴딩](03_nyquist_folding.md)
