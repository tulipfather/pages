# 🎙️ Module 1. 디지털 마이크(DMIC) PDM 프로토콜과 데시메이션

본 장에서는 외부 아날로그 음향 신호가 디지털 MEMS(Micro-Electro-Mechanical Systems) 마이크로폰을 통해 1비트 PDM(Pulse Density Modulation) 고밀도 스트림으로 변환되는 물리적 메커니즘을 규명하고, 변환된 디지털 스트림이 하드웨어 파이프라인 상에서 데시메이션(Decimation) 과정을 거쳐 최종 24비트 Q1.23 PCM 데이터로 변환되는 전 과정을 학술적으로 고찰합니다.

---

## 1. 아날로그-디지털 변환 체계에서 PDM 채택의 당위성

오디오 시스템에서 널리 사용되는 PCM(Pulse Code Modulation) 데이터 포맷은 마이크로폰 센서 캡슐 수준에서 직접 디지털 신호로 출력하기에 상당한 하드웨어적 제약이 존재합니다.

* **전송 인터페이스 핀 수 및 전송선 잡음의 한계:** 24비트 PCM 신호를 병렬로 전송하려면 최소 24개의 전송 라인이 확보되어야 합니다. 이는 공간적 제약이 극심한 인공와우 기기 내에서 물리적으로 수용할 수 없습니다. I2S 또는 SPI와 같은 직렬 통신 구조를 적용하더라도, 마이크로폰 캡슐 내부에 고정밀 고정소수점 멀티비트 ADC(Analog-to-Digital Converter)와 내부 클럭 복구 회로가 통합되어야 하므로 소자의 물리적 부피가 증가하고 소모 전력이 크게 상승합니다.
* **PDM의 저전력 및 연결 간결성:** PDM(Pulse Density Modulation)은 클럭(CLK) 라인과 데이터(DATA) 라인이라는 단 **2개의 물리적 핀**만으로 양방향 혹은 단방향 데이터 전송을 지원합니다. 마이크로폰 내부에는 극저전력으로 동작하는 1비트 시그마-델타 변조기(Sigma-Delta Modulator)만을 집적하여 마이크로폰 모듈의 초소형화와 배터리 수명 극대화를 동시에 달성할 수 있습니다.
* **전자기 왜곡에 대한 복원력:** 1비트 디지털 펄스열은 감쇄나 위상 왜곡이 발생하더라도 판정 임계 전압(Threshold Voltage)을 기반으로 고수준/저수준(High/Low) 신호를 명확히 식별할 수 있습니다. 결과적으로 인체 이식 또는 귓바퀴 후면에 장착되는 인공와우 기기 내부의 아날로그 전도 잡음 및 전자기 간섭(EMI) 환경에서 매우 강인한 전기적 잡음 면역성을 제공합니다.

---

## 2. 1비트 PDM의 수학적 원리: 밀도 기반의 아날로그 신호 표상

PDM 변조 방식은 아날로그 입력 전압의 순간 진폭을 **1비트 이산 펄스의 밀도(Pulse Density)**로 표현합니다.
* 아날로그 입력 전압이 양의 최댓값 ($+1.0$)에 도달할 때: 출력 비트스트림 상에 논리적 '1'이 극도로 조밀하게 밀집됩니다 (`11111111`).
* 아날로그 입력 전압이 평형 상태 ($0.0$)에 위치할 때: 논리적 '1'과 '0'이 일정한 주기로 균등하게 교차합니다 (`10101010`).
* 아날로그 입력 전압이 음의 최댓값 ($-1.0$)에 도달할 때: 출력 비트스트림 상에 논리적 '0'이 극도로 조밀하게 밀집됩니다 (`00000000`).

### 📊 PDM 펄스 밀도 변환 시각화

