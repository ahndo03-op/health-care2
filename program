import streamlit as st
import plotly.graph_objects as go
import plotly.express as px
import pandas as pd
import numpy as np

# 페이지 기본 설정
st.set_page_config(page_title="NutriGuard - 맞춤 건강 식단 & 혈당 예측", page_icon="🥗", layout="wide")

st.title("🥗 NutriGuard: 초가공식품 탈출 & 맞춤 식단 가이드")
st.caption("신체 정보와 목표에 맞는 영양 비율 및 식품별 혈당 변화 예측 차트를 제공합니다.")

# -------------------------------------------------------------------
# 1. 사이드바 - 사용자 정보 입력
# -------------------------------------------------------------------
st.sidebar.header("📋 사용자 신체 정보 입력")

gender = st.sidebar.radio("성별", ["남성", "여성"])
age = st.sidebar.number_input("나이 (세)", min_value=10, max_value=100, value=25)
height = st.sidebar.number_input("키 (cm)", min_value=100.0, max_value=230.0, value=170.0)
weight = st.sidebar.number_input("몸무게 (kg)", min_value=30.0, max_value=200.0, value=65.0)

activity_level = st.sidebar.selectbox(
    "일일 활동량",
    ["비활동적 (운동 안 함)", "보통 (주 1~3회 운동)", "활동적 (주 4~5회 운동)", "매우 활동적 (매일 강한 운동)"]
)

goal = st.sidebar.selectbox(
    "건강 목표",
    ["체중 및 혈당 유지", "체중 감량 (체지방 감소)", "근육 증량"]
)

# -------------------------------------------------------------------
# 2. 기초대사량(BMR) 및 일일 칼로리/영양소 계산
# -------------------------------------------------------------------
if gender == "남성":
    bmr = 10 * weight + 6.25 * height - 5 * age + 5
else:
    bmr = 10 * weight + 6.25 * height - 5 * age - 161

activity_multipliers = {
    "비활동적 (운동 안 함)": 1.2,
    "보통 (주 1~3회 운동)": 1.375,
    "활동적 (주 4~5회 운동)": 1.55,
    "매우 활동적 (매일 강한 운동)": 1.725
}
tdee = bmr * activity_multipliers[activity_level]

if goal == "체중 감량 (체지방 감소)":
    target_calories = tdee - 400
    carbs_ratio, protein_ratio, fat_ratio = 40, 35, 25  # 탄/단/지 비율 (%)
elif goal == "근육 증량":
    target_calories = tdee + 300
    carbs_ratio, protein_ratio, fat_ratio = 50, 30, 20
else:
    target_calories = tdee
    carbs_ratio, protein_ratio, fat_ratio = 50, 20, 30

bmi = weight / ((height / 100) ** 2)

# 목표 칼로리 기반 영양소 g 수 환산 (탄수화물/단백질: 4kcal/g, 지방: 9kcal/g)
carbs_g = int((target_calories * (carbs_ratio / 100)) / 4)
protein_g = int((target_calories * (protein_ratio / 100)) / 4)
fat_g = int((target_calories * (fat_ratio / 100)) / 9)

# -------------------------------------------------------------------
# 3. 메인 화면 탭 구성
# -------------------------------------------------------------------
tab1, tab2, tab3 = st.tabs(["📊 맞춤 영양 & 영양소 비율", "🍱 권장 식단표", "📈 혈당 스파이크 예측 & TMI"])

