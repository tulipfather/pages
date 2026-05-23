# ⚡ Module 7. 실수 최적화 RDFT와 전극 채널 에너지 추출

본 장에서는 인공와우(Cochlear Implant)의 실시간 동작 시 전력 효율성을 극대화하기 위해 하드웨어 연산량과 메모리를 50% 절감하는 **실수 입력 이산 푸리에 변환(RDFT, Real Discrete Fourier Transform)**의 대칭적 특징과 언팩(Unpack) 최적화 알고리즘을 분석합니다. 또한, 연산된 주파수 에너지 스펙트럼 성분을 인간 청각의 물리적 비선형 인지 척도에 부합하도록 22개의 달팽이관 전극 채널로 통합 추출하는 **비닝(Binning) 매핑 기법**을 상술하며, FPU가 생략된 프로세서 상에서 **CLZ(Count Leading Zeros)** 명령어를 기반으로 고속 정수 로그 연산을 처리하는 참조 설계를 고찰합니다.

---

## 1. 실수 입력 신호의 켤레 대칭성(Conjugate Symmetry)과 257개 주파수 빈(Bin)의 원리

우리가 수집하여 처리하는 물리적인 목소리 및 환경 음향 데이터는 수학적으로 모두 실수(Real Number) 차원입니다. 실수 신호를 이산 푸리에 변환(DFT)할 때 유도되는 고유한 대칭성 규칙인 **켤레 대칭성(Conjugate Symmetry)**을 파악하고 주파수 분해 공간을 최적화합니다.

### A. 켤레 대칭성의 수학적 정의와 대칭 투영

512점 실수 데이터를 DFT 연산하면 결과물로 512개의 복소수 데이터 $X[k]$가 도출됩니다. 이때, 대역 중앙의 **나이퀴스트 주파수(Nyquist Frequency, $k=256$)** 성분을 경계선으로 하여 스펙트럼의 전반부와 후반부는 다음과 같은 완벽한 데칼코마니 대칭 관계가 형성됩니다.
$$X[512 - k] = X^*[k] \quad (0 < k < 256)$$
*(여기서 $X^*[k]$는 $X[k]$의 켤레 복소수입니다.)*

* **켤레 복소수 대칭쌍의 물리적 거동:**
  * 실수부(Real Part)의 진폭은 나이퀴스트 축을 대칭선으로 하여 거울 대칭을 이룹니다 ($Re(X[512-k]) = Re(X[k])$).
  * 허수부(Imaginary Part)의 값은 진폭의 크기는 완벽히 일치하되 **대칭선 방향을 축으로 부호가 반전**됩니다 ($Im(X[512-k]) = -Im(X[k])$).

<svg viewBox="0 0 800 240" width="100%" height="240" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="800" height="240" rx="8" style="fill: #f8f9fa !important; stroke: #e9ecef !important; stroke-width: 2 !important;" />

  <!-- Symmetric Spectrum Axes -->
  <g transform="translate(80, 180)">
    <!-- Base Line -->
    <line x1="0" y1="0" x2="640" y2="0" style="stroke: #495057 !important; stroke-width: 2 !important;" />
    <line x1="0" y1="5" x2="0" y2="-150" style="stroke: #495057 !important; stroke-dasharray: 2 2 !important;" />
    <line x1="320" y1="5" x2="320" y2="-150" style="stroke: #d32f2f !important; stroke-width: 2 !important;" />
    <line x1="640" y1="5" x2="640" y2="-150" style="stroke: #495057 !important; stroke-dasharray: 2 2 !important;" />

    <!-- Axis Ticks & Labels -->
    <text x="0" y="20" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #212529 !important; font-weight: bold !important; text-anchor: middle !important;">k = 0 (DC)</text>
    <text x="320" y="20" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #d32f2f !important; font-weight: bold !important; text-anchor: middle !important;">k = 256 (Nyquist, 8kHz)</text>
    <text x="640" y="20" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #212529 !important; font-weight: bold !important; text-anchor: middle !important;">k = 511 (16kHz)</text>

    <!-- Spectrum Magnitude (Real symmetric curve) -->
    <!-- Left side -->
    <path d="M 0,-10 C 40,-20 80,-120 120,-80 C 160,-40 200,-50 240,-110 C 280,-170 300,-10 320,-5" style="fill: none !important; stroke: #228be6 !important; stroke-width: 3 !important;" />
    <!-- Right side (Symmetric Mirror) -->
    <path d="M 640,-10 C 600,-20 560,-120 520,-80 C 480,-40 440,-50 400,-110 C 360,-170 340,-10 320,-5" style="fill: none !important; stroke: #228be6 !important; stroke-dasharray: 4 2 !important; stroke-width: 3 !important; stroke-opacity: 0.7 !important;" />

    <!-- Annotations -->
    <rect x="70" y="-140" width="130" height="40" rx="3" style="fill: #e7f5ff !important; stroke: #228be6 !important; stroke-dasharray: 2 2 !important;" />
    <text x="135" y="-125" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #1c7ed6 !important; font-weight: bold !important; text-anchor: middle !important;">유효 데이터 영역</text>
    <text x="135" y="-110" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #1c7ed6 !important; text-anchor: middle !important;">k = 0 ~ 256 (257 Bins)</text>

    <rect x="440" y="-140" width="130" height="40" rx="3" style="fill: #fff5f5 !important; stroke: #fa5252 !important; stroke-dasharray: 2 2 !important;" />
    <text x="505" y="-125" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #c92a2a !important; font-weight: bold !important; text-anchor: middle !important;">대칭 영역 (거울)</text>
    <text x="505" y="-110" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #c92a2a !important; text-anchor: middle !important;">연산 및 저장 불필요</text>
  </g>