<svg viewBox="0 0 800 240" width="100%" height="240" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="800" height="240" rx="10" style="fill: #f8f9fa !important; stroke: #e9ecef !important; stroke-width: 2 !important;" />
  
  <!-- Analog Wave -->
  <path d="M 50 120 Q 225 10, 400 120 T 750 120" style="fill: none !important; stroke: #007bff !important; stroke-width: 3 !important; stroke-dasharray: 4 !important;" />
  <text x="60" y="40" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #007bff !important; font-weight: bold !important;">아날로그 전압 파형 (Analog Signal)</text>

  <!-- PDM Pulses for Positive Peak -->
  <!-- Densely spaced 1s -->
  <g style="fill: #28a745 !important;">
    <!-- Center (0.0) -->
    <rect x="50" y="150" width="4" height="40" rx="1"/>
    <rect x="70" y="150" width="4" height="40" rx="1"/>
    <rect x="90" y="150" width="4" height="40" rx="1"/>
    <rect x="110" y="150" width="4" height="40" rx="1"/>
    
    <!-- Positive Peak (+1.0) -->
    <rect x="160" y="150" width="4" height="40" rx="1"/>
    <rect x="170" y="150" width="4" height="40" rx="1"/>
    <rect x="180" y="150" width="4" height="40" rx="1"/>
    <rect x="190" y="150" width="4" height="40" rx="1"/>
    <rect x="200" y="150" width="4" height="40" rx="1"/>
    <rect x="210" y="150" width="4" height="40" rx="1"/>
    <rect x="220" y="150" width="4" height="40" rx="1"/>
    <rect x="230" y="150" width="4" height="40" rx="1"/>
    <rect x="240" y="150" width="4" height="40" rx="1"/>
    
    <!-- Going Down -->
    <rect x="290" y="150" width="4" height="40" rx="1"/>
    <rect x="310" y="150" width="4" height="40" rx="1"/>
    <rect x="330" y="150" width="4" height="40" rx="1"/>
    <rect x="350" y="150" width="4" height="40" rx="1"/>
    
    <!-- Negative Peak (-1.0) - sparse or no pulses -->
    <rect x="490" y="150" width="4" height="40" rx="1"/>
    <rect x="570" y="150" width="4" height="40" rx="1"/>
    
    <!-- Center (0.0) -->
    <rect x="670" y="150" width="4" height="40" rx="1"/>
    <rect x="690" y="150" width="4" height="40" rx="1"/>
    <rect x="710" y="150" width="4" height="40" rx="1"/>
    <rect x="730" y="150" width="4" height="40" rx="1"/>
  </g>
  
  <line x1="50" y1="200" x2="750" y2="200" style="stroke: #6c757d !important; stroke-width: 2 !important;" />
  <text x="60" y="220" style="font-family: Segoe UI, Arial !important; font-size: 12 !important; fill: #28a745 !important; font-weight: bold !important;">1비트 PDM 출력 (High=1, Low=0) - 전압이 높을수록 촘촘함</text>
</svg>

---

## 3. 데시메이션(Decimation) 필터: 대역 제한 및 표본율 변환 파이프라인

디지털 신호처리 영역에서 데시메이션(Decimation)은 고속으로 과표본화(Oversampling)된 디지털 신호의 대역을 통과 대역 내로 제한하고, 표본화율(Sampling Rate)을 낮추는 전 과정을 지칭합니다.
인공와우 타겟 프로세서는 $1.024\text{ MHz}$ 클럭 속도의 고밀도 1비트 PDM 입력을 수용하여, 저역통과 필터링을 수행한 후 시스템 기준 주파수인 $16\text{ kHz}$ 표본으로 다운샘플링합니다. (이때의 데시메이션 비율은 $R = 64$가 됩니다.)

### ⚙️ 하드웨어 처리 파이프라인 블록 다이어그램

