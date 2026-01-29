# KPS 2025 스키마 ERD (DDL 기반) — 검증본

본 문서는 사용자가 제공한 MariaDB DDL(내보내기 스크립트) 내용을 기준으로 ERD를 구성한 **검증본**입니다.

- DDL에 **FOREIGN KEY 제약이 거의/전혀 선언되어 있지 않아**, 관계선은 **컬럼명/PK 패턴 기반으로 “추정”**하여 표기했습니다.
- 따라서, 이 문서의 관계는 **설계 의도 또는 실제 JOIN 키와 다를 수 있으며**, 운영 쿼리(SELECT/JOIN) 또는 애플리케이션 코드 기준으로 최종 확정이 필요합니다.

---

## 1. 검증 결과 요약

### 1) PK/복합PK 구조 검증
- 각 테이블의 `PRIMARY KEY (...)` 구문을 기준으로 PK를 표기했습니다.
- 복합PK 테이블(예: `..._assign`, `..._quiz`, `..._atchfile` 등)은 DDL과 동일한 PK 컬럼 조합을 사용했습니다.

### 2) FK 부재로 인한 관계 “추정” 원칙
관계는 다음 규칙으로 추정했습니다.

- `QUIZCODE` → `tbl_quiz_quiz.QUIZCODE`
- `TESTCODE` → `tbl_quiz_test.TESTCODE`
- `TLFCODE` → `tbl_quiz_tlf.TLFCODE`
- `...PLAN_SEQ` → 해당 계획 테이블의 `...PLAN_SEQ`
- `MNUCD` → `tbl_mnu.MNUCD`, `DISPCD` → `tbl_disp.DISPCD`, `MNU_DISP_SEQ` → `tbl_mnu_disp.MNU_DISP_SEQ`
- `QUIZ_HISTORY_SEQ` → `tbl_quiz_quiz_history.QUIZ_HISTORY_SEQ` 또는 `tbl_quiz_test_history.TEST_HISTORY_SEQ` 등 (이력 테이블별 상이)

### 3) 데이터 타입 불일치(중요)
DDL 상 다음과 같은 **ID 타입 불일치**가 존재합니다(관계 확정 시 반드시 확인 필요).

- `tbl_user.USERID` 는 `bigint` 인데,
  - 일부 테이블에서 `USERID` / `..._USERID` / `CID` 등이 `varchar(50)` 또는 `bigint`로 혼재
  - 예: `tbl_user_accctrl.USERID`(varchar), `tbl_user_class.USERID`(varchar) 등

이 경우 실제 조인은:
- `LOGINID`(varchar) 기반인지,
- `USERID`(bigint)를 문자열로 저장한 것인지,
- 또는 별도 사용자키가 있는지
를 확인해야 합니다.

---

## 2. 테이블 분류(도메인 기준)

### A. 접근제어/로그
- `tbl_accctrl`
- `tbl_user_accctrl`
- `tbl_conn_log`
- `tbl_butn_log`
- `tbl_butn_log_quizmov`
- `tbl_butn_log_quizmov_dtl`

### B. 사용자/이력/전문분야
- `tbl_user`
- `tbl_user_career`
- `tbl_user_cert`
- `tbl_user_certgetfield`
- `tbl_user_schcaree`
- `tbl_user_resume_history`
- `tbl_user_panalti`
- `tbl_user_class`

### C. 메뉴/화면/권한
- `tbl_mnu`
- `tbl_disp`
- `tbl_mnu_disp`
- `tbl_mnu_perm`

### D. 자격(분류/과목/난이도)
- `tbl_cert`
- `tbl_cert_dtl`
- `tbl_cert_hard2`
- `tbl_cert_lecture`

### E. 공통(업무 지침/참고/임상사례)
- `tbl_comm_taskmanual`
- `tbl_comm_taskrefdata`
- `tbl_comm_taskclncase`

### F. 계획(개발/심사/출제/채점/정리)
- 개발: `tbl_makeqplan`, `tbl_makeqplan_*`
- 심사: `tbl_invqplan`, `tbl_invqplan_*`
- 출제: `tbl_quizselplan`, `tbl_quizselplan_*`
- 채점: `tbl_markplan`, `tbl_markplan_*`
- 정리: `tbl_revqplan`, `tbl_revqplan_*`

### G. 출제 자동조건
- `tbl_quizsel_autocond`
- `tbl_quizsel_autocond_cls`
- `tbl_quizsel_autocond_cls_dtl`
- `tbl_quizsel_autocond_dtl`

