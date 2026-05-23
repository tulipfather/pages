# ⏳ Module 4. 실시간 FIFO 버퍼링과 지연(Latency) 극복 및 임펄스 응답

본 장에서는 매 $1\text{ ms}$(16샘플)마다 새로운 음성 데이터 샘플이 실시간으로 입력되는 인공와우(Cochlear Implant) 하드웨어 환경 하에서, 메모리 이동 오버헤드 없이 512샘플 크기의 FFT 분석 프레임을 실시간으로 유지 및 갱신하는 **원형 링버퍼(Circular Ring Buffer) 메커니즘**을 상술합니다. 아울러, 오디오 신호처리 과정 상에서 필연적으로 수반되는 집단 지연(Group Delay)을 상쇄하기 위한 **비대칭형 창 함수(Asymmetric Window)** 설계와 LTI 시스템의 고유한 동적 특성을 규명하는 **임펄스 응답(Impulse Response)**의 수리적 개념을 분석하고 C언어로 검증합니다.

---

## 1. 메모리 오버헤드 최소화를 위한 원형 버퍼(Ring Buffer)와 포인터 연산

실시간 이산 신호처리 시스템에서 매 프레임 전송 주기($1\text{ ms}$)마다 버퍼 내 기존 496개의 누적 샘플 데이터를 물리적으로 이동(Shift Copy)시키는 구조는 연산 자원이 제약된 초저전력 DSP 아키텍처 관점에서 불필요한 메모리 대역폭 소모와 전력 낭비를 초래합니다.

이를 극복하기 위해 물리적 주소가 고정된 버퍼 구조 상에 신규 표본만을 순환적으로 덮어쓰고, 프레임의 논리적 시작 지점을 가리키는 지시자만 갱신하는 **원형 링버퍼(Circular Ring Buffer)** 기법을 도입합니다.

### 🔄 512포인트 링버퍼의 포인터 동작 시각화

<svg viewBox="0 0 800 240" width="100%" height="240" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="800" height="240" rx="8" style="fill: #f8f9fa !important; stroke: #e9ecef !important; stroke-width: 2 !important;" />

  <!-- Circular Buffer Representation -->
  <circle cx="200" cy="120" r="80" style="fill: none !important; stroke: #6c757d !important; stroke-width: 8 !important;" />
  <circle cx="200" cy="120" r="70" style="fill: none !important; stroke: #e9ecef !important; stroke-width: 12 !important;" />

  <!-- Segments on Circle -->
  <!-- Write Pointer -->
  <line x1="200" y1="120" x2="250" y2="70" marker-end="url(#arrow-wp)" style="stroke: #d32f2f !important; stroke-width: 3 !important;" />
  <circle cx="250" cy="70" r="6" style="fill: #d32f2f !important;" />
  <text x="265" y="65" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #d32f2f !important; font-weight: bold !important;">Write Pointer (쓰기)</text>

  <!-- Oldest Read pointer concept -->
  <line x1="200" y1="120" x2="130" y2="150" style="stroke: #007bff !important; stroke-width: 2 !important;" />
  <text x="70" y="165" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #007bff !important; font-weight: bold !important;">Oldest Data (읽기 시작)</text>

  <!-- Memory Layout representation -->
  <rect x="420" y="70" width="340" height="40" rx="4" style="fill: #e1f5fe !important; stroke: #0288d1 !important; stroke-width: 2 !important;" />
  <!-- Buffer Cells -->
  <line x1="462.5" y1="70" x2="462.5" y2="110" style="stroke: #0288d1 !important; stroke-width: 1 !important;" />
  <line x1="505" y1="70" x2="505" y2="110" style="stroke: #0288d1 !important; stroke-width: 1 !important;" />
  <line x1="547.5" y1="70" x2="547.5" y2="110" style="stroke: #0288d1 !important; stroke-width: 1 !important;" />
  <line x1="680" y1="70" x2="680" y2="110" style="stroke: #0288d1 !important; stroke-width: 1 !important;" />
  <line x1="722.5" y1="70" x2="722.5" y2="110" style="stroke: #0288d1 !important; stroke-width: 1 !important;" />
  
  <text x="441.25" y="95" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #01579b !important; text-anchor: middle !important;">0</text>
  <text x="483.75" y="95" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #01579b !important; text-anchor: middle !important;">1</text>
  <text x="526.25" y="95" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #01579b !important; text-anchor: middle !important;">2</text>
  <text x="613.75" y="95" style="font-family: Segoe UI, Arial !important; font-size: 18 !important; fill: #0288d1 !important; text-anchor: middle !important;">. . .</text>
  <text x="701.25" y="95" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #01579b !important; text-anchor: middle !important;">510</text>
  <text x="743.75" y="95" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #01579b !important; text-anchor: middle !important;">511</text>
  
  <!-- Index Masking note -->
  <text x="590" y="140" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #333 !important; text-anchor: middle !important; font-weight: bold !important;">비트 연산 마스킹 최적화</text>
  <text x="590" y="160" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #666 !important; text-anchor: middle !important;">index = pointer &amp; 511; (나머지 % 연산 대체)</text>

  <!-- Marker definition -->
  <defs>
    <marker id="arrow-wp" viewBox="0 0 10 10" refX="5" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M 0 0 L 10 5 L 0 10 z" style="fill: #d32f2f !important;" />
    </marker>
  </defs>
