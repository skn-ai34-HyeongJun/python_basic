# KBO 직관가이드 챗봇 — 3차 프로젝트 데이터 수집 계획

작성 기준일: 2026-09-02 (일정: 8/31~9/17, 2.5주)
GitHub: SKNETWORKS-FAMILY-AICAMP/SKN34-3rd-5Team

역할 분배·산출물 담당자 매핑은 팀 회의 후 확정 예정. 이 문서는 "무엇을 모아야 하는지"와 "프론트엔드/DB 스택"만 정리.

## 0. 3차 범위 재확인
Notion 3차 목표는 "내외부 문서 기반 RAG 질의응답 시스템 구현"이다. 매칭·예측게임·포인트·굿즈 BM 등은 4차 확장 범위이므로, 3차 수집 대상은 아래 챗봇 코어 기능에 필요한 데이터로 한정한다. 응원가(응원문화) 관련 데이터는 저작권 이슈로 수집 범위에서 제외했고, 팀순위/TOP5 중에서는 팀 순위만 수집 대상으로 한정했다.

## 1. 수집해야 할 데이터 카테고리 (구장 표준 스키마)

10개 구단 · 9개 구장(두산·LG는 잠실 공동 홈구장) 기준으로 아래 필드를 전부 채운다.

| 카테고리 | 세부 필드 | 설명 |
| --- | --- | --- |
| 구장/홈팀 기본정보 | 구장명, 홈팀, 개장연도, 수용인원 | 구장 식별 기본값 |
| 좌석 구역 정보 | 구역명, 시야 특징, 가격대, 지붕 유무 | 구역별 특징을 챗봇이 추천할 때 근거로 사용 |
| 교통·주차 안내 | 대중교통 노선(지하철/버스), 주차장 위치·요금 | 직관 전 이동 계획용 |
| 반입 규정 | 반입 금지 물품, 허용 물품 목록 | KBO 공식 관람 규정 기반 |
| 먹거리·주변 콘텐츠 | 구장 내 명물 먹거리, 주변 맛집·핫플레이스, 포토존·이벤트 | 새 메인주제(직관 코스/여행) 반영해서 주변 맛집·핫플까지 범위 확장 |
| 주요 선수/추천 선수 | 팀별 핵심 선수, 포지션별 볼거리, 입문자 추천 관전 포인트 | "이 팀 처음이면 이 선수 주목" 문구용 |
| 기초 규칙 | 이닝/아웃/스트라이크/볼넷 등 최소 기본 용어 | 심화 해설 아님, 관람 전 예습용 요약 수준으로 범위 제한 |
| 팀 순위 | 구단 순위 | 실시간 아님, 경기 종료 후 하루 1~2회 배치 수집 대상. 투수/타자 TOP5는 수집하지 않음 |
| 시즌 일정 | 팀별 홈경기 일정(날짜·시간·상대팀) | 캘린더 안내용 |
| 좌석 후기/평점(참고용) | 구역별 시야/편의성 후기 텍스트 | 3차는 조회만, 실제 UGC 축적은 4차 |
| 원정 응원 가이드 | 원정 구장별 원정팀 응원석 위치, 입장 동선, 원정팀 응대 문화 | 응원가가 아닌 좌석·동선 안내 위주. 기존 팬 심화 정보 |

## 2. 데이터·API 출처 종합표