</svg>

### B. 대칭 경계축의 포함에 따른 257개 주파수 빈 도출

수학적인 켤레 대칭성에 따라 절반의 데이터인 256개가 생성되는 것으로 보이나, 두 개 경계축을 상호 포함해야 하므로 총 유효 스펙트럼 기둥은 **$0 \sim 256$번까지 총 257개**로 수렴합니다.

1. **시작 경계 $k=0$ (DC 성분):** 시간축 데이터의 평균 오프셋에 부합하는 직류(Direct Current) 성분입니다. 대칭 투영의 시작 기점으로 삼으며, 허수부를 상실한 **순수 실수 스칼라**입니다.
2. **최종 경계 $k=256$ (나이퀴스트 대역):** 샘플링 주파수의 완전한 절반인 $8\text{ kHz}$ 성분입니다. 대칭 투영의 최종 축이자 회전 벡터 거울면이 위치하는 지점으로, 이 주파수 성분 역시 허수 값이 부재한 **순수 실수 스칼라**가 됩니다.
3. **매개 구간 $k=1 \dots 255$:** 복소 에너지 성분이 안정적으로 발현되는 실수와 허수 복소 결합 평면 구간이며, 반대편 대역인 $k=257 \dots 511$ 영역과 정확히 켤레 대칭을 이루어 매핑됩니다.

---

## 2. 512점 실수 입력 RDFT의 256점 복소 FFT 치환 및 언팩(Unpacking) 최적화

초저전력 배터리 동작을 담보해야 하는 인공와우 칩 내부에서는 512 실수 샘플 분석을 위해 범용 512점 복소 푸리에 변환(Complex FFT) 엔진을 구동하는 비효율을 배제합니다. 대신 **512 실수 데이터를 256 복소 버퍼에 구겨 넣는 패킹(Packing)을 거쳐 256점 복소 FFT를 계산한 뒤, 이의 출력물에 수학적 역변환(Unpack) 공식을 가해 512 실수 입력 스펙트럼의 고유 절반(257개 유효 빈)을 고속 산출합니다.**

### 💾 512 실수 샘플의 256 복소 버퍼 매핑(Packing) 레이아웃

512 실수 입력 표본 $x[n]$의 짝수 인덱스(Even)는 가상의 복소 데이터 실수부(`real`)에, 홀수 인덱스(Odd)는 허수부(`imag`)에 1대1 교차 매핑하여 256 크기의 복소 구조체 배열 $y[n]$으로 구성합니다.