# -------------------------------------------------------------------
# [TAB 1] 영양 가이드 및 영양소 비율 차트
# -------------------------------------------------------------------
with tab1:
    st.subheader("💡 나의 신체 지표 & 하루 권장 영양 가이드")
    
    col1, col2, col3, col4 = st.columns(4)
    col1.metric("BMI 지수", f"{bmi:.1f} kg/m²")
    col2.metric("기초대사량 (BMR)", f"{int(bmr)} kcal")
    col3.metric("유지 칼로리 (TDEE)", f"{int(tdee)} kcal")
    col4.metric("목표 일일 칼로리", f"{int(target_calories)} kcal")

    st.markdown("---")
    
    col_chart, col_limit = st.columns([1, 1])
    
    with col_chart:
        st.write("### 🥗 목표 맞춤 3대 영양소 권장 비율")
        
        # Plotly 도넛 차트 생성
        fig_macro = go.Figure(data=[go.Pie(
            labels=['탄수화물 (Carbs)', '단백질 (Protein)', '지방 (Fat)'],
            values=[carbs_g * 4, protein_g * 4, fat_g * 9],  # 칼로리 기준
            hole=.4,
            marker_colors=['#4CAF50', '#2196F3', '#FF9800'],
            textinfo='label+percent',
            hovertemplate='<b>%{label}</b><br>칼로리: %{value:.0f} kcal<br>비율: %{percent}<extra></extra>'
        )])
        
        fig_macro.update_layout(
            showlegend=False,
            margin=dict(t=20, b=20, l=20, r=20),
            height=300
        )
        st.plotly_chart(fig_macro, use_container_width=True)
        st.info(f"👉 **일일 권장량:** 탄수화물 **{carbs_g}g** | 단백질 **{protein_g}g** | 지방 **{fat_g}g**")

    with col_limit:
        st.write("### ⚠️ 하루 최대 섭취 제한 가이드")
        max_caffeine = 400 if age >= 19 else int(weight * 2.5)
        max_sugar = int(target_calories * 0.05 / 4)
        
        st.warning(f"☕ **카페인:** 최대 **{max_caffeine} mg** / 일 (아메리카노 약 {max_caffeine // 150}잔)")
        st.error(f"🍩 **첨가당/과당:** 최대 **{max_sugar} g** / 일 (각설탕 약 {max_sugar // 3}개)")
        st.info(f"🌾 **정제 탄수화물:** 총 탄수화물의 **30% 이하**로 제한")
        st.error(f"🥓 **가공육:** 일 **50g 미만** 권장 (대장암 위험 감소)")

# -------------------------------------------------------------------
# [TAB 2] 권장 식단표
# -------------------------------------------------------------------
with tab2:
    st.subheader("🍱 목표 맞춤 1일 식단 제안")
    st.write(f"선택하신 **[{goal}]** 목표에 부합하는 클린 식단 조합입니다.")

    if goal == "체중 감량 (체지방 감소)":
        st.success("""
        - **아침:** 사과 1/2개 + 플레인 요거트 + 계란 2개 (삶은 것)
        - **점심:** 현미밥 2/3공기 + 닭가슴살 100g + 쌈채소 & 구운 버섯
        - **간식:** 견과류 한 주먹 (아몬드 10알) + 루이보스 티
        - **저녁:** 두부 구이 1/2모 + 오이, 파프리카 샐러드 (올리브유 드레싱)
        """)
    elif goal == "근육 증량":
        st.success("""
        - **아침:** 잡곡밥 1공기 + 계란프라이 2개 + 시금치나물 + 사과 1개
        - **점심:** 현미 곤약밥 1공기 + 소고기 우둔살 150g + 브로콜리
        - **간식:** 바나나 1개 + 무첨가 두유 1팩
        - **저녁:** 연어 구이 150g + 고구마 1개 + 모듬 채소
        """)
    else:
        st.success("""
        - **아침:** 통밀빵 1장 + 계란 1개 + 방울토마토 10알
        - **점심:** 일반 잡곡밥 1공기 + 생선구이(고등어/삼치) + 나물 반찬 2종
        - **간식:** 제철 과일 1조각
        - **저녁:** 닭안심구이 + 단호박 + 모듬 채소 볶음
        """)

