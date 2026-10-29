# BookOasis Tachimanga 확장 저장소

[BookOasis](https://github.com/leeyj/BookOasis_stable) 서버를 Tachimanga(iOS)와 Tachiyomi/Mihon 계열 앱에서 소스로 쓰기 위한 확장 저장소입니다.

## 저장소 추가

앱의 확장 저장소 설정에 아래 URL을 추가합니다.

```
https://raw.githubusercontent.com/taisetna/bookoasis-tachimanga-repo/main/index.min.json
```

추가한 뒤 확장 목록에서 **BookOasis**를 설치합니다.

APK 매니페스트에는 `tachiyomi.extension.nsfw=1`이 들어 있지만, `index.min.json`에는 `"nsfw":0`으로 적었습니다. Tachimanga의 기반인 Suwayomi는 목록의 `nsfw` 값으로 NSFW 여부를 판단하고, NSFW 확장은 앱 설정에 따라 목록에서 숨겨지기 때문입니다. 자체 서버용인 Keiyoushi의 Komga 확장도 SAFE로 분류되어 있습니다.

## 소스 설정

설치하면 소스가 3개(`BookOasis`, `BookOasis (2)`, `BookOasis (3)`) 생깁니다. 서버를 여러 대 쓰거나 일반/성인 라이브러리를 나눠 등록할 때 씁니다. 하나만 쓴다면 첫 번째 소스만 설정하면 됩니다.

| 항목 | 값 |
|---|---|
| 서버 주소 | `http://<서버 IP 또는 도메인>:5930` (끝에 `/` 없이) |
| 아이디 / 비밀번호 | BookOasis 웹 로그인 계정 |
| 라이브러리 종류 | 일반(`/app-opds`), 성인(`/app-opds-adult`), 둘 다. 성인과 둘 다는 admin 계정이 필요합니다. |
| 표시 이름 | 소스 목록에 보일 별칭 (선택) |

설정을 바꾼 뒤에는 앱을 재시작해야 반영됩니다.

## 제약 사항

- ZIP/CBZ 만화만 표시됩니다. EPUB, PDF, TXT는 나오지 않습니다.
- 서버에서 페이지 수를 아직 계산하지 않은 책(`total_pages=0`, 원격 드라이브에서 스캔 전인 책 등)은 열리지 않습니다. 웹 뷰어로 한 번 열거나 Lazy-Scanner가 처리한 뒤에 읽을 수 있습니다.
- BookOasis 서버의 `/app-opds/api/media/progress` 계열 API는 현재 읽은 위치를 계정과 관계없이 `user_id=1`에 기록합니다. 여러 계정으로 쓰면 진행률이 섞입니다.
- 인증이 Basic Auth이므로 외부망에서 접속할 때는 HTTPS 리버스 프록시를 권장합니다.

## 구성

```
index.min.json   확장 목록 (Keiyoushi 형식)
repo.json        저장소 메타데이터와 서명 지문
apk/             확장 APK
icon/            확장 아이콘 (<pkg>.png)
```

## 확장 정보

| 항목 | 값 |
|---|---|
| 패키지 | `eu.kanade.tachiyomi.extension.all.bookoasis` |
| 버전 | 1.4.34 (versionCode 34, extension lib 1.4) |
| 소스 ID | 8820010100100100, 8820010100100101, 8820010100100102 |
| 서명 | Android Debug 인증서, SHA-256 `887ca73ea517eff6873520f71522e8a7513bce2861f683f5eddc7535199c3af0` |
| 출처 | BookOasis 저장소의 `tools/tachiyomi-all.bookoasis-v1.4.34-debug.apk` (제작: hejerd), 파일 이름만 바꾸고 내용은 그대로 |

## 확장 업데이트 절차

1. 새 APK를 `apk/tachiyomi-all.bookoasis-v<버전>.apk`로 넣고 이전 APK는 지웁니다.
2. `aapt dump badging <apk>`로 `versionCode`와 `versionName`을 확인해 `index.min.json`의 `apk`, `code`, `version`을 고칩니다.
3. 서명 키가 바뀌었다면 `repo.json`의 `signingKeyFingerprint`도 새 인증서의 SHA-256으로 바꿉니다.
   ```
   unzip -p <apk> META-INF/CERT.RSA | openssl pkcs7 -inform DER -print_certs | openssl x509 -noout -fingerprint -sha256
   ```
   서명 키가 다르면 이미 설치된 확장 위에 업데이트할 수 없으므로, 앱에서 확장을 지우고 다시 설치해야 합니다.