<svg viewBox="0 0 800 220" width="100%" height="220" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="800" height="220" rx="10" style="fill: #f8f9fa !important; stroke: #e9ecef !important; stroke-width: 2 !important;" />

  <!-- Blocks -->
  <!-- DMIC PDM -->
  <rect x="30" y="70" width="140" height="80" rx="8" style="fill: #e1f5fe !important; stroke: #0288d1 !important; stroke-width: 2 !important;" />
  <text x="100" y="105" style="font-family: Segoe UI, Arial !important; font-size: 14 !important; fill: #01579b !important; font-weight: bold !important; text-anchor: middle !important;">디지털 마이크</text>
  <text x="100" y="130" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #0288d1 !important; text-anchor: middle !important;">1.024 MHz PDM</text>

  <!-- Arrow 1 -->
  <line x1="170" y1="110" x2="220" y2="110" marker-end="url(#arrow)" style="stroke: #555 !important; stroke-width: 2 !important;" />
  
  <!-- CIC Filter -->
  <rect x="220" y="70" width="160" height="80" rx="8" style="fill: #ffe0b2 !important; stroke: #f57c00 !important; stroke-width: 2 !important;" />
  <text x="300" y="105" style="font-family: Segoe UI, Arial !important; font-size: 14 !important; fill: #e65100 !important; font-weight: bold !important; text-anchor: middle !important;">CIC 필터 (3차)</text>
  <text x="300" y="130" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #f57c00 !important; text-anchor: middle !important;">곱셈 없음 / 64배 감쇄</text>

  <!-- Arrow 2 -->
  <line x1="380" y1="110" x2="430" y2="110" style="stroke: #555 !important; stroke-width: 2 !important;" />
  <text x="405" y="100" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #555 !important; text-anchor: middle !important;">16 kHz Multi-bit</text>

  <!-- Compensation FIR -->
  <rect x="430" y="70" width="160" height="80" rx="8" style="fill: #c8e6c9 !important; stroke: #388e3c !important; stroke-width: 2 !important;" />
  <text x="515" y="100" style="font-family: Segoe UI, Arial !important; font-size: 13 !important; fill: #1b5e20 !important; font-weight: bold !important; text-anchor: middle !important;">보상 FIR 필터</text>
  <text x="515" y="120" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #388e3c !important; text-anchor: middle !important;">통과 대역 감쇄 평탄화</text>
  <text x="515" y="138" style="font-family: Segoe UI, Arial !important; font-size: 10 !important; fill: #388e3c !important; text-anchor: middle !important;">(Half-Band 최적화)</text>

  <!-- Arrow 3 -->
  <line x1="590" y1="110" x2="640" y2="110" style="stroke: #555 !important; stroke-width: 2 !important;" />

  <!-- PCM Output -->
  <rect x="640" y="70" width="130" height="80" rx="8" style="fill: #d1c4e9 !important; stroke: #5e35b1 !important; stroke-width: 2 !important;" />
  <text x="705" y="105" style="font-family: Segoe UI, Arial !important; font-size: 13 !important; fill: #4a148c !important; font-weight: bold !important; text-anchor: middle !important;">Q1.23 PCM 출력</text>
  <text x="705" y="130" style="font-family: Segoe UI, Arial !important; font-size: 11 !important; fill: #5e35b1 !important; text-anchor: middle !important;">1ms당 16개 샘플</text>
  
  <!-- Arrow marker definition -->
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="5" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M 0 0 L 10 5 L 0 10 z" style="fill: #555 !important;" />
    </marker>
  </defs>
</svg>

### 1) 3차 CIC (Cascaded Integrator-Comb) 필터의 연산 기하학
* **동작 정의:** CIC 필터는 고속 PDM 입력 스트림을 저소모 전력 환경에서 실시간으로 복제하기 위해 하드웨어 곱셈기 없이 누산 적분기(Integrator)와 차분기(Comb Filter)의 연쇄 결합만으로 고속 연산을 처리합니다.
* **구조 설계:** 세 단계의 적분기가 고주파 영역에서 동작한 후, 다운샘플러($\downarrow 64$)를 거쳐 출력 단에서 세 단계의 빗살 모양 차분기(Comb)가 연동되어 $16\text{ kHz}$ 신호를 복원합니다.
* **비트 성장률(Bit Growth) 계산:** 1비트 입력 신호가 다단 적분 루프를 거치며 누적됨에 따라 데이터 워드 길이가 증가합니다. 3차 CIC 필터($N_{stage}=3$, $R=64$)에서 발생하는 최대 비트 성장은 다음과 같은 비트 성장 관계식으로 규정됩니다.
  $$B_{out} = 1 + N_{stage} \log_2(R) = 1 + 3 \log_2(64) = 1 + (3 \times 6) = 19\text{ bits}$$

### 2) 보상 FIR (Finite Impulse Response) 필터 및 하프밴드(Half-Band) 최적화
* **CIC 감쇄(Droop) 보정:** CIC 필터의 통과 대역 주파수 특성은 수학적으로 싱크 함수($\text{Sinc}^3$) 곡선을 보입니다. 이로 인하여 인간의 음성 인지에서 핵심이 되는 $3\text{ kHz} \sim 8\text{ kHz}$ 대역에서 원치 않는 이득 왜곡(Droop)이 발생합니다. 이에 후행하는 FIR 보상 필터를 배치하여 주파수 대역 내 이득을 극도로 평탄화(Flat Passband, $\pm0.1\text{ dB}$)되도록 재교정합니다.
* **하프밴드(Half-Band) 최적화 설계:** 보상 FIR 필터링을 실시간으로 가동하기 위해 계수의 대칭성을 적용합니다. 대칭 중심이 짝수 인덱스일 때 홀수 인덱스 탭 계수가 정확히 **$0.0$**이 되는 주파수 대칭 특성을 활용하여 연산 효율을 높입니다. 이 구조를 통해 하드웨어 연산 장치(MAC)는 계수가 $0$인 구간에서 곱셈 및 누산 연산을 무시하고 건너뛰는 바이패스 제어를 수행하여, 연산 소모 전류를 약 50% 수준으로 절감합니다.

---

## 4. PDM-to-PCM 데시메이션 필터 파이썬 시뮬레이션

