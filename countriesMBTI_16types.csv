import streamlit as st
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots

# 페이지 설정
st.set_page_config(
    page_title="MBTI 국가별 분석",
    page_icon="🌍",
    layout="wide"
)

# 타이틀과 설명
st.title("🌍 MBTI 성격 유형별 국가 분석 대시보드")
st.markdown("""
이 대시보드는 158개 국가의 MBTI 16가지 성격 유형 분포 데이터를 분석합니다.
각 MBTI 유형별로 가장 높은 비율을 보이는 국가들을 확인할 수 있습니다.
""")

# 데이터 로드 함수
@st.cache_data
def load_data():
    """기본 파일을 먼저 시도하고, 없으면 None 반환"""
    try:
        df = pd.read_csv('countriesMBTI_16types.csv')
        return df, True  # 성공적으로 로드됨
    except FileNotFoundError:
        return None, False  # 파일이 없음

# 데이터 로드 시도
df, file_loaded = load_data()

# 파일 로드 상태에 따른 처리
if file_loaded:
    st.success("✅ 기본 데이터 파일(countriesMBTI_16types.csv)을 성공적으로 로드했습니다!")
    uploaded_file = None
else:
    st.info("📁 기본 데이터 파일을 찾을 수 없습니다. CSV 파일을 업로드해주세요.")
    uploaded_file = st.file_uploader(
        "MBTI 데이터 CSV 파일을 업로드하세요",
        type=['csv'],
        help="countriesMBTI_16types.csv 파일을 업로드하세요"
    )
    
    if uploaded_file is not None:
        # 업로드된 파일로 데이터 로드
        df = pd.read_csv(uploaded_file)
        st.success("✅ 업로드된 파일을 성공적으로 로드했습니다!")