<svg viewBox="0 0 800 220" width="100%" height="220" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="800" height="220" rx="8" style="fill: #f8f9fa !important; stroke: #e9ecef !important; stroke-width: 2 !important;" />

  <!-- Source Buffer (Real 512) -->
  <text x="40" y="35" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #495057 !important; font-weight: bold !important;">실수 입력 버퍼 (x[n], Length = 512)</text>
  
  <rect x="40" y="45" width="60" height="40" rx="3" style="fill: #e7f5ff !important; stroke: #228be6 !important; stroke-width: 1.5 !important;" />
  <text x="70" y="65" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #1c7ed6 !important; font-weight: bold !important; text-anchor: middle !important;">x[0]</text>
  <text x="70" y="80" style="font-family: Segoe UI, Arial !important; font-size: 9 !important; fill: #1c7ed6 !important; text-anchor: middle !important;">Even</text>

  <rect x="105" y="45" width="60" height="40" rx="3" style="fill: #fff4e6 !important; stroke: #fd7e14 !important; stroke-width: 1.5 !important;" />
  <text x="135" y="65" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #e8590c !important; font-weight: bold !important; text-anchor: middle !important;">x[1]</text>
  <text x="135" y="80" style="font-family: Segoe UI, Arial !important; font-size: 9 !important; fill: #e8590c !important; text-anchor: middle !important;">Odd</text>

  <rect x="170" y="45" width="60" height="40" rx="3" style="fill: #e7f5ff !important; stroke: #228be6 !important; stroke-width: 1.5 !important;" />
  <text x="200" y="65" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #1c7ed6 !important; font-weight: bold !important; text-anchor: middle !important;">x[2]</text>
  <text x="200" y="80" style="font-family: Segoe UI, Arial !important; font-size: 9 !important; fill: #1c7ed6 !important; text-anchor: middle !important;">Even</text>

  <rect x="235" y="45" width="60" height="40" rx="3" style="fill: #fff4e6 !important; stroke: #fd7e14 !important; stroke-width: 1.5 !important;" />
  <text x="265" y="65" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #e8590c !important; font-weight: bold !important; text-anchor: middle !important;">x[3]</text>
  <text x="265" y="80" style="font-family: Segoe UI, Arial !important; font-size: 9 !important; fill: #e8590c !important; text-anchor: middle !important;">Odd</text>

  <text x="320" y="70" style="font-family: Segoe UI, Arial !important; font-size: 18 !important; fill: #adb5bd !important;">...</text>

  <rect x="345" y="45" width="60" height="40" rx="3" style="fill: #e7f5ff !important; stroke: #228be6 !important; stroke-width: 1.5 !important;" />
  <text x="375" y="65" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #1c7ed6 !important; font-weight: bold !important; text-anchor: middle !important;">x[510]</text>
  <text x="375" y="80" style="font-family: Segoe UI, Arial !important; font-size: 9 !important; fill: #1c7ed6 !important; text-anchor: middle !important;">Even</text>

  <rect x="410" y="45" width="60" height="40" rx="3" style="fill: #fff4e6 !important; stroke: #fd7e14 !important; stroke-width: 1.5 !important;" />
  <text x="440" y="65" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #e8590c !important; font-weight: bold !important; text-anchor: middle !important;">x[511]</text>
  <text x="440" y="80" style="font-family: Segoe UI, Arial !important; font-size: 9 !important; fill: #e8590c !important; text-anchor: middle !important;">Odd</text>

  <!-- Packing Arrows -->
  <path d="M 70,90 Q 70,120 140,120" marker-end="url(#arrow-p)" style="fill: none !important; stroke: #228be6 !important; stroke-dasharray: 2 2 !important; stroke-width: 2 !important;" />
  <path d="M 135,90 Q 135,120 180,120" marker-end="url(#arrow-p)" style="fill: none !important; stroke: #fd7e14 !important; stroke-dasharray: 2 2 !important; stroke-width: 2 !important;" />

  <!-- Packed Target Buffer (Complex 256) -->
  <text x="40" y="145" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #495057 !important; font-weight: bold !important;">포장된 복소 버퍼 (y[n], Length = 256)</text>
  
  <rect x="40" y="155" width="220" height="45" rx="4" style="fill: #f1f3f5 !important; stroke: #868e96 !important; stroke-width: 2 !important;" />
  <text x="50" y="172" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #343a40 !important; font-weight: bold !important;">y[0]</text>
  
  <!-- Real part -->
  <rect x="90" y="160" width="70" height="35" rx="2" style="fill: #e7f5ff !important; stroke: #228be6 !important;" />
  <text x="125" y="181" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #1c7ed6 !important; font-weight: bold !important; text-anchor: middle !important;">.real = x[0]</text>
  
  <!-- Imag part -->
  <rect x="165" y="160" width="70" height="35" rx="2" style="fill: #fff4e6 !important; stroke: #fd7e14 !important;" />
  <text x="200" y="181" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #e8590c !important; font-weight: bold !important; text-anchor: middle !important;">.imag = x[1]</text>

  <!-- Array element 2 -->
  <rect x="280" y="155" width="220" height="45" rx="4" style="fill: #f1f3f5 !important; stroke: #868e96 !important; stroke-width: 2 !important;" />
  <text x="290" y="172" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #343a40 !important; font-weight: bold !important;">y[1]</text>
  
  <!-- Real part 2 -->
  <rect x="330" y="160" width="70" height="35" rx="2" style="fill: #e7f5ff !important; stroke: #228be6 !important;" />
  <text x="365" y="181" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #1c7ed6 !important; font-weight: bold !important; text-anchor: middle !important;">.real = x[2]</text>
  
  <!-- Imag part 2 -->
  <rect x="405" y="160" width="70" height="35" rx="2" style="fill: #fff4e6 !important; stroke: #fd7e14 !important;" />
  <text x="440" y="181" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #e8590c !important; font-weight: bold !important; text-anchor: middle !important;">.imag = x[3]</text>

  <text x="520" y="180" style="font-family: Segoe UI, Arial !important; font-size: 18 !important; fill: #adb5bd !important;">...</text>

  <!-- Arrow markers -->
  <defs>
    <marker id="arrow-p" viewBox="0 0 10 10" refX="5" refY="5" markerWidth="4" markerHeight="4" orient="auto-start-reverse">
      <path d="M 0 1.5 L 8 5 L 0 8.5 z" style="fill: #495057 !important;" />
    </marker>
  </defs>