| 구분 | 출처/API | 용도 | 수집방식 | 비용 | 사용 시점 |
| --- | --- | --- | --- | --- | --- |
| 정적 데이터 | KBO 공식 홈페이지 구단정보/기록실 (koreabaseball.com/kbo/league/teamInfo.aspx) | 선수 소개, 기초 규칙 1차 소스 | 크롤링 | 무료 | 3차 |
| 정적 데이터 | KBO 공식 팀순위 페이지 (koreabaseball.com/Record/Ranking/Top5.aspx) | 팀 순위만 사용 (투수·타자 TOP5는 수집 범위 제외) | 크롤링 (robots.txt 차단 확인됨 → 하루 1~2회만) | 무료 | 3차 |
| 정적 데이터 | 각 구단 홈페이지 구장 안내(10개 구단 개별) | 좌석·교통·반입규정 1차 소스 | 크롤링 | 무료 | 3차 |
| 정적 데이터 | KBO 구단별 티켓 예매 정책 정리 (yagu.today/tickets) | 좌석 등급별 예상비용 보강자료 | 크롤링 | 무료 | 3차 |
| 장소 검색 API | 카카오 로컬 API (Kakao Developers) | 구장 좌표 기준 반경 내 맛집·카페·핫플레이스 키워드/카테고리 검색 | REST API | 무료 티어 | 3차(텍스트 정리용)~4차(실시간 검색) |
| 장소 검색 API | 네이버 검색 API (지역 검색) | 맛집·핫플 지역정보 보강, 블로그 후기 포함 | REST API | 무료 할당량(일 25,000건) | 3차~4차 |
| 장소 검색 API | 한국관광공사 TourAPI (공공데이터포털) | 관광지·맛집·축제 공공데이터, 좌표 포함 | REST API | 완전 무료(공공데이터) | 3차~4차 |
| 지도 시각화 API | 카카오맵 JS SDK | 구장·맛집·핫플을 지도에 마커로 표시, 코스 시각화 | JS SDK | 무료 티어 | 4차 |
| 지도 시각화 API | 네이버 지도(NCP Maps) JS SDK | 카카오맵 대안/보조 | JS SDK | 무료 티어 | 4차 |
| 경로 계산 API | 티맵(Tmap) Open API (SK Open API 포털, openapi.sk.com) | 자동차/도보/대중교통 길찾기, POI 검색 — 구장→맛집→핫플 코스 이동 동선 계산 | REST API | 개인 개발자 무료 할당량 | 4차 (권장) |
| 경로 계산 API | 네이버 Directions5 | 티맵 대안, 도보/차량 경로·시간 계산 | REST API | 무료 크레딧 | 4차 |
| 경로 계산 API | 카카오 모빌리티 길찾기 API | 사업자 등록이 필요한 유료 상품이라 우선순위 낮음 | REST API | 유료 | 4차 (보류) |

## 2-1. 출처별 수집 방법 상세

- **[크롤링] KBO 공식 구단정보/기록실**: requests + BeautifulSoup로 팀 코드별 URL을 순회하며 선수단 명단(이름, 포지션, 등번호, 주요 기록) 표를 파싱한다.
- **[크롤링, 제한적] KBO 팀 순위**: robots.txt로 차단된 경로이므로 초 단위 자동 수집은 하지 않는다. requests + BeautifulSoup로 순위표만 파싱하고 투수/타자 TOP5 표는 수집하지 않는다. 배치 스케줄러로 하루 1~2회(경기 종료 후)만 실행하고, on/off 플래그를 둔다.
- **[크롤링] 각 구단 홈페이지 구장 안내**: 구단마다 메뉴 URL이 달라 10개 구단 URL을 먼저 목록화한 뒤 크롤링한다. 좌석 구역도가 이미지로만 제공되는 구단은 사람이 직접 보고 텍스트로 정리한다.
- **[크롤링] yagu.today/tickets**: 좌석 등급별 가격표를 BeautifulSoup으로 파싱한다.
- **[API] 카카오 로컬 API**: developers.kakao.com 가입 → 애플리케이션 등록 → REST API 키 발급 → 키워드 장소 검색 엔드포인트(/v2/local/search/keyword.json) 호출.
- **[API] 네이버 검색 API(지역 검색)**: developers.naver.com 가입 → Client ID/Secret 발급 → 지역 검색 엔드포인트(/v1/search/local.json) 호출.
- **[API] 한국관광공사 TourAPI**: data.go.kr 가입 → 활용신청 → 서비스키 발급 → 지역/위치기반 관광정보 엔드포인트 호출.
- **[SDK] 카카오맵 / 네이버 지도 JS SDK**: 각 개발자 포털에서 키 발급 → 프론트엔드 script 태그 삽입(별도 데이터 수집 아님).
- **[API] 티맵 Open API**: openapi.sk.com 가입 → 앱키 발급 → 길찾기 엔드포인트 호출(자동차/도보/대중교통 각각).
- **[API] 네이버 Directions5**: 네이버 클라우드 플랫폼에서 서비스 신청 → Client ID/Secret 발급 후 호출.
- **[API, 보류] 카카오 모빌리티 길찾기**: 사업자 등록 필요해 학생 프로젝트 단계에서는 신청하지 않음.