### H. 문항/분류/코드/시험지/이력/첨부/검색어
- 문항: `tbl_quiz_quiz`, `tbl_quiz_quiz_history`
- 분류: `tbl_quiz_class`
- 코드: `tbl_quiz_code`, `tbl_quiz_ncode`
- 첨부/참고자료(+이력): `tbl_quiz_atchfile`, `tbl_quiz_atchfile_history`, `tbl_quiz_refdata`, `tbl_quiz_refdata_history`
- 시험지/관계/TOC(+이력): `tbl_quiz_test`, `tbl_quiz_test_history`, `tbl_quiz_test_rel`, `tbl_quiz_test_rel_history`, `tbl_quiz_test_toc`, `tbl_quiz_test_toc_history`
- 시험지 양식: `tbl_quiz_tlf`, `tbl_quiz_tlf_rel`
- 기출/자료: `tbl_quiz_quizuse`, `tbl_quiz_quizuse_data`
- 검색어: `tbl_srchidx_word`
- 고유번호: `tbl_quiz_custquizno`

### I. 성적(응시자)
- `tbl_quiz_info_guser`
- `tbl_quiz_info_guser_act`

### J. 미디어
- `tbl_quiz_lib_mdseqno`
- `tbl_quiz_lib`
- `tbl_quiz_lib_class`
- `tbl_quiz_lib_list`

### K. 학교/출판사
- `tbl_sch_pcompany`

---

## 3. ERD (Mermaid)

> Mermaid 렌더링을 지원하는 뷰어(예: GitHub, 일부 Markdown 편집기)에서 확인하세요.

### 3.1 사용자 · 권한 · 메뉴 · 로그

```mermaid
erDiagram
  tbl_user {
    bigint USERID PK
    varchar USERPERM
    varchar LOGINID
    varchar USERNM
    varchar EMAIL
  }

  tbl_mnu {
    varchar MNUCD PK
    varchar MNUNM
    varchar P_MNUCD
    char USEYN
  }

  tbl_disp {
    varchar DISPCD PK
    varchar TITL
    varchar DISPURL
  }

  tbl_mnu_disp {
    bigint MNU_DISP_SEQ PK
    varchar MNUCD
    varchar DISPCD
    varchar USERPERMCD
    char MASTDISPYN
  }

  tbl_mnu_perm {
    varchar MNUCD PK
    varchar USERPERMCD PK
    char USEYN
  }

  tbl_conn_log {
    bigint CONN_LOG_SEQ PK
    bigint MNU_DISP_SEQ
    varchar CONNIP
    bigint CID
    datetime CDATETIME
  }

  tbl_butn_log {
    bigint BUTN_LOG_SEQ PK
    varchar BUTNKIND
    varchar URL
    varchar BUTNNM
    varchar CONNIP
    bigint CID
    datetime CDATETIME
  }

  tbl_butn_log_quizmov {
    bigint BUTN_LOG_SEQ PK
    varchar CLASSA_BEFORE PK
    varchar CLASSA_AFTER PK
    int QUIZCNT
  }

  tbl_butn_log_quizmov_dtl {
    bigint BUTN_LOG_SEQ PK
    varchar QUIZCODE PK
    varchar CLASSA_BEFORE PK
    varchar CLASSA_AFTER PK
  }

  tbl_accctrl {
    bigint ACCCTRL_SEQ PK
    varchar ACCIP1
    varchar ACCIP2
    varchar ACCIP3
    varchar ACCIP4FROM
    varchar ACCIP4TO
  }

  tbl_user_accctrl {
    int USER_ACCCTRL_SEQ PK
    varchar USERID
    varchar ACCPERD_ST
    varchar ACCPERD_END
    char MAKEQYN
    char INVQYN
    char SELQSTYN
  }

  tbl_mnu ||--o{ tbl_mnu_disp : maps
  tbl_disp ||--o{ tbl_mnu_disp : maps
  tbl_mnu ||--o{ tbl_mnu_perm : perm
  tbl_mnu_disp ||--o{ tbl_conn_log : logs

  tbl_butn_log ||--o{ tbl_butn_log_quizmov : detail
  tbl_butn_log ||--o{ tbl_butn_log_quizmov_dtl : detail
```

---

### 3.2 사용자 부가정보(경력/자격/학력/이력서/패널티/전문분야)

