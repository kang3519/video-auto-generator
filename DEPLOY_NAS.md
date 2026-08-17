# Synology NAS 배포 가이드

이 문서는 `webssd.duckdns.org` NAS에 MoneyPrinterTurbo를 Docker로 띄우는 절차입니다.
아래 명령은 **NAS에 SSH로 접속한 상태(PC 터미널에서)**에서 실행합니다.

## 0. 사전 준비

- Synology DSM → 제어판 → 터미널 및 SNMP → **SSH 서비스 활성화** 체크
- Synology DSM → 패키지 센터 → **Container Manager**(구 Docker) 설치

## 1. SSH 접속 (PC 터미널에서)

```bash
ssh kang3519@webssd.duckdns.org -p 2222
```

## 2. 저장소 클론

```bash
cd /volume1/docker   # 원하는 공유폴더 경로로 변경 가능
git clone https://github.com/kang3519/video-auto-generator.git moneyprinterturbo
cd moneyprinterturbo
```

## 3. 설정 파일 준비 (API 키는 여기서만 입력, git에는 안 올라감)

```bash
cp config.example.toml config.toml
nano config.toml
```

아래 값들을 채웁니다:

```toml
llm_provider = "gemini"
```

```toml
# Google Gemini
gemini_api_key = "여기에_본인_Gemini_API_키"
gemini_model_name = "gemini-flash-latest"
```

```toml
# Pexels
pexels_api_keys = ["여기에_본인_Pexels_API_키"]
```

저장(`Ctrl+O` → Enter) 후 종료(`Ctrl+X`).

> NAS는 일반 인터넷 환경이라 **Edge TTS(무료 음성)와 Pexels 다운로드가 정상 작동**합니다.
> 샌드박스에서 겪었던 WebSocket 차단, 프록시 차단 문제가 없습니다.

## 4. 실행

```bash
docker compose -f docker-compose.nas.yml up -d
```

## 5. 접속 확인

같은 Wi-Fi(LAN)에 연결된 다른 기기(휴대폰, PC)에서:

- **WebUI**: `http://<NAS의 내부IP>:8501`
- **API**: `http://<NAS의 내부IP>:8080`

NAS 내부 IP는 DSM 우측 상단 또는 `ip addr` 명령으로 확인할 수 있습니다.

## 6. 로그 확인 / 중지

```bash
docker compose -f docker-compose.nas.yml logs -f      # 로그 실시간 확인
docker compose -f docker-compose.nas.yml down          # 중지
```

## 7. 업데이트 (나중에 코드가 바뀌었을 때)

```bash
git pull
docker compose -f docker-compose.nas.yml pull   # 최신 이미지 받기
docker compose -f docker-compose.nas.yml up -d
```

## 참고: 외부(인터넷)에서 접속하고 싶다면

- Synology DSM → 제어판 → 외부 액세스 → **역방향 프록시**에서 `webssd.duckdns.org`의
  특정 서브도메인(예: `mpt.webssd.duckdns.org`)을 `localhost:8501`로 연결
- 반드시 DSM 자체의 HTTPS 인증서(Let's Encrypt 무료 인증서 발급 가능)를 적용해서
  평문 HTTP로 외부에 노출되지 않도록 할 것
- 이 단계는 보안 설정이 꽤 중요하니, 진행하고 싶으시면 별도로 말씀해주세요
