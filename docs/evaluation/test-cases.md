# Test Cases — RESTful API Guidelines Skill Evaluation

**작성일:** 2026-03-20
**대상 스킬:** `.claude/skills/restful-api-guidelines.md`
**참조:** `docs/evaluation/coverage-map.md`

---

## 섹션 2: HTTP 기본 규칙

### TC-2-01: URL 소문자 kebab-case

- 규칙: "✅ **필수**: URL 경로에는 소문자 kebab-case를 사용한다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: COVERED

❌ Bad:
```kotlin
@RestController
@RequestMapping("/userProfiles")
class UserProfileController {

    @GetMapping("/{id}/paymentMethods")
    fun getPaymentMethods(@PathVariable id: String): ResponseEntity<List<PaymentMethod>> {
        // URL에 camelCase 사용 — 금지
        val methods = paymentService.findByUserId(id)
        return ResponseEntity.ok(methods)
    }
}
```

✅ Good:
```kotlin
@RestController
@RequestMapping("/user-profiles")
class UserProfileController {

    @GetMapping("/{id}/payment-methods")
    fun getPaymentMethods(@PathVariable id: String): ResponseEntity<List<PaymentMethod>> {
        val methods = paymentService.findByUserId(id)
        return ResponseEntity.ok(methods)
    }
}
```

- 검증 포인트: Writing 모드의 "Plural nouns + kebab-case" 주석 및 `/user-profiles` 예시, Review 체크리스트의 "Lowercase kebab-case used in paths" 항목

---

### TC-2-01a: URL 허용 문자 제한 (ASCII 영소문자/숫자/하이픈만)

- 규칙: "URL 경로 세그먼트에는 ASCII 영소문자, 숫자, 하이픈(`-`)만 사용한다"
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: PARTIAL / Review: PARTIAL
  (coverage-map.md 2.1-5 기준)

❌ Bad:
```
GET /사용자_프로필          # 비ASCII 문자
GET /User-Profiles       # 대문자
GET /user_profiles       # 언더스코어
GET /userProfiles        # camelCase
```

✅ Good:
```
GET /user-profiles
GET /article-categories/123
```

- 검증 포인트: Review 체크리스트 "Lowercase kebab-case used in paths"가 kebab-case를 다루지만 허용 문자 집합(ASCII 영소문자/숫자/하이픈만) 명시가 없어 PARTIAL. 비ASCII 문자 사용 케이스는 탐지 불가.

---

### TC-2-02: 리소스 컬렉션 복수형 명사

- 규칙: "✅ **필수**: 리소스 컬렉션 이름은 복수형 명사를 사용한다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: COVERED

❌ Bad:
```kotlin
@RestController
@RequestMapping("/article")
class ArticleController {

    @GetMapping("/{id}/comment")
    fun getComments(@PathVariable id: String): ResponseEntity<List<Comment>> {
        val comments = commentService.findByArticleId(id)
        return ResponseEntity.ok(comments)
    }
}
```

✅ Good:
```kotlin
@RestController
@RequestMapping("/articles")
class ArticleController {

    @GetMapping("/{id}/comments")
    fun getComments(@PathVariable id: String): ResponseEntity<List<Comment>> {
        val comments = commentService.findByArticleId(id)
        return ResponseEntity.ok(comments)
    }
}
```

- 검증 포인트: Writing 모드의 `/articles`, `/users/{userId}/comments` 복수형 예시, Review 체크리스트의 "Resource names are plural nouns" 항목

---

### TC-2-03: URL에 동사 포함 금지

- 규칙: "❌ **금지**: URL에 동사를 포함하지 않는다. 액션은 HTTP 메서드로 표현한다."
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: PARTIAL / Review: COVERED

❌ Bad:
```kotlin
@RestController
class UserController {

    @PostMapping("/createUser")
    fun createUser(@RequestBody request: CreateUserRequest): ResponseEntity<User> {
        val user = userService.create(request)
        return ResponseEntity.ok(user)
    }

    @GetMapping("/getArticles")
    fun getArticles(): ResponseEntity<List<Article>> {
        return ResponseEntity.ok(articleService.findAll())
    }

    @DeleteMapping("/deleteComment/{id}")
    fun deleteComment(@PathVariable id: String): ResponseEntity<Void> {
        commentService.delete(id)
        return ResponseEntity.noContent().build()
    }
}
```

✅ Good:
```kotlin
@RestController
class UserController {

    @PostMapping("/users")
    fun createUser(@RequestBody request: CreateUserRequest): ResponseEntity<User> {
        val user = userService.create(request)
        val location = URI.create("/users/${user.id}")
        return ResponseEntity.created(location).body(user)
    }
}

@RestController
@RequestMapping("/articles")
class ArticleController {

    @GetMapping
    fun getArticles(): ResponseEntity<List<Article>> {
        return ResponseEntity.ok(articleService.findAll())
    }
}

@RestController
@RequestMapping("/comments")
class CommentController {

    @DeleteMapping("/{id}")
    fun deleteComment(@PathVariable id: String): ResponseEntity<Void> {
        commentService.delete(id)
        return ResponseEntity.noContent().build()
    }
}
```

