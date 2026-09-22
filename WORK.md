아래처럼 정리하면 **개념 → 실습 순서 → 실제 결과**가 자연스럽게 이어져.

# Webpack Performance

## 1. 요청 크기 줄이기

### 1.1 소스코드 크기 줄이기

- JS minify
- CSS minify
- Tree Shaking
- 불필요한 dependency 제거

### 1.2 전송 크기 줄이기

- gzip
- Brotli

### 1.3 이미지 크기 줄이기

- WebP / AVIF
- 이미지 compression
- 적절한 이미지 해상도

### 1.4 CDN 활용

- CloudFront
- Cache
- gzip / Brotli

---

# 2. Webpack Performance 과제 실습 순서

```text
① Production build 전/후 JS 크기 비교
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
```

---

# 3. 소스코드 크기와 전송 크기 구분

## 3.1 소스코드 크기 줄이기

Webpack 빌드 결과물 자체의 크기를 줄이는 것

```text
Webpack
  ↓
minify
  ↓
Tree Shaking
  ↓
불필요한 dependency 제거
```

## 3.2 전송 크기 줄이기

브라우저와 서버 사이에서 실제로 전송되는 데이터의 크기를 줄이는 것

```text
gzip / Brotli
  ↓
전송 데이터 압축
  ↓
네트워크 전송량 감소
```

## 3.3 이미지 크기 줄이기

이미지 파일 자체의 용량을 줄이는 것

```text
WebP / AVIF
  +
Compression
  +
적절한 해상도
  ↓
이미지 파일 크기 감소
```

## 3.4 CDN

사용자와 가까운 서버에서 정적 파일을 제공하고 캐싱 및 압축을 적용

```text
CloudFront
  ├── Cache
  └── gzip / Brotli
```

---

# 4. 과제 실습

## 4.1 Production Build 전/후 JS 크기 비교

### Build 전

```text
dist/

bundle.js       2.4M
bundle.js.map   2.5M
index.html      712B
public/
static/
```

### Production Build

```text
dist/

bundle.js       1.2M
bundle.js.map   1.4M
index.html      633B
public/
static/
```

### 결과

```text
bundle.js

2.4MB
  ↓
1.2MB
```

약 **50% 감소**

> `bundle.js.map`은 소스맵이므로 실제 브라우저가 실행하는 JS 용량을 비교할 때는 `bundle.js`를 기준으로 한다.

---

# 5. JS Minify

## 5.1 목표

브라우저의 동작은 그대로 유지하면서 **실행에 필요한 JS 파일의 크기를 줄이는 것**

제거 또는 축약되는 예:

- 공백
- 줄바꿈
- 주석
- 불필요한 문자
- 가능한 경우 변수명 축약

### Minify 전

```js
function calculateTotal(price, quantity) {
  const total = price * quantity;
  return total;
}
```

### Minify 후

```js
function calculateTotal(e, t) {
  return e * t;
}
```

---

## 5.2 적용 단계

### ① `webpack.config.js`

```js
mode: 'production';
```

↓

### ② `optimization.minimize`

```js
optimization: {
  minimize: true;
}
```

↓

### ③ Production Build

```bash
npm run build:prod
```

↓

### ④ `dist/bundle.js` 확인

코드가 압축되어 있는지 확인

---

## 5.3 Minify 결과

### `minimize: false`

```text
bundle.js   1.2 MB
```

### `minimize: true`

```text
bundle.js   224 KB
```

|             | `minimize: false` | `minimize: true` |            감소 |
| ----------- | ----------------: | ---------------: | --------------: |
| `bundle.js` |            1.2 MB |           224 KB | **약 81% 감소** |

---

# 6. CSS Minify

## 6.1 목표

CSS의 스타일 동작은 그대로 유지하면서 **CSS 파일의 크기를 줄이는 것**

제거 또는 축약되는 예:

- 공백 제거
- 줄바꿈 제거
- 주석 제거
- 불필요한 문자 제거
- 가능한 CSS 값 축약

### Minify 전

```css
.button {
  padding: 10px 20px;
  color: red;
  background-color: white;
}
```

### Minify 후

```css
.button {
  padding: 10px 20px;
  color: red;
  background-color: #fff;
}
```

---