</svg>

### ⚡ 비트 연산 마스킹을 통한 모듈로(Modulo) 산술 최적화
링버퍼의 전체 길이인 $N=512$는 2의 제곱수($2^9$)에 부합합니다. 따라서 나눗셈 연산 회로 및 분기문을 유도하여 상대적으로 실행 사이클이 긴 일반적인 모듈로 연산자(`% 512`)를 적용하는 대신, 논리곱(AND) 비트 명령어인 **`& 511` (16진수 `0x1FF` 마스킹)** 연산을 활용함으로써 단일 기계어 클럭 사이클 내에 경계를 순환 탈출하는 최적화를 완수합니다.
$$\text{pointer} \ \& \ (N - 1) \equiv \text{pointer} \ \% \ N \quad (\text{단, } N \text{은 2의 거듭제곱})$$

---

## 2. 실시간 원형 링버퍼의 C언어 참조 구현

아래 C 소스코드는 매 $1\text{ ms}$마다 획득되는 신규 16개 프레임 표본을 링버퍼에 기입하고, 푸리에 변환 엔진으로 이송하기 위해 가장 오래된 샘플부터 시간 선형 순서대로 512개의 분석 버퍼로 정렬하여 복원하는 참조 알고리즘입니다.

```c
#include <stdio.h>
#include <stdint.h>

#define FFT_SIZE 512
#define HOP_SIZE 16
#define INDEX_MASK (FFT_SIZE - 1)  // 511 (0x1FF)

typedef struct {
    int32_t buffer[FFT_SIZE];      // Q1.23 소수점 데이터를 저장하는 원형 배열
    uint32_t write_idx;            // 신규 표본이 기입될 주소를 가리키는 포인터
} CircularBuffer;

/**
 * @brief 원형 링버퍼 구조체의 메모리 영역을 초기화합니다.
 */
void init_buffer(CircularBuffer *cb) {
    for (int i = 0; i < FFT_SIZE; i++) {
        cb->buffer[i] = 0;
    }
    cb->write_idx = 0;
}

/**
 * @brief 신입 홉(Hop) 샘플 데이터를 링버퍼에 추가 기입합니다.
 * @param cb 원형 버퍼 지시 포인터
 * @param new_samples 홉 사이즈 단위의 입력 배열
 */
void write_samples(CircularBuffer *cb, const int32_t *new_samples) {
    for (int i = 0; i < HOP_SIZE; i++) {
        cb->buffer[cb->write_idx] = new_samples[i];
        // 비트연산 마스킹 최적화를 적용하여 포인터 순환
        cb->write_idx = (cb->write_idx + 1) & INDEX_MASK;
    }
}

/**
 * @brief 고속 푸리에 변환(FFT) 구동을 위한 512샘플 선형 시퀀스를 순차 복원하여 추출합니다.
 * @param cb 원형 버퍼 지시 포인터
 * @param fft_frame_out 추출된 512점 선형 출력 버퍼
 */
void read_fft_frame(const CircularBuffer *cb, int32_t *fft_frame_out) {
    // 링버퍼 구조상 가장 유효 기간이 오래된 데이터의 위치는 현재 기입 포인터 주소와 일치합니다.
    uint32_t read_idx = cb->write_idx; 
    
    for (int i = 0; i < FFT_SIZE; i++) {
        fft_frame_out[i] = cb->buffer[read_idx];
        read_idx = (read_idx + 1) & INDEX_MASK;
    }
}

int main() {
    CircularBuffer cb;
    init_buffer(&cb);

    // Q1.23 데이터 포맷으로 가정된 1ms 분량의 모사 입력 데이터 (16개 샘플)
    int32_t mock_samples[HOP_SIZE];
    for (int i = 0; i < HOP_SIZE; i++) {
        mock_samples[i] = i + 1; // [1, 2, ..., 16] 값 매핑
    }

    // 신입 프레임 기입
    write_samples(&cb, mock_samples);
    printf("16개 신규 표본 수신 및 원형 버퍼 기입 완료. 현재 write_idx 포인터: %u\n", cb.write_idx);

    // 선형 분석 버퍼 시퀀스 복원
    int32_t fft_input[FFT_SIZE];
    read_fft_frame(&cb, fft_input);

    printf("추출된 FFT 프레임 선형 정렬 결과 (초기 32샘플): \n");
    for (int i = 0; i < 32; i++) {
        printf("%d ", fft_input[i]);
    }
    printf("... \n");

    return 0;
}
```