</svg>

### A. 대수적 연립 방정식을 통한 복소 성분 분리 이론

256점 복소 푸리에 변환의 출력 결과 데이터인 $Y[k]$ ($k=0 \dots 255$)는 변환 구조 상 실수 입력 짝수 대역 성분과 홀수 대역 성분이 서로 대수적으로 중첩된 결합 상태를 가집니다.

1. **복소 FFT 출력 스펙트럼의 대수 식 정의:**
   $$Y[k] = X_e[k] + j \cdot X_o[k]$$
2. **켤레 성질을 이용한 대칭 반대 주파수 묶음 식:**
   실수 신호의 고유성 규칙에 따라 $Y$ 버퍼 내 대칭 위치($256-k$번째 인덱스) 데이터의 켤레 복소수($Y^*$)를 구하면 짝수부와 홀수부가 음의 복소 벡터로 전환됩니다.
   $$Y^*[256-k] = X_e[k] - j \cdot X_o[k]$$
3. **짝수 분량($X_e$) 및 홀수 분량($X_o$)의 대수 분리:**
   도출된 2개의 연립 방정식을 가감 결합하여 각 스펙트럼 구성 인자를 완벽하게 개별 획득합니다.
   * **짝수 주파수 스펙트럼:** $X_e[k] = \frac{Y[k] + Y^*[256-k]}{2}$
   * **홀수 주파수 스펙트럼:** $X_o[k] = \frac{Y[k] - Y^*[256-k]}{2j} = -j \frac{Y[k] - Y^*[256-k]}{2}$

### B. 복소 대수 공간 상에서 $j$ 연산자의 나눗셈과 비트 스와핑(Bit Swapping) 최적화
* **허수 나눗셈의 곱셈 치환:** 임의의 수식을 허수 단위 $j$로 제하는 연산은 다음과 같이 대수적으로 $-j$ 곱셉식과 완벽히 일치합니다.
  $$\frac{1}{j} = \frac{1 \cdot (-j)}{j \cdot (-j)} = \frac{-j}{-j^2} = \frac{-j}{-(-1)} = -j$$
* **$-j$ 연산자 곱셈에 따른 레지스터 내 비트 변위:** 임의의 복소수 $A + jB$에 대해 $-j$ 곱 연산을 취해 비트의 물리적 흐름을 추적합니다.
  $$(A + jB) \times (-j) = -jA - j^2 B = -jA - (-1)B = B - jA$$
  즉, 원래 실수부 성분이었던 $A$는 최종 연산물에서 **부호가 뒤집힌 허수부($-A$)**로 위치하고, 본래 허수부였던 $B$는 **실수부($B$)**로 위치가 교차 전이됩니다.
  이 대수적 거동 덕분에 임베디드 연산 코어는 수많은 복소곱 기계어 어셈블리를 처리하는 대신, 단순히 뺄셈 결괏값의 실수 레지스터 변수와 허수 레지스터 변수의 주소만을 뒤집어 대입하는 **레지스터 비트 스와핑(Swapping)**을 수행하여 $\frac{1}{2j}$ 연산을 1클럭 내에 고속 완료합니다.
  * 복원 실수부 대입: $X_{odd}.real = \frac{Y[k].imag - Y^*[256-k].imag}{2}$
  * 복원 허수부 대입: $X_{odd}.imag = -\frac{Y[k].real - Y^*[256-k].real}{2}$

4. **최종 512점 스펙트럼 $X[k]$의 합성:**
   분리된 짝수 및 홀수 개별 스펙트럼에 삼각함수 회전인자 Twiddle Factor ($W_{512}^k = e^{-j\frac{2\pi k}{512}}$)를 곱해 최종 복소 주파수 스펙트럼을 복원합니다.
   $$X[k] = X_e[k] + W_{512}^k \cdot X_o[k] \quad (0 \le k < 256)$$

---

## 3. C언어 기반의 고속 RDFT Unpacking 참조 구현