- 검증 포인트: Writing 모드에 `:action` 패턴 예시는 있으나 "동사 금지" 규칙 자체 명시 없음(PARTIAL), Review 체크리스트의 "No verbs in paths (actions use :action pattern)" 항목

---

### TC-2-04: URL 파일 확장자 금지

- 규칙: "❌ **금지**: URL에 파일 확장자(`.json`, `.xml`)를 포함하지 않는다. 콘텐츠 협상은 `Accept` 헤더를 사용한다."
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: MISSING / Review: COVERED

❌ Bad:
```kotlin
@RestController
class ArticleController {

    @GetMapping("/articles.json")
    fun getArticlesJson(): ResponseEntity<List<Article>> {
        return ResponseEntity.ok(articleService.findAll())
    }

    @GetMapping("/articles/{id}.xml")
    fun getArticleXml(@PathVariable id: String): ResponseEntity<Article> {
        return ResponseEntity.ok(articleService.findById(id))
    }
}
```

✅ Good:
```kotlin
@RestController
@RequestMapping("/articles")
class ArticleController {

    @GetMapping(produces = [MediaType.APPLICATION_JSON_VALUE])
    fun getArticles(
        @RequestHeader("Accept", defaultValue = "application/json") accept: String
    ): ResponseEntity<List<Article>> {
        return ResponseEntity.ok(articleService.findAll())
    }

    @GetMapping("/{id}")
    fun getArticle(@PathVariable id: String): ResponseEntity<Article> {
        return ResponseEntity.ok(articleService.findById(id))
    }
}
```

- 검증 포인트: Writing 모드에 파일 확장자 금지 언급 없음(MISSING), Review 체크리스트의 "No file extensions in URLs" 항목

---

### TC-2-05: URL 2000자 이하

- 규칙: "⚠️ **권장**: URL은 2000자 이하로 유지한다. 그 이상이 필요한 경우 쿼리 파라미터를 요청 본문으로 이동하는 것을 고려한다."
- 규범 수준: ⚠️권장
- 대상 모드: Both
- 스킬 커버: Writing: MISSING / Review: MISSING

❌ Bad:
```kotlin
// 수십 개의 필터 파라미터를 모두 쿼리스트링으로 전달 — URL이 2000자를 초과할 수 있음
@GetMapping("/reports")
fun searchReports(
    @RequestParam status: List<String>,     // ?status=A&status=B&...&status=Z
    @RequestParam category: List<String>,   // &category=...
    @RequestParam tag: List<String>,        // &tag=...  (수백 개)
    @RequestParam region: List<String>      // &region=...
): ResponseEntity<List<Report>> {
    return ResponseEntity.ok(reportService.search(status, category, tag, region))
}

// GET /reports?status=A&status=B&...&tag=tag1&tag=tag2&...&tag=tag200&region=...
// → URL 길이 2000자 초과
```

✅ Good:
```kotlin
// 복잡한 필터 조건은 POST 검색 엔드포인트로 이동
@PostMapping("/reports:search")
fun searchReports(@RequestBody request: ReportSearchRequest): ResponseEntity<List<Report>> {
    return ResponseEntity.ok(reportService.search(request))
}

data class ReportSearchRequest(
    val status: List<String>? = null,
    val category: List<String>? = null,
    val tag: List<String>? = null,
    val region: List<String>? = null
)
```

- 검증 포인트: 스킬의 Writing/Review 모드 모두에서 URL 길이 제한 언급 없음(MISSING)

---

### TC-2-06: 201 Created + Location 헤더

- 규칙: "✅ **필수**: 201 Created 응답에는 `Location` 헤더로 생성된 리소스의 URL을 포함한다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: COVERED

❌ Bad:
```kotlin
@PostMapping("/articles")
fun createArticle(@RequestBody request: CreateArticleRequest): ResponseEntity<Article> {
    val article = articleService.create(request)
    // Location 헤더 없이 200 OK 반환 — 잘못된 패턴
    return ResponseEntity.ok(article)
}
```

✅ Good:
```kotlin
@PostMapping("/articles")
fun createArticle(@RequestBody request: CreateArticleRequest): ResponseEntity<Article> {
    val article = articleService.create(request)
    val location = URI.create("/articles/${article.id}")
    return ResponseEntity.created(location).body(article)
}
```

- 검증 포인트: Writing 모드의 "201 Created + Location header" 명시 및 `ResponseEntity.created(location)` 코드 예시, Review 체크리스트의 "POST create → 201 + Location header" 항목

---

### TC-2-07: GET/HEAD/DELETE에 body 포함 금지

- 규칙: "❌ **금지**: GET, HEAD, DELETE 요청에 요청 본문(body)을 포함하지 않는다."
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: MISSING / Review: MISSING

