# 🌀 Module 6. 복소수와 신호 분석

본 장에서는 기저대역의 실수(Real) 오디오 신호 분석 시 수학적으로 허수(Imaginary Number)를 도입하여 복소 영역에서 신호를 표상해야 하는 물리적 당위성을 규명합니다. 또한, 복소 평면 상의 회전 벡터로서 실수부와 허수부가 가지는 기하학적 정의를 시각화하고, 고정소수점(Q1.23) 환경 하에서 삼각함수 연산장치(FPU) 없이 복소 크기(Magnitude)와 위상(Phase)을 고속 계산하는 C언어 참조 구현을 상세히 고찰합니다.

---

## 1. 실수 도메인 오디오 신호 분석에서의 복소수 도입의 당위성

마이크로폰 음압 변환 센서로부터 획득하는 순시 전압 데이터나 전기 음향적인 스피커 출력단으로 송출하는 아날로그 오디오 파형은 모두 물리 세계의 고유 전압 차원인 실수(Real Number)로 규정됩니다. 이에 따라 실수 성분만을 기준으로 주파수를 분석할 때 유도되는 치명적 한계를 검증하고 복소 신호처리의 필요성을 밝힙니다.

### A. 실수 성분 단독 푸리에 상관 분석의 이론적 한계

주파수 분해 및 분산 분석은 기본적으로 입력 신호 벡터와 특정 주파수를 지니는 정현파(사인 및 코사인) 기준 신호 간의 상호 상관관계(Cross-correlation) 연산입니다. 만약 복소 평면을 도입하지 않고 실수 신호인 코사인(Cosine) 파형 하나만을 기저 함수로 삼아 주파수 상관도를 검출할 경우 다음과 같은 왜곡이 파생됩니다.

1. **위상이 완벽하게 동기화된 경우:** 입력 신호와 기저 코사인 함수의 시작 주기가 정확하게 동치일 때, 상관 적분 계수는 최대 진폭 수준($1.0$)에 부합하게 산출됩니다.
2. **$90^\circ$의 위상 지연이 발생한 경우:** 입력 신호가 기준 코사인 신호에 비해 주기 시간의 4분의 1만큼 지연(위상 편차 $90^\circ$)되어 유입되는 시점에서는 상관 곱이 완전히 무력화됩니다. 삼각함수의 직교성(Orthogonality) 원리에 의해 코사인 함수와 사인 함수의 한 주기 평균 누적 적분은 항상 정확히 $0$으로 수렴하기 때문입니다.
3. **정보의 소멸 왜곡:** 분석 대상 대역 내에 명확히 높은 신호 에너지가 주입되고 있음에도 불구하고, 단지 표본화 창과의 위상 시간차로 인하여 해당 진폭 강도가 $0$으로 연산되어 신호가 소멸하는 치명적인 물리적 왜곡이 생깁니다.

> [!IMPORTANT]
> **직교 위상 복소수(Complex Conjugate Pair)가 제공하는 대책**
> 시간축 상의 위상 지연 및 시작 주기에 관계없이 주파수 대역 내 고유 스펙트럼 에너지 세기를 항시 일정하게 추적하려면, 서로 $90^\circ$ 위상 각도를 유지하며 완벽한 직교성을 확보한 두 개의 기저 함수(코사인과 사인)를 곱해 결합해야 합니다.
> 이때 가산 합성곱 연산 도중 두 직교 변조 성분이 혼합되지 않도록 독립된 기하학적 평면에 격리하여 대수적 연산을 가능케 하는 프레임워크가 바로 복소수 평면입니다.

---

## 2. 2차원 복소 평면과 회전 벡터 라벨 기호 $j$의 물리적 기하학

허수 단위 $j = \sqrt{-1}$를 실수 축 상에서 정의 불가능한 허상의 차원으로 파악하기보다는, 대수 공간 상에서 **"가로 실수축(Re)과 물리적으로 직교하는 세로 독립축(Im)에 위치한 성분임을 명시하는 2차원 좌표 벡터의 방향 지시자(Label)"**로 고찰하는 것이 타당합니다.

### A. 1차원 이산 신호와 2차원 원형 운동의 기하학적 매핑

