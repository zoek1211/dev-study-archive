# 스프링부트 REST API 강의 한 장 정리 (p-14172) {#spring-rest-api}

이 강의는 글·댓글·회원 API를 만들며 **한 가지 구조를 반복**합니다. 엔티티는 DTO로 감추고, 응답은 RsData로 통일하고, HTTP 코드는 AOP로 맞추고, 동작은 테스트로 보장합니다.

- 요청은 컨트롤러 → 서비스 → 엔티티·리포지터리를 거친 뒤 DTO로 바뀌어 JSON으로 나갑니다.[](source:post-controller)
- 조회를 뺀 응답은 RsData(resultCode, msg, data)이고, ResponseAspect가 resultCode 앞자리로 HTTP 상태코드를 정합니다.[](source:aspect)
- 실패는 예외로 던지고, GlobalExceptionHandler가 404·400·401·409 응답으로 바꿉니다.[](source:handler)
- MockMvc 통합 테스트가 응답 JSON의 모든 필드를 검사하고, 테스트가 끝나면 롤백합니다.[](source:post-test)

@answer scope="강의 1~79강 · 저장소 jhs512/p-14172-2 main(071-7 커밋, 71강까지) · 72강 이후는 숙제라 코드 없음"
@meta locale=ko
@source lecture title="slog.gg 14172 - 스프링부트, REST API (강의 목차·설명)" url=https://www.slog.gg/p/14172
@source repo title="GitHub jhs512/p-14172-2 (강의 코드)" url=https://github.com/jhs512/p-14172-2
@source post-controller title="ApiV1PostController.write" path=src/main/java/com/back/domain/post/post/controller/ApiV1PostController.java lines=63:86
@source rsdata title="RsData" path=src/main/java/com/back/global/rsData/RsData.java lines=5:18
@source aspect title="ResponseAspect" path=src/main/java/com/back/global/aspect/ResponseAspect.java lines=19:40
@source handler title="GlobalExceptionHandler" path=src/main/java/com/back/global/globalExceptionHandler/GlobalExceptionHandler.java lines=20:49
@source post-entity title="Post 엔티티" path=src/main/java/com/back/domain/post/post/entity/Post.java lines=20:56
@source comment-write title="ApiV1PostCommentController.write" path=src/main/java/com/back/domain/post/postComment/controller/ApiV1PostCommentController.java lines=106:124
@source post-test title="ApiV1PostControllerTest 글 작성" path=src/test/java/com/back/domain/post/post/controller/ApiV1PostControllerTest.java lines=23:63
@source cors title="WebMvcConfig CORS" path=src/main/java/com/back/global/webMvc/WebMvcConfig.java lines=8:18
@source member-controller title="ApiV1MemberController" path=src/main/java/com/back/domain/member/member/controller/ApiV1MemberController.java lines=38:64

## 강의는 어떤 순서로 쌓이나요? {#course-map}

@workflow id=lecture-map title="4부 79강 학습 지도"
| lane p1 | 1부 REST API (1~32강) |
| lane p2 | 2부 TDD (33~56강) |
| lane p3 | 3부 Swagger·CORS (57~68강) |
| lane p4 | 4부 연습문제 (69~79강) |
| read | 조회 API | backend | p1 |
| dto | DTO 도입 | backend | p1 |
| rsdata | RsData·Aspect | backend | p1 |
| crud | 작성·수정·삭제 | backend | p1 |
| test | 컨트롤러 테스트 | backend | p2 |
| errors | 전역 예외 처리 | backend | p2 |
| docs | Swagger 문서 | external | p3 |
| cors | CORS 허용 | security | p3 |
| member | 회원 API | backend | p4 |
| author | 작성자·권한 | security | p4 |
| read > dto | 엔티티 노출 |
| dto > rsdata | 응답 통일 |
| rsdata > crud | |
| crud > test | 안전망 |
| test > errors | 실패 케이스 |
| errors > docs | |
| docs > cors | 프론트 연동 |
| cors > member | 종합 |
| member > author | actorId | emphasis |

