# Web Vulnerability Diagnosis Checklist

이 문서는 기본 취약점 후보를 식별하기 위한 최소 체크리스트다. 이 문서에 없는 항목도 `security_strategy.md`와 `business_logic_playbook.md` 기준으로 가설화해야 한다.

## 1. 공통 분류 기준

모든 요청은 다음 기준으로 분류한다.

- Method / Host / Path
- Query / Body Parameter
- Content-Type
- Cookie / Authorization 존재 여부
- CSRF Token / Origin / Referer 존재 여부
- 응답 Status Code / Content-Type / Length
- Redirect 여부
- 상태 변경 여부
- 객체 식별자 존재 여부
- 사용자/권한/조직/tenant 식별자 존재 여부
- 파일명/경로/URL 입력값 존재 여부
- 에러 메시지 또는 내부 정보 노출 여부

## 2. 인증 / 세션 관리

후보:

- 로그인 후 접근 기능
- Cookie / Authorization 제거 시 확인 필요한 요청
- 로그아웃 후 재사용 가능 요청
- 세션 만료 후 접근 가능 요청
- URL에 토큰이 포함된 요청

확인 관점:

- 인증 제거 후 민감 데이터 또는 기능 응답이 반환되는가
- 로그아웃/만료 세션으로 기존 요청이 동작하는가
- 인증 실패 시 상태가 변경되는가
- 실패 응답 이후 세션 상태가 오염되는가

## 3. 인가 / IDOR

후보 파라미터:

- id, no, seq, idx
- userId, memberId, memberNo, accountId
- fileId, fileNo, attachId
- boardId, postId, commentId, documentId
- orderId, reservationId, room, time
- orgId, companyId, tenantId

확인 관점:

- A 계정 객체를 B 계정 세션에서 조회/수정/삭제/다운로드 가능한가
- 목록에 없는 객체를 직접 접근 가능한가
- 서버가 객체 소유자와 세션 사용자를 비교하는가
- 관리자 객체와 일반 사용자 객체가 같은 식별자 체계를 쓰는가

## 4. 상태 전이 / 비즈니스 로직

후보:

- 비밀번호 재설정
- 이메일/휴대폰 변경
- 인증번호 검증
- 예약/취소
- 주문/결제/환불
- 승인/반려
- 업로드 후 다운로드

확인 관점:

- 선행 단계 없이 후속 단계 호출 가능한가
- 실패한 선행 단계 이후 후속 단계가 성공하는가
- 완료/취소/삭제된 객체를 다시 조작할 수 있는가
- 동일 토큰/인증번호/쿠폰/예약이 재사용 가능한가
- 성공/실패 메시지와 실제 상태가 일치하는가

## 5. 파일 다운로드

후보 파라미터:

- file, filename, fileName
- path, filePath, savePath, realPath
- fileId, fileSeq, attachId
- number, no, id
- url, src

확인 관점:

- 인증 제거 후 파일이 반환되는가
- 게시글/문서 조회 권한과 파일 다운로드 권한이 일치하는가
- 파일 ID 변경 시 다른 파일이 반환되는가
- 경로 파라미터가 직접 사용되는가
- Content-Disposition에 파일명이 노출되는가

## 6. 파일 업로드

후보:

- multipart/form-data
- upload, attach, file, image, editor, import 경로
- 확장자, MIME Type, 파일 크기 파라미터

확인 관점:

- 서버 측 확장자/MIME/시그니처 검증이 있는가
- 업로드 파일이 웹 경로에서 직접 접근 가능한가
- 업로드 후 URL 또는 저장 경로가 반환되는가
- 파일명이 사용자 입력값을 신뢰하는가
- 인증 없이 업로드 가능한가

## 7. XSS / HTML Injection

후보 입력값:

- 검색어, 제목, 내용, 댓글, 이름, 닉네임
- HTML 에디터 입력값
- URL 파라미터
- JSON 문자열 필드

확인 관점:

- 입력값이 HTML 본문/Attribute/JavaScript/URL/CSS 영역에 반영되는가
- 출력 시 Context-aware Encoding이 적용되는가
- 저장 후 목록/상세/관리자/검색/알림/PDF/엑셀에 재출력되는가
- HTML 렌더링만 확인했는지, 실제 스크립트 실행까지 확인했는지 구분한다

판단:

- HTML 태그 렌더링만 확인: Stored/Reflected HTML Injection 또는 XSS 가능성
- 스크립트 실행까지 확인: XSS Confirmed

## 8. SQL Injection

후보:

- 로그인
- 검색/필터/정렬
- 목록/상세 식별자
- 날짜 범위
- 통계/리포트
- 엑셀 다운로드

확인 관점:

- 특수문자/타입 변경 시 DB 에러 또는 응답 차이가 있는가
- 정렬/필터 파라미터가 허용 목록 검증을 거치는가
- DBMS, SQL 구문, 테이블/컬럼명이 노출되는가
- 단순 500 에러인지 DB 관련 에러인지 구분한다

## 9. NoSQL Injection / JSON 구조 처리

후보:

- JSON Body 기반 검색/필터
- 로그인 또는 사용자 조회
- 배열/객체 형태 파라미터 허용 요청

확인 관점:

- 문자열 위치에 객체/배열 입력 시 검증되는가
- JSON 구조 변경 시 응답 범위가 달라지는가
- 조건 객체가 서버에서 해석되는가

## 10. SSRF / 외부 URL 처리

후보 파라미터:

- url, uri, link, callback, endpoint, target
- imageUrl, fileUrl, webhook, apiUrl, feedUrl

확인 관점:

- 서버가 사용자 입력 URL로 직접 요청하는가
- 허용 도메인 목록이 있는가
- 리다이렉트 후 최종 목적지를 검증하는가
- 네트워크 오류가 응답에 노출되는가

## 11. Open Redirect

후보 파라미터:

- redirect, redirectUrl, returnUrl, next, url, target, continue, destination

확인 관점:

- 외부 도메인이 Location 헤더 또는 클라이언트 이동 로직에 반영되는가
- 로그인/인증/메일 링크와 연결되는가
- 상대경로, 절대 URL, 인코딩 URL 처리가 다른가

## 12. CSRF

후보:

- 모든 상태 변경 요청
- GET 기반 등록/수정/삭제/취소
- 계정/권한/비밀번호/이메일 변경

확인 관점:

- CSRF Token 존재 여부
- Token 서버 검증 여부
- Origin / Referer 검증 여부
- SameSite Cookie 속성
- GET 요청만으로 상태 변경 가능한가
- Stored XSS/HTML Injection과 연계 가능한가

## 13. 정보 노출

후보:

- Stack Trace
- SQL Query / DBMS 정보
- 서버 내부 경로
- 내부 IP / 내부 URL
- API Key / 토큰 / JWT Secret
- Swagger/OpenAPI
- 소스맵 / 백업 / 설정 파일
- HTML/JS 주석 내 민감정보

## 14. API 보안

확인 항목:

- 인증 없는 API 접근
- Method 변경 시 동작
- Content-Type 변경 시 동작
- JSON 필드 추가/삭제 검증
- Mass Assignment
- 과도한 데이터 반환
- Pagination 제한 우회
- Swagger/OpenAPI 노출

## 15. 우선순위 기준

### Critical

- 비인증 상태에서 계정 탈취, 관리자 기능 조작, 민감정보 대량 접근, 중요 상태 변경이 가능함

### High

- 타 사용자 데이터 접근/수정/삭제 가능
- 권한 없는 파일 다운로드 가능
- 중요 계정/권한/예약/결제 상태 변경 가능
- Stored XSS가 관리자 또는 상태 변경 기능과 연계 가능

### Medium

- 제한적 정보 노출
- 일부 권한 검증 미흡
- CSRF 방어 미흡
- Reflected XSS 가능성
- Open Redirect 가능성
- DB/서버 에러 노출

### Low / Info

- 보안 헤더 미흡
- 버전 정보 노출
- 민감도가 낮은 정보 노출
- 보안 설정 개선 사항
