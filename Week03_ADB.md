# Week 3: ADB (Android Debug Bridge) 핵심 요약

## 1. ADB 개요 및 구성

### ADB 위치
- Android SDK 폴더 내 `platform-tools` 하위 폴더에 포함됨

### ADB 3요소 (3 element)
1. **클라이언트 (Client)** - 명령어를 입력하는 터미널
2. **서버 (Server)** - PC와 기기 간 통신 관리
3. **데몬 (Daemon)** - 기기 내부에서 실행되는 프로세스

---

## 2. 장치 연결 및 접속 (Connection)

```bash
# 연결된 장치 목록 확인
adb devices

# 실제 장치 또는 에뮬레이터 쉘(Shell) 접속 (나갈 때는 exit 입력)
adb shell

# 시리얼 번호를 지정하여 특정 장치에 접속
adb -s <serial> shell

# 네트워크(IP)를 통해 ADB를 장치(주로 VM)에 연결
adb connect <ip>

# TCP/UDP 연결 상태 및 포트 확인
netstat -tulpen
```

---

## 3. 네트워크 및 포트 포워딩 (Networking)

```bash
# 포트 포워딩 (Forwarding)
# PC(Host)의 포트로 들어오는 요청을 장치(Device)의 포트로 전달
adb forward tcp:1234 tcp:32154

# 리버스 포워딩 (Reverse)
# 장치(Device)의 요청을 PC(Host)의 포트로 전달
adb reverse tcp:4321 tcp:8000

# 현재 디렉토리를 웹 서버로 구동 (Python 이용)
python3 -m http.server
```

---

## 4. 파일 전송 (File Transfer)

```bash
# ADB Push (PC → 장치)
# PC(Host)에 있는 파일을 장치(Device)로 복사 (경로는 장치 기준)
adb push test.txt /sdcard/

# ADB Pull (장치 → PC)
# 장치(Device)에 있는 파일을 PC(Host)로 복사 (경로는 장치 기준)
adb pull /sdcard/경로/파일명

# test.txt 파일이 제대로 푸시(Push)되었는지 확인
adb shell ls /sdcard/test.txt
```

---

## 5. 기타 유용한 명령어 (Useful Commands)

```bash
# 실행 중인 ADB 서버 종료
adb kill-server

# 앱 설치 (Installing Apps)
adb install <apk파일>

# 특정 장치(IP: 192.168.32.137)에 spacepeng.apk 설치 예시
adb -s 192.168.32.137 install spacepeng.apk

# 안드로이드 로그 확인 (Logcat)
adb logcat

# 장치의 IP 주소 확인 (Wi-Fi 인터페이스)
adb shell ip -f addr show wlan0

# 버그 리포트 생성 (Bugreport)
adb bugreport <파일명>.zip

# 터치 입력 보내기 (Send Touch)
adb input touch <x좌표> <y좌표>

# IP(Wi-Fi)를 통한 ADB 연결
adb connect <ip>:8888

# 백업 (Backup - 시스템 앱 제외, APK 포함)
adb backup -f all -apk -nosystem
```
