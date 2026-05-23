# 📡 Module 3. 나이퀴스트 이론과 주파수 폴딩 (Nyquist & Folding)

본 장에서는 표본화 주파수 **$16\text{ kHz}$**를 기준 규격으로 채택한 인공와우(Cochlear Implant) 시스템 환경 하에서, 아날로그 입력 신호를 연속된 시간 정보에서 이산 데이터로 원복할 때의 수학적 한계인 **나이퀴스트 표본화 이론(Nyquist Sampling Theorem)**과 이 조건을 미충족할 때 일어나는 **주파수 폴딩(Frequency Folding)** 왜곡 현상을 학술적으로 규명합니다.

---

## 1. 나이퀴스트 표본화 이론의 수학적 고찰

연속적인 아날로그 신호를 디지털 연산 프로세서가 인식하고 처리하도록 하기 위해서는 일정한 시간 간격으로 순시 진폭 값을 추출하는 표본화(Sampling)가 수반되어야 합니다.

* **표본화의 최소 요건:** 기저 대역 아날로그 파형의 주파수 및 진폭 정보가 손실 없이 복원되기 위해서는, 최소한 임의 주파수의 정점(Peak)과 기곡점(Valley) 위치에서 각 1회 이상의 표본점 획득이 물리적으로 달성되어야 합니다.
* **수리적 모델:** 
  $$f_s > 2 \cdot f_{max} \quad (\text{표본화 주파수 } f_s\text{는 입력 신호의 차단 주파수 } f_{max}\text{의 2배를 초과})$$
* **나이퀴스트 주파수 (Nyquist Frequency, $f_N$):** 
  디지털 시스템 상에서 대역 제한을 가하지 않고 앨리어싱(Aliasing) 왜곡 없이 분석 및 표상할 수 있는 수학적·물리적 상한 한계 주파수입니다.
  $$f_N = \frac{f_s}{2}$$
  인공와우 타겟 사양에서 표본화율 $f_s = 16\text{ kHz}$이므로, 시스템의 나이퀴스트 주파수는 정확히 **$8\text{ kHz}$**로 고정됩니다. 즉, $8\text{ kHz}$를 초과하는 대역 외부 고주파 성분이 안티-앨리어싱(Anti-aliasing) 필터의 감쇄 없이 아날로그-디지털 변환 장치(ADC)로 유입되면, 디지털 복원 데이터 상에 원치 않는 왜곡이 투영됩니다.

### 📈 샘플링 주파수와 복원 가능 여부 시각화

<svg viewBox="0 0 800 240" width="100%" height="240" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="800" height="240" rx="8" style="fill: #f8f9fa !important; stroke: #e9ecef !important; stroke-width: 2 !important;" />

  <!-- Grid lines -->
  <line x1="50" y1="120" x2="750" y2="120" style="stroke: #ccc !important; stroke-width: 1 !important;" />
  
  <!-- Left Side: Sufficiently Sampled (fs = 4 * f) -->
  <path d="M 50 120 Q 93.75 30, 137.5 120 T 225 120 T 312.5 120 T 400 120" style="fill: none !important; stroke: #28a745 !important; stroke-width: 3 !important;" />
  <!-- Sample Dots -->
  <circle cx="50" cy="120" r="5" style="fill: #d32f2f !important;" />
  <circle cx="93.75" cy="30" r="5" style="fill: #d32f2f !important;" />
  <circle cx="137.5" cy="120" r="5" style="fill: #d32f2f !important;" />
  <circle cx="181.25" cy="210" r="5" style="fill: #d32f2f !important;" />
  <circle cx="225" cy="120" r="5" style="fill: #d32f2f !important;" />
  <circle cx="268.75" cy="30" r="5" style="fill: #d32f2f !important;" />
  <circle cx="312.5" cy="120" r="5" style="fill: #d32f2f !important;" />
  <circle cx="356.25" cy="210" r="5" style="fill: #d32f2f !important;" />
  <circle cx="400" cy="120" r="5" style="fill: #d32f2f !important;" />
  <text x="225" y="230" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #28a745 !important; font-weight: bold !important; text-anchor: middle !important;">정상 샘플링 (충분히 많은 점 획득)</text>

  <!-- Divider -->
  <line x1="420" y1="20" x2="420" y2="220" style="stroke: #6c757d !important; stroke-width: 2 !important; stroke-dasharray: 4 !important;" />

  <!-- Right Side: Under Sampled / Aliasing -->
  <path d="M 440 120 Q 477.5 30, 515 120 T 590 120 T 665 120 T 740 120" style="fill: none !important; stroke: #dc3545 !important; stroke-width: 1 !important; stroke-dasharray: 3 !important;" />
  <!-- Reconstructed Fake wave (Lower frequency) -->
  <path d="M 440 120 Q 590 30, 740 120" style="fill: none !important; stroke: #fd7e14 !important; stroke-width: 3 !important;" />
  <!-- Sparse Sample Dots -->
  <circle cx="440" cy="120" r="5" style="fill: #d32f2f !important;" />
  <circle cx="515" cy="120" r="5" style="fill: #d32f2f !important;" />
  <circle cx="590" cy="30" r="5" style="fill: #d32f2f !important;" />
  <circle cx="665" cy="120" r="5" style="fill: #d32f2f !important;" />
  <circle cx="740" cy="120" r="5" style="fill: #d32f2f !important;" />
  <text x="590" y="230" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #fd7e14 !important; font-weight: bold !important; text-anchor: middle !important;">샘플링 부족 (왜곡된 저주파 주황색 선으로 복원됨)</text>