다음 C 프로그래밍 코드는 256점 복소 푸리에 변환 출력을 입력받아, 메모리 룩업 테이블(LUT) 형태의 회전인자 데이터 테이블 $W_{512}^k$를 곱하여 최종 257개 주파수 빈의 물리 복소 스펙트럼으로 역교정하는 실시간 언팩(Unpacking) 함수 구현체입니다.

```c
#include <stdio.h>
#include <stdint.h>
#include <math.h>

#define FFT_SIZE_COMPLEX 256 // 256점 복소 FFT 크기
#define RDFT_OUT_BINS 257     // 최종 실수 주파수 빈 개수 (512 / 2 + 1)
#define Q23_SCALE (1 << 23)

typedef struct {
    int32_t real;
    int32_t imag;
} Q23_Complex;

// Twiddle Factor (회전인자) LUT: W_512^k = cos(2*pi*k/512) - j*sin(2*pi*k/512)
// Q1.23 고정소수점 규격으로 메모리에 LUT 생성
static Q23_Complex twiddle_lut_512[FFT_SIZE_COMPLEX];

/**
 * @brief 시스템 초기 기동 시 고정소수점 회전인자 LUT 데이터를 사전 연산하여 테이블을 구축합니다.
 */
void init_twiddle_lut() {
    for (int k = 0; k < FFT_SIZE_COMPLEX; k++) {
        double angle = 2.0 * M_PI * k / 512.0;
        twiddle_lut_512[k].real = (int32_t)(cos(angle) * Q23_SCALE);
        twiddle_lut_512[k].imag = (int32_t)(-sin(angle) * Q23_SCALE); // -j * sin(angle)
    }
}

/**
 * @brief 64비트 정밀도 정수 연산 레지스터를 활용한 오버플로우 방지형 복소 곱셈을 수행합니다.
 * @note C 표준에서 부호 있는 타입(signed type)의 우측 시프트(>>) 연산은 구현 정의(Implementation-defined) 사항입니다.
 *       본 코드는 ARM Compiler, GCC, Clang 등 부호 있는 우측 시프트 시 부호 비트를 유지하는 산술 우측 시프트(ASR, Arithmetic Right Shift)를
 *       보장하는 DSP 및 임베디드 대상 컴파일러 환경을 기본 전제로 작성되었습니다.
 */
Q23_Complex q23_complex_mult(Q23_Complex a, Q23_Complex b) {
    Q23_Complex res;
    // 임시 64비트 레지스터 가산 곱을 유도하여 이중 곱 스케일(Q2.46) 수용
    int64_t re1 = (int64_t)a.real * b.real;
    int64_t re2 = (int64_t)a.imag * b.imag;
    int64_t im1 = (int64_t)a.real * b.imag;
    int64_t im2 = (int64_t)a.imag * b.real;

    // (ac - bd) 연산 후 23비트 산술 우측 시프트(ASR)하여 Q1.23으로 원복 (컴파일러가 ASR을 보장함)
    res.real = (int32_t)((re1 - re2) >> 23);
    // (ad + bc) 연산 후 23비트 산술 우측 시프트(ASR)하여 Q1.23으로 원복 (컴파일러가 ASR을 보장함)
    res.imag = (int32_t)((im1 + im2) >> 23);
    return res;
}

/**
 * @brief 256점 복소 FFT 출력 데이터를 복사하여 257개의 RDFT 주파수 스펙트럼 빈으로 언팩 복원합니다.
 * @param Y_in 256점 복소 푸리에 변환 완료 배열
 * @param X_out 최종적으로 복원 정렬할 257개 크기의 출력 배열
 * @note C 표준에서 부호 있는 타입의 우측 시프트(>>) 연산은 구현 정의(Implementation-defined) 사항입니다.
 *       본 코드는 부호 있는 우측 시프트 시 부호 비트를 보존하는 산술 우측 시프트(ASR)를 보장하는 임베디드 컴파일러 환경을 전제로 합니다.
 */
void rdft_unpack(const Q23_Complex* Y_in, Q23_Complex* X_out) {
    // 1. 거울 대칭축의 시작 기점 (k = 0, DC) 및 최종 경계선 (k = 256, Nyquist) 처리
    // 두 성분은 켤레 허수가 소멸하는 실수축 성분입니다.
    X_out[0].real = Y_in[0].real + Y_in[0].imag;
    X_out[0].imag = 0; 

    X_out[FFT_SIZE_COMPLEX].real = Y_in[0].real - Y_in[0].imag;
    X_out[FFT_SIZE_COMPLEX].imag = 0; 

    // 2. 주파수 스펙트럼 중간 영역 복원 루프 (k = 1 ~ 255)
    for (int k = 1; k < FFT_SIZE_COMPLEX; k++) {
        int mirror_idx = FFT_SIZE_COMPLEX - k; // 대칭 거울 인덱스 맵 산출

        Q23_Complex y_k = Y_in[k];
        Q23_Complex y_mirror_conj;
        y_mirror_conj.real = Y_in[mirror_idx].real;
        y_mirror_conj.imag = -Y_in[mirror_idx].imag; // Conjugate 대수 처리

        // 3. Xe 및 Xo 대수 성분 분리 처리 (컴파일러 ASR 보장 하에 나누기 2를 위한 1비트 우측 시프트 수행)
        Q23_Complex X_even;
        X_even.real = (y_k.real + y_mirror_conj.real) >> 1; 
        X_even.imag = (y_k.imag + y_mirror_conj.imag) >> 1;

        // Xo = -j * (y[k] - y*[256-k]) / 2
        int32_t diff_re = y_k.real - y_mirror_conj.real;
        int32_t diff_im = y_k.imag - y_mirror_conj.imag;
        
        // j의 대수 기하적 특성에 의거한 레지스터 스와핑 및 부호 반전 결합
        // (컴파일러 ASR 보장 하에 나누기 2를 위한 1비트 우측 시프트 수행)
        Q23_Complex X_odd;
        X_odd.real = diff_im >> 1;
        X_odd.imag = -diff_re >> 1;

        // 4. LUT Twiddle Factor 승산 및 위상 회전 가산
        Q23_Complex W = twiddle_lut_512[k];
        Q23_Complex rotated_odd = q23_complex_mult(X_odd, W);

        // 5. 나비연산 조립을 통한 주파수 스펙트럼 합성 복원
        X_out[k].real = X_even.real + rotated_odd.real;
        X_out[k].imag = X_even.imag + rotated_odd.imag;
    }
}

int main() {
    init_twiddle_lut();

    // FFT 연산 완료 시퀀스 데이터 가정 매핑
    Q23_Complex Y_fft[FFT_SIZE_COMPLEX];
    for (int i = 0; i < FFT_SIZE_COMPLEX; i++) {
        Y_fft[i].real = (int32_t)(0.12f * Q23_SCALE);
        Y_fft[i].imag = (int32_t)(-0.05f * Q23_SCALE);
    }

    Q23_Complex X_rdft[RDFT_OUT_BINS];
    rdft_unpack(Y_fft, X_rdft);

    printf("실수 입력 RDFT Unpacking 연산 검증:\n");
    printf("  DC 성분 (Bin 0)    : Re = %f, Im = %f\n", (float)X_rdft[0].real / Q23_SCALE, (float)X_rdft[0].imag / Q23_SCALE);
    printf("  임의 대역 (Bin 128): Re = %f, Im = %f\n", (float)X_rdft[128].real / Q23_SCALE, (float)X_rdft[128].imag / Q23_SCALE);
    printf("  나이퀴스트 (Bin 256): Re = %f, Im = %f\n", (float)X_rdft[256].real / Q23_SCALE, (float)X_rdft[256].imag / Q23_SCALE);

    return 0;
}
```

