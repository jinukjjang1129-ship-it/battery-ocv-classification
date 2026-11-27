# 배터리 OCV 사전 불량 분류

OCV1~3으로 최종 저전압 판정을 예측하고, 생산일 분할과 임곗값에 따른 미검출·과검을 비교하는 코드입니다.

## 문제와 구현

최종 측정값 OCV4로 정답을 정하는 데이터에서 OCV4를 입력에 포함하면 사전 예측 성능을 평가할 수 없습니다. 입력은 OCV1~3과 차분으로 제한하고, 생산일 또는 그룹으로 학습·검증·평가 자료를 나눕니다. 결측 처리와 표준화는 학습 자료 내부에서 수행합니다.

로지스틱 회귀, 결정나무, 랜덤포레스트, XGBoost를 비교합니다. 검증셋 AP로 모델을 선택하고 검증셋 F1으로 임곗값을 정합니다. 평가 결과에는 Precision, Recall, F1, AP, PR-AUC, 혼동행렬을 저장합니다.

## 구성과 실행

- `src/battery_ocv/`: 특징 생성, 자료 분할, 모델 비교와 평가
- `train_model.py`: 실험 CLI
- `app.py`: Streamlit 분석·판정 화면
- `tests/`: 누수 방지와 분할·지표 회귀 테스트

```bash
python -m venv .venv
source .venv/bin/activate
pip install -e .
python train_model.py /path/to/private/sample.xlsx --header 1 --output results/local
streamlit run app.py
pip install pytest ruff
pytest -q
ruff check src tests app.py train_model.py
```

입력에는 OCV1, OCV2, OCV3, OK_NG와 생산일자가 필요합니다. 원자료와 과거 모델 pickle은 포함하지 않았습니다. 사용 권한이 있는 자료를 로컬에서 준비해야 합니다.

## 검증과 한계

2026-11-26 코드 회귀 테스트 8개와 Ruff 검사가 통과했습니다. 비공개 데이터로 전체 모델 실험을 재실행한 결과는 아닙니다. 기존 Recall 0.98·Precision 0.94는 업무 데이터와 교육용 샘플의 동일성이 확인되지 않아 직접 재현 검증을 보류합니다.

측정 간격과 단위가 확인되지 않은 차분을 물리적 변화율이나 가속도로 해석하지 않습니다. Precision·Recall은 공정의 6σ/5σ 보증과 다른 지표입니다. σ 수준에는 결함 기회 정의, 허용 유출률, 대표성 있는 대규모 검증이 필요합니다.

생성된 그래프와 개별 사례의 공개 여부는 입력 자료에 따라 검토해야 합니다. 이번 코드 배포에는 실제 데이터에서 생성한 그림·측정값을 포함하지 않았습니다.