</svg>

---

## 2. 주파수 폴딩(Frequency Folding) 현상 및 앨리어싱

나이퀴스트 주파수 $f_N = 8\text{ kHz}$ 한계를 초과하여 유입된 고역 신호 성분이 이산 시간 축 상에서 표현될 때 발생하는 스펙트럼 도메인의 대칭성 왜곡 현상을 **주파수 폴딩(Frequency Folding)**이라고 칭합니다.

* **접힘 및 앨리어싱 메커니즘:** 복구 불가능한 고주파 성분이 샘플링 경계 주파수($8\text{ kHz}$)를 거울 반사선(Reflection Line)으로 삼아 데칼코마니 형태로 뒤집히며, 가청 대역(주파수 스펙트럼의 내부 대역)으로 하강하여 고주파와 연관이 없는 다른 주파수로 변환 투영되는 성질을 갖습니다.
* **수리적 모델:**
  $$f_{folded} = | f_{in} - N \cdot f_s | \quad (\text{단, } N \text{은 } f_{folded} \le \frac{f_s}{2}\text{를 만족하는 임의의 정수})$$

### 📉 12 kHz 신호가 4 kHz로 꺾여 들어오는 폴딩 현상 시각화

<svg viewBox="0 0 800 220" width="100%" height="220" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="800" height="220" rx="8" style="fill: #f8f9fa !important; stroke: #e9ecef !important; stroke-width: 2 !important;" />

  <!-- Coordinates Axis -->
  <line x1="50" y1="160" x2="750" y2="160" style="stroke: #000 !important; stroke-width: 2 !important;" />
  <line x1="50" y1="160" x2="50" y2="30" style="stroke: #000 !important; stroke-width: 2 !important;" />

  <!-- Frequency Points Labels -->
  <text x="50" y="180" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #333 !important; text-anchor: middle !important;">0 Hz</text>
  <text x="225" y="180" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #333 !important; text-anchor: middle !important;">4 kHz (관측 지점)</text>
  <circle cx="225" cy="160" r="4" style="fill: #007bff !important;" />
  
  <!-- Folding Pivot Line (Nyquist - 8 kHz) -->
  <line x1="400" y1="160" x2="400" y2="40" style="stroke: #dc3545 !important; stroke-width: 2 !important; stroke-dasharray: 4 !important;" />
  <text x="400" y="180" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #dc3545 !important; font-weight: bold !important; text-anchor: middle !important;">8 kHz (Nyquist 한계선)</text>
  
  <text x="575" y="180" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #777 !important; text-anchor: middle !important;">12 kHz (원래 입력)</text>
  <circle cx="575" cy="160" r="4" style="fill: #6c757d !important;" />

  <!-- Arrow indicating the Folding reflection -->
  <!-- 12 kHz to 8 kHz -->
  <path d="M 575 140 Q 487.5 70, 400 120" marker-end="url(#arrow-f)" style="fill: none !important; stroke: #fd7e14 !important; stroke-width: 2 !important; stroke-dasharray: 3 !important;" />
  <!-- 8 kHz to 4 kHz -->
  <path d="M 400 120 Q 312.5 170, 225 140" marker-end="url(#arrow-f)" style="fill: none !important; stroke: #fd7e14 !important; stroke-width: 3 !important;" />
  <text x="400" y="80" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #fd7e14 !important; font-weight: bold !important; text-anchor: middle !important;">경계면을 기준으로 접혀 반사됨</text>

  <!-- Arrow marker -->
  <defs>
    <marker id="arrow-f" viewBox="0 0 10 10" refX="5" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M 0 0 L 10 5 L 0 10 z" style="fill: #fd7e14 !important;" />
    </marker>
  </defs>