❌ Bad:
```kotlin
@RestController
@RequestMapping("/articles")
class ArticleController {

    // GET에 body 포함 — 금지
    @GetMapping("/search")
    fun searchArticles(@RequestBody request: SearchRequest): ResponseEntity<List<Article>> {
        return ResponseEntity.ok(articleService.search(request))
    }

    // DELETE에 body 포함 — 금지
    @DeleteMapping("/{id}")
    fun deleteArticle(
        @PathVariable id: String,
        @RequestBody reason: DeleteReasonRequest
    ): ResponseEntity<Void> {
        articleService.delete(id, reason)
        return ResponseEntity.noContent().build()
    }
}
```

✅ Good:
```kotlin
@RestController
@RequestMapping("/articles")
class ArticleController {

    // 검색 조건은 쿼리 파라미터로 전달
    @GetMapping
    fun searchArticles(
        @RequestParam(required = false) q: String?,
        @RequestParam(required = false) status: String?
    ): ResponseEntity<List<Article>> {
        return ResponseEntity.ok(articleService.search(q, status))
    }

    // DELETE는 body 없이 처리, 부가 정보가 필요하면 별도 액션으로 분리
    @DeleteMapping("/{id}")
    fun deleteArticle(@PathVariable id: String): ResponseEntity<Void> {
        articleService.delete(id)
        return ResponseEntity.noContent().build()
    }
}
```

- 검증 포인트: 스킬의 Writing/Review 모드 모두에서 GET/HEAD/DELETE body 금지 규칙 없음(MISSING)

---

### TC-2-08: 오류 상황에 200 OK 반환 금지

- 규칙: "❌ **금지**: 오류 상황에 200 OK를 반환하지 않는다."
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: MISSING / Review: COVERED

❌ Bad:
```kotlin
@GetMapping("/articles/{id}")
fun getArticle(@PathVariable id: String): ResponseEntity<Map<String, Any>> {
    val article = articleService.findById(id)
    if (article == null) {
        // 오류 상황인데 200 OK 반환 — 금지
        return ResponseEntity.ok(mapOf(
            "success" to false,
            "error" to "Article not found"
        ))
    }
    return ResponseEntity.ok(mapOf("success" to true, "data" to article))
}
```

✅ Good:
```kotlin
@GetMapping("/articles/{id}")
fun getArticle(@PathVariable id: String): ResponseEntity<Article> {
    val article = articleService.findById(id)
        ?: throw ApiException(
            status = HttpStatus.NOT_FOUND,
            type = "https://api.example.com/errors/resource-not-found",
            title = "Resource not found",
            detail = "The requested article could not be found.",
            instance = "/articles/$id"
        )
    return ResponseEntity.ok(article)
}
```

- 검증 포인트: Writing 모드에 "200 OK로 에러 반환 금지" 규칙 명시 없음(MISSING), Review 체크리스트의 "200 not returned for error conditions" 항목

---

### TC-2-09: 서버 상태 변경에 쿼리 파라미터 사용 금지

- 규칙: "❌ **금지**: 서버 상태를 변경하는 작업에 쿼리 파라미터를 사용하지 않는다."
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: MISSING / Review: MISSING

❌ Bad:
```kotlin
// 쿼리 파라미터로 서버 상태 변경 — 금지
@PostMapping("/articles")  // GET → POST로 변경
fun publishArticle(@RequestParam action: String, @RequestParam id: String): ResponseEntity<Article> {
    if (action == "publish") {
        return ResponseEntity.ok(articleService.publish(id))
    }
    return ResponseEntity.badRequest().build()
}
```

✅ Good:
```kotlin
// 상태 변경은 POST + 액션 패턴 사용
@PostMapping("/articles/{id}:publish")
fun publishArticle(@PathVariable id: String): ResponseEntity<Article> {
    val article = articleService.publish(id)
    return ResponseEntity.ok(article)
}

@PostMapping("/articles/{id}:archive")
fun archiveArticle(@PathVariable id: String): ResponseEntity<Article> {
    val article = articleService.archive(id)
    return ResponseEntity.ok(article)
}
```

- 검증 포인트: 스킬의 Writing/Review 모드 모두에서 서버 상태 변경에 쿼리 파라미터 사용 금지 규칙 없음(MISSING)

---

### TC-2-10: 동일 파라미터 반복으로 배열 값 전달

- 규칙: "✅ **필수**: 동일한 파라미터 이름을 반복하여 배열 값을 전달한다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: MISSING / Review: PARTIAL

❌ Bad:
```kotlin
// 쉼표 구분 문자열로 배열 값 전달 — 비표준
@GetMapping("/articles")
fun getArticles(
    @RequestParam tags: String  // ?tags=tech,design,backend
): ResponseEntity<List<Article>> {
    val tagList = tags.split(",")
    return ResponseEntity.ok(articleService.findByTags(tagList))
}

// 또는 brackets 표기법 — 비표준
// GET /articles?tags[]=tech&tags[]=design
```

✅ Good:
```kotlin
// 동일 파라미터를 반복하여 배열 전달
@GetMapping("/articles")
fun getArticles(
    @RequestParam(required = false) tag: List<String>?
    // GET /articles?tag=tech&tag=design&tag=backend
): ResponseEntity<List<Article>> {
    return ResponseEntity.ok(articleService.findByTags(tag ?: emptyList()))
}
```