## 3. 수집 시 주의사항
- KBO 팀 순위 페이지는 robots.txt로 크롤링이 차단된 경로이므로 초 단위 연동 대신 하루 1~2회 배치 수집으로 요청 빈도를 최소화하고, 투수·타자 TOP5는 수집하지 않는다.
- 구단마다 홈페이지 정보 공개 수준이 달라 품질 편차 발생 가능 — 부족한 구단은 공식 SNS·공지사항으로 보완하고 출처·최종 갱신일 표기
- 모든 문서에 고유 ID 부여(추후 추가/수정/삭제/재인덱싱 대비), metadata에 최소 `source`, `team`, `stadium`, `category`, `updated_at` 포함
- 카카오/네이버/티맵/공공데이터포털 API 키는 심사·발급에 며칠 걸릴 수 있으므로 4차 착수 전에 미리 신청

## 4. 수집 우선순위 (1주차 목표)
1. 구장/좌석/교통/반입규정 (10개 구단) — 챗봇 핵심 조회 기능이라 최우선
2. 주요 선수 정보 (10개 구단)
3. 먹거리/주변 맛집·핫플 (신규 메인주제 반영분 포함, 카카오 로컬/TourAPI로 보강 가능)
4. 기초 규칙 요약본 (공통 문서 1개)
5. 팀 순위, 시즌일정, 원정가이드 — 2주차 병행 가능

## 5. 프론트엔드 스택 결정
- Streamlit 프로토타입 단계는 사용하지 않는다. 처음부터 React로 간다.
- 빌드 툴체인(CRA/Vite) 없이 순수 `<script>` + `.html` 방식으로 React를 붙인다 — HTML 파일에 React/ReactDOM(UMD 빌드)을 CDN `<script>`로 로드하고, 같은 파일 또는 별도 `.js`에 컴포넌트 코드를 작성하는 구조.
- 백엔드(Django REST, `/api/chat` 등)는 원래 계획대로 그대로 두고, 이 React 화면이 `fetch`/`axios`로 API를 직접 호출하는 구조로 프로토타입을 만든다.
- 즉 "기획안 4차 몫이던 React"를 3차 프로토타입 단계부터 앞당겨 쓰는 셈이라, 개발 순서는 (1) Django API(RAG 질의응답 엔드포인트) 먼저 동작 확인 → (2) 같은 API를 호출하는 React 채팅 UI(script+html) 붙이기 순서로 진행하면 된다.

## 6. DB 스택 결정: MySQL + Chroma (Neo4j 사용 안 함)

- **Chroma (벡터DB)**: 의미 기반 유사도 검색 전용. 구장 설명, 선수 소개, 반입 규정처럼 자연어로 된 비정형 안내 텍스트를 임베딩해 RAG로 검색한다.
- **MySQL (관계형DB)**: 팀 순위, 직관 기록, 좌석 후기 평점처럼 값이 정확히 맞아야 하는 정형 데이터를 저장·조회한다. 기존 기획안은 PostgreSQL이었지만 팀에서 MySQL 경험이 많아 MySQL로 대체 — Django ORM이 둘 다 지원해서 설정만 바꾸면 되고 스키마는 그대로 재사용 가능하다.
- **Neo4j는 쓰지 않는다**: 그래프 탐색(관계 기반 추천)이 필요한 GraphRAG를 할 때만 필요한데, 우리 서비스는 단순 조회형 질문이 대부분이라 불필요. Notion 가이드의 "처음부터 GraphRAG/Agentic RAG 넣지 않기" 원칙과도 일치한다.
- API 레이어(Django)가 질문 유형을 보고 두 저장소 중 어디를 조회할지 라우팅한다 (정형 데이터 질문 → MySQL 직접 조회 / 비정형 안내 질문 → Chroma RAG 검색).

> **※ 아래 ERD와 테이블 설계는 예시로 보여주기 위한 초안이다.** 실제 필드/관계는 팀 회의 결과에 따라 바뀔 수 있으니 참고용으로만 보고 확정된 내용으로 오해하지 않도록 한다.

