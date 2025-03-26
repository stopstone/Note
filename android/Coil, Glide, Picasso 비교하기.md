# Coil, Glide, Picasso 비교하기
안드로이드에서 이미지를 로딩할때 활용되는 이미지 로딩 라이브러리( Coil, Gilde, Piccasso )에 대해 비교 분석하며  
레이아웃, 컴포즈를 쓸때도 뭐가 더 좋은지 알아봅니다.  
어떤 기준으로 라이브러리를 선정하면 좋을지에 대해 정리해둔 문서입니다.  

### 1. 이미지 로딩 라이브러리를 왜 사용할까?

이미지를 직접 처리하려면 다음과 같은 복잡한 로직이 필요합니다.  
-   네트워크 요청으로 이미지 바이트를 가져오기
-   Bitmap 디코딩 및 리사이징
-   RecyclerView에서의 이미지 재활용 처리
-   메모리 절약을 위한 캐시(LruCache), 디스크 캐시 직접 구현 등

#### 예시 코드 (직접 이미지 로딩 구현)

```
fun loadImageManually(url: String, imageView: ImageView) {
    CoroutineScope(Dispatchers.IO).launch {
        try {
            val connection = URL(url).openConnection() as HttpURLConnection
            connection.doInput = true
            connection.connect()
            val inputStream = connection.inputStream
            val bitmap = BitmapFactory.decodeStream(inputStream)

            val resized = Bitmap.createScaledBitmap(bitmap, imageView.width, imageView.height, true)

            withContext(Dispatchers.Main) {
                imageView.setImageBitmap(resized)
            }
        } catch (e: Exception) {
            Log.e("ManualImageLoader", "Error loading image", e)
        }
    }
}
```

네트워크로부터 받은 이미지 url을 비트맵 변환을 거쳐 ImageView에 그리는 코드입니다.  
이미지 로딩 라이브러리를 사용하면 이러한 복잡하고 귀찮은 코드를 줄일 수 있습니다.  

라이브러리를 사용하면 위의 과정이 추상화되어 다음과 같은 이점이 있습니다.    
-   간단한 코드로 이미지 로딩 가능
-   메모리/디스크 캐싱 자동화
-   리사이징, 변형, GIF, 애니메이션 지원
-   View 재활용, 이미지 요청 취소 등 최적화 포함

---

### 2. 주요 이미지 로딩 라이브러리 개요 및 특징

| 항목 | Picasso | Glide | Coil |
| --- | --- | --- | --- |
| 개발사 | Square | bumptech (Google 기여) | Instacart + Jetpack Team |
| Min SDK | 14 | 14 | 14 |
| 라이브러리 크기 | 약 121KB | 약 440KB | 작음 (Kotlin-first) |
| GIF 지원 | ❌ 미지원 | ⭕ 지원 | ⭕ 확장 라이브러리 필요 (coil-gif)|
| 주요 특징 | 간단한 API, 자동 캐싱, `.fit()` | 다양한 포맷 지원, 고성능 캐싱 | Kotlin-first, Coroutine 기반, Compose 지원 |
| 기본 Bitmap 포맷 | ARGB\_8888 | RGB\_565 (변경 가능) | ARGB\_8888 |
| Compose 지원 | 미흡 | 일부 지원 (Glide Compose 별도) | 완전 지원 (AsyncImage) |

#### 코드 예시

**Picasso (XML 기반)**

```
Picasso.get()
    .load("https://example.com/image.jpg")
    .placeholder(R.drawable.placeholder)
    .error(R.drawable.error)
    .fit()
    .into(imageView)
```

**Glide (XML 기반)**

```
Glide.with(this)
    .load("https://example.com/image.jpg")
    .placeholder(R.drawable.placeholder)
    .error(R.drawable.error)
    .centerCrop()
    .into(imageView)
```

**Coil (XML + Kotlin)**

```
imageView.load("https://example.com/image.jpg") {
    placeholder(R.drawable.placeholder)
    error(R.drawable.error)
    transformations(RoundedCornersTransformation(16f))
}
```

**Coil (Compose)**

```
AsyncImage(
    model = "https://example.com/image.jpg",
    contentDescription = null,
    placeholder = painterResource(R.drawable.placeholder),
    error = painterResource(R.drawable.error)
)
```