```mermaid
erDiagram
  tbl_user {
    bigint USERID PK
    varchar LOGINID
  }

  tbl_user_career {
    bigint USERID PK
    int ORDERNO PK
    varchar WORKPLACENM
    varchar POSITIONNM
  }

  tbl_user_cert {
    bigint USERID PK
    int ORDERNO PK
    varchar CERTNM
    varchar GETYYMM
  }

  tbl_user_certgetfield {
    bigint USERID PK
    varchar CERTGETFIELDCD PK
    varchar ABROADCERTGETFIELDNM
  }

  tbl_user_schcaree {
    bigint USERID PK
    int ORDERNO PK
    varchar SCHDGREE
    varchar SCHNM
  }

  tbl_user_resume_history {
    bigint USERID PK
    datetime CDATETIME PK
    varchar RESUMEFILEPATH
    varchar RESUMEORGNFILENM
  }

  tbl_user_panalti {
    int USER_PANALTI_SEQ PK
    varchar USERID
    varchar PANALTILVL
    varchar PANALTIPERD_ST
    varchar PANALTIPERD_END
  }

  tbl_user_class {
    varchar USERID PK
    varchar CLASS1 PK
    varchar CLASS2 PK
    varchar CLASS3 PK
  }

  tbl_user ||--o{ tbl_user_career : has
  tbl_user ||--o{ tbl_user_cert : has
  tbl_user ||--o{ tbl_user_certgetfield : has
  tbl_user ||--o{ tbl_user_schcaree : has
  tbl_user ||--o{ tbl_user_resume_history : has
```

---

### 3.3 자격(기본/상세/난이도/과목)

```mermaid
erDiagram
  tbl_cert {
    varchar CLASS1 PK
    varchar CLASS2 PK
    varchar CLASS3 PK
    varchar CERTNM
    varchar PGSGGB
    int QUIZCNT
    decimal PASSSCORE
  }

  tbl_cert_dtl {
    varchar CLASS1 PK
    varchar CLASS2 PK
    varchar CLASS3 PK
    varchar AUTOCOND_DTLKIND_CD PK
    varchar AUTOCOND_DTLKIND_CDSUB PK
    int ORDERNO
  }

  tbl_cert_hard2 {
    varchar CLASS1 PK
    varchar CLASS2
    varchar CLASS3
  }

  tbl_cert_lecture {
    varchar CLASS1 PK
    varchar CLASS2 PK
    varchar CLASS3 PK
    varchar CLASS4 PK
    int PRD
    int QUIZCNT
  }

  tbl_cert ||--o{ tbl_cert_dtl : detail
  tbl_cert ||--o{ tbl_cert_lecture : lecture
```

---

### 3.4 공통 업무 지침/참고자료/임상사례

```mermaid
erDiagram
  tbl_comm_taskmanual {
    varchar TASKMANUALCD PK
    varchar ATCHFILEPATH
    varchar ATCHORGNFILENM
  }

  tbl_comm_taskrefdata {
    varchar TASKSTEPCD PK
    bigint PLAN_SEQ PK
    varchar CLASS3 PK
    int ORDERNO PK
    varchar ATCHFILEPATH
    varchar ATCHORGNFILENM
  }

  tbl_comm_taskclncase {
    varchar TASKSTEPCD PK
    bigint PLAN_SEQ PK
    varchar CLASS3 PK
    varchar QUIZCODE PK
    varchar QUIZPATH
    varchar CUSTQUIZNO
  }
```

---

### 3.5 개발계획(MAKEQ) · 배정/범위/문항/참고