@steps id=lecture-steps
| s-read | 조회 API | 4~5, 10강. 글·댓글 목록과 단건을 JSON으로 응답합니다. 양방향 관계 때문에 Jackson이 무한 루프에 빠지는 순환참조 문제를 겪습니다. | lecture-map | read |
| s-dto | DTO 도입 | 7~9강. 프론트가 필드 이름을 바꿔 달라고 할 때 엔티티를 고치면 DB까지 바뀝니다. 그래서 PostDto(record)를 두고 엔티티를 감춥니다. | lecture-map | dto |
| s-rsdata | RsData·Aspect | 13~19, 25~28강. resultCode·msg·data를 담는 RsData를 만들고, 제너릭으로 data 타입을 정합니다. ResponseAspect가 HTTP 코드를 맞춥니다. | lecture-map | rsdata |
| s-crud | 작성·수정·삭제 | 11~12, 16, 21~24, 31~32강. GET 삭제를 DELETE로 바꾸고, POST·PUT 본문은 ReqBody record와 @Valid로 받습니다. | lecture-map | crud |
| s-test | 컨트롤러 테스트 | 33~47강. MockMvc로 요청을 보내고 응답을 검증하는 2단계 구조를 익힙니다. 요구사항이 바뀌면 테스트부터 고칩니다. | lecture-map | test |
| s-errors | 전역 예외 처리 | 48~56강. 없는 글은 404, 잘못된 본문은 400으로 응답합니다. 마지막에 댓글 작성을 TDD로 구현합니다. | lecture-map | errors |
| s-docs | Swagger 문서 | 57~64강. SpringDoc이 코드를 읽어 OpenAPI 문서를 만들고, Swagger UI에서 바로 호출해 볼 수 있습니다. | lecture-map | docs |
| s-cors | CORS 허용 | 65~68강. 다른 출처에서 fetch하면 브라우저가 막습니다. 서버가 허용할 출처를 알려 줘야 합니다. | lecture-map | cors |
| s-member | 회원 API | 69~71강. 회원가입(409 중복)과 내 정보(401 미인증)를 구현합니다. 저장소 코드는 여기까지입니다. | lecture-map | member |
| s-author | 작성자·권한 | 72~79강. 글·댓글에 작성자를 붙이고, 본인만 수정·삭제하게 만드는 숙제입니다. | lecture-map | author |

## 글 작성 요청 하나는 어떻게 처리되나요? {#write-flow}

@sequence id=write-sequence title="POST /api/v1/posts 처리 순서"
| actor client | 프론트·Postman | frontend |
| actor mvc | Spring MVC·Jackson | backend |
| actor controller | ApiV1PostController | backend |
| actor service | PostService | backend |
| actor db | PostRepository·H2 | database |
| actor aspect | ResponseAspect | backend |
| phase | 요청 해석 |
| client | mvc | POST JSON {title, content} |
| mvc | controller | PostWriteReqBody (@Valid 통과) |
| phase | 비즈니스 로직 |
| controller | service | write(title, content) |
| service | db | save(new Post) |
| db | service | Post (id 발급) | return |
| service | controller | Post | return |
| phase | 응답 만들기 |
| controller | aspect | RsData("201-1", PostDto) | return |
| aspect | mvc | setStatus(201) | emphasis |
| mvc | client | 201 + JSON | return |

@steps id=write-steps
| w-client | 프론트·Postman | 본문은 JSON입니다. Content-Type: application/json 헤더가 필요합니다. | write-sequence | client |
| w-mvc | Spring MVC·Jackson | @RequestBody가 JSON을 PostWriteReqBody로 바꿉니다(역직렬화). @Valid 검사에 실패하면 컨트롤러는 실행되지 않고 400이 나갑니다. | write-sequence | mvc |
| w-controller | ApiV1PostController | 서비스를 부르고, 결과 엔티티를 PostDto로 감싸 RsData에 담습니다. @Transactional이 붙어 있습니다. | write-sequence | controller |
| w-service | PostService | new Post를 만들어 저장합니다. 비즈니스 규칙이 늘어나면 여기에 들어갑니다. | write-sequence | service |
| w-db | PostRepository·H2 | IDENTITY 전략이라 save 시점에 INSERT가 나가고 id가 생깁니다. | write-sequence | db |
| w-aspect | ResponseAspect | RsData를 돌려주는 액션 메서드를 감쌉니다. "201-1"의 앞자리 201을 HTTP 상태코드로 설정합니다. | write-sequence | aspect |

@code source=post-controller language=java

