# NAON-SPAM-FILTER-MANAGE

스팸필터 **관리 프로젝트**입니다. 관리 백엔드(Spring Boot)와 관리 화면(Svelte)을 관리 Docker 1개로 배포합니다.

> **기획 단계입니다. 아직 코드가 없습니다.**
> 초기 기획 코드(관리 서비스가 Rspamd·James를 직접 연동하던 구조)는 다시 쓰지 않기로 해 저장소에서 뺐습니다.

## 구조

```
관리자 → 서버 Nginx → [관리 Docker: 관리 백엔드 + 관리 화면]
                          │ 서버 DB 읽기·쓰기
                          └ 엔진 API 호출 (격리 해제·원문 조회·학습 반영·엔진 상태)
```

- 엔진(메일 수신·판정·격리·전달)은 별도 저장소 `NAON-SPAM-FILTER-ENGINE`입니다.
- 두 프로젝트는 **서버 DB**와 **엔진 API**로만 연결됩니다. 관리 백엔드는 Rspamd·James를 직접 호출하지 않습니다.

## 문서

두 저장소를 같은 상위 폴더에 나란히 두면 아래 경로가 맞습니다.

| 문서 | 내용 |
|---|---|
| `CLAUDE.md` | 관리 프로젝트 결정, 소스 구성 계획 |
| `../NAON-SPAM-FILTER-ENGINE/docs/SHARED.md` | 두 프로젝트 공동 결정 (서버 DB, 엔진 API, 한쪽이 멈췄을 때, 진행 단계) |
| `../NAON-SPAM-FILTER-ENGINE/docs/FUNC_SPEC.md` | 엔진·관리 전체 세부 명세 (관리 작업: 3-9, 7-2, 8장, 9장 MANAGE 열) |
| `docs/dashboard-sample.html` | 초기 기획 때 만든 대시보드 시안 (참고용, 브라우저로 바로 열기) |