```mermaid
erDiagram
  tbl_makeqplan {
    bigint MAKEQPLAN_SEQ PK
    varchar MAKEQPLANNM
    varchar CLASS1
    varchar CLASS2
    varchar YYYY
    int NO
  }

  tbl_makeqplan_cmit_assign {
    bigint MAKEQPLAN_SEQ PK
    varchar CLASS3 PK
    bigint MAKEQ_USERID PK
    char REQYN
    datetime REQDATETIME
  }

  tbl_makeqplan_rng_field {
    bigint MAKEQPLAN_SEQ PK
    varchar CLASS3 PK
    int CLSSTEP
  }

  tbl_makeqplan_rng_field_quiztype {
    bigint MAKEQPLAN_SEQ PK
    varchar CLASS3 PK
    varchar IOSTYPE2 PK
  }

  tbl_makeqplan_rng_field_quizcnt {
    bigint MAKEQPLAN_SEQ PK
    varchar CLASS3 PK
    varchar CLASSA PK
    varchar IOSTYPE2 PK
    int CLNCASENO PK
    int HARD2_1_QUIZCNT
    int HARD2_2_QUIZCNT
    int HARD2_3_QUIZCNT
    int HARD2_4_QUIZCNT
    int HARD2_5_QUIZCNT
    int HARD2_ZERO_QUIZCNT
  }

  tbl_makeqplan_quiz {
    bigint MAKEQPLAN_SEQ PK
    varchar CLASS3 PK
    bigint MAKEQ_USERID PK
    varchar QUIZCODE PK
    varchar GOAL_IOSTYPE2
    varchar GOAL_HARD2
    varchar GOAL_CLASSA
    bigint CLNCASENO
  }

  tbl_makeqplan_cmit_goalquizcnt {
    bigint MAKEQPLAN_SEQ PK
    varchar CLASS3 PK
    bigint MAKEQ_USERID PK
    varchar CLASSA PK
    varchar IOSTYPE2 PK
    bigint CLNCASENO PK
    int HARD2_1_QUIZCNT
    int HARD2_2_QUIZCNT
    int HARD2_3_QUIZCNT
    int HARD2_4_QUIZCNT
    int HARD2_5_QUIZCNT
    int HARD2_ZERO_QUIZCNT
  }

  tbl_makeqplan_cmit_refdata_atchfile {
    bigint MAKEQPLAN_SEQ PK
    varchar CLASS3 PK
    bigint MAKEQ_USERID PK
    int ORDERNO PK
    varchar ATCHFILEPATH
    varchar ATCHORGNFILENM
  }

  tbl_makeqplan_cmit_refdata_clncase {
    bigint MAKEQPLAN_SEQ PK
    varchar CLASS3 PK
    bigint MAKEQ_USERID PK
    varchar QUIZCODE PK
    varchar QUIZPATH
    varchar CUSTQUIZNO
  }

  tbl_makeqplan ||--o{ tbl_makeqplan_cmit_assign : assigns
  tbl_makeqplan ||--o{ tbl_makeqplan_rng_field : scope
  tbl_makeqplan ||--o{ tbl_makeqplan_rng_field_quiztype : scopeType
  tbl_makeqplan ||--o{ tbl_makeqplan_rng_field_quizcnt : scopeCount
  tbl_makeqplan ||--o{ tbl_makeqplan_quiz : includes
  tbl_makeqplan ||--o{ tbl_makeqplan_cmit_goalquizcnt : goal
  tbl_makeqplan ||--o{ tbl_makeqplan_cmit_refdata_atchfile : refFiles
  tbl_makeqplan ||--o{ tbl_makeqplan_cmit_refdata_clncase : refCases
```

---

### 3.6 심사계획(INVQ) · 배정/범위/문항/참고

```mermaid
erDiagram
  tbl_invqplan {
    bigint INVQPLAN_SEQ PK
    bigint MAKEQPLAN_SEQ
    varchar INVQPLANNM
    varchar CLASS1
    varchar CLASS2
    varchar YYYY
    int NO
  }

  tbl_invqplan_cmit_assign {
    bigint INVQPLAN_SEQ PK
    varchar CLASS3 PK
    bigint INVQ_USERID PK
    varchar REQYN
  }

  tbl_invqplan_rng_field {
    bigint INVQPLAN_SEQ PK
    varchar CLASS3 PK
    int CLSSTEP
  }

  tbl_invqplan_rng_field_quiztype {
    bigint INVQPLAN_SEQ PK
    varchar CLASS3 PK
    varchar IOSTYPE2 PK
  }

  tbl_invqplan_quiz {
    bigint INVQPLAN_SEQ PK
    varchar CLASS3 PK
    varchar QUIZCODE PK
    bigint MAKEQPLAN_SEQ
    bigint EACHINVQ1_CMITID
    bigint EACHINVQ2_CMITID
    bigint WITHINVQ_CMITID
  }

  tbl_invqplan_refdata_atchfile {
    bigint INVQPLAN_SEQ PK
    varchar CLASS3 PK
    bigint INVQ_USERID PK
    int ORDERNO PK
    varchar ATCHFILEPATH
    varchar ATCHORGNFILENM
  }

  tbl_invqplan ||--o{ tbl_invqplan_cmit_assign : assigns
  tbl_invqplan ||--o{ tbl_invqplan_rng_field : scope
  tbl_invqplan ||--o{ tbl_invqplan_rng_field_quiztype : scopeType
  tbl_invqplan ||--o{ tbl_invqplan_quiz : includes
  tbl_invqplan ||--o{ tbl_invqplan_refdata_atchfile : refFiles
```