### MySQL 테이블 (예시안)

| 테이블 | 주요 필드 | 설명 |
| --- | --- | --- |
| user | user_id(PK), username, email, password_hash, created_at | 회원 계정 |
| team | team_id(PK), team_name, region | 구단 마스터 |
| stadium | stadium_id(PK), stadium_name, open_year, capacity, transport_info, banned_items | 구장 마스터 |
| team_stadium | team_id(PK,FK), stadium_id(PK,FK) | 두산·LG처럼 한 구장을 공동 홈으로 쓰는 경우 대비한 N:M 연결 |
| seat_zone | zone_id(PK), stadium_id(FK), zone_name, view_desc, price_range, has_roof | 구장별 좌석 구역 |
| player | player_id(PK), team_id(FK), name, position, is_recommended | 선수 정보 |
| team_ranking_cache | ranking_id(PK), team_id(FK), rank, wins, losses, draws, updated_at | KBO 팀 순위 배치 수집 캐시 |
| home_schedule | schedule_id(PK), team_id(FK), stadium_id(FK), opponent_team_id(FK), game_date, game_time | 팀별 홈경기 일정 |
| attendance_record | record_id(PK), user_id(FK), stadium_id(FK), opponent_team_id(FK), game_date, result, memo | 사용자 직관 기록 |
| seat_review | review_id(PK), zone_id(FK), user_id(FK), rating, comment, created_at | 구역별 좌석 후기·평점 |
| chat_session | session_id(PK), user_id(FK), started_at | 대화 세션 |
| chat_message | message_id(PK), session_id(FK), role, content, created_at | 대화 메시지 이력 |
| vector_doc_ref | doc_id(PK, VARCHAR), source_table, source_id, category, updated_at | MySQL row ↔ Chroma 청크 매핑 (다형 참조라서 강한 FK는 걸지 않음) |
| attraction_reference (4차 예정) | - | 구장 주변 맛집·핫플레이스 마스터 데이터 |
| course / course_stop (4차 예정) | - | 사용자가 만든 직관 코스와 방문 순서 |

**vector_doc_ref의 역할**: Chroma에 청크를 저장할 때 metadata에 doc_id를 함께 넣고, 이 doc_id로 vector_doc_ref를 조회하면 "이 답변 근거가 MySQL의 어느 테이블/행에서 왔는지" 추적할 수 있다. 특정 구단·구장 데이터가 갱신됐을 때 재인덱싱 대상을 doc_id 기준으로 빠르게 찾을 수 있어 전체 재인덱싱 대신 증분 인덱싱이 가능해진다.

모든 Document(Chroma 청크)에는 최소 `id`, `source`, `team`, `stadium`, `category`, `updated_at` 메타데이터를 부여한다.

## 7. RAG 파이프라인 뼈대

인덱싱: 전처리 → Document(메타데이터) → 청킹(500~1000토큰, overlap 10~20%) → 임베딩 → Chroma persist. 최초 1회 전체 인덱싱, 트레이드/개편시만 증분.

서빙: 서버 기동시 VectorDB/Retriever/LLM 1회 초기화 후 재사용 → 질문임베딩 → Retrieve(top-k 4~5, 메타데이터 필터) → (필요시 고유명사는 Hybrid) → Context 제한(구단당 ~400토큰) → Few-shot 프롬프트 → Streaming 생성. Reranking은 검색품질 문제 확인될 때만 추가. Retrieval/LLM/Total 시간 분리 기록.

### LangChain 체인 구성 개념 요약
- Retriever = "질문(string) → 관련 문서 리스트"를 반환하는 Runnable. `vectorstore.as_retriever(search_kwargs={"k":5,"filter":{...}})`로 생성.
- LCEL로 `{"context": retriever | format_docs, "question": RunnablePassthrough()} | prompt | llm | StrOutputParser()` 형태로 파이프 연결.
- top-k=4~5, 메타데이터 필터, Few-shot 프롬프트, Context 길이 제한, Retrieve→Generate 단순 구조로 시작하는 이유는 각각 "정확도", "비용/속도", "디버깅 용이성"을 확보하기 위함.