- 검증 포인트: Writing 모드에 동일 파라미터 반복으로 배열 전달 패턴 없음(MISSING), Review 체크리스트의 "Repeated same parameter treated as OR condition" 항목은 의미(OR 조건)를 다루며 인코딩 방식은 간접적(PARTIAL)

---

### TC-2-11: Content-Type 헤더 포함

- 규칙: "✅ **필수**: 요청 본문이 있는 경우 `Content-Type` 헤더를 포함한다." / "✅ **필수**: 응답 본문이 있는 경우 `Content-Type` 헤더를 포함한다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: PARTIAL / Review: MISSING

❌ Bad:
```kotlin
// Bad — Content-Type 헤더를 명시하지 않고 응답 본문을 반환
@PostMapping
fun createArticle(@RequestBody request: CreateArticleRequest): ResponseEntity<Article> {
    val article = articleService.create(request)
    val location = URI.create("/articles/${article.id}")
    // produces 없이 ResponseEntity로만 반환 시 프레임워크 설정에 따라 Content-Type 누락 가능
    return ResponseEntity.created(location).body(article)
}

// 또는 raw HttpServletResponse 사용 시:
@PostMapping("/raw")
fun createArticleRaw(response: HttpServletResponse, @RequestBody request: CreateArticleRequest) {
    val article = articleService.create(request)
    // Content-Type 헤더 누락
    response.outputStream.write(objectMapper.writeValueAsBytes(article))
}
```

✅ Good:
```kotlin
// Good — Content-Type 명시
@PostMapping(produces = [MediaType.APPLICATION_JSON_VALUE])
fun createArticle(@RequestBody request: CreateArticleRequest): ResponseEntity<Article> {
    val article = articleService.create(request)
    val location = URI.create("/articles/${article.id}")
    return ResponseEntity.created(location)
        .contentType(MediaType.APPLICATION_JSON)
        .body(article)
}
```

- 검증 포인트: Spring Boot는 `@RestController` + `produces` 설정으로 자동 처리되나, HTTP 규칙상 응답 본문이 있는 모든 응답에 Content-Type 헤더가 있어야 함. Writing 모드는 에러 응답 `application/problem+json`만 언급하고 일반 요청/응답 Content-Type 규칙 없음(PARTIAL), Review 모드에 요청/응답 Content-Type 체크 항목 없음(MISSING)

---

## 섹션 3: REST 원칙

### TC-3-01: null 값 필드 응답 포함 금지

- 규칙: "❌ **금지**: 응답에 null 값 필드를 포함하지 않는다. 값이 없는 필드는 응답에서 제외한다."
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: MISSING / Review: COVERED

❌ Bad:
```kotlin
data class ArticleResponse(
    val id: String,
    val title: String,
    val content: String,
    val deletedAt: String? = null,   // null 값이 응답에 포함됨
    val publishedAt: String? = null  // null 값이 응답에 포함됨
)

// JSON 직렬화 결과:
// { "id": "123", "title": "제목", "content": "...", "deletedAt": null, "publishedAt": null }
```

✅ Good:
```kotlin
@JsonInclude(JsonInclude.Include.NON_NULL)
data class ArticleResponse(
    val id: String,
    val title: String,
    val content: String,
    val deletedAt: String? = null,
    val publishedAt: String? = null
)

// JSON 직렬화 결과 (null 필드 제외):
// { "id": "123", "title": "제목", "content": "..." }
```

- 검증 포인트: Writing 모드에 null 값 필드 제외 규칙 없음(MISSING), Review 체크리스트의 "Null-valued fields excluded from response" 항목

---

### TC-3-02: 읽기 전용 필드 요청 본문 무시

- 규칙: "✅ **필수**: 서버가 관리하는 읽기 전용 필드(`id`, `createdAt`, `updatedAt`)를 클라이언트가 요청 본문에 포함하더라도 이를 무시한다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: PARTIAL / Review: MISSING

❌ Bad:
```kotlin
@PostMapping("/articles")
fun createArticle(@RequestBody request: Map<String, Any>): ResponseEntity<Article> {
    // 클라이언트가 보낸 id, createdAt을 그대로 사용 — 금지
    val article = Article(
        id = request["id"] as? String ?: UUID.randomUUID().toString(),
        title = request["title"] as String,
        createdAt = request["createdAt"] as? Instant ?: Instant.now(),
        updatedAt = request["updatedAt"] as? Instant ?: Instant.now()
    )
    articleRepository.save(article)
    val location = URI.create("/articles/${article.id}")
    return ResponseEntity.created(location).body(article)
}
```

✅ Good:
```kotlin
// 클라이언트 요청 (id 포함)
// POST /articles
// { "id": "hack-123", "title": "제목", "content": "내용", "createdAt": "2020-01-01T00:00:00Z" }
//
// 서버는 위 요청에서 id, createdAt을 무시하고 서버가 직접 생성함

@PostMapping("/articles")
fun createArticle(@RequestBody request: CreateArticleRequest): ResponseEntity<Article> {
    // 클라이언트가 id, createdAt, updatedAt을 보내더라도 무시하고 서버가 생성
    val article = Article(
        id = UUID.randomUUID().toString(),
        title = request.title,
        content = request.content,
        createdAt = Instant.now(),
        updatedAt = Instant.now()
    )
    articleRepository.save(article)
    val location = URI.create("/articles/${article.id}")
    return ResponseEntity.created(location).body(article)
}

data class CreateArticleRequest(
    val title: String,
    val content: String
    // id, createdAt, updatedAt은 요청 DTO에 포함하지 않음
)
```