---

### 3.7 출제계획(QUIZSEL) · 분야/배정/문항번호/문항/참고/최종인쇄/자동조건

```mermaid
erDiagram
  tbl_quizselplan {
    bigint QUIZSELPLAN_SEQ PK
    varchar QUIZSELPLANNM
    varchar CLASS1
    varchar CLASS2
    varchar YYYY
    int NO
    varchar PGSGGB
  }

  tbl_quizselplan_field {
    bigint QUIZSELPLAN_SEQ PK
    varchar CLASS3 PK
    varchar TESTCODE_A
    varchar TESTCODE_B
    bigint QUIZSEL_AUTOCOND_SEQ
  }

  tbl_quizselplan_cmit_assign {
    bigint QUIZSELPLAN_SEQ PK
    varchar CLASS3 PK
    bigint QUIZSEL_USERID PK
    char QUIZSELROL_CMITYN
    char QUIZSELCLOSYN
  }

  tbl_quizselplan_cmit_quiznoassign {
    bigint QUIZSELPLAN_SEQ PK
    varchar CLASS3 PK
    varchar IOSTYPE2 PK
    int TESTQUIZNO PK
    bigint QUIZSEL_USERID
    varchar CLASSA
    decimal ASSIGNPOINTS
  }

  tbl_quizselplan_quiz {
    bigint QUIZSELPLAN_SEQ PK
    varchar CLASS3 PK
    int QUIZSELSET_TYPE PK
    varchar QUIZCODE PK
    varchar CLASSA
    varchar IOSTYPE2
    int TESTQUIZNO
    int final_testquizno
  }

  tbl_quizselplan_cmit_refdata_atchfile {
    bigint QUIZSELPLAN_SEQ PK
    varchar CLASS3 PK
    bigint QUIZSEL_USERID PK
    int ORDERNO PK
    varchar ATCHFILEPATH
    varchar ATCHORGNFILENM
  }

  tbl_quizselplan_cmit_refdata_clncase {
    bigint QUIZSELPLAN_SEQ PK
    varchar CLASS3 PK
    bigint QUIZSEL_USERID PK
    varchar QUIZCODE PK
  }

  tbl_quizselplan_lastprint {
    varchar CLASS1 PK
    varchar CLASS2 PK
    varchar CLASS3 PK
    varchar YYYY PK
    int NO PK
    varchar PGSGGB PK
    varchar TESTTYPE PK
    bigint QUIZSELPLAN_SEQ
    int QUIZCNT
  }

  tbl_quizselplan_lastprint_atchfile {
    varchar CLASS1 PK
    varchar CLASS2 PK
    varchar CLASS3 PK
    varchar YYYY PK
    int NO PK
    varchar PGSGGB PK
    varchar TESTTYPE PK
    int ORDERNO PK
    varchar ATCHFILEPATH
    varchar ATCHORGNFILENM
  }

  tbl_quizsel_autocond {
    int QUIZSEL_AUTOCOND_SEQ PK
    varchar STDYY
    varchar TITL
    varchar CLASS1
    varchar CLASS2
    varchar CLASS3
  }

  tbl_quizsel_autocond_cls {
    int QUIZSEL_AUTOCOND_SEQ PK
    varchar CLASSA PK
    int TESTQUIZNO_ST
    int TESTQUIZNO_END
    int QUIZCNT
  }

  tbl_quizsel_autocond_dtl {
    int QUIZSEL_AUTOCOND_SEQ PK
    varchar AUTOCOND_DTLKIND_CD PK
    varchar AUTOCOND_DTLKIND_CDSUB PK
    int GOALQUIZCNT
  }

  tbl_quizsel_autocond_cls_dtl {
    int QUIZSEL_AUTOCOND_SEQ PK
    varchar CLASSA PK
    varchar AUTOCOND_DTLKIND_CD PK
    varchar AUTOCOND_DTLKIND_CDSUB PK
    int GOALQUIZCNT
  }

  tbl_quizselplan ||--o{ tbl_quizselplan_field : field
  tbl_quizselplan ||--o{ tbl_quizselplan_cmit_assign : assigns
  tbl_quizselplan ||--o{ tbl_quizselplan_cmit_quiznoassign : numbering
  tbl_quizselplan ||--o{ tbl_quizselplan_quiz : includes
  tbl_quizselplan ||--o{ tbl_quizselplan_cmit_refdata_atchfile : refFiles
  tbl_quizselplan ||--o{ tbl_quizselplan_cmit_refdata_clncase : refCases

  tbl_quizsel_autocond ||--o{ tbl_quizsel_autocond_cls : cls
  tbl_quizsel_autocond ||--o{ tbl_quizsel_autocond_dtl : dtl
  tbl_quizsel_autocond_cls ||--o{ tbl_quizsel_autocond_cls_dtl : clsDtl
```