---

## 3. 실시간 시스템 지연 억제: 대칭형 창 함수와 비대칭형 창 함수의 비교 분석

인공와우 이식 수술을 받은 환자가 시각 인지 피드백(말하는 화자의 입술 움직임)과 청각 신경 자극 신호 간의 비동기적 어색함이나 어지럼증을 겪지 않도록 하기 위해서는 시스템의 **전체 전달 지연(Total Latency)**을 극소화하여야 합니다.

### A. 대칭형 Hann 윈도우의 군지연(Group Delay) 특성
기본적인 대칭형 Hann 창 함수는 시간축 상에서 에너지 가중 분포의 정점(Peak)이 정확히 프레임의 중심축($N/2 = 256$ 샘플)에 위치합니다. 
수학적 정의에 의하여 대칭형 선형 위상 시스템의 고유한 군지연(Group Delay)은 시간 영역 상의 무게 중심과 일치합니다.
$$\text{Latency}_{\text{group}} = \frac{N}{2 \cdot f_s} = \frac{256}{16,000 \text{ Hz}} = 16 \text{ ms}$$
* 이 $16\text{ ms}$ 물리적 지연 시간에 아날로그-디지털 변환 지연, DSP 분석 연산 클럭 지연, 그리고 피부 투과 고주파 송수신단 전송 지연이 합산되면 총 지연 시간이 $20\text{ ms}$를 능가하게 됩니다. 이는 인간이 비동기 왜곡을 인지하기 시작하는 임계치를 초과하여 환자에게 인지 장애를 유발할 수 있습니다.

### B. 비대칭형 창 함수(Asymmetric Window)를 활용한 지연 단축 메커니즘
비대칭형 창 함수는 전주기 주파수 누설 억제 특성은 유지하되, 시간 영역 상에서의 에너지 정점의 위치를 가장 최근 유입된 표본 영역인 우측 경계면 방향으로 치우치도록 왜곡 설계한 창 함수입니다.

### 📈 대칭 윈도우 vs 비대칭 윈도우 시간축 가중치 비교