1. **1차원 선형 센서 투영의 정보 유실:**
   * 회전 운동하는 물체의 움직임을 1차원 시간 단면축에만 투영하면 날개 끝단은 가로선 좌표선 상에서 왕복 단순 조화 진동 운동($X = \cos\theta$)을 하는 것으로만 관측됩니다.
   * 이 제한된 정보 하에서는 물체가 **시계 방향(양의 회전)으로 돌고 있는지, 반시계 방향(음의 회전)으로 돌고 있는지** 방향 판별이 불가능하며, 속도가 $0$이 되는 양방향 정점 위치에서 순간 가속 정보를 분실하게 됩니다.
2. **2차원 직교 성분의 결합 유도:**
   * 회전체의 움직임을 위와 옆 양방향에서 동시에 투영하여 각각의 직교 정보를 독립 획득합니다($X = \cos\theta, \ Y = \sin\theta$).
   * 이제 물체의 상태 점을 2차원 좌표 벡터 $(X, Y)$로 합성하여 처리하면 회전 모멘텀의 방향 및 순시 회전각을 완벽히 식별할 수 있습니다.
3. **복소 라벨 $X + jY$의 정의:**
   * 대수적 수식 전개 과정에서 $(X, Y)$ 순서쌍 표기 대신 대수적 결합을 위해 세로축 성분 값 뒤에 독립축 라벨 $j$를 부착하여 하나의 통일된 수식으로 선언합니다.
   * $j$는 곱해지는 성분을 가로 실수축에서 **반시계 방향으로 $90^\circ$ 회전 이동**시키는 기하학적 회전 연산자로 기능합니다.

### 📊 1D 진동 신호의 2D 복소 회전 벡터 투영