## 코드는 어떤 부품으로 나뉘나요? {#structure}

@architecture id=code-structure title="p-14172-2 구성 요소"
| client | 프론트 앱·Postman | frontend |
| cors | WebMvcConfig (CORS) | security |
| ctrl | ApiV1*Controller | backend |
| aspect | ResponseAspect | backend |
| advice | GlobalExceptionHandler | backend |
| svc | PostService·MemberService | backend |
| entity | Post·PostComment·Member | backend |
| repo | JpaRepository | backend |
| db | H2 (dev 파일·test 메모리) | database |
| dto | DTO·RsData | backend |
| docs | SpringDoc·Swagger UI | external |
| client > cors | fetch |
| cors > ctrl | 허용된 출처 |
| ctrl > svc | 호출 |
| svc > entity | 상태 변경 |
| svc > repo | 저장·조회 |
| repo > db | SQL |
| ctrl > dto | 변환 |
| aspect > ctrl | 상태코드 | dashed |
| advice > ctrl | 예외 처리 | dashed |
| docs > ctrl | 문서 생성 | dashed |

| 패키지 | 역할 | 대표 파일 |
| --- | --- | --- |
| `domain/post/post` | 글: 컨트롤러·서비스·엔티티·DTO·리포지터리 | `ApiV1PostController`, `Post`, `PostDto` |
| `domain/post/postComment` | 댓글: 글 서비스와 글 엔티티를 거쳐 다룸 | `ApiV1PostCommentController`, `PostComment` |
| `domain/member/member` | 회원가입·내 정보 | `ApiV1MemberController`, `MemberService` |
| `global/rsData` | 공통 응답 형식 | `RsData` |
| `global/aspect` | RsData 응답의 HTTP 코드 맞추기 | `ResponseAspect` |
| `global/globalExceptionHandler` | 예외 → 실패 응답 | `GlobalExceptionHandler` |
| `global/jpa/entity` | id·생성일·수정일 공통 필드 | `BaseEntity` |
| `global/initData` | 글 3개·댓글 5개 샘플 데이터 | `BaseInitData` |
| `global/webMvc`, `global/springDoc` | CORS, API 문서 설정 | `WebMvcConfig`, `SpringDocConfig` |

댓글 리포지터리는 따로 없습니다. 댓글은 Post 엔티티의 comments 리스트로 추가·삭제하고, cascade와 orphanRemoval이 DB에 반영합니다.[](source:post-entity)

## 테이블은 어떻게 생겼나요? {#data-model}

@erd id=board-model title="현재 코드의 테이블 (71강 기준)"
| entity member | Member |
| entity post | Post |
| entity post_comment | PostComment |
| member.id | int | pk |
| member.username | varchar | uk |
| member.password | varchar |
| member.name | varchar |
| member.create_date | datetime |
| member.modify_date | datetime |
| post.id | int | pk |
| post.title | varchar |
| post.content | varchar |
| post.create_date | datetime |
| post.modify_date | datetime |
| post_comment.id | int | pk |
| post_comment.post_id | int | fk | post.id |
| post_comment.content | varchar |
| post_comment.create_date | datetime |
| post_comment.modify_date | datetime |
| post_comment > post | many:one | 댓글이 속한 글 |

@callout level=note title="member와 글은 아직 연결되지 않았습니다"
문제 4(73강)에서 post와 post_comment에 author(작성자) 외래키를 추가하는 것이 숙제입니다.
id, create_date, modify_date는 BaseEntity에서 상속받는 공통 칼럼입니다.

@code source=post-entity language=java

## 응답은 왜 RsData로 감싸나요? {#rsdata}

작성·수정·삭제는 성공 여부와 메시지를 알려 줘야 해서 RsData를 씁니다. 조회는 실패가 드물어서 DTO를 그대로 돌려줍니다(14강). 조회에 RsData를 써도 틀리지 않지만 과합니다.

| 쓰는 형태 | 언제 | 예 |
| --- | --- | --- |
| `RsData<Void>` | 성공·실패 여부만 알릴 때 | 글 수정·삭제, 회원가입 |
| `RsData<PostDto>` | 결과 데이터 하나를 함께 줄 때 | 글 작성 |
| `RsData<XxxResBody>` | 돌려줄 값이 둘 이상일 때 | 25강의 totalCount (40강에서 제거) |
| DTO 그대로 | 조회 성공 | 글 목록·단건, 내 정보 |

