# 고등어 어황 예보 대시보드 (시범운영)

「한눈에 확인하는 AI 기반 고등어 어황 예보 서비스」 시범 화면.
**정식 공표용이 아닌 참고자료**입니다.

## 이 저장소는 배포 전용

`index.html` 은 **직접 편집하지 마십시오.** 원본과 생성기는 아래에 있습니다.

```
LAB/FisheryFC/고등어/어황예측AI/
├── dashboard.html      ← 원본
└── dashboard_data.py   ← 데이터 생성기(DATA 블록 교체)
```

## 갱신 절차 (분기마다)

```bash
cd LAB/FisheryFC/고등어/어획량예측
conda run -n lab python -m engine.pipeline forecast-cal {분기}     # 운영 전망(먼저)
cd ../어황예측AI
conda run -n lab python 분포/sdm_forecast.py {분기}                # 어획 집중도
conda run -n lab python 양/monthly_forecast.py {분기}              # 월별
conda run -n lab python dashboard_data.py {분기}                   # 화면 데이터 + 이 저장소 index.html 갱신
cd /Users/yoon/APP/fishery-forecast && git add -A && git commit -m "..." && git push
```

`dashboard_data.py` 가 이 폴더의 `index.html` 까지 자동으로 갱신합니다.

## 배포

Vercel 정적 배포(빌드 없음). 저장소를 Vercel 프로젝트에 연결하면 push 시 자동 배포됩니다.