<svg viewBox="0 0 800 320" width="100%" height="320" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="800" height="320" rx="8" style="fill: #f8f9fa !important; stroke: #e9ecef !important; stroke-width: 2px !important;" />

  <!-- Left: 2D Complex Plane -->
  <g transform="translate(180, 160)">
    <!-- Grid and Axes -->
    <circle cx="0" cy="0" r="100" style="fill: none !important; stroke: #dee2e6 !important; stroke-dasharray: 4 4 !important; stroke-width: 1.5px !important;" />
    <line x1="-130" y1="0" x2="130" y2="0" style="stroke: #adb5bd !important; stroke-width: 2px !important;" />
    <line x1="0" y1="-130" x2="0" y2="130" style="stroke: #adb5bd !important; stroke-width: 2px !important;" />
    
    <!-- Axis Labels -->
    <text x="140" y="5" style="fill: #495057 !important; font-family: Segoe UI, Arial !important; font-size: 12 !important; font-weight: bold !important;">실수축 (Re, Cos)</text>
    <text x="5" y="-135" style="fill: #495057 !important; font-family: Segoe UI, Arial !important; font-size: 12 !important; font-weight: bold !important;">허수축 (Im, j * Sin)</text>
    
    <!-- Rotating Vector (Theta = 45 degrees) -->
    <!-- 45 deg = 70.7, -70.7 (SVG Y-axis is inverted) -->
    <line x1="0" y1="0" x2="70.7" y2="-70.7" marker-end="url(#arrow)" style="stroke: #228be6 !important; stroke-width: 4px !important;" />
    <circle cx="70.7" cy="-70.7" r="6" style="fill: #1c7ed6 !important;" />
    
    <!-- Project Lines -->
    <line x1="70.7" y1="-70.7" x2="70.7" y2="0" style="stroke: #fa5252 !important; stroke-width: 2px !important; stroke-dasharray: 3 3 !important;" />
    <line x1="70.7" y1="-70.7" x2="0" y2="-70.7" style="stroke: #12b886 !important; stroke-width: 2px !important; stroke-dasharray: 3 3 !important;" />
    
    <!-- Projections values -->
    <rect x="25" y="5" width="45" height="18" rx="3" style="fill: #ffe3e3 !important;" />
    <text x="47" y="18" style="fill: #c92a2a !important; font-family: Segoe UI, Arial !important; font-size: 10 !important; font-weight: bold !important; text-anchor: middle !important;">Re = Cosθ</text>
    
    <rect x="-65" y="-45" width="55" height="18" rx="3" style="fill: #e6fcf5 !important;" />
    <text x="-37" y="-32" style="fill: #087f5b !important; font-family: Segoe UI, Arial !important; font-size: 10 !important; font-weight: bold !important; text-anchor: middle !important;">Im = Sinθ</text>
    
    <!-- Angle Theta -->
    <path d="M 30,0 A 30,30 0 0,0 21.2,-21.2" style="fill: none !important; stroke: #228be6 !important; stroke-width: 2px !important;" />
    <text x="35" y="-12" style="fill: #1971c2 !important; font-family: Segoe UI, Arial !important; font-size: 12 !important; font-weight: bold !important;">위상(θ)</text>
    
    <!-- Magnitude Line Label -->
    <text x="25" y="-45" style="fill: #1971c2 !important; font-family: Segoe UI, Arial !important; font-size: 12 !important; font-weight: bold !important;">크기 (Mag)</text>
  </g>

  <!-- Right: 1D Time Signals -->
  <g transform="translate(420, 40)">
    <!-- Time-domain Grid -->
    <line x1="0" y1="0" x2="0" y2="240" style="stroke: #adb5bd !important; stroke-width: 2px !important;" />
    <line x1="0" y1="120" x2="320" y2="120" style="stroke: #adb5bd !important; stroke-dasharray: 2 2 !important;" />
    <line x1="0" y1="60" x2="320" y2="60" style="stroke: #dee2e6 !important; stroke-dasharray: 4 4 !important;" />
    <line x1="0" y1="180" x2="320" y2="180" style="stroke: #dee2e6 !important; stroke-dasharray: 4 4 !important;" />
    
    <!-- Cosine wave (Red, Real) -->
    <path d="M 0,60 Q 40,60 80,120 T 160,180 T 240,120 T 320,60" style="fill: none !important; stroke: #fa5252 !important; stroke-width: 2px !important; stroke-opacity: 0.8 !important;" />
    
    <!-- Sine wave (Green, Imaginary) -->
    <path d="M 0,120 Q 40,60 80,120 T 160,180 T 240,120 T 320,120" style="fill: none !important; stroke: #12b886 !important; stroke-width: 2px !important; stroke-opacity: 0.8 !important;" />
    
    <!-- Axis Labels -->
    <text x="325" y="125" style="fill: #495057 !important; font-family: Segoe UI, Arial !important; font-size: 11 !important;">시간 (Time)</text>
    <text x="240" y="50" style="fill: #c92a2a !important; font-family: Segoe UI, Arial !important; font-size: 11 !important; font-weight: bold !important;">Re (실수 성분)</text>
    <text x="240" y="210" style="fill: #087f5b !important; font-family: Segoe UI, Arial !important; font-size: 11 !important; font-weight: bold !important;">Im (허수 성분)</text>
    
    <circle cx="40" cy="70.7" r="5" style="fill: #228be6 !important;" />
    <line x1="40" y1="0" x2="40" y2="240" style="stroke: #228be6 !important; stroke-width: 1.5px !important; stroke-dasharray: 2 2 !important;" />
    <text x="45" y="20" style="fill: #1971c2 !important; font-family: Segoe UI, Arial !important; font-size: 11 !important; font-weight: bold !important;">관측 시점</text>
  </g>

  <!-- Definition of Arrow Marker -->
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="6" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M 0 1.5 L 8 5 L 0 8.5 z" style="fill: #228be6 !important;" />
    </marker>
  </defs>
</svg>

---

## 3. 복소 평면 좌표에서의 크기(Magnitude) 및 위상(Phase)의 수학적 성질

복소 평면 상에 임의의 주파수 분석 좌표 정보가 결정되면, 이에 대한 해석학적 연산을 통해 핵심 물리 특성치를 추출합니다.

### 📐 1. 스펙트럼 크기 (Magnitude)
* **대수적 정의:** 복소 평면 상의 원점 $(0,0)$에서 데이터 포인트 좌표 $(Re, Im)$를 종점으로 하는 회전 위상 벡터의 **직선 거리(길이)**입니다.
* **수리적 모델 (피타고라스 정리):**
  $$\text{Magnitude } |X| = \sqrt{Re^2 + Im^2}$$