resultCode "201-1"은 "HTTP 코드-세부 상황 번호"입니다. 생성자가 앞자리를 잘라 statusCode에 넣고, statusCode는 @JsonIgnore라 JSON에는 나가지 않습니다.[](source:rsdata)

@code source=rsdata language=java

resultCode만 바꿔서는 HTTP 응답코드가 바뀌지 않습니다(27강). 매번 ResponseEntity를 쓰는 대신, ResponseAspect가 RsData를 돌려주는 모든 액션 메서드 뒤에서 상태코드를 설정합니다(28강).

@code source=aspect language=java

## 실패하면 어떤 응답이 나가나요? {#errors}

컨트롤러는 실패를 직접 처리하지 않고 예외를 던집니다. GlobalExceptionHandler가 예외 종류별로 RsData와 HTTP 코드를 만듭니다.

| 상황 | 예외 | HTTP · resultCode |
| --- | --- | --- |
| 없는 글·댓글 번호 | `NoSuchElementException` (Optional.get) | 404 · 404-1 |
| 제목·내용 누락, 길이 위반 | `MethodArgumentNotValidException` (@Valid) | 400 · 400-1 |
| JSON 형식이 깨짐 | `HttpMessageNotReadableException` | 400 · 400-1 |
| username 중복 | `MemberDuplicateUsernameException` | 409 · 409-1 |
| actorId 없음·없는 회원 | `UnauthenticatedException` | 401 · 401-1 |
| 작성자가 아님 (숙제) | `AccessDeniedException` (아직 없음) | 403 · 403-1 |

@code source=handler language=java

@callout level=note title="예외 응답에는 ResponseAspect가 끼지 않습니다"
핸들러는 ResponseEntity로 상태코드를 직접 정합니다.
48~49강은 @ResponseStatus와 ResponseEntity 둘 다 정답이라고 설명합니다.
Aspect를 예외 핸들러까지 넓히는 실험은 lab/049-exception-handler 브랜치에 있습니다.

## 헷갈리기 쉬운 JPA·웹 개념은? {#concepts}

| 질문 | 답 | 강의 |
| --- | --- | --- |
| 왜 엔티티를 바로 응답하지 않나요? | 프론트 요구로 이름을 바꿀 때 엔티티와 DB를 건드리지 않으려고, 그리고 Jackson 순환참조를 피하려고 DTO를 씁니다. | 4, 7~9강 |
| 왜 액션 메서드에 @Transactional을 붙이나요? | 더티체킹과 orphanRemoval은 트랜잭션 안에서만 DB에 반영됩니다. 없으면 댓글 삭제가 실패합니다. | 11~12강 |
| 글을 지우면 댓글은요? | cascade REMOVE가 댓글을 먼저 지웁니다. | 16강 |
| 댓글 작성 직후 id가 0인 이유는? | INSERT가 트랜잭션 끝까지 미뤄집니다. flush()로 바로 INSERT합니다. flush는 커밋이 아니라 롤백할 수 있습니다. | 56강 |
| 경로에 v1을 넣는 이유는? | API 버저닝입니다. 새 버전을 내도 기존 클라이언트가 계속 동작합니다. 타임리프는 서버가 화면까지 만들어서 필요 없습니다. | 6강 |
| 작성 성공은 201이어야 하나요? | 201이 가장 정확하지만 200이어도 문제는 없습니다. 이 강의는 Aspect로 201을 맞춥니다. | 26~28강 |
| BindingResult는 왜 안 쓰나요? | REST API에서는 검증 실패를 예외로 받아 전역 핸들러에서 처리합니다. 입력 검증은 프론트도 합니다. | 23, 50강 |
| CORS는 누가 막나요? | 브라우저입니다. 다른 출처로 fetch하면 서버가 allowedOrigins로 허락해야 응답을 읽을 수 있습니다. | 66~68강 |

댓글 작성은 flush가 왜 필요한지 보여 주는 코드입니다.[](source:comment-write)

@code source=comment-write language=java

## 컨트롤러 테스트는 어떻게 쓰나요? {#testing}

