import streamlit as st
import numpy as np
import matplotlib.pyplot as plt

# 1. 웹 페이지 기본 설정
st.set_page_config(page_title="물리II x 지학II 외계행성 탐사 시뮬레이터", layout="wide")

st.title("🪐 물리학II × 지구과학II 융합 탐구: 이체 운동과 시선 속도(도플러 효과) 분석기")
st.markdown("""
이 프로그램은 **물리학II의 역학적 개념(공통 질량 중심, 이체 운동, 도플러 효과)**과 **지구과학II의 천문 관측 개념(시선 속도 변화를 이용한 외계행성 탐사)**을 융합하여 시각화한 대시보드입니다.
사이드바의 물리량을 조절하며 별과 행성이 질량 중심을 기준으로 어떻게 상호작용하는지, 그리고 지구에서 그것이 어떻게 관측되는지 분석해 보세요.
""")

# 2. 사이드바 제어판 (물리 변수 입력)
st.sidebar.header("🛠️ 물리 및 천문 매개변수 설정")

# 중심 별의 질량 (태양 질량 기준)
M_star = st.sidebar.slider("중심 별의 질량 ($M_\\odot$)", 0.5, 2.0, 1.0, 0.1)

# 외계행성의 질량 (목성 질량 기준, 1 M_jupiter ~ 0.001 M_sun)
m_planet_mj = st.sidebar.slider("외계행성 질량 ($M_{Jupiter}$)", 1.0, 20.0, 5.0, 0.5)
# 목성 질량을 태양 질량 단위로 변환 (1 M_jup ≈ 0.000954 M_sun)
m_planet = m_planet_mj * 0.000954

# 행성의 공전 주기 (일 단위 -> 년 단위 변환)
period_days = st.sidebar.slider("행성의 공전 주기 (일, Days)", 2.0, 20.0, 4.0, 0.5)
T = period_days / 365.25 # 년 단위

# 궤도 경사각 (지구 시선 방향과 궤도면이 이루는 각도, 90도이면 시선 방향과 일치)
inclination_deg = st.sidebar.slider("궤도 경사각 ($i$, 도)", 0, 90, 90, 5)
i_rad = np.radians(inclination_deg)

# 현재 공전 궤도 위상(시간) 조절 슬라이더 -> 행성 궤도 운동을 두드러지게 관찰하는 핵심 위젯
st.sidebar.markdown("---")
st.sidebar.subheader("⏰ 실시간 궤도 운동 추적")
phase = st.sidebar.slider("공전 위상 (0 ~ 1 주행)", 0.0, 1.0, 0.0, 0.01)
theta = 2 * np.pi * phase # 라디안 각도

# 3. 물리 및 천문 이론 수식 계산 (물리II x 지학II 융합식)
# [지학II] 케플러 제3법칙 변형식을 이용한 공전 궤도 장반경(a) 계산: a^3 = (M_star + m_planet) * T^2
a = ((M_star + m_planet) * (T**2))**(1/3) # AU 단위

# [물리II] 이체 문제(Two-body problem)에 따른 공통 질량 중심(Barycenter)으로부터의 거리 계산
# M_star * r_star = m_planet * r_planet, r_star + r_planet = a
r_star = a * (m_planet / (M_star + m_planet))
r_planet = a * (M_star / (M_star + m_planet))

# [물리II] 공전 속도 계산 (v = 2 * pi * r / T)
# AU/year 단위를 m/s 단위로 변환 (1 AU ≈ 1.496e11 m, 1 year ≈ 3.154e7 s)
au_to_m = 1.496e11
yr_to_s = 3.154e7

v_star_m_s = (2 * np.pi * r_star * au_to_m) / (T * yr_to_s)
v_planet_m_s = (2 * np.pi * r_planet * au_to_m) / (T * yr_to_s)

# [지학II] 시선 속도(Radial Velocity) 계산: v_r = v_star * sin(i) * cos(theta)
# 지구 관측자가 -Y축 먼 곳에 있다고 가정하면, X축 방향 성분 또는 Y축 성분으로 시선 방향 정의 가능
# 여기서는 관측자가 아래쪽(-Y 방향)에서 바라본다고 설정하여, 별이 다가올 때(-)와 멀어질 때(+)를 정의함
v_radial = v_star_m_s * np.sin(i_rad) * np.sin(theta)

# [물리II] 빛의 도플러 효과와 파장 변화량 계산 (기준 파장: 수소 알파선 656.3 nm)
c = 3.0e8 # 빛의 속도 (m/s)
lambda_0 = 656.3 # nm
delta_lambda = lambda_0 * (v_radial / c)

