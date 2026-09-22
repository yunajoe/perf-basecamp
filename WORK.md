1. 소스코드 크기
   ├── JS minify
   ├── CSS minify
   ├── Tree Shaking
   └── 불필요한 dependency 제거

2. 전송 크기
   ├── gzip
   └── Brotli

3. 이미지 크기
   ├── WebP / AVIF
   ├── 이미지 compression
   └── 적절한 이미지 해상도

4. CDN
   ├── CloudFront
   ├── Cache
   └── gzip / Brotli

   ***

## Webpack performance 과제를 하는 거라면 이 순서가 좋습니다.

① production build 전/후 JS 크기 비교
↓
② JS minify 확인
↓
③ CSS minify 적용
↓
④ Tree Shaking 확인
↓
⑤ gzip 적용 전/후 크기 비교
↓
⑥ 이미지 PNG/JPEG → WebP 변환
↓
⑦ 이미지 compression
↓
⑧ CloudFront 캐싱 + 압축
↓
⑨ Chrome Network에서 실제 transferred size 비교

## 특히 "소스코드 크기 줄이기"와 "요청 크기 줄이기"를 구분하는 게 중요합니다.

소스코드 크기 줄이기
→ Webpack
→ minify / tree shaking

전송 크기 줄이기
→ gzip / Brotli
→ CloudFront

이미지 크기 줄이기
→ WebP / AVIF
→ compression

## 과제 실습하기

### 1. production build 전/후 JS 크기 비교

total 10072
-rw-r--r--@ 1 yunajoe staff 2.4M 9월 22 19:47 bundle.js
-rw-r--r--@ 1 yunajoe staff 2.5M 9월 22 19:47 bundle.js.map
-rw-r--r--@ 1 yunajoe staff 712B 9월 22 19:47 index.html
drwxr-xr-x@ 3 yunajoe staff 96B 9월 21 20:57 public
drwxr-xr-x@ 6 yunajoe staff 192B 9월 21 20:57 static
yunajoeui-MacBookPro:perf-basecamp yunajoe$

For more info visit https://webpack.js.org/guides/code-splitting/

webpack 5.111.0 compiled with 3 warnings in 1094 ms
yunajoeui-MacBookPro:perf-basecamp yunajoe$ ls -lh dist
total 5280
-rw-r--r--@ 1 yunajoe staff 1.2M 9월 22 19:54 bundle.js
-rw-r--r--@ 1 yunajoe staff 1.4M 9월 22 19:54 bundle.js.map
-rw-r--r--@ 1 yunajoe staff 633B 9월 22 19:54 index.html
drwxr-xr-x@ 3 yunajoe staff 96B 9월 21 20:57 public
drwxr-xr-x@ 6 yunajoe staff 192B 9월 21 20:57 static
yunajoeui-MacBookPro:perf-basecamp yunajoe$

### 2. JS minify 확인 (목표: 브라우저가 실행할 JS 파일의 용량을 줄이는 것)

#### 2.1 목표:브라우저가 실행할 JS 파일의 용량을 줄이는 것

- 코드의 동작은 그대로 유지하면서, 브라우저 실행에 필요 없는 문자들을 제거
- 공백, 줄바꿈, 주석, 불필요한 문자, 가능한 경우 변수명 축약

```javascript
function calculateTotal(price, quantity) {
  const total = price * quantity;

  return total;
}
```

```javascript
function calculateTotal(e, t) {
  return e * t;
}
```

#### 2.2 단계

1. webpack.config.js
   mode: 'production'
   ↓
1. optimization.minimize: true
   ↓
1. production build
   ↓
1. dist/bundle.js 확인
   → 코드가 압축되어 있는지 확인
   ↓
1. development 2.4MB
   production 1.2MB
   → 약 50% 감소

#### 2.3 결과

npm run build:prod
ls -lh dist

total 3464
-rw-r--r--@ 1 yunajoe staff 224K 9월 22 20:29 bundle.js
-rw-r--r--@ 1 yunajoe staff 1.7K 9월 22 20:29 bundle.js.LICENSE.txt
-rw-r--r--@ 1 yunajoe staff 1.5M 9월 22 20:29 bundle.js.map
-rw-r--r--@ 1 yunajoe staff 592B 9월 22 20:29 index.html
drwxr-xr-x@ 3 yunajoe staff 96B 9월 21 20:57 public
drwxr-xr-x@ 6 yunajoe staff 192B 9월 21 20:57 static

|             | `minimize: false` | `minimize: true` |            감소 |
| ----------- | ----------------: | ---------------: | --------------: |
| `bundle.js` |            1.2 MB |           224 KB | **약 81% 감소** |
