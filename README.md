# 고등어 어황 예보 대시보드 (시범운영)

「한눈에 확인하는 AI 기반 고등어 어황 예보 서비스」 시범 화면.
**정식 공표용이 아닌 참고자료**입니다.

## 서비스 고도화 기획

웹앱 배포 위치는 이 저장소에 유지합니다. 모바일 메뉴, 주별 관측 연계,
조업 참고정보, 기관별 협의사항 및 체장 표본설계는 아래 기준 문서에서 관리합니다.

- [어황예보서비스 고도화 기획 — 로컬 작업공간 문서](/Users/yoon/FCST/Chubmackerel/docs/어황예보서비스_고도화_기획.md)

위 링크는 로컬 경로이며 GitHub·배포 사이트에서는 열리지 않습니다.
기획 원본은 `~/FCST/Chubmackerel/docs/어황예보서비스_고도화_기획.md` 한 곳에서 갱신합니다.
`전망 / 현황` 두 메뉴로 정리하고 현황에 주간 실적과 조업 참고정보를 통합했습니다.
모바일에서는 지도 격자를 누르면 상세 패널이 열립니다. 현황 안에서 `어획 / 체장`을 전환합니다.
- 어획 진행: 90~100% 근접, 100% 초과~150% 초과, 150% 초과 큰 폭 초과.
- 분산조업 후보: 분기 중반 이후 누적 50% 이하이면서 선택 주간 말 기준 최근 7일 안에 양수 어획이 확인된 격자.
- 체장: 과거 5개년 같은 분기 FL 표본의 25·75백분위로 구간을 구분. 화면에는 2026Q3 기준 `27cm 미만 / 27~32.4cm / 32.4cm 초과`를 표시하며 지도 숫자는 관측 중앙체장(cm).
- 측정 1개부터 조사일과 함께 표시하며 과거 관측은 점선 처리. 체장 구분은 성숙 판정과 별개입니다.

지도와 해역 상세의 `표시 기준·판단 근거` 버튼에서 색 구간·분포 미표시 원인·운영 임계값·체장 기준 자료를 확인합니다. 상세에는 선택 해역의 값과 판정 근거를 병기합니다.

표시 기준은 시범 운영 설정이며 조업 제한·허용을 뜻하지 않습니다. 조업당 30마리는 검증 전 잠정 수집안입니다.

## 이 저장소는 배포 전용

`index.html` 은 **직접 편집하지 마십시오.** 원본과 생성기는 아래에 있습니다.

```
~/FCST/Chubmackerel/어황예측AI/
├── dashboard.html      ← 원본
├── dashboard_data.py   ← 데이터 생성기(DATA 블록 교체)
└── dashboard_observations.py ← FisheryDB 주별 관측 집계
```

## 주별 관측 갱신 (전망값 보존)

FisheryDB 가공이 완료되면 현재 화면의 분기를 기준으로 실행합니다.

```bash
cd ~/FCST/Chubmackerel/어황예측AI
conda run -n lab python dashboard_data.py --observations-only
```

이 명령은 기존 전망 DATA를 유지하고 주별 관측만 갱신한 뒤 배포용 `index.html`을 동기화합니다.
진행률의 분모는 저장된 권역 전망과 분포 배분으로 내부 산출하며, 기존 지도와 불일치하면 중단합니다. 모델을 재실행하지 않습니다.
원본 CSV의 선박 식별자·개체별 측정값·격자별 절대 어획량은 내보내지 않습니다.
관측 주간은 월요일~일요일이며 분기 경계에서는 분기 내 날짜만 집계합니다.
관측이 없는 주·격자는 어획 0 또는 미성어 위험 낮음으로 처리하지 않습니다.
Git push와 실제 Vercel 배포는 별도 단계입니다.

## 갱신 절차 (분기마다)

```bash
cd ~/FCST/Chubmackerel/어획량예측
conda run -n lab python -m engine.pipeline forecast-cal {분기}     # 운영 전망(먼저)
cd ../어황예측AI
conda run -n lab python 분포/sdm_forecast.py {분기}                # 어획 집중도
conda run -n lab python 양/monthly_forecast.py {분기}              # 월별
conda run -n lab python dashboard_data.py {분기}                   # 화면 데이터 + 이 저장소 index.html 갱신
cd /Users/yoon/APP/fishery-forecast && git add -A && git commit -m "..." && git push
```

`dashboard_data.py` 가 이 폴더의 `index.html` 까지 자동으로 갱신합니다.

체급 카드는 같은 분기의 `체급예측/예보/outputs/size_forecast_{분기}.csv`가 있고 동해·서해·남해
3권역이 모두 유효할 때만 자동 표시됩니다. 현재 Q3 화면에 Q4 체급값을 섞지 않으며, 파일·권역·분기가
맞지 않으면 카드가 숨겨집니다. `index.html`의 `sizeForecast`를 손으로 고치지 마십시오.

## 배포

Vercel 정적 배포(빌드 없음). 저장소를 Vercel 프로젝트에 연결하면 push 시 자동 배포됩니다.
