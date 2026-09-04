# 대전교통 배차 데이터

배차알리미용 전체 JSON 배포 저장소입니다. 앱 소스·서명키·사용자 알람 예약은 포함하지 않습니다.
현재 데이터: **dataVersion 3 / 2026.09.04-v3**, schemaVersion 1.

## 공개 배포 주소

- [저장소](https://github.com/Garim12/daejeon-transport-data)
- [업데이트 manifest](https://garim12.github.io/daejeon-transport-data/dispatch/manifest.json)
- [최신 전체 시간표 (v3)](https://garim12.github.io/daejeon-transport-data/dispatch/data_v3.json)
- [최초 기준 시간표 (v1)](https://garim12.github.io/daejeon-transport-data/dispatch/data_v1.json)
- [JSON Schema v1](https://garim12.github.io/daejeon-transport-data/dispatch/schema/dispatch_schema_v1.json)

**main /docs**에서 HTTPS로 배포합니다. 2026-09-04에 manifest와 최신 v3 JSON의 HTTP 응답 및 다운로드 SHA-256 일치를 다시 확인했습니다. HTML 홈페이지가 아닌 JSON 피드이므로 루트 주소 대신 위 파일 주소를 사용합니다.

## 파일 구조

```text
docs/dispatch/manifest.json
docs/dispatch/data_vN.json
docs/dispatch/schema/dispatch_schema_v1.json
```

manifest는 버전·변경내용·적용 안내일·데이터 파일명·SHA-256을 담은 작은 색인입니다.
data_vN.json은 해당 버전의 전체 데이터이며 변경분만 담지 않습니다. 이전 파일은 유지합니다.

v1은 2026-08-23 사진을 기준으로 기존 앱에 전사되어 있던 8개 노선·87개 차번을 변환한 데이터입니다.
평일 894건 / 토요일 754건 / 휴일 717건이며 출발지 예외·공동배차 운수회사·선택 차번 제한을 보존합니다.
차번은 실제 차량 번호판이 아닌 노선별 운행 순번입니다. seed의 2000-01-01은 기술적 날짜 하한이고 실제 개통일이 아닙니다.

## 새 버전 만들기 (Windows)

앱 프로젝트의 `tool/dispatch_data/` 도구를 사용합니다. 먼저 이전 전체 JSON을 새 `data_vN.json`으로 복사한 후 수정합니다.

1. dataVersion을 반드시 증가시키고 versionName을 작성합니다. 이미 배포한 번호는 재사용하지 않습니다.
2. 노선/차번 내부 ID를 표시 번호와 분리해 유지합니다. 노선 재배분은 activeUntil/activeFrom으로 처리하고 과거 노선을 삭제하지 않습니다.
3. 시간표 변경은 이전 시간표의 effectiveUntil을 새 적용일 전날로 닫고, 새 ID의 시간표를 추가합니다. 같은 요일 유형의 기간은 겹치지 않아야 합니다.
4. 첫 출발지 예외와 operator/selectable을 빠뜨리지 마세요. 다음날 운행은 dayOffset=1과 00:20 등의 HH:mm 시각을 사용합니다.
   차번을 추가할 때 vehicles에 새 내부 ID를 추가하고 vehicleCount를 저장 차번 수에 맞춥니다. 과거 시간표의 assignments는 그대로 두고 새 기간의 시간표에만 새 차번을 배정합니다. 종료 차번은 삭제하지 않고 새 시간표의 배정에서 제외합니다. 화면의 운행 차번 수는 날짜별 배정/선택 가능 여부로 계산합니다.
5. 앱 프로젝트 폴더에서 다음을 실행합니다. 마지막 인수들은 사용자에게 표시할 실제 변경내용입니다.

```powershell
dart run tool/dispatch_data/validate.dart ../daejeon-transport-data/docs/dispatch/data_v2.json ../daejeon-transport-data/docs/dispatch/data_v1.json
dart run tool/dispatch_data/publish.dart ../daejeon-transport-data/docs/dispatch/data_v2.json 2026-09-15 "실제 변경한 시간표 내용을 여기에 작성"
```

두 번째 명령은 검증, 이전 버전/기간 보존 검사, 원본 파일 바이트의 SHA-256 계산, manifest 갱신을 한 번에 수행합니다. 오류가 있으면 manifest를 교체하지 않습니다. JSON Schema는 편집기에서도 사용할 수 있으나 중복 ID·참조·기간 충돌 등은 앱과 동일한 Dart 검증기로 추가 검사해야 합니다.

manifest effectiveFrom은 안내용 적용일입니다. 실제 시간표는 각 노선·시간표의 적용 기간과 사용자가 선택한 근무 날짜로 결정합니다. 미리 다운로드해도 적용일 전날에는 이전 시간표가 선택되려면 새 파일에 이전 기간 데이터가 포함되어 있어야 합니다.

## 배포

데이터 저장소의 **main**에 새 JSON과 manifest를 같은 commit으로 push합니다. GitHub 저장소 Settings → Pages → Deploy from a branch → main /docs → Save를 사용합니다. 실제 배포가 끝난 뒤 manifest와 데이터 URL의 HTTP 응답과 해시를 확인하세요. 같은 dataVersion 파일의 내용을 바꾸지 마세요.

앱은 약 12시간 간격 WorkManager와 앱 실행/복귀, 사용자의 수동 확인으로 manifest만 조회합니다. 새 파일은 사용자가 변경내용을 보고 승인해야 다운로드·검증·DB Transaction으로 적용됩니다. mandatory는 향후 확장용이며 현재 강제 적용하지 않습니다.

네트워크/서버 장애 시 기존 로컬 데이터를 유지합니다. 업데이트는 이미 예약된 알람을 변경하지 않으며, 새 시간표를 사용하려면 해당 근무를 명시적으로 다시 선택/적용해야 합니다.

앱에는 이전 정상 버전 복구 기능이 있습니다. 복구는 알람 예약을 변경하지 않습니다. 복구한 경우에도 이미 적용했던 dataVersion을 재사용하지 않으며, 관리자는 수정 내용을 포함한 더 큰 번호의 새 버전을 배포해야 합니다.

## 보안/주의

- 이 저장소와 Pages의 배차 데이터는 공개 배포 대상입니다. 개인정보·실제 운전자 명단·API 키·서명키를 추가하지 마세요.
- SHA-256은 전송 파일의 무결성 검사이며, 저장소 계정 자체가 탈취된 경우를 막는 전자서명은 아닙니다. GitHub 계정 2단계 인증과 저장소 변경 권한을 관리하세요.
- 한 날짜에 해당하는 모든 요일 유형과 필요한 운행을 유지하세요. 오류가 있으면 앱은 새 버전을 적용하지 않습니다.
- GitHub Pages 장애는 업데이트 지연일 뿐 실제 알람 발화의 필수 서버 장애가 아닙니다.

<!-- DISPATCH_STATUS_START -->
## 현재 배차 데이터

- 데이터 버전: v3
- 버전 이름: 2026.09.04-v3
- 적용일: 2026-09-04
- 최신 파일: data_v3.json

### 업데이트 내역

- 공식 Excel 노선 색상 기준 정렬 정보 추가
- 대전교통 8개 노선의 시간표와 공동배차 담당 순번 재검증
- 704 평일 6번차 공식 시각 14:54 · 17:24 · 21:16 검증
<!-- DISPATCH_STATUS_END -->