---

## 4. 주파수 스펙트럼 빈의 비선형 전극 채널 에너지 추출(Binning)

수학적 언팩을 거치며 균일 주파수 해상도를 지닌 257개의 개별 주파수 빈이 확보되지만, 인공와우 환자의 달팽이관 내벽에 외과적으로 수용 및 삽입되는 미세 기계 전극선의 개수는 물리적 접촉 한계로 인해 **16~22개 대역**에 불과합니다. 이에 따라 257개 빈 에너지를 22개 대역의 대표 스케일 에너지로 결합하는 **Binning** 연산이 필수적으로 요구됩니다.

### A. 달팽이관의 주파수 국소성(Tonotopic Organization)과 로그 스케일 필터뱅크 설계
* 인간 청각의 물리적 내벽인 달팽이관은 주파수를 감지할 때 고조파 영역으로 전진할수록 넓은 주파수 대역을 뭉뚱그려(둔감하게) 인지하는 비선형 로그 척도(Tonotopic 규칙)를 가집니다.
* 이를 모사하기 위해, 낮은 주파수를 자극하는 전극 대역(Channel 0)은 257개 주파수 빈 중 단 2~3개의 정밀 대역만을 묶어 예리하게 에너지를 관리하며, 높은 대역을 타겟하는 전극(Channel 21)은 20~30여 개의 광범위한 고역 스펙트럼 빈을 통째로 가산 통합하도록 로그 경계 테이블을 정의합니다.

### B. 22개 전극 채널 에너지 적분 및 로그 압축 C언어 구현

다음 코드는 복소 주파수 스펙트럼 에너지를 전달받아, 지정된 비선형 로그 전극 맵핑 테이블에 준하여 대역 에너지를 산출하고 데시벨 단위로 로그 수축하는 실시간 연산 코드입니다.