# -------------------------------------------------------------------
# [TAB 3] 혈당 스파이크 예측 차트 & TMI
# -------------------------------------------------------------------
with tab3:
    st.subheader("📈 음식을 먹은 후 내 혈당은 어떻게 변할까? (혈당 스파이크 시뮬레이션)")
    st.caption("초가공식품/정제 탄수화물과 복합 탄수화물(클린 식단) 섭취 후 2시간 동안의 예상 혈당 변화 비교 그래프입니다.")

    # 혈당 변화 시뮬레이션 데이터 생성 (0분 ~ 120분)
    time = np.linspace(0, 120, 13)  # 10분 간격
    fasting_glucose = 90  # 공복 혈당 기준 (mg/dL)

    # 1. 초가공식품 / 액상과당 (급격한 상승 후 추락 - 혈당 스파이크)
    glucose_upf = fasting_glucose + 85 * np.sin(np.pi * time / 70) * np.exp(-time / 90)
    # 2. 정제 탄수화물 (흰쌀, 흰빵)
    glucose_refined = fasting_glucose + 60 * np.sin(np.pi * time / 80) * np.exp(-time / 100)
    # 3. 건강한 복합 탄수화물 + 단백질/채소 (완만한 상승)
    glucose_healthy = fasting_glucose + 30 * np.sin(np.pi * time / 100) * np.exp(-time / 120)

    df_glucose = pd.DataFrame({
        '시간(분)': time,
        '초가공식품 & 액상과당 (탄산음료, 탕후루 등)': glucose_upf,
        '정제 탄수화물 (흰쌀밥, 빵, 라면)': glucose_refined,
        '통곡물 & 채소 식단 (현미밥, 샐러드 등)': glucose_healthy
    })

    # Plotly 라인 차트 생성
    fig_glucose = go.Figure()

    fig_glucose.add_trace(go.Scatter(
        x=df_glucose['시간(분)'], y=df_glucose['초가공식품 & 액상과당 (탄산음료, 탕후루 등)'],
        mode='lines+markers', name='❌ 초가공식품 / 액상과당',
        line=dict(color='#FF1744', width=3)
    ))

    fig_glucose.add_trace(go.Scatter(
        x=df_glucose['시간(분)'], y=df_glucose['정제 탄수화물 (흰쌀밥, 빵, 라면)'],
        mode='lines+markers', name='⚠️ 정제 탄수화물',
        line=dict(color='#FF9100', width=2, dash='dash')
    ))

    fig_glucose.add_trace(go.Scatter(
        x=df_glucose['시간(분)'], y=df_glucose['통곡물 & 채소 식단 (현미밥, 샐러드 등)'],
        mode='lines+markers', name='✅ 건강한 복합 탄수화물 식단',
        line=dict(color='#00E676', width=3)
    ))

    # 위험 임계선 추가 (140 mg/dL 이상 식후 혈당 주의 구간)
    fig_glucose.add_hline(
        y=140, line_dash="dot", line_color="red",
        annotation_text="혈당 스파이크 위험 기준선 (140 mg/dL)", 
        annotation_position="top left"
    )

    fig_glucose.update_layout(
        title="<b>식후 시간에 따른 혈당 변화(mg/dL) 추이 비교</b>",
        xaxis_title="섭취 후 시간 (분)",
        yaxis_title="혈당 수치 (mg/dL)",
        hovermode="x unified",
        legend=dict(yanchor="top", y=0.99, xanchor="left", x=0.01),
        height=450
    )

    st.plotly_chart(fig_glucose, use_container_width=True)

    st.markdown("""
    > **💡 혈당 스파이크(Blood Sugar Spike)란?**  
    > 음식을 먹은 뒤 혈당이 급격하게 올랐다가 내리는 현상을 말합니다. 혈당이 급상승하면 췌장에서 인슐린이 과다 분비되어 **지방 축적, 식곤증, 가짜 배고픔**을 유발하고, 장기적으로는 **젊은 당뇨병 및 뇌졸중/심합병증**의 원인이 됩니다.
    """)

    st.markdown("---")
    st.subheader("🧠 카드 뉴스형 영양 TMI")
    
    col_tmi1, col_tmi2 = st.columns(2)
    with col_tmi1:
        with st.expander("❓ 초가공식품(UPF)이 대장암 위험을 높이는 이유"):
            st.write("유화제와 산화방지제 등 인공 첨가물은 대장 내 유익균을 감소시키고 장 점막에 지속적인 미세 염증을 일으킵니다.")
    with col_tmi2:
        with st.expander("❓ 액상과당이 일반 설탕보다 위험한 이유"):
            st.write("액상과당은 인슐린/렙틴 호르몬 자극 없이 간으로 바로 이동하여 **비알코올성 지방간**과 **내장 지방**을 급격히 만듭니다.")