## 6.2 CSS Minify의 효과

```text
CSS
 ↓
Minify
 ↓
CSS 파일 크기 감소
 ↓
네트워크 전송량 감소
 ↓
CSS 다운로드 시간 감소
 ↓
페이지 로딩 성능 개선
```

---

# 7. CSS 파일 분리

## 7.1 기존 방식

기존 Webpack 설정:

```js
{
  test: /\.css$/i,
  use: ['style-loader', 'css-loader']
}
```

Webpack이 CSS를 처리하는 과정:

```text
CSS 파일
  ↓
css-loader
  ↓
JS가 이해할 수 있는 형태로 변환
  ↓
style-loader
  ↓
브라우저에서 <style> 태그로 CSS 삽입
```

결과적으로:

```text
bundle.js
├── JavaScript
└── CSS
```

브라우저가 JS를 실행하면서 CSS를 `<style>` 태그로 페이지에 삽입한다.

---

## 7.2 CSS 파일 추출

CSS를 별도의 파일로 만들기 위해 패키지를 설치한다.

```bash
npm install -D mini-css-extract-plugin css-minimizer-webpack-plugin
```

### `webpack.config.js`

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const Dotenv = require('dotenv-webpack');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');

module.exports = {
  entry: './src/index.tsx',

  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx']
  },

  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, '/dist'),
    clean: true
  },

  devServer: {
    hot: true,
    open: true,
    historyApiFallback: true
  },

  devtool: 'source-map',

  plugins: [
    new HtmlWebpackPlugin({
      template: './index.html'
    }),

    new CopyWebpackPlugin({
      patterns: [{ from: './public', to: './public' }]
    }),

    new Dotenv(),

    new MiniCssExtractPlugin({
      filename: 'styles.css'
    })
  ],

  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/i,
        exclude: /node_modules/,
        use: {
          loader: 'ts-loader'
        }
      },

      {
        test: /\.css$/i,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      },

      {
        test: /\.(eot|svg|ttf|woff|woff2|png|jpg|gif)$/i,
        loader: 'file-loader',
        options: {
          name: 'static/[name].[ext]'
        }
      }
    ]
  },

  optimization: {
    minimize: true,

    minimizer: ['...', new CssMinimizerPlugin()]
  }
};
```

---

## 7.3 CSS 분리 결과

다시 빌드:

```bash
npm run build:prod
```

기존:

```text
bundle.js
```

에서 변경 후:

```text
bundle.js
styles.css
```

### 구조

```text
변경 전

bundle.js
├── JS
└── CSS


변경 후

bundle.js   → JavaScript
styles.css  → CSS
```

### 각각의 역할

```text
MiniCssExtractPlugin
→ CSS를 styles.css로 분리

CssMinimizerPlugin
→ styles.css를 minify
```

**CSS 분리와 CSS minify는 서로 다른 작업이다.**

---

# 8. CSS Minify 실습 결과

## 8.1 Minify 전

```bash
ls -lh dist/styles.css
```

```text
styles.css   12K
```

## 8.2 Minify 후

```bash
npm run build:prod
```

```bash
ls -lh dist/styles.css
```

```text
styles.css   9.5K
```

## 8.3 결과 비교

| 상태      |      `styles.css` |
| --------- | ----------------: |
| Minify ❌ |           **12K** |
| Minify ✅ |          **9.5K** |
| 감소량    | **약 20.8% 감소** |

```text
12K
 ↓
9.5K
```

약 **20.8% 감소**

---

# 9. 현재까지 실습 결과

| 최적화     | 적용 전 | 적용 후 |              결과 |
| ---------- | ------: | ------: | ----------------: |
| JS Minify  |  1.2 MB |  224 KB |   **약 81% 감소** |
| CSS Minify |   12 KB |  9.5 KB | **약 20.8% 감소** |

### 현재 `dist`

```text
bundle.js    183 KB
styles.css   9.5 KB
index.html   629 B
```

> JS가 `224 KB → 183 KB`로 줄어든 것은 CSS를 `styles.css`로 분리하면서 CSS가 `bundle.js`에서 빠진 영향도 포함되어 있다. 따라서 JS Minify의 순수 효과를 비교할 때는 `1.2 MB → 224 KB` 비교를 사용한다.