* **물리적 해석:** 
  * 위상각 $\theta$의 진행에 따라 복소값 $Re$와 $Im$은 연속 교차하는 삼각함수 형태로 계속 변합니다. 
  * 그러나 이 둘을 제곱하여 가산한 후 제곱근을 취한 위상 벡터의 궤적 반경은 $\theta$에 의존하지 않고 항상 **일정한 상수값**을 유지합니다.
  * 즉, 분석 창과의 진입 오프셋(위상)에 대해 불변하는 해당 대역의 **순수한 진폭 세기(볼륨 에너지)**를 대변하며, 주파수 스펙트럼 상에서 묘사하고자 하는 실질적인 소리의 에너지가 됩니다.

### 🧭 2. 위상각 (Phase)
* **대수적 정의:** 가로 실수축의 양의 방향을 기준선으로 하여 회전 위상 벡터가 벌어진 반시계 방향의 **편각($\theta$)**입니다.
* **수리적 모델:**
  $$\text{Phase } \theta = \text{atan2}(Im, Re)$$
* **물리적 해석:** 시간 도메인 상에서 입력 주기 신호가 기준 시간 축에 비해 임베디드 오프셋만큼 지연되었는지를 측정하는 **시간 축 편차 정보**를 제공합니다.

---

## 4. Q1.23 고정소수점 환경 하에서의 복소수 크기 연산과 C언어 최적화 구현

초저전력으로 가동되는 임베디드 오디오 DSP 코어는 유효 배터리 마진에 의해 전력 소모가 극심한 `float`/`double` 연산이나 고정 소수 정밀도가 누락된 기본 `math.h` 라이브러리의 `sqrt()` 연산을 수행하기 곤란합니다. 따라서 24비트 Q1.23 데이터 포맷의 범주를 보전하며 정수 산술 명령어만으로 구동되는 **고속 정수 제곱근(Fixed-Point Integer Square Root)** 알고리즘이 적용됩니다.

### A. 제곱합 연산 시의 오버플로우 임계와 가드 비트 확장
두 개의 Q1.23 성분 $Re$와 $Im$을 승산한 후 더하는 단순 제곱합 공식 $Re^2 + Im^2$은 심각한 오버플로우 오류를 내포합니다.
* 개별 데이터 $Re, Im$은 각각 절대 임계치 $1.0$ 이하이므로 곱셈 자체는 한계 대역 내에 위치하지만, 두 제곱 결과의 가산합은 최대 $Re^2 + Im^2 \approx 1.0 + 1.0 = 2.0$에 접근하므로 고정소수점 양의 한계치인 $0.99999988$을 현저하게 초과합니다.
* 이에 대안으로 임시 누산기로 **64비트 정수 공간(`int64_t`)**을 선언하고 스케일 비트를 상향 격리하여 제곱합을 확보한 뒤 제곱근 연산을 취합니다.

### B. Q1.23 산술 곱셈 시의 Q2.46 포맷 변환 및 제곱근에 의한 스케일 자동 복원
Q1.23 스케일을 가지는 복소 데이터를 64비트 버퍼에서 연산 시 비트 폭은 다음과 같은 수학적 거동을 보입니다.
1. **소수부의 누적 확장:** 소수부 23비트 크기 상호 간의 곱셈에 의해, 소수점 이하 자리 폭은 정확히 $23 + 23 = 46\text{비트}$로 확장됩니다.
2. **정수부 부호 확장 (Q2 포맷):** 두 곱셉 산출물의 가산합은 최대 임계 스케일 $2.0$을 보전하기 위해 부호 비트를 포함하여 상위 정수부 2비트를 점유하므로 **Q2.46 포맷**으로 전이됩니다. 이는 양/음의 정수 동작 영역 $[-2.0, +2.0)$ 범주를 포함하여 오버플로우를 원천 방지합니다.
3. **제곱근 연산에 의한 고속 스케일 원복:**
   Q2.46 규격의 정밀도를 갖춘 임시 값에 정수 제곱근(SQRT)을 취하면 다음과 같이 정리됩니다.
   $$\sqrt{x \cdot 2^{46}} = \sqrt{x} \cdot 2^{23}$$
   즉, 소수부 가중치 $2^{46}$가 제곱근 연산자를 거침으로써 논리 시프트 연산 추가 없이 정확히 기존 **Q1.23 스케일($2^{23}$)**로 자동 환원되는 지극히 우아한 대수적 정합성이 성립합니다.

