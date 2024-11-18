# 웹 성능 최적화로 로딩 속도를 N배 더 빠르게 만들기

오늘날 웹사이트와 애플리케이션의 성능은 사용자가 사이트에 머물지, 이탈할지를 결정하는 핵심 요소가 되었다.
디지털 환경에서 사용자 기대치는 점차 높아지고, 기업들은 경쟁 우위를 확보하기 위해 웹 성능 최적화를 중요하게 고려하고 있다.

# 웹 성능 최적화의 중요성

### 비즈니스 관점

웹사이트의 로딩 속도는 기업의 비즈니스 성과에 직접적으로 영향을 미친다. 사용자는 웹 페이지 로딩이 몇 초 이상 지연될 경우 이탈할 가능성이 높아지며, 이는 고객 손실로 이어질 수 있다.

<figure align="center">
  <img src="./images/google_research_loading_speed_impact.png" alt="Google/SOASTA Research, 2017." />
  <figcaption><em>Google/SOASTA Research, 2017.</em></figcaption>
</figure>

실제로 구글의 연구에 따르면, 페이지 로딩 시간이 1초에서 3초로 늘어날 때 페이지 이탈률이 32% 증가하며, 10초로 늘어날 경우 그 비율은 123%까지 급격히 상승한다. 로딩 시간이 길어질수록 방문자가 웹 페이지를 떠나는 비율이 높아진다. 반면에, 빠른 웹 페이지는 더 많은 고객 유입과 유지로 이어진다.

#### 검색 엔진 최적화 (SEO)

비즈니스 관점에서 SEO는 중요한 마케팅 수단이다. 구글은 2021년부터 웹 성능 지표가 검색 순위에 영향을 미친다고 발표했으며, 이는 로딩 속도가 빠른 사이트가 더 높은 검색 순위에 노출될 가능성이 크다는 것을 의미한다. 빠른 로딩 속도를 유지함으로써 검색 엔진에서의 가시성을 높이고 더 많은 트래픽을 확보할 수 있다.

#### 사용자 경험

사용자 경험은 웹 성능과 밀접하게 연결되어 있다. 특히 로딩 속도는 사용자의 첫인상과 사이트에 머무는 시간을 결정하는 중요한 요소이다. 사용자가 페이지 로딩 중 불편함을 느끼지 않도록 콘텐츠가 자연스럽게 표시되고, 탐색 시 지연이 최소화되면 더 나은 사용자 경험을 제공할 수 있다. 이는 고객 만족도를 높이고, 재방문율과 전환율 향상에 기여할 수 있다.

<br />
<br />

# 웹 성능 최적화하기

프론트엔드 성능은 크게 두 가지 요소로 나눌 수 있다. 하나는 **로딩 성능**이고, 다른 하나는 **렌더링 성능**이다. 로딩 성능은 페이지가 사용자에게 얼마나 빨리 보이는지를 의미하며, 렌더링 성능은 사용자가 웹 페이지와 상호작용할 때 얼마나 빠르고 효율적으로 반응하는지를 나타낸다. 이 두 성능 요소에서 문제를 찾아 해결하면 웹 페이지가 처음 뜨는 속도와 사용자 반응 속도가 빨라진다. 이를 통해 사용자는 페이지에 더 빨리 접속하고 원활하게 이용할 수 있어, 웹사이트의 이탈률을 낮추고 전환율을 높이는 데 도움이 된다.

## 렌더링 최적화

렌더링 최적화는 사용자의 상호작용에 대해 웹 페이지가 얼마나 빠르고 효율적으로 반응할 수 있는지를 개선하는 과정이다. 대표적인 작업으로는 불필요한 DOM 업데이트 최소화, CSS 애니메이션 최적화, 레이아웃 및 리페인트를 줄이는 기술 등이 있다. 렌더링 최적화를 통해 사용자의 스크롤, 버튼 클릭, 애니메이션 등의 상호작용에 더 매끄럽고 즉각적인 반응을 제공할 수 있다.

하지만 이번 글에서는 주로 로딩 속도에 초점을 맞출 예정이므로, 렌더링 최적화에 대해서는 다루지 않는다

## 로딩 최적화