<svg viewBox="0 0 800 240" width="100%" height="240" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="800" height="240" rx="8" style="fill: #f8f9fa !important; stroke: #e9ecef !important; stroke-width: 2 !important;" />

  <!-- Axis -->
  <line x1="80" y1="180" x2="720" y2="180" style="stroke: #555 !important; stroke-width: 2 !important;" />
  <text x="80" y="200" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #555 !important; text-anchor: middle !important;">0 (오래된 데이터)</text>
  <text x="400" y="200" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #555 !important; text-anchor: middle !important;">256 (중앙)</text>
  <text x="720" y="200" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #555 !important; text-anchor: middle !important;">511 (최신 데이터)</text>

  <!-- Symmetric Window (Hann) -->
  <path d="M 80 180 C 240 40, 560 40, 720 180" style="fill: none !important; stroke: #007bff !important; stroke-width: 3 !important;" />
  <circle cx="400" cy="110" r="5" style="fill: #007bff !important;" />
  <text x="400" y="90" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #007bff !important; font-weight: bold !important; text-anchor: middle !important;">대칭 윈도우 피크 (지연 16ms)</text>

  <!-- Asymmetric Window -->
  <path d="M 80 180 C 350 180, 550 40, 720 180" style="fill: none !important; stroke: #28a745 !important; stroke-width: 3 !important;" />
  <circle cx="610" cy="110" r="5" style="fill: #28a745 !important;" />
  <text x="610" y="90" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #28a745 !important; font-weight: bold !important; text-anchor: middle !important;">비대칭 피크 (지연 4~8ms)</text>
</svg>

* **군지연 저감 효과:** 비대칭형 창 함수는 최근 $1\text{ ms} \sim 10\text{ ms}$ 이내로 유입된 오디오 샘플 신호에 더 높은 가중치를 주어 스펙트럼 분석을 단행하므로, 창 함수의 유효 중심 좌표가 우측(최신 샘플 방면)으로 대폭 전향하게 됩니다. 이 설계 최적화를 통해 사이드로브 누설 억제력을 희생하지 않으면서도 시스템 군지연을 **$4 \sim 8\text{ ms}$** 영역으로 축소하여 실시간 음성 자극 전달 피드백 루프를 보장합니다.

---

## 4. 선형 시불변(LTI) 시스템의 수학적 고유성: 임펄스 응답(Impulse Response)

임펄스 응답(Impulse Response)은 임의의 선형 시불변(LTI, Linear Time-Invariant) 시스템이나 디지털 필터 단에 가상으로 완벽한 과도 자극인 이산 임펄스 신호를 가했을 때 출력되는 시스템 시간 도메인의 고유 응답 신호입니다.

* **디렉 이산 임펄스 함수 ($\delta[n]$) 정의:** $n=0$ 시점에 크기 $1.0$을 보이고, 그 외의 모든 시간 지점에서는 $0.0$의 스칼라를 갖는 이상적인 디지털 물리 자극입니다.
  $$\delta[n] = \begin{cases} 1, & n = 0 \\ 0, & n \ne 0 \end{cases}$$
* **임펄스 응답 ($h[n]$)의 성질:** LPF 또는 BPF 필터 회로에 입력 $x[n] = \text{δ}[n]$을 가하여 출력되는 출력 파형 $y[n] = h[n]$을 구함으로써 얻어집니다.
* **수리적 유용성:** 디렉 이산 임펄스는 주파수 도메인 전 대역($0\text{ Hz} \sim 8\text{ kHz}$)에 대해 동일한 진폭 강도($1.0$)로 전 주파수를 고르게 동시 자극하는 특성을 갖습니다. 따라서 구동된 시간축 상의 임펄스 출력 $h[n]$을 푸리에 변환하면, 해당 LTI 시스템이 내포한 고유한 **주파수 전달 함수(Frequency Response)**를 수리적 왜곡 없이 통째로 규명할 수 있습니다.

### 💻 C언어를 이용한 디지털 필터의 임펄스 응답 계측 시뮬레이션