</svg>

### 🧮 실시간 인공와우 시스템에서의 물리적 영향 분석
만일 안티-앨리어싱 아날로그 필터나 CIC 보상 필터단의 차단 특성이 불충분하여 $12\text{ kHz}$의 전자기 유도성 고주파 스파이크성 잡음이 $16\text{ kHz}$ ADC 입력단으로 들어올 경우:
$$f_{folded} = | 12\text{ kHz} - 1 \cdot 16\text{ kHz} | = |-4\text{ kHz}| = 4\text{ kHz}$$
* **영향 검증:** 고주파 잡음 $12\text{ kHz}$는 시스템 내부에서 저주파 대역 대각인 **$4\text{ kHz}$** 주파수 노이즈로 둔갑하여 복원됩니다. 이는 환자 뇌에 심각한 청각적 왜곡 및 해상도 손실을 발생시키므로, 표본화 전 단계에서 아날로그 저역통과 필터(LPF) 또는 PDM 데시메이션 단계의 디지털 LPF 성능 평가는 설계상 매우 중요한 파라미터가 됩니다.

---

## 3. 0 dBFS 풀스케일 (dBFS - Decibels relative to Full Scale)

디지털 신호 처리 장치 내부에서 음향 강도를 정량화할 때, 물리적인 절대 음압(dBSPL) 대신 유효 비트스트림이 허용하는 최대 동적 폭을 기준으로 수치화하는 상대 측정 단위를 **dBFS**라 합니다.

* **최대 디지털 상한 ($0\text{ dBFS}$):** Q1.23 데이터 규격이 담을 수 있는 최댓값인 절댓값 $1.0$(레지스터 상의 `0x7FFFFF` / `0x800000` 마스킹 값)에 1대1 매핑되는 절대 전력 기준점입니다.
* **로그 표현식:** 디지털 음원의 순시 전력은 항상 $0\text{ dBFS}$ 이하에 머물기 때문에 대다수 소수 표현은 음수 값을 취합니다.
  $$\text{Volume (dBFS)} = 20\log_{10} \left( \frac{|x|}{x_{max}} \right)$$
  (예: 정상 발화 신호의 순시 최대치가 $x = 0.01$일 때, $20\log_{10}(0.01) = -40\text{ dBFS}$로 산정됩니다.)

### 💥 클리핑 (Clipping) 왜곡
여러 대역의 필터 결합이나 가산 합 연산의 누적으로 인해 최종 스칼라 결과가 $0\text{ dBFS}$($\pm1.0$) 스케일을 초과하게 되는 순간, 그 이상의 파형은 평탄하게 잘려 나가는 클리핑 왜곡을 초래합니다. 이는 주파수 도메인 상에 유해한 기수 고조파(Odd Harmonics) 노이즈를 다량 분출하여 가청 감도를 크게 해칩니다.

---

## 4. 하드웨어적 오버플로우 제어: 가드 비트 누산기

다단 합성곱 필터 연산 및 실시간 FFT 분석 시 가산 루프가 수없이 겹치면서 스칼라 합이 $1.0$(즉, $0\text{ dBFS}$)을 순간적으로 돌과하여 산술 오염이 일어날 가능성이 매우 큽니다.

이를 방지하기 위해 DSP 코어 아키텍처에 구현된 **Guard Bit Accumulator**는 데이터 폭을 일반 24비트 범주보다 추가로 설계한 확장형 헤드룸을 가집니다.

### 💾 56비트 누산기(Accumulator) 비트 구조 및 연산 마진