---

### 💻 C언어 기반 Q1.23 복소수 크기(Magnitude) 연산 구현

아래 C언어 소스코드는 Q1.23 실수부/허수부를 입력받아 64비트 곱셈 누산 영역 내에서 안전하게 피가산 연산을 수행하고, 비트 이진 탐색 기법(Digit-by-digit 개평법)을 기초로 하는 정수 제곱근 알고리즘을 사용해 최종 복소 크기 스칼라를 환원하는 고신뢰성 코드입니다.

```c
#include <stdio.h>
#include <stdint.h>

#define Q23_SCALE (1 << 23)

typedef struct {
    int32_t real; // Q1.23 포맷 실수부
    int32_t imag; // Q1.23 포맷 허수부
} Q23_Complex;

/**
 * @brief 64비트 무부호 정수 대상 CLZ(Count Leading Zeros) 기반 고속 제곱근 연산 (Integer SQRT)
 * @details 기존의 `while (one > value)` 순차 시프트 루프는 미세 잡음이나 무음 구간(value가 매우 작은 경우)에서
 *          불필요하게 많은 조건 분기 및 시프트 클럭을 소비하는 단점이 있습니다.
 *          이를 개선하기 위해 하드웨어 가속 명령어인 CLZ를 호출하여 단일 클럭 사이클 내에($O(1)$) 루프 시작 지수 `one`을 초기화합니다.
 * 
 * 💡 [작동 원리]
 * 1. 하드웨어 명령어(또는 비트 조작 최적화 함수)를 활용하여 leading zero의 수 `lz`를 구합니다.
 * 2. 최상위 유효 비트(MSB) 위치인 `63 - lz`를 구한 후, 짝수 지수만 탐색하는 알고리즘 특성에 맞춰 `shift = (63 - lz) & ~1` 연산으로 비트 시작 위치를 정렬합니다.
 * 3. 룩업용 비트 `one`을 결정한 후, 1비트씩 논리합 조건에 맞춰 제곱근 몫 `res`를 갱신합니다.
 */
uint64_t int_sqrt64_optimized(uint64_t value) {
    if (value == 0) return 0;

    int lz;
#if defined(__GNUC__) || defined(__clang__)
    // GCC 및 Clang 계열 컴파일러의 64비트 CLZ 내장 함수
    lz = __builtin_clzll(value);
#elif defined(_MSC_VER)
    // MSVC 계열 64비트 비트 탐색 내장 함수
    unsigned long index;
    if (_BitScanReverse64(&index, value)) {
        lz = 63 - index;
    } else {
        lz = 64;
    }
#else
    // 컴파일러 독립적 휴대용(Portable) 소프트웨어 CLZ 알고리즘
    uint64_t temp = value;
    lz = 0;
    if (temp <= 0x00000000FFFFFFFFULL) { lz += 32; temp <<= 32; }
    if (temp <= 0x0000FFFFFFFFFFFFULL) { lz += 16; temp <<= 16; }
    if (temp <= 0x00FFFFFFFFFFFFFFULL) { lz += 8;  temp <<= 8;  }
    if (temp <= 0x0FFFFFFFFFFFFFFFULL) { lz += 4;  temp <<= 4;  }
    if (temp <= 0x3FFFFFFFFFFFFFFFULL) { lz += 2;  temp <<= 2;  }
    if (temp <= 0x7FFFFFFFFFFFFFFFULL) { lz += 1; }
#endif

    // 제곱근 탐색을 시작할 최대 짝수 멱수 비트 계산
    int shift = (63 - lz) & ~1;
    uint64_t one = (uint64_t)1 << shift;
    uint64_t res = 0;

    while (one != 0) {
        if (value >= res + one) {
            value -= res + one;
            res = (res >> 1) + one;
        } else {
            res >>= 1;
        }
        one >>= 2;
    }
    return res;
}

/**
 * @brief Q1.23 고정소수점 복소수의 Magnitude(크기)를 오버플로우 없이 안전하게 산출합니다.
 * @param c 입력 Q1.23 복소 구조체 데이터
 * @return int32_t Q1.23 포맷의 복소 크기 결과값
 */
int32_t q23_complex_magnitude(Q23_Complex c) {
    // 1. 64비트 부호 정수 공간으로 연산 레지스터 확장 후 승산 진행
    // Q1.23 x Q1.23 = Q2.46 포맷 변환 발생
    int64_t real_sq = (int64_t)c.real * c.real;
    int64_t imag_sq = (int64_t)c.imag * c.imag;

    // 2. 가산 합 도중 오버플로우 전이 방지를 위한 누산
    // 합산 상한은 임계 스케일 2.0 (Q2.46 포맷 유지)
    uint64_t sum_sq = (uint64_t)(real_sq + imag_sq);

    // 3. 고속 정수 제곱근 호출을 통한 크기 도출 (CLZ 최적화 버전)
    // Q2.46 스케일의 입력을 제곱근 처리하여 Q1.23 규격으로 자동 복구
    uint64_t magnitude_q23 = int_sqrt64_optimized(sum_sq);

    // 4. 레지스터 표현 한계 보정을 위한 포화 연산(Saturation) 처리
    // 가산 결과가 1.0을 상회할 수 있으므로 Q1.23 최대 상한인 0x7FFFFF로 상한 억제
    if (magnitude_q23 > 0x7FFFFF) {
        return 0x7FFFFF; 
    }

    return (int32_t)magnitude_q23;
}

int main() {
    // 예제 1: Re = 0.6, Im = 0.8 (기하학적 크기는 정확히 1.0으로 수렴해야 함)
    Q23_Complex val1;
    val1.real = (int32_t)(0.6f * Q23_SCALE);
    val1.imag = (int32_t)(0.8f * Q23_SCALE);

    int32_t mag1 = q23_complex_magnitude(val1);
    double mag1_float = (double)mag1 / Q23_SCALE;

    printf("결과 1 검증:\n");
    printf("  입력 성분: Re = %d (0.6f), Im = %d (0.8f)\n", val1.real, val1.imag);
    printf("  Q1.23 크기 출력값: 0x%06X (Dec: %d)\n", mag1 & 0xFFFFFF, mag1);
    printf("  실수 복원 크기: %f (목표치: 1.000000)\n\n", mag1_float);

    // 예제 2: Re = 0.3, Im = -0.4 (크기는 0.5로 수렴해야 함)
    Q23_Complex val2;
    val2.real = (int32_t)(0.3f * Q23_SCALE);
    val2.imag = (int32_t)(-0.4f * Q23_SCALE);

    int32_t mag2 = q23_complex_magnitude(val2);
    double mag2_float = (double)mag2 / Q23_SCALE;

    printf("결과 2 검증:\n");
    printf("  입력 성분: Re = %d (0.3f), Im = %d (-0.4f)\n", val2.real, val2.imag);
    printf("  Q1.23 크기 출력값: 0x%06X (Dec: %d)\n", mag2 & 0xFFFFFF, mag2);
    printf("  실수 복원 크기: %f (목표치: 0.500000)\n", mag2_float);

    return 0;
}
```

---

## 5. 인공와우(Cochlear Implant) 하드웨어단에서의 연산 당위성

1. **위상 독립적 볼륨 스케일링:** 음압 파형 유입 시, 입력 신호의 시작 타이밍(위상각) 편차 왜곡을 배제하고 오직 가청 주파수의 순수한 **볼륨 압축 에너지**만을 상시 획득합니다.
2. **신경 전극선 전류 자극 강도 결정:** 주파수별 Magnitude 연산 결과값은 최종 단의 비선형 볼륨 맵핑 함수를 통과하여, 달팽이관 내부에 부착된 22개 물리 신경 전극선 채널에 투여될 **실시간 전류 인가 파라미터**로 직접 조율 제어됩니다.

---

➡️ **다음 단계:** [Module 7. 실수 최적화 RDFT와 전극 채널 에너지 추출](07_rdft_energy.md)