---

### 3.8 채점계획(MARK) · 배정/문항/답안지/대상자/개별채점/참고

```mermaid
erDiagram
  tbl_markplan {
    bigint MARKPLAN_SEQ PK
    varchar MARKQPLANNM
    varchar CLASS1
    varchar CLASS2
    varchar YYYY
    int NO
  }

  tbl_markplan_cmit_assign {
    bigint MARKPLAN_SEQ PK
    varchar CLASS3 PK
    bigint MARK_USERID PK
    varchar REQYN
  }

  tbl_markplan_quiz {
    varchar QUIZCODE PK
    varchar CLASS3 PK
    bigint MARKPLAN_SEQ PK
    varchar TESTTYPE PK
    bigint QUIZ_HISTORY_SEQ
  }

  tbl_markplan_sht {
    bigint MARKPLAN_SEQ PK
    varchar CLASS3 PK
    varchar TESTTYPE PK
    varchar ANSFILEPATH
    varchar ANSORGNFILENM
  }

  tbl_markplan_exmne {
    varchar EXMNE_NO PK
    bigint MARK_PLAN_SEQ PK
    varchar CLASS1
    varchar CLASS2
    varchar CLASS3
    varchar EMP_NO
    varchar NM
  }

  tbl_markplan_cmit_quiz_eachmark {
    bigint MARKPLAN_SEQ PK
    varchar CLASS3 PK
    varchar QUIZCODE PK
    varchar TESTTYPE PK
    bigint MARK_USERID PK
    varchar APYEXAMNO PK
    decimal EACHMARK_HITPOINT
  }

  tbl_markplan_cmit_refdata_atchfile {
    bigint MARKPLAN_SEQ PK
    varchar CLASS3 PK
    bigint MARK_USERID PK
    int ORDERNO PK
    varchar ATCHFILEPATH
    varchar ATCHORGNFILENM
  }

  tbl_markplan ||--o{ tbl_markplan_cmit_assign : assigns
  tbl_markplan ||--o{ tbl_markplan_quiz : includes
  tbl_markplan ||--o{ tbl_markplan_sht : sheets
  tbl_markplan ||--o{ tbl_markplan_exmne : examinee
  tbl_markplan ||--o{ tbl_markplan_cmit_quiz_eachmark : eachMark
  tbl_markplan ||--o{ tbl_markplan_cmit_refdata_atchfile : refFiles
```

---

### 3.9 정리계획(REVQ) · 배정/범위/문항/참고

```mermaid
erDiagram
  tbl_revqplan {
    bigint REVQPLAN_SEQ PK
    varchar REVQPLANNM
    varchar CLASS1
    varchar CLASS2
    varchar YYYY
    int NO
  }

  tbl_revqplan_cmit_assign {
    bigint REVQPLAN_SEQ PK
    varchar CLASS3 PK
    bigint REVQ_USERID PK
    char REQYN
  }

  tbl_revqplan_rng_field {
    bigint REVQPLAN_SEQ PK
    varchar CLASS3 PK
    int CLSSTEP
  }

  tbl_revqplan_rng_field_quiztype {
    bigint REVQPLAN_SEQ PK
    varchar CLASS3 PK
    varchar IOSTYPE2 PK
  }

  tbl_revqplan_quiz {
    bigint REVQPLAN_SEQ PK
    varchar CLASS3 PK
    varchar QUIZCODE PK
    bigint REVQ_USERID
    varchar QUIZSTAT
    varchar REVQSTAT
  }

  tbl_revqplan_refdata_atchfile {
    bigint REVQPLAN_SEQ PK
    varchar CLASS3 PK
    bigint REVQ_USERID PK
    int ORDERNO PK
    varchar ATCHFILEPATH
    varchar ATCHORGNFILENM
  }

  tbl_revqplan ||--o{ tbl_revqplan_cmit_assign : assigns
  tbl_revqplan ||--o{ tbl_revqplan_rng_field : scope
  tbl_revqplan ||--o{ tbl_revqplan_rng_field_quiztype : scopeType
  tbl_revqplan ||--o{ tbl_revqplan_quiz : includes
  tbl_revqplan ||--o{ tbl_revqplan_refdata_atchfile : refFiles
```