```c
#include <stdio.h>
#include <stdint.h>
#include <math.h>

#define RDFT_OUT_BINS 257
#define COCHLEAR_CHANNELS 22
#define Q23_SCALE (1 << 23)

typedef struct {
    int16_t start_bin;
    int16_t end_bin;
} ChannelBoundary;

// 각 채널이 담당할 주파수 빈(Bin)의 로그 매핑 경계 좌표 정의
static ChannelBoundary ch_bounds[COCHLEAR_CHANNELS] = {
    {2, 3},     // Ch 0 (저역 대역폭 좁은 구간)
    {4, 5},     // Ch 1
    {6, 7},     // Ch 2
    {8, 9},     // Ch 3
    {10, 12},   // Ch 4
    {13, 15},   // Ch 5
    {16, 18},   // Ch 6
    {19, 22},   // Ch 7
    {23, 26},   // Ch 8
    {27, 31},   // Ch 9
    {32, 37},   // Ch 10
    {38, 44},   // Ch 11
    {45, 52},   // Ch 12
    {53, 61},   // Ch 13
    {62, 72},   // Ch 14
    {73, 85},   // Ch 15
    {86, 100},  // Ch 16
    {101, 118}, // Ch 17
    {119, 139}, // Ch 18
    {140, 164}, // Ch 19
    {165, 194}, // Ch 20
    {195, 255}  // Ch 21 (고역 대역폭 매우 넓은 구간)
};

/**
 * @brief 257개의 스펙트럼 에너지를 22개 전극 채널 전력으로 변환하고 로그형 데시벨로 매핑 압축합니다.
 * @param X 입력 RDFT 복소수 데이터 배열
 * @param ch_energy_db 22개 채널 출력이 적재될 Q16.15 포맷 데시벨 배열
 */
void extract_electrode_energy(const Q23_Complex* X, int32_t* ch_energy_db) {
    for (int ch = 0; ch < COCHLEAR_CHANNELS; ch++) {
        int start = ch_bounds[ch].start_bin;
        int end = ch_bounds[ch].end_bin;

        // 1. 해당 전극 채널 스펙트럼 윈도우 내부의 제곱합 누적
        // 대칭 곱 가산(Re^2 + Im^2) 시 64비트 정수형 누산기를 사용하여 오버플로우 방지
        uint64_t band_power = 0;
        for (int k = start; k <= end; k++) {
            int64_t re_sq = (int64_t)X[k].real * X[k].real;
            int64_t im_sq = (int64_t)X[k].imag * X[k].imag;
            band_power += (uint64_t)(re_sq + im_sq); // Q2.46 포맷 누산 진행
        }

        // 2. 가중치 스케일 규격 복원
        double power_float = (double)band_power / ((uint64_t)1 << 46);
        
        // 로그 에러 방지를 위한 미세 보정치(Epsilon) 가산
        if (power_float < 1e-12) {
            power_float = 1e-12;
        }

        // 3. dB = 10 * log10(Power) 데시벨 변환 연산
        double db = 10.0 * log10(power_float);

        // 4. 자극 전류 매핑을 위한 Q16.15 고정소수점 스케일링 인코딩
        ch_energy_db[ch] = (int32_t)(db * 32768.0);
    }
}
```

---

## 5. 초저전력 임베디드 코어를 위한 CLZ(Count Leading Zeros) 고속 정수 로그 근사법

물리적 전력 한계로 인해 부동소수점 계산 장치(FPU) 및 표준 음향 로그 연산 함수(`log10`)의 작동이 차단된 저전력 임베디드 오디오 칩에서는 하드웨어 단의 단일 클럭 명령어인 **CLZ(Count Leading Zeros)** 비트 연산을 응용해 $O(1)$의 결정론적 속도로 로그 데시벨 값을 극도로 정밀하게 근사 추출합니다.

### A. 비트 수준에서의 정수 로그($\log_2$) 고속 근사 메커니즘
1. **정수 범위 판정:** 32비트 임의의 이산 신호 $x$의 최상위 활성 비트의 물리적 기하 지점은 $31 - \text{clz}(x)$ 공식과 정확히 수렴하며, 이는 대수적 $\log_2(x)$의 정수부 파트와 동치입니다.
2. **소수 영역의 선형 보간(Linear Interpolation):** 활성 최상위 비트(MSB) 하위단에 배열된 하위 비트 데이터 스트림을 강제로 소수점 이하 자리수로 표상합니다.
   $$\text{fractional} = \frac{x - 2^{\text{int\_part}}}{2^{\text{int\_part}}}$$
   이 대수적 처리는 부동소수점 산술 FPU 모듈 없이 단순 비트 마스킹과 논리 시프트(Bitwise Shift)만으로 극상의 소수 영역 로그 해상도를 근사적으로 확보합니다.