- 검증 포인트: Writing 모드의 Standard Resource Fields에 "not modifiable by client" 언급 있으나 무시 동작 명시 불충분(PARTIAL), Review 체크리스트에 읽기 전용 필드 무시 동작 체크 없음(MISSING)

---

### TC-3-03: RFC 7807 에러 응답 구조

- 규칙: "✅ **필수**: 모든 에러 응답은 RFC 7807 / RFC 9457 (Problem Details for HTTP APIs) 표준을 따른다." / "✅ **필수**: 에러 응답의 `Content-Type`은 `application/problem+json`을 사용한다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: COVERED

❌ Bad:
```kotlin
// 비표준 에러 응답 구조
@ExceptionHandler(ResourceNotFoundException::class)
fun handleNotFound(ex: ResourceNotFoundException): ResponseEntity<Map<String, Any>> {
    return ResponseEntity.status(HttpStatus.NOT_FOUND)
        .contentType(MediaType.APPLICATION_JSON)  // application/json — 잘못됨
        .body(mapOf(
            "success" to false,
            "message" to ex.message,
            "code" to "NOT_FOUND"
        ))
}
```

✅ Good:
```kotlin
@ExceptionHandler(ResourceNotFoundException::class)
fun handleNotFound(
    ex: ResourceNotFoundException,
    request: HttpServletRequest
): ResponseEntity<ProblemDetail> {
    return ResponseEntity.status(HttpStatus.NOT_FOUND)
        .contentType(MediaType.parseMediaType("application/problem+json"))
        .body(ProblemDetail(
            type = "https://api.example.com/errors/resource-not-found",
            title = "Resource not found",
            status = 404,
            detail = ex.message ?: "The requested resource could not be found.",
            instance = request.requestURI
        ))
}
```

- 검증 포인트: Writing 모드의 Error Response Format Template (RFC 7807/9457 구조 + `Content-Type: application/problem+json` 명시), Review 체크리스트의 "Error responses use RFC 7807/9457 Problem Details structure" 및 "Content-Type: application/problem+json used" 항목

---

### TC-3-04: DELETE 성공 시 204 No Content

- 규칙: "✅ **필수**: 삭제 성공 시 204 No Content를 반환한다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: COVERED

❌ Bad:
```kotlin
@DeleteMapping("/articles/{id}")
fun deleteArticle(@PathVariable id: String): ResponseEntity<Map<String, String>> {
    articleService.delete(id)
    // 200 OK + body 반환 — 잘못된 패턴
    return ResponseEntity.ok(mapOf("message" to "Article deleted successfully"))
}
```

✅ Good:
```kotlin
@DeleteMapping("/articles/{id}")
fun deleteArticle(@PathVariable id: String): ResponseEntity<Void> {
    articleService.delete(id)
    return ResponseEntity.noContent().build()
}
```

- 검증 포인트: Writing 모드의 HTTP Method to Status Code Mapping 표 "DELETE | 204 No Content" 및 `ResponseEntity.noContent().build()` 코드 예시, Review 체크리스트의 "DELETE success → 204 (no body)" 항목

---

### TC-3-05: POST 201 Created + 리소스 반환

- 규칙: "✅ **필수**: 새 리소스 생성 성공 시 201 Created와 생성된 리소스를 반환한다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: COVERED

❌ Bad:
```kotlin
@PostMapping("/articles")
fun createArticle(@RequestBody request: CreateArticleRequest): ResponseEntity<Map<String, String>> {
    val article = articleService.create(request)
    // 200 OK, Location 헤더 없음, 생성된 리소스 미반환
    return ResponseEntity.ok(mapOf("id" to article.id))
}
```

✅ Good:
```kotlin
@PostMapping("/articles")
fun createArticle(@RequestBody request: CreateArticleRequest): ResponseEntity<Article> {
    val article = articleService.create(request)
    val location = URI.create("/articles/${article.id}")
    return ResponseEntity.created(location).body(article)
}
```

- 검증 포인트: Writing 모드의 "POST (create) | 201 Created + Location header" 매핑 및 `createArticle` 코드 예시, Review 체크리스트의 "POST create → 201 + Location header" 항목

---

### TC-3-06: 유효성 검사 실패 시 모든 오류 한 번에 반환

- 규칙: "⚠️ **권장**: 유효성 검사 실패 시 모든 오류 필드를 한 번에 반환한다 (하나씩 반환하지 않는다)."
- 규범 수준: ⚠️권장
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: COVERED

