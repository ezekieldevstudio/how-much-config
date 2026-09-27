# HowMuch 공개 운영 설정

앱: `C:/dev/EzekielDev/how-much-app` — [앱 README](../how-much-app/README.md). 이 저장소는 GitHub Pages 정적 파일이며 서버·DB·Secret 저장소가 아니다.

## 공개 파일

기본 URL: <https://ezekieldevstudio.github.io/how-much-config/>

| 파일 / 공개 경로 | 역할 |
| --- | --- |
| `config.json` | `schemaVersion: 1`, 기능 flags, app 메타데이터, `content.<종류>.version/file` 참조 |
| `notices.json` | `{ version, items }`. 날짜는 `publishedAt`; enabled=false 제외, pinned 우선, 날짜 최신순 |
| `faq.json` | `{ version, items }`. enabled=false 제외, order 오름차순 |
| `recommendations.json` | `{ version, items }`. 현재 version=0, items=[]; 쿠팡/추천 flags 기본 OFF |
| `privacy-policy.html` | 외부 브라우저에서 여는 개인정보처리방침 |

각 공개 주소는 기본 URL 뒤에 파일명을 붙인다. 예: <https://ezekieldevstudio.github.io/how-much-config/config.json>.
현재 notices=1, faq=1, recommendations=0이며 config 참조도 같다. 파일명은 앱이 고정하며 임의 URL을 config에 넣는다고 새 파일을 읽지 않는다.

## 변경 / 버전 규칙

1. 바꿀 콘텐츠 JSON을 수정한다. id는 해당 파일 안에서 고유하고 비어 있지 않아야 한다. 기존 항목 수정은 id를 유지한다.
2. 그 JSON의 최상위 `version`을 지금까지 배포한 값보다 큰 정수로 올린다.
3. `config.json`의 같은 `content.<종류>.version`을 정확히 일치시킨다. 다른 콘텐츠 version은 불필요하게 올리지 않는다. 공지 날짜는 `publishedAt`을 사용한다.
4. flags/app 메타데이터만 변경하면 콘텐츠 version 증가는 필요 없다. `schemaVersion`은 배포 횟수가 아니라 스키마 버전이며 1을 유지한다. `updatedAt`은 앱의 갱신 판정 기준이 아니다.
5. 개인정보처리방침 HTML은 JSON 콘텐츠 version 비교 대상이 아니다.

앱은 유효한 config를 갱신한 후 **원격 콘텐츠 version > 캐시 version**인 파일만 다운로드한다. 파일 내부 version과 config 참조가 다르거나 JSON/항목 검증에 실패하면 이전 콘텐츠를 유지한다. 같은 version의 본문만 수정하면 기존 사용자는 갱신받지 않는다.

## 캐시 / 적용 범위

- 시작할 때 로컬 캐시부터 사용한다. 앱 시작/foreground 전환에서 마지막 확인 시도 후 24시간이 지난 경우만 원격 확인한다. 24시간마다 백그라운드에서 자동 요청하는 타이머는 없다.
- **실패한 요청도 24시간 재확인 제한에 포함**된다. 서버 수정 후 앱 재시작만으로 즉시 재시도되지 않는다. 실패 시 이전 캐시, 캐시가 없으면 앱 기본값을 사용한다.
- config만 먼저 반영되고 일부 콘텐츠 갱신은 실패할 수 있다. 다음 허용된 확인에서 다시 시도한다. 긴급 flags OFF도 즉시 모든 기기에 전파되지는 않는다.
- `maintenanceMode`, `minVersion`, `latestVersion`은 현재 상태에 **저장만** 된다. 점검 차단, 강제 업데이트, 스토어 이동 enforcement는 없다.
- 개인정보처리방침 URL은 **앱의 HTTPS `expo.extra.privacyPolicyUrl` > remote `app.privacyPolicyUrl` > 앱 기본값** 순서다. 현재 앱 설정이 우선하므로 remote URL만 바꿔도 설치 앱의 연결 주소는 바뀌지 않는다. 동일 URL의 HTML 본문 교체와 URL 변경을 구분한다.
- 쿠팡/추천 flag와 AdMob flag는 독립이다. 추천 ON은 실제 화면 연결/데이터 검증도 별도로 확인한다. 현재 기록 리스트는 AdMob을 사용하므로 JSON flags만으로 추천 카드 UI가 새로 생긴다고 가정하지 않는다.

## 배포 / 복구

1. 기존 미커밋 변경을 확인하고 필요한 JSON/HTML만 편집한다. JSON parse, 중복 id, config와 콘텐츠 version 일치를 검사한다. 앱 저장소에서 `node --test tests/remoteConfig.test.cjs`와 전체 테스트를 실행한다.
2. 콘텐츠와 config 참조를 같은 커밋으로 검토하고 배포 브랜치에 push한다. 현재 로컬 브랜치는 main이다. GitHub Settings → Pages의 실제 Source/브랜치/폴더 또는 배포 workflow를 먼저 확인한다. 로컬 main 존재만으로 Pages가 main/root 배포라고 단정하지 않는다.
3. GitHub Pages 배포 완료를 확인하고 공개 URL들의 HTTP 응답·JSON version·HTML을 확인한다. 분리 배포가 필요하면 콘텐츠 먼저, config 참조는 나중에 공개한다. 캐시/배포 중 일시 불일치로 실패한 앱은 다음 24시간 허용 시점까지 지연될 수 있다.
4. **콘텐츠 롤백은 과거 본문을 더 높은 version으로 재배포**한다. 예: version 2가 잘못됐으면 이전 내용을 version 3으로 만들고 config 참조도 3으로 올린다. 단순 git revert로 version을 1로 내리면 이미 2를 받은 앱이 수용하지 않는다.
5. flags만 복구하는 경우 이전 boolean을 다시 배포한다. HTML만 복구하면 이전 본문을 같은 주소로 재배포하고 Pages/브라우저 캐시를 확인한다. 과거 커밋 전체를 되돌려 콘텐츠 version까지 낮추지 않는다.
6. 복구 결과는 공개 파일과 실제 앱 양쪽에서 확인한다. 운영 기기의 앱 데이터 삭제/재설치를 캐시 우회 기본 절차로 사용하지 않는다. 기록이 유실될 수 있다.

이번 문서 정리는 로컬 코드/JSON 기준이며 Pages 설정 변경이나 실제 publish를 수행하지 않았다.

## 공개 설정 / Secret

현재 루트 `.env*` 파일과 이를 소비하는 빌드 도구가 없다. `.env` 하나는 공개 클라이언트 설정용으로 커밋 가능하지만 정적 JSON에 자동 반영되지 않는다. `.env.*`는 제외하고 `.env.example`만 공개 예시로 허용한다.

`keys/`, `secrets/`, `*.jks`, `*.keystore`, `signing.properties`, `credentials.json`은 제외한다. 앱 서명키·비밀번호·API Secret·사용자 사진/진단 ZIP을 공개 저장소/Pages에 넣지 않는다. ignore는 추적 중인 파일과 과거 커밋을 제거하지 않는다.
