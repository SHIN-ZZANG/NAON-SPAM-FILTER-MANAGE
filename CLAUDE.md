# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 상태 (먼저 읽을 것)

- **기획 단계다. 저장소에 코드가 없다.**
  - 초기 기획 코드(관리 서비스가 Rspamd·James를 직접 연동하던 구조)는 다시 쓰지 않기로 해 저장소에서 뺐다. 개발은 새로 시작한다.
- **이 저장소는 관리 프로젝트(`NAON-SPAM-FILTER-MANAGE`)다.** 엔진 프로젝트(`NAON-SPAM-FILTER-ENGINE`)는 별도 저장소다.
- 문서는 ENGINE 저장소에 있다. 두 저장소를 같은 상위 폴더에 나란히 두어야 이 경로가 맞다.
  - **`../NAON-SPAM-FILTER-ENGINE/docs/SHARED.md`** — 두 프로젝트가 함께 지키는 결정: 서버 DB(테이블별 쓰는 쪽), 엔진 API, 한쪽이 멈췄을 때, 진행 단계. 서버 DB나 엔진 API를 건드리는 작업 전에 반드시 읽는다.
  - `../NAON-SPAM-FILTER-ENGINE/docs/FUNC_SPEC.md` — 엔진·관리 전체 세부 명세. 관리 작업은 3-9, 7-2, 8장 MANAGE 부분, 9장 MANAGE 열.
  - 우선순위: 공동 결정은 `SHARED.md`, 관리 결정은 이 파일이 `FUNC_SPEC.md`보다 우선이다. 어긋남을 발견하면 사용자에게 알린다.

## 관리 프로젝트 결정

```
관리자 → 서버 Nginx → [관리 Docker: 관리 백엔드 + 관리 화면]
                          │ 서버 DB 읽기·쓰기 (SHARED.md 3장)
                          └ 엔진 API 호출 (SHARED.md 4장)
```

- **관리 Docker 1개**: 관리 백엔드(Spring Boot) + 관리 화면(Svelte). 화면은 jar 안에 넣어 함께 제공한다.
- **서버 Nginx**(Docker 밖, 서버에 별도 구성, 우리 산출물 아님)가 관리자 요청을 관리 Docker로 전달한다. Nginx에 화면 파일을 두지 않는다.
- **관리 백엔드 역할**: 관리자 로그인, 권한 확인, 격리함 열람·해제·삭제 기록, 처리 이력·통계 조회, 정책·규칙·허용/차단 목록 관리.
  - 엔진만 할 수 있는 일(격리 해제, 원문 조회, 학습 반영, 엔진 상태)은 엔진 API를 호출한다. 관리 백엔드는 Rspamd·James를 직접 호출하지 않는다.
- **관리자 계정**: 스팸필터 전용 계정(서버 DB 저장, 관리 백엔드가 로그인 처리). 그룹웨어 SSO 아님.
- **엔진이 응답하지 않을 때**: 화면에 "엔진 응답 없음"을 표시하고, 엔진이 필요한 기능(해제·원문 보기·학습)은 막는다. 나머지 기능은 그대로 쓴다(SHARED.md 5장).
- 서버 DB는 Docker 밖, 서버에 별도 구성한다. 관리 Docker에 DB 컨테이너를 넣지 않는다.

## 소스 구성 (개발 시작 때 이대로 만든다)

- 한 저장소에 `api/`(Spring Boot, 관리 백엔드)와 `web/`(Svelte, 관리 화면)를 둔다.
  - 빌드 결과물(`target/`, `dist/`, `node_modules/`)은 커밋하지 않는다.
- 개발: `api`(8080)와 `web`(Vite 5173, `/api`를 8080으로 프록시)을 따로 띄운다.
  - 엔진 API 호출은 mock 구현으로 대체할 수 있게 한다(엔진 없이 관리 화면 개발).
- 배포: Dockerfile에서 `web`을 빌드해 `api`의 static에 넣고 jar를 만든 뒤 이미지 하나로 만든다.
  - 관리 Docker 구성(Dockerfile, compose)은 이 저장소에 둔다.
  - 서버 DB 주소(`SPAMFILTER_DB_URL`)·엔진 API 주소(`ENGINE_API_URL`)는 환경변수로 받는다.

## 작업 원칙

- 결정된 사항만 반영한다. 권한 등급, 비밀번호 정책 같은 세부 운영 사항은 사용자가 요청할 때까지 만들지 않는다.
- 운영할 구성 요소(서버·컨테이너)를 늘리는 제안은 꼭 필요할 때만, 선택지로 낸다.
- 스팸스나이퍼(현재 사용 중인 상용 제품)에서는 **우리가 등록한 설정**(허용/차단 목록, 자체 규칙, 기준값)만 이관한다. 업체의 스팸 패턴·규칙은 가져오지 않는다.
- 서버 DB 테이블이나 엔진 API를 바꾸면 ENGINE도 영향을 받는다. 바꾸기 전에 사용자에게 알리고, 결정되면 `SHARED.md`에 반영한다.
