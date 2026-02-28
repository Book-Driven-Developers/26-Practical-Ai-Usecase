- 방향성을 정하는게 일
    - 방향성도 ai에게 물어봐서 의도를 정리할 수 있다.
    - 응집도 결정은 개발자가 할 수 있는 상태

## 5. 리뷰

## 리뷰 - 레거시 x AI

<aside>

Review에 Image를 추가할 수 있도록 구현해

[요구사항]

- Review 작성 시 image 목록을 보낼 수 있도록 구현해
    - Image Review는 포인트를 500원 더 지급하도록 구현해
- Review 목록 조회 시 Image Review만 모아보기 기능을 추가해
- 이미 작성된 Image Review는 이미지를 모두 삭제하고 수정할 순 없도록 구현해

[정책]

- Image는 리뷰 작성 시 필수는 아니야
- Image는 최대 5개까지 등록이 가능해
</aside>

Claude code 테스트

[CLAUDE.md](http://CLAUDE.md) 에 추가한 가이드라인

```java
### 메소드화 지침
- 메소드 안에 메소드화 할 수 있는 코드 블록들이 있을 경우 메소드화 한다.
  - 예를들어, 메소드 안에 조건문이 있거나 메소드화 하는 것이 더 가독성이 좋게 되는 경우.
```

변경 전

```kotlin
fun addReview(user: User, target: ReviewTarget, content: ReviewContent): Long {
        val reviewKey = reviewPolicyValidator.validateNew(user, target)
        val reviewId = reviewManager.add(reviewKey, target, content)
        pointHandler.earn(user, PointType.REVIEW, reviewId, PointAmount.REVIEW)
        if (content.imageUrls.isNotEmpty()) {
            pointHandler.earn(user, PointType.REVIEW, reviewId, PointAmount.IMAGE_REVIEW)
        }
        return reviewId
    }
    
    ...
```

변경 후

```kotlin
fun addReview(user: User, target: ReviewTarget, content: ReviewContent): Long {
        val reviewKey = reviewPolicyValidator.validateNew(user, target)
        val reviewId = reviewManager.add(reviewKey, target, content)
        earnReviewPoints(user, reviewId, hasImages = content.imageUrls.isNotEmpty())
        return reviewId
    }

private fun earnReviewPoints(user: User, reviewId: Long, hasImages: Boolean) {
        pointHandler.earn(user, PointType.REVIEW, reviewId, PointAmount.REVIEW)
        if (hasImages) {
            pointHandler.earn(user, PointType.REVIEW, reviewId, PointAmount.IMAGE_REVIEW)
        }
    }
```

CLAUDE.md에 가이드라인을 적지 않고 리팩토링을 요청할 경우

변경 전

```kotlin
    fun findRateSummary(target: ReviewTarget): RateSummary {
        return reviewFinder.findRateSummary(target)
    }

    fun findReviews(target: ReviewTarget, offsetLimit: OffsetLimit, imageOnly: Boolean = false): Page<Review> {
        return reviewFinder.find(target, offsetLimit, imageOnly)
    }
    
    ...
    
    fun addReview(user: User, target: ReviewTarget, content: ReviewContent): Long {
        val reviewKey = reviewPolicyValidator.validateNew(user, target)
        val reviewId = reviewManager.add(reviewKey, target, content)
        pointHandler.earn(user, PointType.REVIEW, reviewId, PointAmount.REVIEW)
        if (content.imageUrls.isNotEmpty()) {
            pointHandler.earn(user, PointType.REVIEW, reviewId, PointAmount.IMAGE_REVIEW)
        }
        return reviewId
    }
    
    fun removeReview(user: User, reviewId: Long): Long {
        val isImageReview = reviewFinder.hasImages(reviewId)
        val deletedReviewId = reviewManager.delete(user, reviewId)
        pointHandler.deduct(user, PointType.REVIEW, deletedReviewId, PointAmount.REVIEW)
        if (isImageReview) {
            pointHandler.deduct(user, PointType.REVIEW, deletedReviewId, PointAmount.IMAGE_REVIEW)
        }
        return deletedReviewId
    }
```

변경 후

```kotlin
fun findReviews(target: ReviewTarget, offsetLimit: OffsetLimit, imageOnly: Boolean = false): Page<Review> =
        reviewFinder.find(target, offsetLimit, imageOnly)

fun addReview(user: User, target: ReviewTarget, content: ReviewContent): Long {
        val reviewKey = reviewPolicyValidator.validateNew(user, target)
        val reviewId = reviewManager.add(reviewKey, target, content)
        earnReviewPoint(user, reviewId, hasImage = content.imageUrls.isNotEmpty())
        return reviewId
    }

private fun earnReviewPoint(user: User, reviewId: Long, hasImage: Boolean) {
        pointHandler.earn(user, PointType.REVIEW, reviewId, PointAmount.REVIEW)
        if (hasImage) pointHandler.earn(user, PointType.REVIEW, reviewId, PointAmount.IMAGE_REVIEW)
    }
```

> 결론: guideline을 써 줬을 때의 변화와 쓰지 않았을 때 큰 차이는 없는 걸로 관찰됨.
→ 가이드라인으로 컨텍스트 관리를 해 보아야 guideline의 위력을 실감 할 수 있을 것 같음.
>

https://code.claude.com/docs/en/memory

앤트로픽 공식 문서에

CLADUE.md는 **짧고** 인간이 읽기 쉬워야 하고, 크게 공통 적용되는 규칙만 넣길 권장.

복잡한 프로젝트일 시 .claude/rules/ 로 쪼개면 좋다.

```kotlin
.claude/
  CLAUDE.md
  rules/
    general.md
    backend.md
    testing.md
    security.md
    api-controller.md
```

`.claude/rules/*.md` 는 프로젝트 메모리처럼 자동 로드할 수 있고, 특정 경로에만 적용되게 만들 수 있다.

예를 들어 src/main/java/**/controller/** 에만 적용되는 컨트롤러 규칙을 적용할 수 있다.

### 규칙 준수력 & 문맥 유지력

```kotlin
@core\core-api\src\main\kotlin\io\dodn\commerce\core\api\controller\v1\ReviewController.kt @core\core-api\src\main\kotlin\io\dodn\commerce\core\domain\ReviewService.kt           
  @storage\db-core\src\main\kotlin\io\dodn\commerce\storage\db\core\ReviewRepository.kt 를 먼저 읽고 현재 구조를 요약해줘.                                                         
  그 다음 리뷰를 영어로 번역해주는 기능을 추가해줘.                                                                                                                                 
  변경 전 어떤 파일을 바꿀지 먼저 계획을 제시해 줘  
```

● Read 3 files (ctrl+o to expand)

● 현재 구조 요약

ReviewController

├── GET  /v1/reviews           → ReviewService.findReviews()

├── POST /v1/reviews           → ReviewService.addReview()

├── PUT  /v1/reviews/{id}      → ReviewService.updateReview()

└── DELETE /v1/reviews/{id}   → ReviewService.removeReview()

ReviewService (Business Layer)
├── findRateSummary  → ReviewFinder
├── findReviews      → ReviewFinder
├── addReview        → PolicyValidator → Manager → PointHandler
├── updateReview     → PolicyValidator → Manager
└── removeReview     → Finder → Manager → PointHandler
private: earnReviewPoint / deductReviewPoint

ReviewRepository (Data Access Layer)
├── 파생 쿼리 3종 (findByIdAndUserId 등)
└── @Query: findWithImages... (이미지 리뷰 페이징)

---

번역 기능 추가 계획

API 설계
GET /v1/reviews/{reviewId}/translate
특정 리뷰의 content를 영어로 번역해 반환합니다.

변경 파일 목록

┌──────┬────────────────────────────┬──────────────────────────────────────────────────────────────────────┐
│ 구분 │            파일            │                              변경 내용                               │
├──────┼────────────────────────────┼──────────────────────────────────────────────────────────────────────┤
│ 신규 │ ReviewTranslator.kt        │ Logic Layer @Component. 외부 번역 API 호출                           │
├──────┼────────────────────────────┼──────────────────────────────────────────────────────────────────────┤
│ 신규 │ TranslatedReview.kt        │ 번역 결과 개념 객체 { reviewId, originalContent, translatedContent } │
├──────┼────────────────────────────┼──────────────────────────────────────────────────────────────────────┤
│ 신규 │ ReviewTranslateResponse.kt │ Response DTO, ReviewTranslateResponse.of(TranslatedReview)           │
├──────┼────────────────────────────┼──────────────────────────────────────────────────────────────────────┤
│ 수정 │ ReviewService.kt           │ translateReview(reviewId) 추가, ReviewTranslator 주입                │
├──────┼────────────────────────────┼──────────────────────────────────────────────────────────────────────┤
│ 수정 │ ReviewController.kt        │ GET /v1/reviews/{reviewId}/translate 엔드포인트 추가                 │
└──────┴────────────────────────────┴──────────────────────────────────────────────────────────────────────┘

흐름

Controller → ReviewService.translateReview(reviewId)
→ ReviewFinder.find(reviewId)       # 기존 Finder 재사용
→ ReviewTranslator.translate(text)  # 외부 API 호출
→ TranslatedReview 반환

ReviewFinder는 수정하지 않습니다. 이미 단건 조회 find(reviewId) 가 있으면 재사용하고, 없으면 ReviewRepository를 직접 사용합니다.

ReviewTranslator는 외부 번역 API(RestTemplate 기반)를 호출하는 Logic Layer 컴포넌트로 만들고, 실제 API 키는 application.yml 프로퍼티로 분리합니다.

---

진행할까요? 진행 전 확인하고 싶은 사항이 있으면 말씀해 주세요.
(번역 API 종류 - Google / DeepL / 기타, 또는 Stub 구현 여부 등)

✻ Cogitated for 53s

- 코드 수정 전에 탐색하는가
    - O
- 계획을 먼저 세우는가
    - X
- controller/service/repository 역할을 지키는가
    - O
- 불필요한 리팩터링을 안 하는가
    - O
    - `ReviewFinder는 수정하지 않습니다. 이미 단건 조회 find(reviewId) 가 있으면 재사용하고, 없으면 ReviewRepository를 직접 사용합니다.`