# 데이터가 로드된 경우에만 분석 진행
if df is not None:
    
    # MBTI 유형 리스트
    mbti_types = ['INFJ', 'ISFJ', 'INTP', 'ISFP', 'ENTP', 'INFP', 
                  'ENTJ', 'ISTP', 'INTJ', 'ESFP', 'ESTJ', 'ENFP', 
                  'ESTP', 'ISTJ', 'ENFJ', 'ESFJ']
    
    # 사이드바에서 MBTI 유형 선택
    st.sidebar.header("🎯 분석 옵션")
    selected_mbti = st.sidebar.selectbox(
        "MBTI 유형을 선택하세요:",
        mbti_types,
        index=mbti_types.index('INFP')  # 기본값: INFP (가장 높은 평균 비율)
    )
    
    # TOP N 국가 수 선택
    top_n = st.sidebar.slider("상위 몇 개국을 표시할까요?", 5, 20, 10)
    
    # 메인 대시보드
    col1, col2 = st.columns([2, 1])
    
    with col1:
        st.subheader(f"📊 {selected_mbti} 유형 비율이 높은 국가 TOP {top_n}")
        
        # 선택된 MBTI 유형의 상위 국가들
        top_countries = df.nlargest(top_n, selected_mbti)[['Country', selected_mbti]]
        top_countries[f'{selected_mbti}_percent'] = top_countries[selected_mbti] * 100
        
        # 막대 그래프
        fig_bar = px.bar(
            top_countries, 
            x='Country', 
            y=f'{selected_mbti}_percent',
            title=f'{selected_mbti} 유형 비율 상위 {top_n}개국',
            labels={f'{selected_mbti}_percent': '비율 (%)', 'Country': '국가'},
            color=f'{selected_mbti}_percent',
            color_continuous_scale='viridis'
        )
        fig_bar.update_layout(
            xaxis_tickangle=-45,
            height=500,
            showlegend=False
        )
        st.plotly_chart(fig_bar, use_container_width=True)
    
    with col2:
        st.subheader("📈 상위 10개국 상세 정보")
        
        # 상위 10개국 테이블
        display_df = top_countries.head(10).copy()
        display_df[f'{selected_mbti} 비율'] = display_df[f'{selected_mbti}_percent'].round(1).astype(str) + '%'
        display_df = display_df[['Country', f'{selected_mbti} 비율']].reset_index(drop=True)
        display_df.index = display_df.index + 1
        
        st.dataframe(display_df, use_container_width=True)
        
        # 선택된 유형의 전 세계 평균
        global_avg = df[selected_mbti].mean() * 100
        st.metric(
            label=f"{selected_mbti} 전 세계 평균", 
            value=f"{global_avg:.1f}%"
        )
    
    # 추가 분석 섹션
    st.subheader("🌐 전체 MBTI 유형별 글로벌 평균 비교")
    
    # 전체 MBTI 유형의 평균 계산
    mbti_averages = {}
    for mbti in mbti_types:
        mbti_averages[mbti] = df[mbti].mean() * 100
    
    # 평균 비율로 정렬
    sorted_mbti = sorted(mbti_averages.items(), key=lambda x: x[1], reverse=True)
    
    # 수평 막대 그래프
    mbti_names = [item[0] for item in sorted_mbti]
    mbti_values = [item[1] for item in sorted_mbti]
    
    fig_global = px.bar(
        x=mbti_values,
        y=mbti_names,
        orientation='h',
        title='MBTI 유형별 전 세계 평균 비율',
        labels={'x': '평균 비율 (%)', 'y': 'MBTI 유형'},
        color=mbti_values,
        color_continuous_scale='plasma'
    )
    fig_global.update_layout(
        height=600,
        yaxis={'categoryorder': 'total ascending'}
    )
    st.plotly_chart(fig_global, use_container_width=True)
    
    # 히트맵 섹션
    st.subheader("🗺️ 국가별 MBTI 분포 히트맵 (상위 20개국)")
    
    # 상위 20개국 선택 (전체 MBTI 분산이 높은 국가들)
    df_numeric = df[mbti_types]
    df['variance'] = df_numeric.var(axis=1)
    top_20_countries = df.nlargest(20, 'variance')
    
    # 히트맵 데이터 준비
    heatmap_data = top_20_countries[['Country'] + mbti_types].set_index('Country')
    heatmap_data = heatmap_data * 100  # 퍼센트로 변환
    
    # 히트맵 생성
    fig_heatmap = px.imshow(
        heatmap_data.values,
        x=heatmap_data.columns,
        y=heatmap_data.index,
        title="국가별 MBTI 분포 히트맵 (분포가 다양한 상위 20개국)",
        labels=dict(x="MBTI 유형", y="국가", color="비율 (%)"),
        color_continuous_scale='RdYlBu_r'
    )
    fig_heatmap.update_layout(height=700)
    st.plotly_chart(fig_heatmap, use_container_width=True)
    
    # 데이터 요약 정보
    with st.expander("📋 데이터셋 정보"):
        st.write(f"**총 국가 수:** {len(df)}개")
        st.write(f"**MBTI 유형 수:** {len(mbti_types)}개")
        st.write(f"**가장 흔한 유형:** {sorted_mbti[0][0]} ({sorted_mbti[0][1]:.1f}%)")
        st.write(f"**가장 드문 유형:** {sorted_mbti[-1][0]} ({sorted_mbti[-1][1]:.1f}%)")
        
        # 샘플 데이터 보기
        st.write("**데이터 샘플:**")
        st.dataframe(df.head(), use_container_width=True)

else:
    st.info("""
    👆 **파일 업로드가 필요합니다:**
    
    **방법 1 (권장):** 
    - `countriesMBTI_16types.csv` 파일을 앱과 같은 폴더에 위치시키세요
    - 앱이 자동으로 데이터를 로드합니다
    
    **방법 2:**
    - 위의 파일 업로더를 사용해서 CSV 파일을 업로드하세요
    
    **주요 기능:**
    - 📊 MBTI 유형별 상위 국가 막대 그래프
    - 📈 전체 MBTI 유형 글로벌 평균 비교
    - 🗺️ 국가별 MBTI 분포 히트맵
    - 📋 상세 통계 정보
    """)

# 푸터
st.markdown("---")
st.markdown(
    "💡 **Made with Streamlit** | 데이터 출처: MBTI 국가별 분포 연구 데이터"
)
