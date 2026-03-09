# WeatherCat
-> 사용자의 현재 위치를 기반으로 날씨를 분석하여 고양이의 상태를 보여주는 웹 서비스입니다.
## 주요 기능
1. 실시간 날씨 수집: OpenWeatherMap API를 활용한 정확한 가상 데이터 조회
2. 위치 기반 서비스: 브라우저 Geolocation API를 통한 자동 지역 감지
3. 감성 UI: 날씨 상태(기온, 하늘 상태)에 따른 동적 고양이 캐릭터 노출

## Tech Stack
Backend : FastAPI, Uvicorn
Frontend : Vanilla Js , HTML 5, CSS3
Library : Requests, CORSMIddleware


날씨 어플/
├── venv/             # 가상환경
├── app/
│   └── main.py       # FastAPI 백엔드 로직
└── index.html        # 프론트엔드 화면