```c
#include <stdio.h>
#include <stdint.h>

#define Q23_SCALE (1 << 23)
#define FLOAT_TO_Q23(f) ((int32_t)((f) * Q23_SCALE))
#define Q23_TO_FLOAT(q) ((float)(q) / Q23_SCALE)

// --- 1. 부동소수점(Float) 기반 참조 구현 ---

/**
 * @brief 1차 저역통과 무한 임펄스 응답(IIR) 필터를 거동합니다.
 * @details 차분 방정식: y[n] = 0.8 * y[n-1] + 0.2 * x[n]
 * @param x 신규 입력 샘플 x[n]
 * @param prev_y 직전 상태 출력 값 y[n-1]
 * @return float 현재 필터 연산 출력 값 y[n]
 */
float lowpass_filter_float(float x, float *prev_y) {
    float y = 0.8f * (*prev_y) + 0.2f * x;
    *prev_y = y;
    return y;
}

// --- 2. Q1.23 고정소수점(Fixed-Point) 기반 포팅 구현 ---

/**
 * @brief Q1.23 고정소수점 곱셈 연산 (반올림 포함)
 */
int32_t q23_multiply(int32_t a, int32_t b) {
    int64_t temp = (int64_t)a * (int64_t)b;
    temp += 0x400000; // 반올림 보정
    return (int32_t)(temp >> 23);
}

/**
 * @brief 1차 저역통과 IIR 필터의 고정소수점(Q1.23) 구현체
 * @details 차분 방정식: y[n] = 0.8 * y[n-1] + 0.2 * x[n]
 *          0.8의 Q1.23 표현 = 6,710,886 (0x666666)
 *          0.2의 Q1.23 표현 = 1,677,722 (0x19999A)
 * @param x_q 신입 고정소수점 입력 x[n]
 * @param prev_y_q 직전 고정소수점 출력 y[n-1] 저장 포인터
 */
int32_t lowpass_filter_q23(int32_t x_q, int32_t *prev_y_q) {
    int32_t term1 = q23_multiply(6710886, *prev_y_q); // 0.8 * y[n-1]
    int32_t term2 = q23_multiply(1677722, x_q);       // 0.2 * x[n]
    int32_t y_q = term1 + term2;                      // 덧셈 연산
    *prev_y_q = y_q;
    return y_q;
}

int main() {
    // --- 1. 부동소수점 임펄스 응답 계측 ---
    float prev_y_f = 0.0f;
    float impulse_output_f[16];

    for (int n = 0; n < 16; n++) {
        float x = (n == 0) ? 1.0f : 0.0f; // 이산 Dirac Delta 입력
        impulse_output_f[n] = lowpass_filter_float(x, &prev_y_f);
    }

    // --- 2. 고정소수점(Q1.23) 임펄스 응답 계측 ---
    int32_t prev_y_q = 0;
    int32_t impulse_output_q[16];

    for (int n = 0; n < 16; n++) {
        int32_t x_q = (n == 0) ? FLOAT_TO_Q23(1.0f) : 0; // Q1.23 이산 임펄스 입력
        impulse_output_q[n] = lowpass_filter_q23(x_q, &prev_y_q);
    }

    // --- 3. 두 구현체의 결과 비교 출력 ---
    printf("1차 IIR 저역통과 필터의 임펄스 응답 h[n] 시뮬레이션 비교:\n");
    printf("%-5s | %-12s | %-12s (Hex) | %-12s\n", "Index", "Float", "Q1.23 (Raw)", "Q1.23 (Float)");
    printf("-------------------------------------------------------------\n");
    for (int n = 0; n < 16; n++) {
        float q_to_f = Q23_TO_FLOAT(impulse_output_q[n]);
        printf("h[%2d] | %-12.6f | 0x%08X   | %-12.6f\n", 
               n, impulse_output_f[n], impulse_output_q[n], q_to_f);
    }

    return 0;
}
```
* **결과 해설:** 
  - 부동소수점과 고정소수점(Q1.23) 필터 모두에서 단 1회 인가된 에너지 $1.0$의 임펄스가 피드백 적분 루프를 회전하면서 지수적으로 하강하는 지수형 감쇄 파형($0.2000 \to 0.1600 \to 0.1280 \to \dots$)을 균일하게 출력합니다.
  - 두 구현체의 소수점 미세 자리 편차는 $10^{-7}$ 이하 수준으로 제어되며, 이는 Q1.23 고정소수점 산술이 24비트 저전력 임베디드 오디오 환경에서 매우 정밀하게 소리를 가공할 수 있음을 실험적으로 증명합니다. 이 과도 출력 상태 $h[n]$이 해당 LTI 시스템의 동적 거동 특성을 대표하는 수학적 고유 인자(임펄스 응답)가 됩니다.

---

➡️ **다음 단계:** [Module 5. 과도기 윈도우 & FFT](05_fft_windowing.md)