로딩 최적화는 웹 페이지가 사용자에게 콘텐츠를 얼마나 빨리 제공할 수 있는지를 개선하는 과정이다. 이는 사용자의 초기 경험에 큰 영향을 미친다. 로딩 속도를 최적화하는 방법은 여러 가지가 있으며, 그중 몇 가지 주요 방법을 살펴보겠다.

### 1. 요청 크기 줄이기

웹 페이지가 로딩되는 동안 브라우저는 서버로부터 다양한 리소스를 요청한다. 이때 각 요청의 크기가 클수록 전송 시간이 길어지기 때문에, 파일 크기를 줄여 요청 크기를 최소화하는 것이 중요하다.

### 1-1. JavaScript 파일 압축 및 난독화

크기가 큰 JavaScript 파일은 로딩 속도를 저하시킬 수 있다. 이를 해결하기 위해 JavaScript 코드에서 불필요한 공백, 주석, 줄바꿈 등을 제거하고, 변수명과 함수명을 난독화하여 파일 크기를 줄일 수 있다.

#### TerserWebpackPlugin

Webpack의 [`TerserWebpackPlugin`](https://webpack.js.org/plugins/terser-webpack-plugin/)은 v5부터 압축과 난독화 작업을 프로덕션 모드에서 자동으로 수행한다. 개발 모드에서는 소스 맵과 디버깅 편의성을 위해 `TerserWebpackPlugin`이 자동으로 설정되지 않는다.

<figure align="center">
  <img src="./images/bundle_dev_mode_local.png" alt="bundle.js dev mode" />
</figure>

위 사진은 로컬에서 개발 모드로 실행했을 때 생성된 bundle.js 파일의 일부이다. 파일의 크기는 2.25MB이다. 개발 모드에서는 다음과 같은 특징이 있다.

- 코드가 읽기 쉽고 구조화되어 있다.
- 변수명과 함수명이 원본 그대로 유지되어 있다.
- 주석이 보존되어 있어 코드의 의도를 이해하기 쉽다.
- 모듈 구조와 의존성을 명확히 볼 수 있다.

이러한 특징은 개발자가 디버깅하고 코드를 이해하기 쉽게 만들어준다.

<figure align="center">
  <img src="./images/bundle_prod_mode_local.png" alt="bundle.js prod mode" />
</figure>

위 사진은 로컬에서 프로덕션 모드로 실행했을 때 생성된 bundle.js 파일의 일부이다. 파일의 크기는 1.27MB로 줄어들었다. 프로덕션 모드에서는 다음과 같은 특징이 있다.

- 난독화
  - 변수명과 함수명이 짧고 의미 없는 문자로 대체되었다.
  - 코드 구조가 복잡해져 원본 코드를 유추하기 어려워졌다.
- 최소화
  - 모든 불필요한 공백, 줄 바꿈, 주석이 제거되었다.
  - 가능한 경우 코드가 한 줄로 압축되었다.
  - 긴 변수명이 짧은 이름으로 대체되어 파일 크기가 줄어들었다.

이런 최적화 과정으로 파일 크기가 크게 줄어들어 웹 페이지의 로딩 속도가 향상되고, 원본 코드도 보호되는 효과가 있다.

<br />

### 1-2. CSS 압축 및 파일 분리

#### CssMinimizerPlugin

JavaScript와 마찬가지로 CSS 파일에도 불필요한 공백과 주석이 포함될 수 있다. 이를 제거하여 파일 크기를 줄이고 웹 페이지의 로딩 속도를 향상시킬 수 있다. CSS 파일의 최소화 과정에서는 Webpack의 [`CssMinimizerPlugin`](https://webpack.js.org/plugins/css-minimizer-webpack-plugin/)을 사용할 수 있다. 이 플러그인은 불필요한 공백, 주석, 줄바꿈 등을 제거해 CSS 파일을 최소화하고, 전송 데이터의 양을 줄여 웹 페이지의 로딩 성능을 개선한다.

#### MiniCssExtractPlugin

CSS 파일을 JavaScript에서 분리하는 것도 중요한 최적화 방법 중 하나이다. CSS를 JavaScript와 함께 번들링하지 않고 별도의 파일로 분리하면 필요한 CSS만 로드하고 불필요한 데이터 전송을 줄일 수 있다. 이를 위해 Webpack의 [`MiniCssExtractPlugin`](https://webpack.js.org/plugins/mini-css-extract-plugin/)을 사용할 수 있다. 이 플러그인은 CSS를 JavaScript에서 분리하여 `<style>` 태그 대신 `<link>` 태그로 HTML에 포함시키는 방식으로 동작한다. 이를 통해 초기 로딩 속도를 더욱 개선할 수 있지만, CSS 파일을 별도로 요청해야 하므로 HTTP 요청 수가 늘어나는 단점이 있다.

[Webpack 공식 문서](https://webpack.js.org/plugins/mini-css-extract-plugin/#recommended)에서는 빠른 개발 피드백과 실시간 CSS 반영을 위해 `style-loader`를 사용하고, 프로덕션 환경에서는 최적화를 위해 `MiniCssExtractPlugin`을 사용하는 것을 권장하고 있다.

`MiniCssExtractPlugin`은 일반적인 CSS 파일이나 CSS 모듈을 처리하는 데 적합하지만, CSS-in-JS 방식으로 작성된 스타일(Emotion, Styled Components 등)은 이 방식으로 추출할 수 없다. CSS-in-JS는 스타일을 JavaScript 코드 내에서 정의하고 런타임에 생성하기 때문에 빌드 시점에 이를 추출하는 것이 어렵다.

#### CssMinimizerPlugin과 MiniCssExtractPlugin 설정 방법

아래는 `CssMinimizerPlugin`과 `MiniCssExtractPlugin`을 Webpack에서 설정하는 예시이다.

```shell
npm install css-minimizer-webpack-plugin -D
npm install mini-css-extract-plugin -D
```

```js
// webpack.config.js

const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");

module.exports = {
  module: {
    rules: [
      {
        test: /.s?css$/,
        use: [MiniCssExtractPlugin.loader, "css-loader", "sass-loader"],
      },
    ],
  },
  optimization: {
    minimizer: [
      // webpack v5의 경우 기존 minimizer를 확장하려면 아래 줄의 주석을 해제한다.
      // `...`,
      new CssMinimizerPlugin(),
    ],
  },
  plugins: [new MiniCssExtractPlugin()],
};
```

<br />

### 1-3. HTTP 압축 방식 설정

현대 웹 애플리케이션은 리치 미디어, 대용량 JavaScript 파일, 다양한 서드파티 라이브러리 등을 포함하고 있어 전송되는 리소스의 크기가 커지는 경향이 있다. 이에 따라 HTTP 요청 시 더 많은 대역폭을 소모하게 되며, 사용자는 느린 로딩 속도를 경험할 수 있다. 이러한 문제를 해결하기 위한 효율적인 방법 중 하나는 서버에서 전송하는 리소스를 압축하여 클라이언트에 전달하는 것이다. HTTP 압축은 서버에서 이루어지며, 클라이언트는 압축된 데이터를 해제함으로써 네트워크 사용량을 줄이고 페이지 로딩 시간을 단축하는 효과를 얻을 수 있다.

#### Gzip 압축

Gzip은 가장 널리 사용되는 HTTP 압축 방식으로, 1992년에 처음 소개된 이후 거의 모든 주요 웹 브라우저와 서버에서 지원되고 있다. Gzip은 반복적인 문자열 패턴을 찾아 이를 단축된 형태로 저장하는 방식으로 작동한다. Gzip의 장점 중 하나는 높은 호환성이다. 대부분의 웹 브라우저와 서버는 Gzip 압축을 기본적으로 지원하기 때문에 클라이언트가 압축을 요청하지 않더라도 서버는 압축하지 않은 데이터를 제공할 수 있다.

#### Brotli 압축

Brotli는 2015년에 구글에서 개발한 HTTP 압축 알고리즘으로, Gzip보다 더 높은 압축률을 제공하는 것으로 알려져 있다. 특히 텍스트 기반 리소스에서 Gzip 대비 약 20-30% 더 작은 파일 크기를 제공할 수 있다. 또한, Brotli는 HTTP/2 환경에서 더 효율적으로 작동하도록 설계되었기 때문에 최신 웹 기술 스택을 사용하는 애플리케이션에서는 특히 더 효과적일 수 있다.

#### Gzip과 Brotli의 비교

Gzip과 Brotli는 모두 텍스트 기반 리소스의 압축에 적합하지만 성능 측면에서는 차이가 있다. Brotli는 Gzip보다 압축률이 더 우수하여 더 작은 파일 크기를 제공할 수 있다. 그러나 호환성 측면에서는 여전히 Gzip이 더 널리 사용된다. 대부분의 브라우저와 서버는 Gzip을 기본적으로 지원하며, Brotli의 지원은 상대적으로 제한적이다.

#### 클라이언트 및 서버 간의 압축 협상

서버는 클라이언트가 지원하는 압축 방식을 감지하고, 이에 따라 적절한 리소스를 제공할 수 있다. 클라이언트는 HTTP 요청 헤더의 `Accept-Encoding` 필드를 통해 지원하는 압축 방식을 명시하며, 서버는 이 정보를 기반으로 Gzip 또는 Brotli와 같은 압축된 리소스를 전송한다. 브라우저가 Brotli를 지원하는 경우 서버는 Brotli로 압축된 파일을 제공하고, 그렇지 않은 경우 Gzip으로 대체한다. 이 과정을 통해 클라이언트와 서버는 최적의 압축 방식을 협상할 수 있다.

```
GET /index.html HTTP/1.1
Host: www.example.com
Accept-Encoding: br, gzip
```

서버는 클라이언트의 요청에 따라 응답 헤더에서 압축 방식을 명시하여 리소스를 전송한다.

```
HTTP/1.1 200 OK
Content-Encoding: br
```

<br />

### 1-4. 이미지 최적화

이미지 파일은 웹 페이지에서 가장 큰 용량을 차지하는 경우가 많기 때문에, 이를 효율적으로 관리하면 페이지 로딩 시간을 크게 단축할 수 있다. 이미지 최적화의 핵심은 이미지의 용량을 줄이고, 적절한 포맷을 선택하며, 사용자에게 필요한 크기만큼만 제공하는 것이다.

#### 적절한 이미지 포맷 선택

각 이미지 포맷은 고유한 특성과 용도를 가지고 있으며, 이미지 종류에 따라 적절한 포맷을 선택하는 것이 중요하다.

<figure align="center">
  <img src="./images/image_format_comparison.png" alt="이미지 포맷 비교" />
</figure>

- JPEG는 널리 사용되는 이미지 포맷으로, 주로 복잡한 이미지를 표현하는 데 적합하다. 이 포맷은 손실 압축 방식을 사용하여 파일 크기를 줄이면서도 적정 수준의 이미지 품질을 유지할 수 있다. 그러나 JPEG는 압축 과정에서 일부 정보가 손실되므로, 반복적으로 편집하거나 저장할 경우 품질 저하가 발생할 수 있다는 단점이 있다. 그럼에도 불구하고 많은 웹사이트에서 JPEG는 파일 크기를 최소화하면서 고품질 이미지를 제공하는 데 여전히 인기 있는 선택이다.
- PNG는 무손실 압축을 지원하는 이미지 포맷으로, 고품질 이미지를 제공하며 투명도를 지원하는 특징이 있다. 이에 따라 로고, 아이콘 및 그래픽 디자인에서 자주 사용된다. PNG는 색상 깊이가 뛰어나 세밀한 디테일을 유지할 수 있지만, JPEG에 비해 파일 크기가 더 크고 복잡한 이미지에서는 비효율적일 수 있어 사용 목적에 따라 적절한 선택이 필요하다.
- WebP는 Google이 개발한 최신 이미지 포맷으로, JPEG와 PNG의 장점을 결합한 형태이다. WebP는 손실 및 무손실 압축 모두를 지원하며, 같은 품질의 이미지를 JPEG보다 약 25-34% 더 작은 크기로 저장할 수 있다. 이 포맷은 특히 웹에서 로딩 속도를 개선하는 데 유리하며, 투명도와 애니메이션 기능도 지원한다. 그러나 일부 구형 브라우저에서는 지원되지 않기 때문에 호환성을 고려해야 한다.
- AVIF는 최신 이미지 포맷으로, AV1 비디오 코덱을 기반으로 한다. 이 포맷은 뛰어난 압축 효율성을 자랑하며, 같은 이미지 품질을 유지하면서 JPEG보다 최대 50% 더 작은 파일 크기를 제공할 수 있다. AVIF는 고해상도 이미지와 HDR을 지원하며, 특히 현대적인 웹사이트에서 성능 최적화를 위한 유망한 선택으로 떠오르고 있다. 그러나 AVIF의 채택은 아직 초기 단계이며, 일부 구형 브라우저와 이미지 편집 도구에서는 지원되지 않을 수 있으므로 주의가 필요하다.

<br />
<br />

## 2. 필요한 것만 필요한 때에 요청하기

웹 애플리케이션의 성능을 최적화하는 중요한 방법 중 하나는 사용자가 필요로 할 때에만 리소스를 요청하는 것이다. 이를 통해 초기 로딩 속도를 개선하고, 불필요한 데이터 전송을 줄일 수 있다. 이를 실현하기 위한 대표적인 방법으로는 코드 스플리팅과 React의 `Lazy Loading` 및 `Suspense`가 있다.

### 2-1. 코드 스플리팅

코드 스플리팅은 애플리케이션의 JavaScript 파일을 여러 개의 청크로 나누어, 사용자가 필요로 할 때에만 해당 청크를 로드하는 기술이다.

#### SplitChunksPlugin

Webpack에서는 [`SplitChunksPlugin`](https://webpack.kr/plugins/split-chunks-plugin/#root)을 사용하여 공통 모듈을 별도로 청크로 분리하고, 애플리케이션의 크기를 줄일 수 있다.

```js
// webpack.config.js

module.exports = {
  //...
  optimization: {
    // ...,
    splitChunks: {
      chunks: "all",
    },
  },
};
```

`chunks: 'all'`은 모든 청크를 분리하여 별도의 파일로 로드하도록 설정하는 옵션이다. `chunks` 옵션에는 다음과 같은 값들이 사용될 수 있다.

- `async`: 비동기 chunk만 분리하여 별도의 파일로 로드한다.
- `initial`: 초기 chunk만 분리하여 별도의 파일로 로드한다.
- `all`: 모든 chunk를 분리하여 별도의 파일로 로드한다.

여기까지만 설정했을 때 다음과 같은 오류가 발생한다.

```
✖ ｢wdm｣: Error: Conflict: Multiple chunks emit assets to the same filename bundle.js (chunks 792 and 166)
```

이 오류는 Webpack 설정에서 여러 청크가 동일한 파일 이름으로 출력되도록 설정되어 있어 충돌이 발생하는 것이다.

이를 해결하려면 Webpack의 설정에서 `output.filename`을 변경하여 각 청크마다 고유한 파일 이름을 지정해야 한다. 예를 들어, [name].[contenthash].js와 같은 형식으로 파일 이름을 설정할 수 있다.

```js
// webpack.config.js

module.exports = {
  output: {
    filename: "[name].[contenthash].js",
  },
};
```

- `[name].[contenthash].js`와 `[name].bundle.js`의 차이
  - `[name].[contenthash].js`: 이 설정은 파일 이름에 컨텐츠 해시를 포함된다. 컨텐츠 해시는 파일의 내용에 따라 생성되므로, 파일의 내용이 변경되면 해시도 변경되기 때문에 브라우저 캐싱을 효과적으로 사용할 수 있다.
  - `[name].bundle.js`: 이 설정은 파일 이름에 컨텐츠 해시를 포함하지 않는다. 따라서 파일의 내용이 변경되어도 파일 이름이 변경되지 않기 때문에 브라우저 캐싱을 효과적으로 사용할 수 없다.

<br />

### 2-2. React Lazy Loading

React에서는 `React.lazy`와 `Suspense`를 사용하여 컴포넌트를 지연 로딩할 수 있다. 이 두 가지를 활용하면, 특정 컴포넌트를 실제로 필요할 때만 로드하여 초기 렌더링 성능을 개선할 수 있다.

아래 코드는 예시로 만든 React 프로젝트이다. Home과 Search 두 개의 페이지가 있으며, `react-router-dom`을 사용하여 페이지 간 라우팅을 처리하고 있다.

```tsx
import { BrowserRouter as Router, Route, Routes } from "react-router-dom";

const App = () => {
  return (
    <Router>
      <NavBar />
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/search" element={<Search />} />
      </Routes>
      <Footer />
    </Router>
  );
};

export default App;
```

이를 특정 페이지에서 사용되는 파일만 불러오는 방식으로 초기 로딩 속도를 개선할 수 있다. 이때 사용하는 개념은 `dynamic import`와 `React.lazy`이다.

`dynamic import`는 EcmaScript 모듈을 비동기적으로 동적으로 불러오는 방법이다. `dynamic import`를 사용하면 의존하는 모듈을 하나의 번들에 포함하지 않고, 코드 실행 시 해당 모듈이 필요할 때만 로드할 수 있다. 그러나 `dynamic import`는 모듈을 `Promise` 형태로 반환하기 때문에, 로드된 모듈을 사용하려면 `then()`을 통해 비동기적으로 처리해야 하는 번거로움이 있다.

```jsx
import("./SomeComponent").then((module) => {
  const SomeComponent = module.default;
  // 컴포넌트를 여기서 사용
});
```

`React.lazy`는 이러한 비동기 처리와 `Promise` 객체를 신경 쓰지 않고도 동적으로 컴포넌트를 불러올 수 있도록 도와준다. `React.lazy`를 사용하면 아래와 같이 간단하게 컴포넌트를 불러올 수 있다.

```jsx
const SomeComponent = React.lazy(() => import("./SomeComponent"));
```

그리고 이를 `Suspense`로 감싸면, 컴포넌트를 비동기적으로 로드하고 로드되는 동안 로딩 화면을 표시하는 방식으로 처리할 수 있다.

```jsx
import { BrowserRouter as Router, Route, Routes } from "react-router-dom";

import { lazy, Suspense } from "react";

const App = () => {
  return (
    <Router>
      <NavBar />
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/search" element={<Search />} />
        </Routes>
      </Suspense>
      <Footer />
    </Router>
  );
};

export default App;
```

위 코드에서는 `React.lazy`를 사용해 Home과 Search 컴포넌트를 동적으로 불러오고, `Suspense`를 이용해 로딩 중에 표시할 대체 콘텐츠를 지정하고 있다. 이를 통해 초기 로딩 시간을 단축하고, 사용자가 실제로 필요로 하는 코드만 로드하여 성능을 최적화할 수 있다.

이 코드를 적용한 후, `webpack-bundle-analyzer`를 사용해 번들 크기를 분석하면 Home과 Search 페이지의 소스 코드가 각각 분리된 것을 확인할 수 있다.
| splitChunks 사용 후 | React.lazy 사용 후 |
| ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| ![splitChunks 사용 후 ](./images/splitChunks_buldle_analysis.png) | ![React.lazy 사용 후](./images/react_lazyloading_bundle_analysis.png) |

| Home 소스 코드                                                                     | Search 소스 코드                                                                     |
| ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| ![Home 소스 코드](./images/home_bundle_analysis.png) (601.16c1283ec74895a77004.js) | ![Search 소스 코드](./images/home_bundle_analysis.png) (356.f674814de8d3de5a160d.js) |

<br />
<br />

## 마치며

웹 성능 최적화는 사용자 경험부터 비즈니스 성과, 검색 엔진 순위까지 여러 면에서 큰 영향을 미친다.
빠르고 효율적인 웹 페이지는 사용자의 이탈을 줄여주고, 기업의 매출이나 성장에도 긍정적인 효과를 줄 수 있다. 또한, 검색 엔진 순위에서도 유리한 위치를 잡는 데 도움이 된다.

이 글에서 다룬 최적화 기법을 적용하면 로딩 속도가 개선되어 사이트 성능 향상에 기여할 수 있을 것이다.
웹 성능 최적화는 단번에 끝나는 작업이 아니라 지속적으로 관리하고 모니터링해야 할 과제이기 때문에, 이를 염두에 두고 웹 페이지의 성능을 꾸준히 점검해 나가는 것이 중요하다.

<br />
<br />

## 참고 자료

- [Using Modern Image Formats: AVIF And WebP](https://www.smashingmagazine.com/2021/09/modern-image-formats-avif-webp/)
- [성능 최적화 1 — 번들 크기 줄이기 (React, Webpack, Minify, Code Splitting)](https://medium.com/@uk960214/성능-최적화-1-번들-크기-줄이기-react-webpack-minify-code-splitting-e2391e7e5f1b)
