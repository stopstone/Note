# Compose에서 WebView 구성해보기

> Jetpack Compose에서 WebView를 사용하는 방법에 대해 알아보겠습니다.   
> 예제로 앱바를 포함하고 있는 웹뷰 화면을 구성하고  
> WebViewClient의 활용에 대해 자세히 설명드리겠습니다.  

## 1. 기본 WebView 구성

Compose에서 WebView를 사용하려면 `AndroidView`를 통해 네이티브 WebView를 래핑해야 합니다.

```kotlin
@Composable
fun WebViewScreen(
    url: String,
    modifier: Modifier = Modifier
) {
    AndroidView(
        modifier = modifier.fillMaxSize(),
        factory = { context ->
            WebView(context).apply {
                settings.apply {
                    domStorageEnabled = true // DOM Storage 활성화 (localStorage, sessionStorage 사용 가능)
                    mixedContentMode = WebSettings.MIXED_CONTENT_ALWAYS_ALLOW // HTTP/HTTPS 혼합 콘텐츠 허용
                    loadWithOverviewMode = true // 페이지를 전체 화면에 맞게 로드
                    useWideViewPort = true // 뷰포트 메타 태그 지원
                    builtInZoomControls = true // 줌 컨트롤 활성화
                    displayZoomControls = false // 줌 컨트롤 UI 숨김
                }
                loadUrl(url)
            }
        }
    )
}
```

## 2. 앱바가 있는 WebView 구성

상단에 앱바가 있는 웹뷰 화면이 필요한 경우가 있습니다.  
해당 화면은 다음과 같이 구성할 수 있습니다

```kotlin
@Composable
fun WebViewWithAppBar(
    onNavigateBack: () -> Unit,
    title: String,
    url: String,
    modifier: Modifier = Modifier,
) {
    var canGoBack by remember { mutableStateOf(false) }
    var webView: WebView? by remember { mutableStateOf(null) }
    var isLoading by remember { mutableStateOf(true) }

    // 뒤로가기 처리 - WebView 내에서 뒤로가기 가능할 때만 활성화
    // canGoBack이 true일 때만 BackHandler가 동작하여 WebView의 goBack() 호출
    // WebView 히스토리가 없으면 앱의 기본 뒤로가기 동작 수행
    BackHandler(enabled = canGoBack) {
        webView?.goBack()
    }

    Column(
        modifier = modifier.fillMaxSize()
    ) {
        // 상단 앱바
        TopAppBar(
            title = { Text(text = title) },
            navigationIcon = {
                IconButton(onClick = onNavigateBack) {
                    Icon(
                        imageVector = Icons.Default.ArrowBack,
                        contentDescription = "뒤로가기"
                    )
                }
            }
        )

        // WebView 영역
        Box(
            modifier = Modifier.weight(1f)
        ) {
            AndroidView(
                modifier = Modifier.fillMaxSize(),
                factory = { context ->
                    WebView(context).apply {
                        webView = this
                        webViewClient = object : WebViewClient() {
                            // 페이지 로딩 시작 시 호출 - 로딩 상태 true로 설정
                            override fun onPageStarted(
                                view: WebView?,
                                url: String?,
                                favicon: Bitmap?
                            ) {
                                super.onPageStarted(view, url, favicon)
                                isLoading = true
                            }

                            // 페이지 로딩 완료 시 호출
                            override fun onPageFinished(
                                view: WebView?,
                                url: String?
                            ) {
                                super.onPageFinished(view, url)
                                canGoBack = view?.canGoBack() ?: false
                                isLoading = false
                            }
                        }
                        
                        settings.apply {
                            domStorageEnabled = true // DOM Storage 활성화 (localStorage, sessionStorage 사용 가능)
                            mixedContentMode = WebSettings.MIXED_CONTENT_ALWAYS_ALLOW // HTTP/HTTPS 혼합 콘텐츠 허용
                            loadWithOverviewMode = true // 페이지를 전체 화면에 맞게 로드
                            useWideViewPort = true // 뷰포트 메타 태그 지원
                            builtInZoomControls = true // 줌 컨트롤 활성화
                            displayZoomControls = false // 줌 컨트롤 UI 숨김
                        }
                        loadUrl(url)
                    }
                },
                // Compose 재구성 시 호출 - WebView 상태 업데이트
                update = { view ->
                    canGoBack = view.canGoBack()
                }
            )

            // 로딩 인디케이터
            if (isLoading) {
                CircularProgressIndicator(
                    modifier = Modifier.align(Alignment.Center)
                )
            }
        }
    }
}
```

## 3. WebViewClient 상세 설명

WebViewClient는 웹 페이지 로딩 과정을 제어하고 다양한 이벤트를 처리할 수 있게 해주는 클래스입니다.

### 주요 메서드들

#### 3.1 onPageStarted()
페이지 로딩이 시작될 때 호출됩니다.

```kotlin
// 페이지 로딩 시작 시 호출 - 로딩 상태 true로 설정
override fun onPageStarted(
    view: WebView?,
    url: String?,
    favicon: Bitmap?
) {
    super.onPageStarted(view, url, favicon)
    // 로딩 시작 처리
    isLoading = true
}
```

#### 3.2 onPageFinished()
페이지 로딩이 완료될 때 호출됩니다.

```kotlin
// 페이지 로딩 완료 시 호출 - 로딩 상태 false, 뒤로가기 가능 여부 업데이트
override fun onPageFinished(
    view: WebView?,
    url: String?
) {
    super.onPageFinished(view, url)
    // 로딩 완료 처리
    isLoading = false
    canGoBack = view?.canGoBack() ?: false
}
```

#### 3.3 shouldOverrideUrlLoading()
새로운 URL로 이동할 때 호출됩니다. 외부 브라우저로 열지, WebView 내에서 계속 로드할지 결정할 수 있습니다.

```kotlin
// URL 로딩 시 호출 - 외부 브라우저로 열지 WebView에서 처리할지 결정
override fun shouldOverrideUrlLoading(
    view: WebView?,
    request: WebResourceRequest?
): Boolean {
    val url = request?.url.toString()
    
    // 특정 도메인은 외부 브라우저로 열기
    if (url.startsWith("https://play.google.com")) {
        val intent = Intent(Intent.ACTION_VIEW, Uri.parse(url))
        view?.context?.startActivity(intent)
        return true
    }
    
    // 기본적으로 WebView에서 처리
    return false
}
```

#### 3.4 onReceivedError()
페이지 로딩 중 오류가 발생했을 때 호출됩니다.

```kotlin
// 페이지 로딩 오류 발생 시 호출 - 오류 상태 처리
override fun onReceivedError(
    view: WebView?,
    request: WebResourceRequest?,
    error: WebResourceError?
) {
    super.onReceivedError(view, request, error)
    // 오류 처리
    showErrorSnackBar("페이지를 불러올 수 없습니다.")
}
```