### B. 밑 변환 공식을 응용한 정수형 데시벨($\log_{10}$) 유도
로그 밑 변환 정리에 의거하여 $\log_{10}(x) = \log_2(x) \times \log_{10}(2)$ 가 유도됩니다.
* 이때 상수 비례 계수 $\log_{10}(2) \approx 0.30103$이 성립하며, 정수 연산 처리를 위해 Q15 고정소수점 포맷 계수로 양자화하면 $0.30103 \times 32768 \approx 9864$가 확보됩니다.
* 따라서 고속 연산으로 구한 정수 $\log_2$ 값에 **상수 $9864$를 곱해준 후 15비트 산술 우측 시프트** 처리하면, FPU 없이 단일 기계어 처리 사이클 내에 $\log_{10}$ 연산 결과를 환산 획득합니다.

### C. FPU 배제를 위한 순수 고정소수점 데시벨 변환 C언어 참조 구현

```c
#include <stdio.h>
#include <stdint.h>

/**
 * @brief 컴파일러 빌트인 CLZ 함수를 감싸 다양한 플랫폼 이식성을 보장합니다.
 */
static inline int32_t count_leading_zeros(uint32_t x) {
    if (x == 0) return 32;
    return __builtin_clz(x); // ARM 및 주요 DSP 코어에서 단일 클럭 명령어 매핑
}

/**
 * @brief 32비트 무부호 정수값의 log2를 정수 산술만으로 계산하여 Q16.15 포맷으로 변환합니다.
 * @return int32_t log2 계산 결과 (Q16.15 포맷)
 */
int32_t q15_log2(uint32_t x) {
    if (x == 0) return 0; // log(0) 안전 제어

    int32_t lz = count_leading_zeros(x);
    int32_t int_part = 31 - lz; // 정수 스케일 확정

    // 소수점 비트 추출 및 Q15 비트 필드로의 스위칭 보간
    int32_t frac_part;
    if (int_part >= 15) {
        frac_part = (x & ((1 << int_part) - 1)) >> (int_part - 15);
    } else {
        frac_part = (x & ((1 << int_part) - 1)) << (15 - int_part);
    }

    return (int_part << 15) + frac_part;
}

/**
 * @brief Q2.46 대역 에너지를 입력받아 정수 산술 clz 만으로 데시벨(dB) 수준을 Q16.15 포맷으로 고속 출력합니다.
 * @param power_q46 Q2.46 포맷의 복소 누적 전력치
 * @return int32_t 데시벨 계산치 (Q16.15 포맷)
 */
int32_t q15_power_to_db(uint64_t power_q46) {
    if (power_q46 == 0) {
        return -120 * 32768; // 음향 분석의 하한 한계인 -120 dB 출력 차단
    }

    int32_t lz = __builtin_clzll(power_q46);
    uint32_t norm_power;
    int32_t scale_shift;

    // 데이터 유효 범위를 32비트 정수 공간으로 정렬 정량화
    if (lz < 32) {
        norm_power = (uint32_t)(power_q46 >> (32 - lz));
        scale_shift = (32 - lz) - 46;
    } else {
        norm_power = (uint32_t)(power_q46 << (lz - 32));
        scale_shift = -(lz - 32) - 46;
    }

    // 1단계: log2(x) 계산 (Q16.15)
    int32_t log2_val = q15_log2(norm_power);
    
    // 2단계: 정규화 시프트 이동 분해량에 따른 정수 스케일 가산 보정
    log2_val += (scale_shift << 15);

    // 3단계: 밑 변환 데시벨 공식 통합곱: dB = 10 * log10(P) = 10 * log2(P) * log10(2)
    // 비례 스칼라 계수 10 * log10(2) = 3.0103. Q15 스케일 인코딩치 = 3.0103 * 32768 = 98641
    // 주의: 결과의 >> 15 시프트는 컴파일러가 부호 비트를 보존하는 산술 우측 시프트(ASR)를 보장함을 전제로 합니다.
    int64_t db_q15 = ((int64_t)log2_val * 98641) >> 15;

    return (int32_t)db_q15;
}
```

이와 같은 CLZ 비트 보간 기법을 응용하면, 고가의 실수 연산 프로세서 유닛 없이도 **인공와우 22개 채널의 순시 가청 볼륨 에너지를 단 20~30여 어셈블리 실행 사이클 내에 실시간 고속 연산 완료**할 수 있습니다.

---

➡️ **다음 단계:** [📖 DSP 마스터 커리큘럼 로드맵](dsp_curriculum.md)