❌ Bad:
```kotlin
// 첫 번째 오류만 반환하고 나머지는 무시
@PostMapping("/articles")
fun createArticle(@RequestBody request: CreateArticleRequest): ResponseEntity<Article> {
    if (request.title.isBlank()) {
        throw ValidationException("title", "제목은 필수 입력값입니다.")
        // content 오류는 검사하지 않음 — 클라이언트가 여러 번 요청해야 함
    }
    if (request.content.length < 10) {
        throw ValidationException("content", "본문은 10자 이상이어야 합니다.")
    }
    // ...
}
```

✅ Good:
```kotlin
@PostMapping("/articles")
fun createArticle(@RequestBody @Valid request: CreateArticleRequest): ResponseEntity<Article> {
    // Bean Validation + 전역 예외 핸들러로 모든 오류를 한 번에 수집
    val article = articleService.create(request)
    val location = URI.create("/articles/${article.id}")
    return ResponseEntity.created(location).body(article)
}

@ExceptionHandler(MethodArgumentNotValidException::class)
fun handleValidation(
    ex: MethodArgumentNotValidException,
    request: HttpServletRequest
): ResponseEntity<ProblemDetail> {
    val fieldErrors = ex.bindingResult.fieldErrors.map { error ->
        ProblemDetail.FieldError(
            field = error.field,
            message = error.defaultMessage ?: "Invalid value"
        )
    }
    return ResponseEntity.badRequest()
        .contentType(MediaType.parseMediaType("application/problem+json"))
        .body(ProblemDetail(
            type = "https://api.example.com/errors/validation-failed",
            title = "Validation failed",
            status = 400,
            detail = "Request data validation failed.",
            instance = request.requestURI,
            errors = fieldErrors
        ))
}
```

- 검증 포인트: 스킬의 RFC 7807 템플릿에 `errors` 배열 확장 필드가 있으나, 유효성 실패 시 '모든 오류를 한 번에 반환해야 한다'는 명시적 지침이 Writing Mode에 없고 Review 체크리스트에도 "All validation errors returned at once" 항목이 없음(MISSING). TC-3-03이 RFC 7807 구조(3.4-1/3.4-2)를 별도로 커버한다.

---

### TC-3-07: 에러 응답에 내부 정보 노출 금지

- 규칙: "❌ **금지**: 에러 응답에 스택 트레이스, 내부 시스템 경로, DB 오류 메시지 등 내부 구현 정보를 노출하지 않는다."
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: MISSING / Review: COVERED

❌ Bad:
```kotlin
@ExceptionHandler(Exception::class)
fun handleException(ex: Exception, request: HttpServletRequest): ResponseEntity<Map<String, Any>> {
    return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
        .body(mapOf(
            "error" to ex.message,
            // 스택 트레이스 노출 — 금지
            "stackTrace" to ex.stackTraceToString(),
            // 내부 시스템 경로 노출 — 금지
            "path" to "/var/app/services/article-service/src/main/kotlin/...",
            // DB 오류 메시지 노출 — 금지
            "cause" to "org.postgresql.util.PSQLException: duplicate key value violates unique constraint"
        ))
}
```

✅ Good:
```kotlin
@ExceptionHandler(Exception::class)
fun handleException(ex: Exception, request: HttpServletRequest): ResponseEntity<ProblemDetail> {
    // 내부 정보는 서버 로그에만 기록
    logger.error("Unhandled exception", ex)

    return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
        .contentType(MediaType.parseMediaType("application/problem+json"))
        .body(ProblemDetail(
            type = "https://api.example.com/errors/internal-server-error",
            title = "Internal server error",
            status = 500,
            detail = "An unexpected error occurred. Please try again later.",
            instance = request.requestURI,
            traceId = MDC.get("traceId")
        ))
}
```

- 검증 포인트: Writing 모드에 스택 트레이스/내부 정보 노출 금지 가이드 없음(MISSING), Review 체크리스트의 "Internal implementation details (stack traces, DB errors) not exposed" 항목

---

### TC-3-08: 표준 리소스 필드 구조

- 규칙: "✅ **필수**: 리소스 스키마는 일관된 구조를 유지한다." (id, createdAt, updatedAt 표준 필드)
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: MISSING

❌ Bad:
```json
{
  "articleId": "123",
  "title": "RESTful API 설계",
  "content": "...",
  "created": "2024-01-15 09:00:00",
  "modified": "2024-01-20 14:30:00"
}
```

✅ Good:
```json
{
  "id": "123",
  "title": "RESTful API 설계",
  "content": "...",
  "createdAt": "2024-01-15T09:00:00Z",
  "updatedAt": "2024-01-20T14:30:00Z"
}
```

- 검증 포인트: Writing 모드의 Standard Resource Fields에 id/createdAt/updatedAt 표준 구조 정의(COVERED), Review 체크리스트에 표준 필드 구조 검증 항목 없음(MISSING)

---

## 섹션 4: JSON 규칙

### TC-4-01: JSON 필드 이름 camelCase

- 규칙: "✅ **필수**: JSON 필드 이름은 camelCase를 사용한다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: COVERED

❌ Bad:
```kotlin
data class ArticleResponse(
    val article_id: String,      // snake_case — 금지
    val user_name: String,       // snake_case — 금지
    val created_at: String,      // snake_case — 금지
    val is_published: Boolean    // snake_case — 금지
)

// JSON 출력: { "article_id": "123", "user_name": "john", "created_at": "...", "is_published": true }
```

