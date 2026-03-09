# 🐾 WeatherCat: 위치 기반 고양이 날씨 서비스

> **사용자의 현재 위치를 기반으로 날씨를 분석하여 귀여운 고양이의 상태를 보여주는 웹 서비스입니다.**

## 🌟 주요 기능
* **실시간 날씨 수집**: OpenWeatherMap API를 활용한 정확한 기상 데이터 조회
* **위치 기반 서비스**: 브라우저 Geolocation API를 통한 자동 지역 감지
* **감성 UI**: 날씨 상태(기온, 하늘 상태)에 따른 동적 고양이 캐릭터 노출

## 🛠 Tech Stack
* **Backend**: FastAPI, Uvicorn
* **Frontend**: Vanilla JS, HTML5, CSS3
* **Library**: Requests, CORSMiddleware

## 📂 프로젝트 구조

#프론트엔드
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Weather Cat - 내 주변 고양이 날씨</title>
    <style>
        body { font-family: 'Malgun Gothic', sans-serif; display: flex; justify-content: center; align-items: center; height: 100vh; background-color: #fdf6e3; margin: 0; }
        .card { background: white; padding: 30px; border-radius: 30px; box-shadow: 0 15px 35px rgba(0,0,0,0.1); text-align: center; width: 320px; border: 4px solid #ffcc00; }
        .emoji { font-size: 100px; margin-bottom: 15px; }
        h1 { color: #444; font-size: 28px; margin: 10px 0; }
        .status-text { color: #666; font-size: 18px; font-weight: bold; margin: 15px 0; background: #fff9db; padding: 10px; border-radius: 15px; min-height: 50px; display: flex; align-items: center; justify-content: center; }
        .temp { font-size: 24px; color: #ff6b6b; font-weight: bold; }
        button { background: #ffcc00; color: #444; border: none; padding: 12px 25px; border-radius: 50px; cursor: pointer; margin-top: 20px; font-weight: bold; font-size: 16px; transition: 0.3s; }
        button:hover { background: #ffa500; transform: scale(1.05); }
    </style>
</head>
<body>
    <div class="card">
        <div id="emoji" class="emoji">🐈</div>
        <h1 id="city">위치 파악 중...</h1>
        <div id="status" class="status-text">고양이가 날씨를 확인하러 갔어요.</div>
        <p>현재 기온: <span id="temp" class="temp">-</span>°C</p>
        <button onclick="getLocationAndWeather()">내 위치 날씨 새로고침 🐾</button>
    </div>

    <script>
        // 날씨 정보를 가져와서 화면을 업데이트하는 함수
        async function fetchWeather(lat = null, lon = null, city = "Asan") {
            try {
                let url = 'http://127.0.0.1:8000/weather-cat';
                
                // 좌표가 있으면 좌표 기반, 없으면 도시 기반으로 쿼리스트링 생성
                if (lat && lon) {
                    url += `?lat=${lat}&lon=${lon}`;
                } else {
                    url += `?city=${city}`;
                }

                const response = await fetch(url);
                const data = await response.json();

                if (data.error) throw new Error(data.message);

                // DOM 요소 업데이트
                document.getElementById('emoji').innerText = data.emoji;
                document.getElementById('city').innerText = data.city || "알 수 없는 지역";
                document.getElementById('status').innerText = data.cat_status;
                document.getElementById('temp').innerText = data.temp;

            } catch (error) {
                console.error('날씨 호출 에러:', error);
                document.getElementById('status').innerText = '데이터를 가져오지 못했어요 😿';
            }
        }

        // 브라우저 지오로케이션으로 좌표를 가져오는 함수
        function getLocationAndWeather() {
            document.getElementById('status').innerText = '위치를 찾는 중...';
            
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(
                    (position) => {
                        const lat = position.coords.latitude;
                        const lon = position.coords.longitude;
                        fetchWeather(lat, lon);
                    },
                    (error) => {
                        console.error("위치 권한 거부 또는 오류:", error);
                        document.getElementById('status').innerText = '위치 권한이 거부되어 기본값(아산)을 보여줍니다.';
                        fetchWeather(); // 권한 거부 시 기본값으로 호출
                    }
                );
            } else {
                alert("이 브라우저는 위치 정보를 지원하지 않습니다.");
                fetchWeather();
            }
        }

        // 페이지 로드 시 즉시 실행
        window.onload = getLocationAndWeather;
    </script>
</body>
</html>
# 백엔드 main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
import requests

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"], 
    allow_methods=["*"],
    allow_headers=["*"],
)

API_KEY = "631d69f32edb781f331d64c5c7146ac0"

@app.get("/weather-cat")
def get_weather_info(city: str = None, lat: float = None, lon: float = None):
    # 1. URL 결정 로직 (좌표 우선 -> 도시 이름 -> 기본값 아산 순서)
    if lat is not None and lon is not None:
        # 좌표 기반 호출 주소 (lat, lon 파라미터 사용)
        url = f"https://api.openweathermap.org/data/2.5/weather?lat={lat}&lon={lon}&appid={API_KEY}&units=metric"
    else:
        # 도시 이름 기반 호출 주소
        target_city = city if city else "Asan"
        url = f"https://api.openweathermap.org/data/2.5/weather?q={target_city}&appid={API_KEY}&units=metric"
    
    # 2. 날씨 데이터 가져오기
    response = requests.get(url)
    data = response.json()
    
    # API 응답 성공 확인 (200 OK)
    if response.status_code != 200:
        return {"error": "데이터를 가져올 수 없습니다.", "message": data.get("message")}

    # 3. 데이터에서 필요한 정보 추출
    temp = data['main']['temp']
    weather = data['weather'][0]['main']
    # API가 돌려주는 실제 지역 이름 (이걸 써야 null이 안 나옵니다)
    display_name = data.get('name') 
    
    # 4. 날씨에 따른 고양이 상태 결정
    if temp > 28:
        status, emoji = "더워서 녹아버린 액체 고양이", "🫠"
    elif "Cloud" in weather:
        status, emoji = "구름 구경하는 멍한 고양이", "☁️"
    elif "Rain" in weather:
        status, emoji = "창밖 비 구경하는 감성 고양이", "☔"
    else:
        status, emoji = "기분 좋아서 꾹꾹이 하는 고양이", "🐾"

    # 5. 결과 반환
    return {
        "city": display_name,
        "temp": temp,
        "weather": weather,
        "cat_status": status,
        "emoji": emoji
    }

