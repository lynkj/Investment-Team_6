# Module 1: 투자 유니버스와 데이터

투자론연구 팀과제 — 체계적 자산운용 시스템의 설계, 구현 및 백테스팅

---

## 개요

백테스트 기간의 각 시점마다 **그 시점에 실제로 투자 가능했던 종목**으로 유니버스를 구성하고,
이후 모듈(2~9)이 공통으로 사용할 데이터를 생성한다.

---

## 실행 방법

```bash
pip install pandas numpy pyarrow requests finance-datareader
```

`Module1_DataCollection.ipynb`를 순서대로 실행하면 `data/processed/`에 산출물이 저장된다.
첫 실행 시 KRX 시세 파일(약 200MB)을 다운로드하므로 시간이 걸린다.
이후 실행부터는 `data/raw/`에 캐시된 파일을 재사용한다.

---

## 산출물 (`data/processed/`)

| 파일 | 설명 |
|---|---|
| `universe_monthly.parquet` | 월별 유니버스 (72회 × 200종목). `effective_date` 기준으로 이후 모듈과 결합 |
| `kospi_daily.parquet` | 편입 이력 336종목의 일별 시세·수익률 (퇴출 종목 포함) |
| `security_master.csv` | 종목별 첫/마지막 거래일, 기간 내 퇴출 여부 |
| `filter_log.csv` | 72회 유니버스 산정 시 필터 단계별 종목 수 기록 |
| `benchmark_daily.parquet` | KOSPI200 일별 종가 (FinanceDataReader) |
| `daily_close_wide.csv` | 종가 표 (행=거래일, 열=종목명) — 엑셀 확인용 |
| `daily_return_wide.csv` | 수익률 표 (행=거래일, 열=종목명) — 엑셀 확인용 |

---

## 주요 설정

| 항목 | 값 |
|---|---|
| 데이터 적재 기간 | 2018.01 ~ 2025.12 |
| 백테스트 기간 | 2020.01 ~ 2025.12 |
| 유니버스 크기 | 시총 상위 200종목 |
| 리밸런싱 주기 | 월말 재산정, 다음 거래일부터 적용 |
| 최소 상장일수 | 252거래일 (약 1년) |
| 거래정지 기준 | 최근 20거래일 중 정지 없음 |
| 유동성 기준 | 최근 60거래일 평균 거래대금 하위 20% 제외 |

---

## 편향 통제

**생존편향 (Elton, Gruber & Blake, 1996)**
매월 그 시점에 상장돼 있던 종목으로 유니버스를 다시 산정한다.
기간 중 상장폐지·합병된 종목 9개(오렌지라이프, 메리츠화재 등)도 퇴출 직전까지 포함된다.

**전진편향 (Harvey & Liu, 2015)**
모든 필터 지표는 t일 종가까지의 과거 데이터로만 계산하고, 유니버스는 t+1일부터 적용한다.
수정주가(미래의 분할·배당을 소급 반영)는 필터에 사용하지 않는다.

---

## 데이터 출처

- **KRX 일별 시세**: [FinanceData/marcap](https://github.com/FinanceData/marcap) — 상장폐지 종목 포함, 연도별 parquet
- **KOSPI200 지수**: [FinanceDataReader](https://github.com/FinanceData/FinanceDataReader)

---

## 참고 문헌

- Elton, E., Gruber, M. and Blake, C. (1996). Survivorship Bias and Mutual Fund Performance. *Review of Financial Studies*, 9(4).
- Harvey, C.R. and Liu, Y. (2015). Backtesting. *Journal of Portfolio Management*, 42(1).
- Korajczyk, R. and Sadka, R. (2004). Are Momentum Profits Robust to Trading Costs? *Journal of Finance*, 59(3).