✅ Good:
```kotlin
data class ArticleResponse(
    val articleId: String,
    val userName: String,
    val createdAt: String,
    val isPublished: Boolean
)

// JSON 출력: { "articleId": "123", "userName": "john", "createdAt": "...", "isPublished": true }
```

- 검증 포인트: Writing 모드의 JSON Field Naming Rules "Correct example" / "Incorrect example" 비교, Review 체크리스트의 "All fields are camelCase" 항목. 이 케이스는 4.1-2(필드 이름 영소문자로 시작) 규칙도 부분적으로 커버한다. snake_case 예시는 모두 영소문자로 시작하므로 4.1-2 위반 패턴(예: `UserId`) 의 별도 TC가 필요할 수 있다.

---

### TC-4-02: 필드명 약어 금지

- 규칙: "❌ **금지**: 필드 이름에 약어를 남용하지 않는다. 명확한 전체 단어를 우선한다."
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: MISSING / Review: MISSING

❌ Bad:
```json
{
  "usr": "john",
  "ts": "2024-01-20T10:00:00Z",
  "cnt": 5,
  "desc": "A short description",
  "qty": 10,
  "amt": 1500
}
```

✅ Good:
```json
{
  "username": "john",
  "timestamp": "2024-01-20T10:00:00Z",
  "count": 5,
  "description": "A short description",
  "quantity": 10,
  "amount": 1500
}
```

- 검증 포인트: 스킬의 Writing/Review 모드 모두에서 약어 남용 금지 규칙 없음(MISSING)

---

### TC-4-03: Boolean 필드 is/has/can 접두사

- 규칙: "✅ **필수**: Boolean 필드 이름은 `is`, `has`, `can` 등의 접두사를 사용한다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: COVERED

❌ Bad:
```json
{
  "active": true,
  "admin": false,
  "editPermission": true,
  "verified": true,
  "subscribed": false
}
```

✅ Good:
```json
{
  "isActive": true,
  "isAdmin": false,
  "canEdit": true,
  "isVerified": true,
  "hasSubscription": false
}
```

- 검증 포인트: Writing 모드의 `"isActive": true` 예시 및 JSON 필드 규칙, Review 체크리스트의 "Boolean fields use is/has/can prefix" 항목

---

### TC-4-04: Boolean은 JSON true/false 사용

- 규칙: "✅ **필수**: Boolean 값에는 JSON `true`/`false`를 사용한다. 문자열 `\"true\"`/`\"false\"` 또는 숫자 `1`/`0`을 사용하지 않는다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: PARTIAL / Review: MISSING

❌ Bad:
```json
{
  "isActive": "true",
  "hasPermission": "false",
  "canEdit": 1,
  "isVerified": 0
}
```

✅ Good:
```json
{
  "isActive": true,
  "hasPermission": false,
  "canEdit": true,
  "isVerified": false
}
```

- 검증 포인트: Writing 모드에 `"isActive": true` 코드 예시 있으나 문자열/"1"/"0" 금지 명시 없음(PARTIAL), Review 체크리스트에 Boolean JSON true/false 체크 항목 없음(MISSING)

---

### TC-4-05: 큰 정수 문자열 반환

- 규칙: "⚠️ **권장**: JavaScript의 안전한 정수 범위(2^53 - 1)를 초과하는 큰 정수는 문자열로 반환한다."
- 규범 수준: ⚠️권장
- 대상 모드: Both
- 스킬 커버: Writing: MISSING / Review: MISSING

❌ Bad:
```json
{
  "id": 9007199254740993,
  "snowflakeId": 1234567890123456789,
  "count": 42,
  "price": 19.99
}
```

✅ Good:
```json
{
  "id": "9007199254740993",
  "snowflakeId": "1234567890123456789",
  "count": 42,
  "price": 19.99
}
```

- 검증 포인트: 스킬의 Writing/Review 모드 모두에서 큰 정수 문자열 반환 규칙 없음(MISSING)

---

### TC-4-06: 날짜/시간 RFC 3339 형식 및 UTC

- 규칙: "✅ **필수**: 모든 날짜/시간 값은 RFC 3339 형식(ISO 8601 프로파일)의 문자열로 표현한다." / "✅ **필수**: 서버 응답의 모든 시간 값은 UTC(`Z`)로 반환한다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: COVERED

❌ Bad:
```json
{
  "createdAt": "2024-01-20 10:00:00",
  "updatedAt": "Jan 20, 2024 3:00 PM",
  "scheduledAt": "2024/01/25 09:30:00+09:00",
  "publishedAt": "20240120T150000Z"
}
```

✅ Good:
```json
{
  "createdAt": "2024-01-20T10:00:00Z",
  "updatedAt": "2024-01-20T15:00:00Z",
  "scheduledAt": "2024-01-25T00:30:00Z",
  "birthDate": "1990-05-15"
}
```