---

### 3.10 문항/분류/코드/시험지/첨부/이력/검색어/기출

```mermaid
erDiagram
  tbl_quiz_quiz {
    varchar QUIZCODE PK
    varchar TITLE
    varchar CUSTQUIZNO
    varchar CLASSA
    varchar SCHCD
    varchar IOSTYPE2
    varchar IOSKIND
    bigint MAKEQPLAN_SEQ
    bigint INVQPLAN_SEQ
    bigint QUIZSELPLAN_SEQ
    bigint MARKPLAN_SEQ
    bigint REVQPLAN_SEQ
    bigint QUIZ_HISTORY_SEQ
  }

  tbl_quiz_quiz_history {
    bigint QUIZ_HISTORY_SEQ PK
    varchar QUIZCODE
    varchar TITLE
    varchar CLASSA
  }

  tbl_quiz_class {
    varchar CLASSA PK
    varchar SCHCD PK
    varchar NAME1
    varchar SORTKEY
  }

  tbl_quiz_code {
    varchar CDMAIN PK
    varchar CDSUB PK
    varchar SCHCD PK
    varchar NAME1
  }

  tbl_quiz_ncode {
    varchar NCODEA PK
    varchar SCHCD PK
    varchar NAME1
  }

  tbl_quiz_atchfile {
    varchar QUIZCODE PK
    int ORDERNO PK
    varchar ATCHFILEPATH
    varchar ORGNFILENM
  }

  tbl_quiz_atchfile_history {
    bigint QUIZ_HISTORY_SEQ PK
    varchar QUIZCODE PK
    int ORDERNO PK
    varchar ATCHFILEPATH
    varchar ORGNFILENM
  }

  tbl_quiz_refdata {
    varchar QUIZCODE PK
    int ORDERNO PK
    varchar REFDATA_BOOKNM
    varchar REFDATA_PAGE
  }

  tbl_quiz_refdata_history {
    bigint QUIZ_HISTORY_SEQ PK
    varchar QUIZCODE PK
    int ORDERNO PK
    varchar REFDATA_BOOKNM
    varchar REFDATA_PAGE
  }

  tbl_srchidx_word {
    varchar WORD PK
    varchar QUIZCODE PK
  }

  tbl_quiz_custquizno {
    bigint QUIZID PK
    varchar QUIZCODE
    varchar CLASS1
    varchar CLASS2
    varchar CLASS3
    varchar CUSTQUIZNO
  }

  tbl_quiz_test {
    varchar TESTCODE PK
    varchar TITLE
    varchar TLFCODE
    int QUIZCNT
    int TESTTIME
  }

  tbl_quiz_test_history {
    bigint TEST_HISTORY_SEQ PK
    varchar TESTCODE
    varchar TLFCODE
    int QUIZCNT
  }

  tbl_quiz_test_rel {
    varchar TESTCODE PK
    smallint ORDERNO PK
    varchar QUIZCODE
    int QUIZNO
    decimal ASSIGNPOINTS
  }

  tbl_quiz_test_rel_history {
    bigint TEST_HISTORY_SEQ PK
    varchar TESTCODE PK
    smallint ORDERNO PK
    varchar QUIZCODE
    int QUIZNO
  }

  tbl_quiz_test_toc {
    varchar TOCCD PK
    varchar TESTCODE PK
    int TOC_ORDERNO
    varchar TOCNM
  }

  tbl_quiz_test_toc_history {
    bigint TEST_HISTORY_SEQ PK
    varchar TOCCD PK
    varchar TESTCODE PK
    int TOC_ORDERNO
    varchar TOCNM
  }

  tbl_quiz_tlf {
    varchar TLFCODE PK
    varchar TITLE
    varchar FILEPATH
    varchar TLFKIND
  }

  tbl_quiz_tlf_rel {
    varchar TLFCODE PK
    varchar TLFPAGEGRP PK
    varchar FILEPATH
  }

  tbl_quiz_quizuse {
    varchar QUIZCODE PK
    varchar CLASS1 PK
    varchar CLASS2 PK
    varchar CLASS3 PK
    varchar YYYY PK
    int NO PK
    varchar PGSGGB PK
    varchar EXAMTYPE PK
    varchar TESTCODE
    int HITCNT
    int SVCCNT
  }

  tbl_quiz_quizuse_data {
    varchar CLASS1 PK
    varchar CLASS2 PK
    varchar CLASS3 PK
    varchar YYYY PK
    int NO PK
    varchar PGSGGB PK
    varchar EXAMTYPE PK
    varchar FILEPATH
    varchar ORGNFILENM
  }

  tbl_quiz_quiz }o--|| tbl_quiz_class : classA
  tbl_quiz_quiz ||--o{ tbl_quiz_atchfile : files
  tbl_quiz_quiz ||--o{ tbl_quiz_refdata : ref
  tbl_quiz_quiz ||--o{ tbl_srchidx_word : index

  tbl_quiz_quiz ||--o{ tbl_quiz_quiz_history : history
  tbl_quiz_quiz_history ||--o{ tbl_quiz_atchfile_history : files_hist
  tbl_quiz_quiz_history ||--o{ tbl_quiz_refdata_history : ref_hist

  tbl_quiz_test ||--o{ tbl_quiz_test_rel : contains
  tbl_quiz_test_rel }o--|| tbl_quiz_quiz : quiz

  tbl_quiz_tlf ||--o{ tbl_quiz_tlf_rel : pages
  tbl_quiz_test }o--|| tbl_quiz_tlf : usesTemplate

  tbl_quiz_quizuse }o--|| tbl_quiz_quiz : usedQuiz
```

