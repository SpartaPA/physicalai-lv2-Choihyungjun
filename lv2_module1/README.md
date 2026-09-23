## 장비 · 환경

| 구분 | 내용 |
|---|---|
| 호스트 | 라즈베리파이 (hostname: pa777) |
| OS | Ubuntu Server 22.04.5 LTS (Jammy) |
| 아키텍처 | aarch64 (ARM64) |
| 제어기 | OpenCR 1.0 (Board Name: OpenCR R1.0) |
| 모터 | 다이나믹셀 XM430-W210 (모델 1030) |
| 모터 설정 | ID 1, 1 Mbps, Protocol 2.0 |
| 접속 포트 | /dev/ttyACM0 (권한: dialout 그룹) |
| 접속 방식 | PC → 라즈베리파이 SSH |
| 예제 | opencr_position_p (P 위치 제어) |

## 실행 방법

모든 명령은 SSH로 접속한 라즈베리파이에서 실행합니다.

1. 환경·포트 확인
   ```bash
    hostname
    cat /etc/os-release
    uname -m
    df -h "$HOME"
    id -nG                 # dialout 포함 확인
    ls -l /dev/ttyACM*     # OpenCR 포트 확인
    ```

2. 펌웨어 빌드, 업로드

    ```bash 
    # opencr_position_p 컴파일 (arduino-cli)
    BASE="$HOME/pa-opencr-build"
    set -o pipefail
    "$BASE/bin/arduino-cli" --config-file "$BASE/arduino-cli.yaml" \
    compile --fqbn ROBOTIS:OpenCR:OpenCR --jobs 1 \
    --output-dir "$BASE/output" \
    "$BASE/sketches/opencr_position_p" 2>&1 | tee "$BASE/build.log"
    # 업로드 (opencr_ld 업로더)
    PORT=/dev/ttyACM0
    UPLOADER="$BASE/uploader-src/arduino/opencr_develop/opencr_ld/opencr_ld"
    set -o pipefail
    "$UPLOADER" "$PORT" 115200 \
    "$BASE/output/opencr_position_p.ino.bin" 1 \
    2>&1 | tee "$BASE/upload.log"
    # 출력에서 CRC OK / [OK] Download 확인
    ```

3. 목표 입력, 응답 기록
    ```bash
    python3 -m serial.tools.miniterm /dev/ttyACM0 115200 --eol LF
    # 모니터 안에서: s <Kp> <speed_deg_s|max> <angle_deg> 
    ```

## 결과 파일 위치
| 파일 | 내용 |
|---|---|
| results/환경확인.png | 호스트명·Ubuntu 버전·아키텍처·포트·dialout 권한 |
| results/build&upload.png | 펌웨어 빌드 및 업로드(CRC OK / [OK] Download) |
| results/실행.png | 목표 입력 후 측정 로그 (실행 A) |
| report.md | 문제별 설정·증거·해석 |