---

### 3. Compose vs Layout 환경에서의 선택

| 환경 | 추천 라이브러리 | 주요 특징 요약 |
| --- | --- | --- |
| Compose | Coil | 공식 권장, `AsyncImage` 제공, 간결한 문법, 빠른 적용 가능 |
| XML Layout | Glide, Picasso | Glide: 다양한 기능 지원 / Picasso: 가볍고 단순한 API |

---

### 4. 성능 비교 (공식 문서 및 실측 기준 요약)

| 라이브러리 | 초기 로딩 속도 | 캐시 적용 후 속도 | 메모리 사용량 (예상 평균치) |
| --- | --- | --- | --- |
| Picasso | 약 6.4초 | 약 1.6초 | 약 10MB |
| Glide | 약 6.2초 | 약 0.72초 | 약 11MB (포맷 설정 가능) |
| Coil | 약 5.2초 | 약 1.3초 | 약 7MB |

> 위 수치는 공식 문서 기반은 아니며, 실제 결과는 디바이스, 이미지 크기, 네트워크 환경에 따라 다를 수 있습니다.

---

### 5. 상황에 따라 어떤 걸 쓰는 게 좋을까?

| 상황 | 추천 라이브러리 |
| --- | --- |
| 단순한 이미지 로딩 | Picasso, Coil |
| GIF 지원, 커스터마이징 필요 | Glide |
| 최신 Compose 앱 개발 | Coil |
| 메모리 최적화 우선 | Coil, Glide (RGB\_565 설정) |
| 이미지 품질 우선 | Glide (ARGB\_8888 설정) |
| RecyclerView 고빈도 이미지 로딩 | Glide |

---

### ARGB\_8888 와 RGB\_565 가 뭘까?

| 포맷 | 설명 |
| --- | --- |
| ARGB\_8888 | 픽셀당 4바이트 (Alpha + RGB), 고화질, 메모리 많이 사용 |
| RGB\_565 | 픽셀당 2바이트 (RGB), 저화질, 메모리 절약 |

-   ARGB\_8888은 투명도 지원 및 색 표현력 우수하지만 메모리를 많이 사용합니다.
-   RGB\_565는 메모리는 적게 사용하지만 화질이 낮고 투명도 미지원입니다.

**Glide 설정 예시 (ARGB\_8888로 전환):**

```
Glide.with(context)
    .load(url)
    .format(DecodeFormat.PREFER_ARGB_8888)
    .into(imageView)
```

---

### 6. 구글은 어떤 라이브러리를 권장할까?

-   Coil은 Jetpack 팀에서 개발한 공식 이미지 로딩 라이브러리입니다.
-   Google의 공식 Compose 문서에서도 Coil 사용 예시가 가장 많이 제공됩니다.
-   Compose 환경에서는 사실상 Coil이 표준처럼 자리잡고 있습니다.

---

## 참고 자료

-   [Coil 공식 문서](https://coil-kt.github.io/coil)
-   [Glide GitHub](https://github.com/bumptech/glide)
-   [Picasso 공식 문서](https://square.github.io/picasso)
-   [태크민의 우당탕탕 개발 블로그: Glide vs Picasso / Coil](https://jtm0609.tistory.com/259)
-   [jsh-me.log: Image Loader Library 톺아보기](https://velog.io/@jshme/Android-Image-Loader-Library)
-   [DongYeon-Lee: Glide, Picasso 이미지 최적화 (Down Sampling)](https://easternkite.medium.com/android-glide-picasso-%EC%99%80-%EA%B0%99%EC%9D%80-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EB%A1%9C%EB%93%9C%EB%A5%BC-%EC%B5%9C%EC%A0%81%ED%99%94%ED%95%A0%EA%B9%8C-1%ED%8E%B8-down-sampling-4d7c7797c018)
-   [롯데ON 기술 블로그: Glide vs. Coil 메모리 비교](https://techblog.lotteon.com/glide-vs-coil-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%82%AC%EC%9A%A9%EB%9F%89-%EB%B9%84%EA%B5%90-cb93cffb9fc0)
-   [박상권의 삽질 블로그: Picasso와 Glide 비교분석](https://gun0912.tistory.com/19)