> [!NOTE]
> **시뮬레이션 모델의 간소화 안내**
> 본문의 이론적 배경과 하드웨어 파이프라인 단락에서는 주파수 차단 성능을 고려한 **3차 CIC 필터**를 기술하고 있습니다. 그러나 아래에 수록된 파이썬 시뮬레이션 코드는 CIC 필터의 기본적인 수학적 원리인 적분 및 다운샘플링 거동을 직관적으로 이해할 수 있도록, 이동 평균(Moving Average) 구조로 축소한 **1차 CIC 단순화 모델**로 구현되었습니다.

다음 코드는 1차 시그마-델타 변조기 모델을 통해 1비트 PDM 비트스트림을 생성하고, 하드웨어 데시메이션 및 다운샘플링 수식을 구현하여 최종 PCM 값으로 원복하는 전 과정을 확인하기 위한 참조 시뮬레이션 코드입니다.

```python
import numpy as np

def generate_pdm_stream(analog_signal, osr=64):
    """
    1차 시그마-델타 변조기(Sigma-Delta Modulator) 모델을 사용하여
    아날로그 전압 파형 데이터를 오버샘플링 PDM 1비트 스트림으로 인코딩합니다.
    
    Parameters:
        analog_signal (np.ndarray): 입력 아날로그 신호
        osr (int): 과표본화 비율 (Over Sampling Ratio)
    """
    pdm = []
    integrator = 0.0
    for sample in analog_signal:
        # 단순 영차 홀드(Zero-Order Hold) 보간 수행
        for _ in range(osr):
            # 피드백 판정 (이산 펄스는 +1.0 또는 -1.0)
            feedback = 1.0 if integrator >= 0.0 else -1.0
            error = sample - feedback
            integrator += error  # 오차 적분 수행
            
            # 이진 펄스로 이송 (1 또는 0)
            pdm_bit = 1 if feedback > 0 else 0
            pdm.append(pdm_bit)
    return np.array(pdm)

def simple_cic_decimation(pdm_stream, decimation_factor=64):
    """
    1차 CIC 필터의 적분-샘플링-차분 특성을 간략화하여
    평균화 필터 및 데시메이션(R=64) 다운샘플링을 거쳐 PCM 신호로 원복합니다.
    """
    pcm_out = []
    # 0과 1의 비트를 양극성 전압 물리 신호 (-1.0, +1.0)로 해석
    bipolar_pdm = pdm_stream * 2.0 - 1.0
    
    # 설정된 데시메이션 비율 단위로 데이터 윈도우 이동 및 적분 평균 산출
    for i in range(0, len(bipolar_pdm), decimation_factor):
        block = bipolar_pdm[i:i+decimation_factor]
        if len(block) < decimation_factor:
            break
        # 블록 단위 총합 평균 연산 (1차 CIC 적분기 필터 거동 모사)
        pcm_val = np.sum(block) / decimation_factor
        pcm_out.append(pcm_val)
        
    return np.array(pcm_out)

# --- 신호 처리 흐름 검증 ---
if __name__ == "__main__":
    # 1. 100 Hz 주파수를 가지는 기저 대역 코사인 입력 아날로그 신호 생성
    t = np.arange(0, 0.01, 1/16000)
    analog_sig = 0.8 * np.cos(2 * np.pi * 100 * t)  # 진폭 0.8 설정
    
    # 2. 16 kHz 신호를 64배 오버샘플링하여 1.024 MHz PDM 스트림으로 생성
    pdm_bits = generate_pdm_stream(analog_sig, osr=64)
    print(f"입력 아날로그 신호 샘플 수: {len(analog_sig)} 개")
    print(f"생성된 PDM 1비트 스트림 크기: {len(pdm_bits)} 비트 ({len(pdm_bits)/(10.0*0.001*1000):.3f} MHz)")
    print(f"PDM 스트림 초반 32비트 패턴: {pdm_bits[:32]}")
    
    # 3. 64배 다운샘플러 및 대역 차분 연산을 통한 16 kHz PCM 원복
    decoded_pcm = simple_cic_decimation(pdm_bits, decimation_factor=64)
    print(f"복원된 PCM 출력 샘플 수: {len(decoded_pcm)} 개 (16 kHz)")
    
    # 원 신호와 복원된 이산 PCM 신호 간 이득 수준 검증
    for i in range(min(5, len(decoded_pcm))):
        print(f"인덱스 {i}: 기준 입력 아날로그 = {analog_sig[i]:.4f} -> 복원된 이산 PCM = {decoded_pcm[i]:.4f}")

---

➡️ **다음 단계:** [Module 2. 24비트 고정소수점(Fixed-Point) Q1.23 수학](02_fixed_point_q1_23.md)