# 4. 상단 정량적 데이터 메트릭 컴포넌트 출력
col_m1, col_m2, col_m3, col_m4 = st.columns(4)
with col_m1:
    st.metric(label="🌌 총 궤도 반지름 (a)", value=f"{a:.4f} AU")
with col_m2:
    st.metric(label="⚖️ 공통 질량 중심-별 거리 ($r_{star}$)", value=f"{r_star * 1000:.4f} x $10^{-3}$ AU")
with col_m3:
    st.metric(label="🏃‍♂️ 별의 실제 공전 속도 ($v_{star}$)", value=f"{v_star_m_s:.2f} m/s")
with col_m4:
    st.metric(label="📡 현재 관측 시선 속도 ($v_r$)", value=f"{v_radial:.2f} m/s")

st.markdown("---")

# 5. 메인 시각화 영역 (두 개의 그래프를 병렬 배치하여 운동성 강조)
col_graph1, col_graph2 = st.columns(2)

with col_graph1:
    st.subheader("🪐 1. 공통 질량 중심 기준 이체 궤도 운동 (우주 부감 뷰)")
    
    fig1, ax1 = plt.subplots(figsize=(6, 6))
    
    # 질량 중심 (0,0) 표시
    ax1.plot(0, 0, 'kx', markersize=10, label="Barycenter (질량중심)")
    
    # 전체 궤도 동선 그리기 (원형 궤도 가정)
    phi_vals = np.linspace(0, 2*np.pi, 200)
    # 별의 궤도 (크기가 작으므로 시각적 식별을 위해 약간 강조하여 그림)
    # 실제 물리 스케일을 유지하되, 별의 이동이 눈에 띄도록 별의 위치 계산
    x_s_orbit = r_star * np.cos(phi_vals)
    y_s_orbit = r_star * np.sin(phi_vals)
    ax1.plot(x_s_orbit, y_s_orbit, 'orange', linestyle='--', alpha=0.7, label="Star Orbit")
    
    # 행성의 궤도
    # 이체 운동 물리 법칙상 별과 행성은 질량 중심을 사이에 두고 항상 180도 반대편에 위치함 (θ와 θ + π)
    x_p_orbit = r_planet * np.cos(phi_vals)
    y_p_orbit = r_planet * np.sin(phi_vals)
    ax1.plot(x_p_orbit, y_p_orbit, 'dodgerblue', linestyle='--', alpha=0.5, label="Planet Orbit")
    
    # 현재 위상에 따른 별과 행성의 실시간 위치 벡터 계산
    # 별의 위치 (행성과 180도 반대 위상)
    x_s = r_star * np.cos(theta + np.pi)
    y_s = r_star * np.sin(theta + np.pi)
    # 행성의 위치
    x_p = r_planet * np.cos(theta)
    y_p = r_planet * np.sin(theta)
    
    # 질량 중심을 통과하는 정렬선 (이체 운동의 핵심 결속 관계 시각화)
    ax1.plot([x_s, x_p], [y_s, y_p], 'gray', linestyle=':', alpha=0.6)
    
    # 천체 마커 배치 (별은 크게, 행성은 작게)
    ax1.plot(x_s, y_s, 'ro', markersize=14, label="Star (중심 별)")
    ax1.plot(x_p, y_p, 'go', markersize=7, label="Exoplanet (외계행성)")
    
    # 지구 관측자 시선 방향 표시 (아래쪽 -Y에서 위쪽 +Y를 바라봄)
    ax1.annotate('', xy=(0, -r_planet*0.5), xytext=(0, -r_planet*1.1),
                arrowprops=dict(facecolor='purple', shrink=0.05, width=2, headwidth=8))
    ax1.text(0, -r_planet*1.2, "👁️ 지구 관측자 시선 (Toward +Y)", color='purple', ha='center', fontsize=10)
    
    # 그래프 스타일링
    ax1.set_xlim(-r_planet*1.3, r_planet*1.3)
    ax1.set_ylim(-r_planet*1.3, r_planet*1.3)
    ax1.set_aspect('equal')
    ax1.set_xlabel("X (AU)", fontsize=9)
    ax1.set_ylabel("Y (AU)", fontsize=9)
    ax1.grid(True, alpha=0.3)
    ax1.legend(loc="upper right", fontsize=8)
    
    st.pyplot(fig1)

