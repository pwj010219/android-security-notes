# Week 11: Content Providers - Path Traversal Attack

## PART 1: 핵심 이론 - Path Traversal (경로 탐색 공격)

### 1. Path Traversal Vulnerability (경로 탐색 취약점)

#### (1) 정의
입력값 검증이 없을 때, 파일 경로를 조작하여 접근 권한이 없는 민감한 파일이나 디렉토리에 접근하는 공격
- 예: `/etc/passwd`, 다른 앱의 데이터

#### (2) 핵심 기호: "../" (Dot-Dot-Slash)
- **의미**: 상위 디렉토리(부모 폴더)로 이동하라
- **원리**: 현재 위치에서 계속 상위로 올라가서 루트(/)까지 간 뒤, 원하는 비밀 경로로 내려감

---

### 2. Content Provider와 파일 접근

#### (1) 역할
CP는 DB뿐만 아니라 파일(File)도 공유할 수 있음

#### (2) 핵심 명령어
- **`content read-uri ...`**: 파일 읽기 (이번 주 핵심!)
- **`content write-uri ...`**: 파일 쓰기

---

### 3. 10주차 vs 11주차 차이점 ⭐ 시험 출제 포인트

#### 10주차 (Sieve 앱)
- **공격 기호**: "///" (슬래시 여러 개)
- **목표**: 권한 검사 우회 (Bypass) → "문지기 속이기"
- **대상**: 데이터베이스 (DB Table)

#### 11주차 (SimpleMusicPlayer 앱)
- **공격 기호**: "../" (상대 경로)
- **목표**: 파일 시스템 탈출 (Escape) → "담벼락 넘기"
- **대상**: 일반 파일 (xml, txt 등)

---

## PART 2: 공격 실습 - SimpleMusicPlayer 털기

### 1. 공격 준비 (Target 정보 파악)

- **앱 패키지명**: `com.example.simplemusicplayer`
- **목표 파일 위치** (안드로이드 국룰 경로):
  ```
  /data/data/com.example.simplemusicplayer/files/mySecretFile
  ```

---

### 2. 공격 명령어 (Path Traversal)

```bash
adb shell content read --uri content://com.example.simplemusicplayer/../../../../../../../../../data/data/com.example.simplemusicplayer/files/mySecretFile
```

#### [해설]
- **`content://com.example...`**: 택배 수신인 주소 (절대 바꾸면 안 됨! 오타 주의!)
- **`../../../...`**: "뒤로 가!"를 연타해서 루트(/)까지 탈출
- **`/data/data/...`**: 탈출 후 진짜 목표물 경로로 진입

#### [주의사항]
- "없는 폴더"를 경유해서 뒤로 갈 수는 없음 (예: `/fake/../` → 에러)
- 확실하게 존재하는 경로(시스템 폴더 등)를 이용하거나, 그냥 묻지마 탈출(../)이 안전함

---

## PART 3: 취약한 코드 vs 안전한 코드 (Secure Coding)

### 1. 취약한 코드 (Vulnerable)

```java
File file = new File(uri.getPath());
```

**문제점**: `uri.getPath()`는 해커가 입력한 "../" 문자열을 그대로 가져옴. 검증 없이 바로 파일을 열어버림.

---

### 2. 안전한 코드 (Secure) ⭐ 중요

이 3가지 요소가 모두 있어야 완벽한 방어 코드입니다.

#### (1) getCanonicalPath() : 경로 정규화 (가면 벗기기)
- "../" 같은 이동 기호를 다 계산해서 "최종 절대 경로"를 구함

#### (2) startsWith() : 안전 구역 검사 (문지기)
- 정규화된 경로가 앱의 허용된 루트 폴더(rootDir)로 시작하는지 확인함

#### (3) SecurityException : 공격 차단 (강제 퇴장) ⭐ 중요
- 검사 결과가 '거짓(공격)'으로 판명되면, 즉시 예외(에러)를 던져서 앱 실행을 중단시킴
- **핵심**: 공격자를 발견 즉시 쫓아내는 역할

---

### 방어 코드 예시

```java
File rootDir = getContext().getFilesDir(); // 안전 구역 (/data/data/.../files)
String canonicalRoot = rootDir.getCanonicalPath();
String canonicalRequest = requestedFile.getCanonicalPath();

// 1. 검사 (문지기)
if (!canonicalRequestedFilePath.startsWith(canonicalRootPath)) {
    // 2. 차단 (퇴장!)
    throw new SecurityException("Access denied!"); 
}
```

---

## PART 4: 코드 상세 분석 (Q&A 정리)

### 1. startsWith()의 검사 원리

**질문**: "어디까지 똑같아야 통과인가?"

**답변**: "안전 구역(rootDir) 경로 문자열과 토씨 하나 틀리지 않고 똑같아야 함."

#### 예시
- **기준**: `/data/data/app/files`
- **요청**: `/data/data/app/files/music.mp3` ✅ → 앞부분 일치!
- **공격**: `/etc/passwd` ❌ → 앞부분 불일치!
- **꼼수**: `/data/data/app/databases` ❌ → 비슷해 보이지만 'files'가 아니라서 차단!

---

### 2. ParcelFileDescriptor.MODE_READ_ONLY

**의미**: "읽기 전용 모드". 박물관 유리관처럼 눈으로 볼 수만 있고 수정은 불가능함.

**실습 과제**: 해커가 파일에 낙서(쓰기)를 하려면 이 코드를 `MODE_WRITE_ONLY` 또는 `MODE_READ_WRITE`로 바꿔야 함.

---

### 3. e.printStackTrace()

**의미**: 에러(Exception) 발생 시, 그 원인과 발생 위치(족보)를 로그캣에 상세히 출력하라는 명령어.

**역할**: 앱이 강제 종료(펑!) 되지 않도록 try-catch로 감싸고, 개발자에게는 에러 내용을 알려주는 안전장치.

---

### 4. rootDir (안전 구역) 생성 원리

```java
getContext().getFilesDir()
```

**의미**: "안드로이드 시스템아, 나(앱)한테 배정된 전용 사물함 위치가 어디야?"

**결과**: 시스템이 `/data/data/[패키지명]/files` 라는 표준 경로를 자동으로 반환해줌.

---

## PART 5: 실습 과제 가이드 (Lab Task)

### 미션 1: 읽기 공격
- 위 [PART 2]의 공격 명령어를 사용하여 "This is my secret text..." 출력 확인

### 미션 2: 방어 코드 작성
- `MusicFileProvider.java` 파일 수정
- `getCanonicalPath()`, `startsWith()`, `SecurityException` 로직 추가

### 미션 3: 쓰기 공격 (Defacement)
- 소스코드에서 `MODE_READ_ONLY`를 `MODE_WRITE_ONLY`로 변경 후 재빌드
- `adb shell content write-uri ...` 명령어로 본인 학번/이름 덮어쓰기
