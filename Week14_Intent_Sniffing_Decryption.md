# Week 14: Intent Sniffing & Password Decryption

## PART 1: 인텐트 스니핑 (Intent Sniffing) 공격

### 1. 개념 및 원리

#### 정의
안드로이드 앱이 수신자를 지정하지 않고 보내는 '암시적 브로드캐스트(Implicit Broadcast)'를 공격자가 가로채는 기법

#### 원리
공격자 앱이 피해자 앱과 동일한 액션(Action)을 리시버에 등록하면 방송을 동시에 수신할 수 있음

#### 위험성
방송에 포함된 민감한 정보(비밀번호, 인증 정보 등)가 노출됨

---

### 2. 공격 준비: 악성 앱 제작 (InsecureBankSniffer)

#### (1) 프로젝트 생성
- **앱 이름**: InsecureBankSniffer
- **패키지명**: edu.univ.insecurebanksniffer

#### (2) MainActivity.java (함정 설치)
- 앱이 활성화될 때(`onResume`) 동적으로 리시버를 등록함
- 필터 설정: `filter.addAction("theBroadcast");`
- **의미**: "theBroadcast"라는 신호가 오면 낚아채겠다

#### (3) SniffReceiver.java (정보 탈취)
방송을 수신했을 때 실행되는 코드

**인텐트에서 데이터 추출**:
```java
String phoneNumber = intent.getStringExtra("phonenumber");
String newPass = intent.getStringExtra("newpass");
```

탈취한 정보를 로그(Log)나 토스트(Toast) 메시지로 출력함

---

### 3. 공격 실행 명령어 (Broadcast Trigger)

공격자 앱을 실행해둔 상태에서, 터미널에 아래 명령어를 입력하여 강제로 방송을 발생시킴

```bash
am broadcast -n com.android.insecurebankv2/com.android.insecurebankv2.MyBroadCastReceiver -a theBroadcast --es "phonenumber" "8502740" --es "newpass" "Hacked@123$"
```

**결과**: 공격자 앱 화면에 "Sniffed Intent => phone: 8502740, newpass: Hacked@123$" 메시지가 뜨면 성공

---

## PART 2: 패스워드 복호화 (Password Decryption)

### 1. 데이터 저장 위치 및 파일 확인

앱은 SharedPreferences를 사용하여 데이터를 XML 파일로 저장함

**파일 경로**:
```
/data/data/com.android.insecurebankv2/shared_prefs/
```

**파일 이름**: `mySharedPreferences.xml`

---

### 2. 저장된 데이터 분석 (Source Code: LoginActivity.java)

- **EncryptedUsername**: 단순히 Base64로 인코딩되어 저장됨 (암호화 아님)
- **superSecurePassword**: AES 알고리즘으로 암호화되어 저장됨 (복호화 필요)

---

### 3. 암호화 키 찾기 (Source Code: CryptoClass.java)

소스코드를 분석하여 하드코딩된 키와 IV를 찾아냄

#### (1) Key (비밀키)
```
"This is the super secret key 123"
```

#### (2) IV (초기화 벡터)
```
{0, 0, 0, ...}  # 16바이트 모두 0
```

#### (3) 알고리즘
```
AES/CBC/PKCS5Padding
```

---

### 4. CyberChef를 이용한 복호화 실습

**사이트**: http://gchq.github.io/CyberChef

#### [CASE 1] 아이디 복구 (Username)

- **Input**: XML에서 복사한 EncryptedUsername 값 (예: `ZGluZXNo`)
- **Recipe**: From Base64
- **Output**: 평문 아이디 (예: `dinesh`)

---

#### [CASE 2] 비밀번호 복구 (Password)

- **Input**: XML에서 복사한 superSecurePassword 값 (예: `j63Qw...`)

**Recipe 설정**:

1. **From Base64**

2. **AES Decrypt**
   - **Key**: `This is the super secret key 123` (Encoding: UTF8)
   - **IV**: `00000000000000000000000000000000` (Encoding: Hex)
   - **Mode**: CBC
   - **Input**: Raw / **Output**: Raw

- **Output**: 평문 비밀번호 (예: `Test1234$`)

---

## PART 3: 실습 과제 (Assignments)

### 1. Intent Sniffing 과제

InsecureBankSniff 앱을 제작하고, `am broadcast` 명령어로 정보를 가로채는 화면 캡처

### 2. Password Decryption 과제

- 앱에서 비밀번호를 변경한 뒤, 저장된 암호문을 추출
- CyberChef를 사용하여 변경된 비밀번호를 평문으로 복구하는 과정 캡처