- 검증 포인트: Writing 모드의 Date/Time 섹션 RFC 3339 형식 예시 및 "Server response (Required: UTC)" 명시, Review 체크리스트의 "Date/time in RFC 3339 format" 및 "All time values in server response are UTC (Z)" 항목

---

### TC-4-07: 오프셋 포함 시간 서버가 UTC로 변환 저장

- 규칙: "✅ **필수**: 클라이언트가 오프셋이 포함된 시간을 전송하면, 서버는 이를 UTC로 변환하여 저장한다. 에러를 반환하지 않는다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: COVERED

❌ Bad:
```kotlin
@PostMapping("/events")
fun createEvent(@RequestBody request: CreateEventRequest): ResponseEntity<Event> {
    val scheduledAt = OffsetDateTime.parse(request.scheduledAt)

    // 오프셋이 포함되면 에러 반환 — 잘못된 처리
    if (scheduledAt.offset != ZoneOffset.UTC) {
        throw ApiException(
            status = HttpStatus.BAD_REQUEST,
            type = "https://api.example.com/errors/invalid-request",
            title = "Invalid time format",
            detail = "Time values must be in UTC format."
        )
    }

    // 또는 오프셋을 무시하고 그대로 저장 — 잘못된 처리
    val event = eventService.create(request.copy(scheduledAt = request.scheduledAt))
    val location = URI.create("/events/${event.id}")
    return ResponseEntity.created(location).body(event)
}
```

✅ Good:
```kotlin
@PostMapping("/events")
fun createEvent(@RequestBody request: CreateEventRequest): ResponseEntity<Event> {
    // 오프셋 포함 시간을 UTC로 정규화하여 저장
    val scheduledAt = OffsetDateTime.parse(request.scheduledAt)
        .withOffsetSameInstant(ZoneOffset.UTC)  // UTC로 변환
        .toInstant()

    val event = eventService.create(
        title = request.title,
        scheduledAt = scheduledAt  // UTC로 변환된 값 저장
    )
    val location = URI.create("/events/${event.id}")
    return ResponseEntity.created(location).body(event)
    // 응답: { "scheduledAt": "2024-01-25T00:30:00Z" }
}
```

- 검증 포인트: Writing 모드의 "offset allowed — server normalizes to UTC" 명시, Review 체크리스트의 "Offset input is normalized to UTC by server (not an error)" 항목

---

### TC-4-08: Unix timestamp 금지

- 규칙: "❌ **금지**: Unix timestamp(epoch milliseconds/seconds)를 기본 시간 형식으로 사용하지 않는다."
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: MISSING

❌ Bad:
```json
{
  "createdAt": 1705744800000,
  "updatedAt": 1705762200,
  "scheduledAt": 1706140200000
}
```

✅ Good:
```json
{
  "createdAt": "2024-01-20T10:00:00Z",
  "updatedAt": "2024-01-20T15:00:00Z",
  "scheduledAt": "2024-01-25T00:30:00Z"
}
```

- 검증 포인트: Writing 모드의 Prohibited 섹션에 "Unix timestamp forbidden" 명시(COVERED), Review 체크리스트에 Unix timestamp 금지 체크 항목 없음(MISSING)

---

### TC-4-09: Enum UPPER_SNAKE_CASE

- 규칙: "✅ **필수**: Enum 값은 UPPER_SNAKE_CASE 문자열을 사용한다."
- 규범 수준: ✅필수
- 대상 모드: Both
- 스킬 커버: Writing: COVERED / Review: COVERED

❌ Bad:
```kotlin
enum class ArticleStatus {
    published,      // 소문자 — 금지
    Draft,          // PascalCase — 금지
    in_review,      // lower_snake_case — 금지
    pendingApproval // camelCase — 금지
}

// JSON: { "status": "published" }
```

✅ Good:
```kotlin
enum class ArticleStatus {
    PUBLISHED,
    DRAFT,
    IN_REVIEW,
    PENDING_APPROVAL
}

// JSON: { "status": "PUBLISHED" }
```

- 검증 포인트: Writing 모드의 `"status": "PUBLISHED"` 예시 및 "enums must be UPPER_SNAKE_CASE" 명시, Review 체크리스트의 "Enum values are UPPER_SNAKE_CASE" 항목

---

### TC-4-10: Enum 숫자/약어 사용 금지

- 규칙: "❌ **금지**: Enum 값으로 숫자나 불명확한 약어를 사용하지 않는다."
- 규범 수준: ❌금지
- 대상 모드: Both
- 스킬 커버: Writing: PARTIAL / Review: MISSING

❌ Bad:
```json
{
  "status": 1,
  "priority": "hi",
  "category": "misc",
  "type": 3
}
```

✅ Good:
```json
{
  "status": "PUBLISHED",
  "priority": "HIGH",
  "category": "MISCELLANEOUS",
  "type": "PREMIUM_MEMBERSHIP"
}
```

- 검증 포인트: Writing 모드에 "enums must be UPPER_SNAKE_CASE" 명시 있으나 숫자/약어 사용 금지 bad case 없음(PARTIAL), Review 체크리스트에 Enum 숫자/약어 금지 체크 항목 없음(MISSING)