<svg viewBox="0 0 800 180" width="100%" height="180" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="800" height="180" rx="8" style="fill: #f8f9fa !important; stroke: #e9ecef !important; stroke-width: 2 !important;" />

  <!-- Guard Bits Group -->
  <rect x="40" y="60" width="160" height="50" rx="4" style="fill: #ffe0b2 !important; stroke: #f57c00 !important; stroke-width: 2 !important;" />
  <text x="120" y="85" style="font-family: Segoe UI, Arial !important; font-size: 13 !important; fill: #e65100 !important; font-weight: bold !important; text-anchor: middle !important;">Guard Bits (8-bit)</text>
  <text x="120" y="103" style="font-family: Segoe UI, Arial !important; font-size: 9 !important; fill: #f57c00 !important; text-anchor: middle !important;">Bit 48 ~ 55</text>

  <!-- Sign / Integer Bit -->
  <rect x="210" y="60" width="60" height="50" rx="4" style="fill: #ffe0b2 !important; stroke: #f57c00 !important; stroke-width: 2 !important;" />
  <text x="240" y="85" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #e65100 !important; font-weight: bold !important; text-anchor: middle !important;">Sign (1)</text>
  <text x="240" y="103" style="font-family: Segoe UI, Arial !important; font-size: 9 !important; fill: #f57c00 !important; text-anchor: middle !important;">Bit 47</text>

  <!-- Binary Point (소수점) -->
  <circle cx="277" cy="95" r="4" style="fill: #d32f2f !important;" />

  <!-- Fractional Part Group -->
  <rect x="285" y="60" width="470" height="50" rx="4" style="fill: #c8e6c9 !important; stroke: #388e3c !important; stroke-width: 2 !important;" />
  <text x="520" y="85" style="font-family: Segoe UI, Arial !important; font-size: 13 !important; fill: #1b5e20 !important; font-weight: bold !important; text-anchor: middle !important;">Fractional Part (47-bit)</text>
  <text x="520" y="103" style="font-family: Segoe UI, Arial !important; font-size: 9 !important; fill: #388e3c !important; text-anchor: middle !important;">Bit 0 ~ 46</text>

  <!-- Explanatory Text -->
  <text x="120" y="145" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #e65100 !important; font-weight: bold !important; text-anchor: middle !important;">최대 256회 덧셈 누적 마진</text>
  <text x="520" y="145" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #1b5e20 !important; font-weight: bold !important; text-anchor: middle !important;">Q1.23 연산 결과가 확장되어 안전하게 저장되는 구간</text>
</svg>

* **구조적 안전성 및 Q9.47 비트 매핑:** 
  1. 두 개의 Q1.23 데이터를 정수 곱셈기에서 승산하면 중간 결과물로 Q2.46(정수부 2비트, 소수부 46비트) 포맷의 raw 데이터가 생성됩니다.
  2. 이때 부호 비트가 중복되는 현상이 발생하는데, DSP 하드웨어 데이터 경로(Datapath)는 이를 제거하기 위해 곱셈기 출력을 **좌측으로 1비트 논리 시프트(Left Shift)**하여 소수부를 47비트로 확장한 **Q1.47 포맷**으로 자동 정렬합니다.
  3. 이 Q1.47 데이터 상위에 누계 오버플로우 방지용 **8비트 가드 비트(Guard Bits)** 필드가 결합하여, 최종 누산기 레지스터는 **정수 및 가드 영역 9비트, 소수부 47비트**로 매핑되는 **Q9.47** 물리 레이아웃을 형성합니다.
* **여유 허용 폭:** 상단에 설계된 **8비트 가드 비트(Guard Bits)** 필드는 누적 연산 결과가 정수 경계를 상회하여 증가하더라도 최대 **$2^8 = 256$회**의 가산 연산 동안 부호 침해를 방지하여 누적 캐리를 안전하게 방어합니다.
* **출력 포화 처리:** 전체 탭의 누산 루프가 종료된 다음, 56비트 임시 결괏값을 다시 24비트 Q1.23 포맷으로 절삭 축소할 때, 스칼라 크기가 $1.0$ 한계를 여전히 넘어선 경우에 한하여 강제로 최댓값인 `0x7FFFFF`로 변환 제어해 주는 **포화 연산(Saturation)**이 동작하여 정수 무결성을 확보합니다.

---

➡️ **다음 단계:** [Module 4. 실시간 FIFO 버퍼링과 지연(Latency) 극복 및 임펄스 응답](04_fifo_latency_impulse.md)