@steps id=test-steps
| t-setup | 환경 준비 | @SpringBootTest로 전체 앱을 띄우고, @ActiveProfiles("test")로 메모리 DB를 씁니다. @Transactional이 테스트마다 롤백합니다. |
| t-request | 요청을 날린다 | MockMvc.perform으로 서버 없이 HTTP 요청을 흉내 냅니다. andDo(print())로 요청과 응답을 출력합니다. |
| t-handler | 어떤 메서드가 받았나 | handler().handlerType과 methodName으로 의도한 액션 메서드가 실행됐는지 확인합니다. |
| t-status | 상태코드 | status().isCreated() 처럼 HTTP 코드를 검사합니다. |
| t-fields | 모든 JSON 필드 | jsonPath로 resultCode, msg, data의 모든 필드를 검사합니다. 프론트와의 약속이 깨지면 바로 알 수 있습니다. |

@code source=post-test language=java

| 검사 범위 | 강의의 판단 |
| --- | --- |
| 응답 JSON의 모든 필드가 있고 형식이 맞는지 | {good:필수} |
| DB를 다시 조회해 값이 정말 맞는지 | {caution:선택} 서비스 테스트에서 해도 됨 (41, 44강) |
| 실패 시나리오 전부 | {caution:대표 1개} 가성비를 따짐 (53강) |
| 모키토로 의존성 가짜 만들기 | {caution:꼭 필요할 때만} 롤백으로 충분한 경우가 많음 (34강) |

## 어떤 엔드포인트가 있나요? {#endpoints}

| 메서드 · 경로 | 하는 일 | 성공 응답 |
| --- | --- | --- |
| `GET /api/v1/posts` | 글 목록 | 200 · `List<PostDto>` |
| `GET /api/v1/posts/{id}` | 글 하나 | 200 · `PostDto` |
| `POST /api/v1/posts` | 글 작성 | 201 · `RsData<PostDto>` |
| `PUT /api/v1/posts/{id}` | 글 수정 | 200 · `RsData<Void>` |
| `DELETE /api/v1/posts/{id}` | 글 삭제 | 200 · `RsData<Void>` |
| `GET /api/v1/posts/{postId}/comments` | 댓글 목록 | 200 · `List<PostCommentDto>` |
| `GET /api/v1/posts/{postId}/comments/{id}` | 댓글 하나 | 200 · `PostCommentDto` |
| `POST /api/v1/posts/{postId}/comments` | 댓글 작성 | 201 · `RsData<PostCommentDto>` |
| `PUT·DELETE /api/v1/posts/{postId}/comments/{id}` | 댓글 수정·삭제 | 200 · `RsData<Void>` |
| `POST /api/v1/members/join` | 회원가입 | 201 · `RsData<Void>` |
| `GET /api/v1/members/me?actorId=1` | 내 정보 | 200 · `MemberDto` |

직접 돌려 볼 때는 `http://localhost:8080/swagger-ui/index.html`에서 호출하거나, H2 콘솔(`jdbc:h2:./db_dev;MODE=MySQL`, 사용자 sa)에서 데이터를 확인합니다.

```bash title="코드 받아서 실행하기"
git clone https://github.com/jhs512/p-14172-2
cd p-14172-2
./gradlew bootRun
```

## 4부 연습문제, 무엇을 확인하며 풀까요? {#practice}

문제 1·2는 70·71강에 정답 커밋이 있습니다.[](source:member-controller) 문제 3부터는 혼자 풀어야 하고, 모든 구현은 TDD로 합니다. actorId가 없으면 401, 작성자가 아니면 403입니다.

- [ ] 문제 1: POST /api/v1/members/join, 중복 username이면 409를 테스트로 확인했다
- [ ] 문제 2: GET /api/v1/members/me 응답에 password가 없다
- [ ] 문제 3: 샘플 회원 6명(system, admin, user1~4)을 비밀번호 12345678로 만든다
- [ ] 문제 4: 글·댓글 엔티티에 author 필드를 추가하고 샘플 작성자를 지정했다
- [ ] 문제 5·6: 글·댓글 작성에 actorId가 없으면 401-1이 나온다
- [ ] 문제 7·8: 남의 글 수정·삭제는 403-1이 나온다
- [ ] 문제 9·10: 남의 댓글 수정·삭제는 403-1이 나온다
- [ ] 각 문제마다 성공 1개, 대표 실패 1개 테스트가 통과한다