with col_graph2:
    st.subheader("📈 2. 별의 관측 시선 속도 곡선 (도플러 그래프)")
    
    fig2, ax2 = plt.subplots(figsize=(6, 6))
    
    # 전체 1주기 동안의 시선 속도 곡선 베이스라인 생성
    phases_line = np.linspace(0, 1, 100)
    v_rad_line = v_star_m_s * np.sin(i_rad) * np.sin(2 * np.pi * phases_line)
    
    ax2.plot(phases_line, v_rad_line, 'purple', linewidth=2.5, label="Radial Velocity Curve")
    
    # 현재 위상에서의 관측 포인트 찍기
    # 청색편이(다가옴, -값)와 적색편이(멀어짐, +값)에 따라 마커 색상 다변화
    if v_radial > 0.5:
        point_color = 'red'
        status_text = f"🔴 적색편이 (멀어지는 중: +{v_radial:.1f} m/s)\nΔλ = +{delta_lambda:.4f} nm"
    elif v_radial < -0.5:
        point_color = 'blue'
        status_text = f"🔵 청색편이 (다가오는 중: {v_radial:.1f} m/s)\nΔλ = {delta_lambda:.4f} nm"
    else:
        point_color = 'black'
        status_text = "⚪ 시선 속도 변화 없음 (횡운동 상태)"
        
    ax2.plot(phase, v_radial, color=point_color, marker='o', markersize=10, label="Current Observation")
    
    # 상하 대칭선 및 중심 가이드라인
    ax2.axhline(0, color='gray', linestyle='--', alpha=0.5)
    
    # 그래프 스타일링
    # Y축 범위 자동 조절 및 고정 (최대 속도 기준)
    y_bound = max(v_star_m_s * np.sin(i_rad) * 1.3, 10.0)
    ax2.set_ylim(-y_bound, y_bound)
    ax2.set_xlabel("공전 위상 (Orbital Phase)", fontsize=9)
    ax2.set_ylabel("별의 시en속도 (Radial Velocity, m/s)", fontsize=9)
    ax2.grid(True, alpha=0.3)
    ax2.legend(loc="upper right", fontsize=8)
    
    # 현재 편이 상태 텍스트 박스 박아주기
    ax2.text(0.05, -y_bound * 0.9, status_text, fontsize=10, 
             bbox=dict(facecolor='white', alpha=0.8, boxstyle='round,pad=0.5'))
    
    st.pyplot(fig2)

# 6. 교과 심층 연계 탐구 가이드 문서화
st.markdown("---")
st.subheader("💡 물리학II & 지구과학II 교과 개념 압축 코멘트 (세특 작성용 소스)")

st.success("""
**🔬 물리학 II 연계 관점: 이체 역학과 에너지 보존**
1. **공통 질량 중심 메커니즘:** 외계행성의 질량이 커질수록($M_{Jupiter}$ 슬라이더를 높일수록) 중심 별이 도는 궤도 반경 $r_{star}$와 실제 공전 속도 $v_{star}$가 정량적으로 증가하여, 공통 질량 중심을 기준으로 한 질량 모멘트 보존 법칙($M_1r_1 = M_2r_2$)이 유효함을 증명할 수 있습니다.
2. **도플러 효과의 물리적 매칭:** 파장의 변화량 $\\Delta \\lambda = \\lambda_0 \\frac{v_r}{c}$ 식에 따라 별의 시선 속도가 최대일 때 분광선 변화폭이 가장 큼을 대시보드 우측 그래프의 파장 변화 값으로 확인할 수 있습니다.

**🌌 지구과학 II 연계 관점: 외계행성 탐사 방법론의 정밀 분석**
1. **궤도 경사각($i$)의 한계성 분석:** 경사각 슬라이더를 `0도`로 설정하면(행성 궤도면을 위에서 수직으로 내려다보는 경우), 우주 부감 뷰에서는 이체 운동이 활발히 일어나지만 도플러 그래프의 시선 속도는 `0`으로 수렴합니다. 이를 통해 **시선 속도 변화 법은 궤도면이 관측자의 시선과 평행할수록(90도에 가까울수록) 유리하고, 수직인 궤도는 발견할 수 없다**는 교과서적 한계를 완벽히 도출해낼 수 있습니다.
2. **Hot Jupiter(뜨거운 목성형 행성) 발견 유리성:** 주기 $T$를 짧게(일 수를 줄이고), 행성 질량을 크게 할수록 시선 속도 곡선의 진폭이 극대화되어 초기 외계행성 탐사에서 뜨거운 목성형 행성들이 왜 주로 발견되었는지 관측 편향(Observation Bias)을 정량적 수치로 해석해낼 수 있습니다.
""")