---

### 3.11 성적(응시자 결과)

```mermaid
erDiagram
  tbl_quiz_info_guser {
    varchar USERID PK
    varchar TESTSETCODE PK
    varchar GROUP1 PK
    varchar GROUP2 PK
    varchar GROUP3 PK
    decimal HITPOINT
    decimal HITCNT
    varchar ANSWERPATH
    datetime CDATETIME
  }

  tbl_quiz_info_guser_act {
    varchar USERID PK
    varchar TESTSETCODE PK
    varchar GROUP1 PK
    varchar GROUP2 PK
    varchar GROUP3 PK
    decimal HITPOINT
    decimal HITCNT
    varchar TEMPANSWERPATH
    datetime CDATETIME
  }
```

---

### 3.12 미디어

```mermaid
erDiagram
  tbl_quiz_lib_mdseqno {
    bigint MDSEQNO PK
  }

  tbl_quiz_lib {
    decimal MDSEQNO PK
    varchar TITLE
    varchar PATH
    varchar PDFFILENAME
  }

  tbl_quiz_lib_class {
    varchar CODEA PK
    char CDGUBUN PK
    varchar NAME
  }

  tbl_quiz_lib_list {
    decimal SEQNO PK
    decimal MDSEQNO PK
    varchar CODEA
    char CDGUBUN
  }

  tbl_quiz_lib_mdseqno ||--|| tbl_quiz_lib : master
  tbl_quiz_lib ||--o{ tbl_quiz_lib_list : maps
  tbl_quiz_lib_class ||--o{ tbl_quiz_lib_list : classify
```

---

### 3.13 학교/출판사

```mermaid
erDiagram
  tbl_sch_pcompany {
    varchar GRADEYY PK
    varchar SUBJECT PK
    varchar GRADE PK
    varchar SCHCD PK
    varchar PCOMPANY
  }
```

---

## 4. 보완 권고(다음 단계)

1) 실제 JOIN 키 확정  
- 운영 SQL/서비스 코드에서 어떤 키로 JOIN 하는지 확인 후 FK 관계를 확정하세요.

2) 타입 정리 또는 캐스팅 정책 명확화  
- `USERID`가 bigint인지 varchar인지 통일하거나, 조인 시 캐스팅 규칙을 정해두는 것이 좋습니다.

3) FK/인덱스 후보 생성  
- 조회가 많은 매핑 테이블(`*_quiz`, `*_assign`, `*_atchfile`)은
  - `(상위SEQ, CLASS3)` 류 복합 인덱스,
  - `(QUIZCODE)` 단일 인덱스
  등을 고려하세요.

