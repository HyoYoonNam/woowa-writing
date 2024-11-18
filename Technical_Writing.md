# 번들 사이즈 최적화를 통한 웹 성능 최적화

### 번들 사이즈가 웹 사이트에 미치는 영향

웹 사이트가 복잡해지면서 번들 사이즈도 함께 증가하는 경향이 있습니다. 특히 사용자 경험을 강화하기 위해 다양한 기능과 복잡한 인터페이스를 추가하면서 자바스크립트, CSS, 이미지 등의 리소스 크기가 커지는 문제가 발생합니다. 이로 인해 초기 로딩 시간이 길어지고, 이는 사용자 경험과 웹 사이트 성능에 부정적인 영향을 미칩니다.

[KissMetrics 연구 결과](https://neilpatel.com/blog/loading-time/)에 따르면 페이지 로드 시간이 1초씩 길어질 때마다 웹 페이지 이탈률이 7%씩 증가한다고 합니다. 전자상거래 사이트의 경우, 이는 매출 감소로 이어질 수 있습니다. 사용자는 로딩 시간이 길어지면 페이지를 떠날 가능성이 높아지고, 특히 구매 과정에서 지연이 발생하면 구매를 포기할 확률이 증가합니다. 실제로 대형 전자상거래 사이트에서 [로딩 속도가 1초만 느려져도 매출에 심각한 영향을 받을 수 있다는 연구 결과](https://www.akamai.com/resources/white-paper/how-web-and-mobile-performance-optimize-conversion-and-user-experience)가 있습니다.

그렇다면 이 문제를 어떻게 해결할 수 있을까요? 바로 번들 사이즈 최적화를 통해 해결할 수 있습니다. 불필요한 코드를 제거하고, 필요한 자원만 즉시 로드할 수 있도록 코드를 최적화하면 네트워크 요청 시간을 줄일 수 있습니다. 이를 통해 초기 로딩 속도를 개선하고, 사용자 이탈률을 낮출 수 있습니다. 번들 사이즈를 줄임으로써 FCP(First Contentful Paint)와 LCP(Largest Contentful Paint) 등의 성능 지표도 개선될 수 있습니다. 결과적으로 번들 사이즈 최적화는 더 나은 사용자 경험을 제공하며 SEO 성능까지 향상시킬 수 있습니다.

<br/>

## 번들 사이즈 및 성능을 확인하는 방법

번들 사이즈와 웹 사이트 성능을 확인하는 방법으로는 여러 가지가 있지만, 그중 가장 기본적인 방법은 **크롬 개발자 도구**를 사용하는 것입니다. 이 도구는 웹 페이지의 성능을 분석하고 번들 크기를 줄이기 위한 최적화 작업을 쉽게 수행할 수 있게 해줍니다.

모든 리소스를 최적화하는 것은 이상적이지만 현실적으로는 어렵기 때문에 우선순위를 설정해야 합니다. 번들 최적화는 특히 **초기 로딩 속도**에 큰 영향을 미치는 자바스크립트와 CSS에 집중하는 것이 좋습니다. 특히, 대용량 라이브러리, 중복 코드, 또는 사용되지 않는 CSS와 자바스크립트는 최적화 우선 대상입니다. 이런 부분을 최적화하면 웹사이트 전체의 성능 향상에 큰 영향을 미칠 수 있습니다.

### 1. 크롬 개발자 도구

크롬 개발자 도구는 네트워크 요청, 리소스 로딩 시간, 자바스크립트 실행 상태 등 웹 성능과 관련된 다양한 데이터를 시각적으로 제공해줍니다. 이를 통해 웹 페이지의 로딩 속도를 저해하는 요인들을 파악하고 개선할 수 있습니다.

- **Network 탭**

  <img src="./image/F12_Network.png" width="80%" />

  Network 탭에서는 페이지가 로드되는 동안 발생하는 모든 네트워크 요청을 확인할 수 있습니다. 이 탭을 사용하면 웹 사이트가 로딩 중인 API 호출, 데이터 요청, 그리고 번들 파일을 포함한 모든 리소스를 분석할 수 있습니다. 각 요청의 크기와 응답 시간을 상세히 제공하기 때문에 불필요하게 큰 번들이나 느린 API 응답으로 인한 성능 저하를 쉽게 파악할 수 있습니다.

<br/>

- **Performance 탭**

  <img src="./image/F12_Performance.png" width="80%" />

  Performance 탭은 웹 페이지의 성능을 더욱 정밀하게 분석할 수 있는 도구입니다. 특정 시점의 이벤트들을 확인하여 렌더링, 페인트, 레이아웃 계산 등 다양한 단계에서 발생하는 성능 저하를 감지할 수 있습니다. 이를 통해 웹 사이트가 어떻게 렌더링되고 있는지, 그리고 번들 크기나 코드의 복잡성이 성능에 어떤 영향을 미치는지 자세히 분석할 수 있습니다.

<br/>

- **Coverage 탭**

  <img src="./image/F12_Coverage.png" width="80%" />

  Coverage 탭은 웹 페이지에서 실제로 사용된 코드와 사용되지 않은 코드를 색상으로 구분해 보여줍니다. 이를 통해 불필요한 코드를 식별하고 제거하여 번들 사이즈를 줄일 수 있습니다.

<br/>

### 2. Lighthouse

<img src="./image/Lighthouse_Logo.png" width="30%" />

Lighthouse는 구글에서 제공하는 오픈소스 자동화 도구로, 웹 사이트의 성능과 접근성, SEO, 프로그레시브 웹 앱(PWA) 지원 여부 등을 종합적으로 분석할 수 있습니다. 특히 웹 성능과 관련된 리포트를 제공하기 때문에 웹 사이트가 얼마나 빠르고 효율적으로 동작하는지 확인하는 데 유용합니다. 크롬 웹 스토어에서 설치하여 크롬 개발자 도구 내에서 실행할 수 있습니다.

<img src="./image/Lighthouse_Report.png" width="80%" />


<br/>

### 3. Webpack Bundle Analyzer

Webpack Bundle Analyzer는 Webpack 번들의 크기를 시각적으로 분석하고 최적화할 수 있는 도구입니다. 이 도구는 번들의 구조를 트리 맵(tree map) 형태로 보여주어, 어떤 모듈이 가장 많은 용량을 차지하는지, 중복된 모듈이 있는지 등을 쉽게 파악할 수 있도록 도와줍니다. 이를 통해 불필요하게 번들 사이즈를 크게 만드는 원인을 찾아내어 번들 최적화 작업을 효율적으로 진행할 수 있습니다.

<img src="./image/Webpack_Bundle_Analyzer.png" width="80%" />

- **Webpack Bundle Analyzer 설치 방법**

  다음의 명령어를 통해 설치한 뒤, 프로젝트의 Webpack 설정에 코드를 추가합니다.
  ```bash
  # NPM
  npm install --save-dev webpack-bundle-analyzer
  # Yarn
  yarn add -D webpack-bundle-analyzer
  ```

  ```js
  // webpack.config.js

  const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

  module.exports = {
    plugins: [
      new BundleAnalyzerPlugin()
    ]
  }
  ```

  자세한 옵션은 [webpack-bundle-analyzer github](https://www.npmjs.com/package/webpack-bundle-analyzer)에서 확인할 수 있습니다.

<br/>

## 번들 사이즈 최적화의 주요 기법

### 1. Webpack mode 설정

React에서 Webpack을 번들러로 사용하는 경우 번들 최적화를 위해 mode를 적절하게 지정하는 것이 중요합니다. [Webpack 공식 문서](https://webpack.js.org/configuration/mode/)에 따르면, Webpack의 mode 옵션은 크게 `development`, `production`, `none` 세 가지가 있고, 기본값으로는 `production`이 지정됩니다.

그렇다면 왜 Webpack의 mode 설정이 중요할까요? Webpack의 mode는 번들의 목적(개발용 또는 배포용)에 따라 최적화 수준을 결정합니다. `production` 모드에서는 코드 난독화, Tree Shaking, 파일 압축 등의 최적화 작업이 자동으로 수행되며, 결과적으로 번들 크기를 줄이고 로딩 속도를 개선합니다. Webpack 버전 5부터는 [TerserWebpackPlugin](https://webpack.js.org/plugins/terser-webpack-plugin/)이 내장되어 있어 추가적인 설정 없이도 자바스크립트 코드가 난독화되고 최적화됩니다. 반대로, `development` 모드는 디버깅과 개발 편의성을 위해 소스 맵 생성, 코드 가독성 유지 등을 기본값으로 적용하여 최적화되지 않은 번들을 생성합니다.


- **`development` 모드와 `production` 모드 비교**

  다음은 동일한 프로젝트를 `development` 모드와 `production` 모드에서 빌드한 결과를 비교한 사례입니다.

  먼저 `development` 모드에서 빌드한 프로젝트 코드를 살펴보겠습니다.

  ```js
  // webpack.config.js

  module.exports = {
    mode: 'development',
    // ... 생략
  };
  ```

  이 모드에서는 코드가 최적화되지 않으며 소스 맵(source map)이 포함되어 디버깅에 도움이 됩니다. 또한, 핫 리로딩 기능 등 개발 중 필요한 기능들이 활성화됩니다. 그러나 최종 배포용으로는 적합하지 않으며 번들 크기가 매우 커질 수 있습니다.

  <br/>

  Webpack Bundle Analyzer에서 확인한 모습은 다음과 같습니다. 

  <img src="./image/BA_development.png" width="80%" />

  위 이미지는 Webpack Bundle Analyzer로 `development` 모드에서 빌드한 번들 구조를 시각화한 결과입니다. 14.06MB의 큰 번들 파일을 확인할 수 있으며, 이는 권장하는 크기(약 244KB) 보다 약 50배 이상 초과합니다.
  
  <br/>

  
  <img src="./image/Lighthouse_development.png" width="80%" />

  또한, S3와 CDN으로 배포 후 Lighthouse로 측정한 성능 지표에서도 Performance 수치가 매우 낮게 나왔습니다. 번들 사이즈가 매우 크기 때문에 LCP 및 TBT(Total Blocking Time)와 같은 주요 성능 지표가 크게 저하되었습니다.

  <br/>

  다음으로 `production` 모드에서 프로젝트 코드를 빌드한 결과를 살펴보겠습니다.

  ```js
  // webpack.config.js

  module.exports = {
    mode: 'production',
    // ... 생략
  };
  ```

  `production` 모드에서는 Webpack이 자동으로 최적화를 수행하여 번들 크기를 줄입니다. 압축과 난독화, Tree Shaking 등이 적용되어 더 작은 번들이 생성됩니다.


  <img src="./image/BA_production.png" width="80%" />

  Webpack Bundle Analyzer로 분석한 결과, 번들 크기는 1.51MB로 `development` 모드와 비교했을 때 약 14배 감소했습니다.
  
  <br/>


  <img src="./image/Lighthouse_production.png" width="80%" />

  Lighthouse로 측정한 성능 지표도 개선되었습니다. Performance 점수가 39점에서 67점으로 상승했으며, 특히 LCP와 TBT가 크게 감소하여 웹 페이지의 초기 로딩 성능이 개선되었습니다. 번들 크기가 줄어듦에 따라 사용자가 더 빠르게 웹 페이지를 볼 수 있고, 상호작용도 빨라졌습니다.

<br/>

 Webpack을 `production` 모드로 설정하면 번들 사이즈가 크게 줄어들고 성능이 향상됩니다. 이를 통해 웹 성능 최적화와 사용자 경험에 긍정적인 영향을 미칩니다. 따라서 배포 환경에서는 `production` 모드를 사용하여 빌드하는 것이 좋습니다.

<br/>


### 2. CSS 최적화

웹 애플리케이션이 점점 더 복잡해지면서 CSS 파일의 크기도 함께 증가하고 있습니다. 특히, 대규모 프로젝트에서는 중복된 스타일, 사용되지 않는 CSS, 그리고 불필요한 공백과 주석이 포함되어 번들 크기가 불필요하게 커지는 경우가 많습니다. 이런 상황은 초기 로딩 속도를 저하시킬 뿐만 아니라, 모바일 환경과 같은 네트워크가 제한된 상황에서 사용자 경험에 부정적인 영향을 미칠 수 있습니다.

CSS 최적화는 이러한 문제를 해결하기 위해 등장했습니다. **CSS 파일의 크기를 줄이고 불필요한 요소를 제거**함으로써 브라우저가 더 빠르게 스타일을 처리할 수 있도록 도와줍니다. 이를 통해 웹 페이지 로딩 시간을 단축하고, 사용자 이탈률을 줄이며, 웹 성능을 전반적으로 향상시킬 수 있습니다.

Webpack과 같은 번들러를 사용하면 CSS 파일을 자바스크립트와 분리하여 관리할 수 있으며, 이를 통해 **필요한 CSS만 로드**하도록 최적화할 수 있습니다. Webpack의 `mini-css-extract-plugin`과 `css-minimizer-webpack-plugin`은 CSS를 독립적인 파일로 추출하고, 파일을 최소화하여 번들 크기를 줄이는 데 효과적인 도구입니다.


```tsx
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');

module.exports = {
  mode: 'production',
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [MiniCssExtractPlugin.loader, 'css-loader'],
      },
    ],
  },
  plugins: [
    new MiniCssExtractPlugin(), // CSS를 별도 파일로 추출
  ],
  optimization: {
    minimizer: [
      '...', // 기본 TerserWebpackPlugin을 유지하기 위해 스프레드 연산자 사용
      new CssMinimizerPlugin(), // CSS 파일 최소화
    ],
  },
};
```

`mini-css-extract-plugin`으로 번들에서 CSS를 별도의 파일로 추출하고, `css-minimizer-webpack-plugin`을 통해 CSS 파일을 최소화합니다.

이때, 주의할 점은 `minimizer` 설정에서 스프레드 연산자 `'...'`를 추가해야 한다는 것입니다. Webpack은 기본적으로 자바스크립트 파일을 최적화하기 위해 `TerserWebpackPlugin`을 내장하고 있습니다. 이 플러그인을 유지하면서 `CssMinimizerPlugin`을 추가하기 위해서는 스프레드 연산자를 사용해야 합니다. 스프레드 연산자를 사용하지 않으면 기본 자바스크립트 최적화 플러그인이 덮어쓰여지고, CSS 최적화만 적용될 수 있기 때문에 주의해야 합니다.

- **CSS 최적화 결과 분석**

  CSS 최적화 전후의 번들 사이즈를 비교해 보겠습니다.

  <img src="./image/BA_production.png" width="80%" />

  CSS 최적화 전에는 `production` mode로 빌드했을 때, 파싱된 번들 사이즈가 **1.51MB**였습니다.

  <img src="./image/CSS_Minify_After.png" width="80%" />

  CSS 최적화 후 같은 방법으로 빌드했을 때, 번들 사이즈는 **1.47MB**로 감소했습니다. 최적화 과정에서 CSS 파일이 별도로 분리되었고, 0.04MB 크기의 CSS 파일이 번들에서 제외되었습니다. 이를 통해 전체 번들 사이즈가 소폭 줄어들었습니다.

  <img src="./image/CSS_Minify_After_Files.png" width="80%" />

<br/>

- **번들 사이즈 감소율**

  최적화 전후의 번들 크기를 정리한 표는 다음과 같습니다.

  | CSS 최적화 전 | CSS 최적화 후 | 감소율 |
  |---------------|---------------|---------|
  | 1.51MB        | 1.47MB        | 2.65%   |

<br/>

- **성능 지표 분석**

  WebpageTest에서 측정한 결과, TTFB(Time To First Byte), FCP, SI(Speed Index), LCP 등의 Web Core Vitals 지표는 거의 변화가 없었습니다. 번들 크기가 약간 줄어들었지만, 감소율이 **2.65%** 에 불과해 실질적인 성능 향상은 미미했습니다.
  |   | 이미지 |
  |----|--------|
  |  WebpageTest CSS 최적화 전  | <img src="./image/CSS_Minify_Before_WebpageTest.png" width="90%" /> |
  | WebpageTest CSS 최적화 후  | <img src="./image/CSS_Minify_After_WebpageTest.png" width="90%" /> |

<br/>

### 3. 코드 스플리팅 (Code Splitting)

더 큰 성능 개선 효과를 얻고 싶다면 코드 스플리팅을 활용할 수 있습니다.

- **코드 스플리팅과 Lazy Loading**
  코드 스플리팅은 초기 로딩 속도를 개선하기 위해 자바스크립트 파일을 여러 개의 작은 번들로 나누어 특정 페이지나 기능에 필요한 코드만 로드하도록 하는 기법입니다. 특히 사용자가 애플리케이션의 특정 기능만을 사용할 때 전체 자바스크립트 파일을 로드하지 않고 필요한 부분만 불러올 수 있어 효율적입니다.

  Lazy Loading은 코드 스플리팅으로 분할된 코드 청크를 실제로 필요한 시점에 비동기적으로 로드하는 방법입니다. 예를 들어, 사용자가 특정 페이지나 기능을 요청하기 전까지는 해당 코드를 로드하지 않습니다.

- **Lazy loading 적용 예시**

  아래 코드는 Lazy Loading을 적용한 프로젝트 코드입니다. `MainPage`를 대표로 소개하고 있으며, 다른 페이지의 코드에도 `lazy`가 적용되어 있습니다.


  ```tsx
  import React, { lazy, Suspense } from 'react';
  import LoadingSpinner from './components/common/LoadingSpinner/LoadingSpinner';
  import QueryErrorBoundary from './components/common/Error/QueryErrorBoundary';
  // ... 기타 import

  // Lazy Loading
  const MainPage = lazy(() => import('./pages/MainPage'));

  const root = ReactDOM.createRoot(document.getElementById('root') as HTMLElement);

  const routes = [
    {
      path: ROUTES.main,
      element: (
        <QueryErrorBoundary>
          <App>
            <Suspense fallback={<LoadingSpinner />}>
              <MainPage />
            </Suspense>
          </App>
        </QueryErrorBoundary>
      ),
    },
    // ... 다른 페이지들
  ];

  export const router = createBrowserRouter(routes, {
    basename: ROUTES.main,
  });

  root.render(
    <React.StrictMode>
      <RouterProvider router={router} />
    </React.StrictMode>,
  );

  ```

  Lazy Loading 적용 후 Bundle Analyzer로 확인했을 때, 다음과 같이 여러 청크로 나뉘어 있음을 확인할 수 있습니다.

  |   | 이미지 |
  |----|--------|
  |  Lazy Loading 적용 전  | <img src="./image/CSS_Minify_After.png" width="90%" /> |
  | Lazy Loading 적용 후  | <img src="./image/Lazy_Loading_After.png" width="90%" /> |

  각 번들 사이즈도 줄어든 것을 볼 수 있습니다. 메인 페이지에서 불러오는 청크 파일 두 종류는 다음과 같은데요, 
  <img src="./image/MainPage_chunk.png" width="80%" />

  Bundle Analyzer를 통해 확인하면 사이즈는 다음과 같습니다.

  |   | 이미지 | 파싱 사이즈 | gzip 압축 사이즈 |
  |----|--------|---|---|
  |  메인 페이지 index.js 청크  | <img src="./image/MainPage_index.png" width="90%" /> | 312.79KB | 109.05KB|
  | 메인 페이지 MainPage 청크  | <img src="./image/MainPage_main.png" width="90%" /> | 10.3KB | 3.54KB|

  WebpageTest 결과도 크게 개선되었습니다.

  |   | 이미지 |
  |----|--------|
  | Lazy Loading 적용 전 WebpageTest |<img src="./image/CSS_Minify_After_WebpageTest.png" width="90%" /> |
  | Lazy Loading 적용 후 WebpageTest |<img src="./image/Lazy_Loading_After_WebpageTest.png" width="90%" /> |

  지표를 자세히 살펴보면 다음과 같습니다.

  | Lazy Loading | FCP | LCP | TBT | CLS | SI |
  |----|---|---|---|---|---|
  | 적용 전 | 5.420s | 6.358s | 0.000s | 0 | 6.423s |
  | 적용 후 | 2.791s | 4.671s | 0.000s | 0 | 4.704s |
  | 개선율 | 48.5% 개선 | 26.5% 개선 | - | - | 26.8% 개선 |

  위의 지표에서 알 수 있듯이, 코드 스플리팅과 Lazy Loading을 적용하면 웹 사이트의 성능이 크게 향상됩니다. 특히 FCP는 약 48.5% 개선되어 사용자에게 초기 콘텐츠를 더 빠르게 제공합니다. LCP와 SI도 각각 **26.5%** 와 **26.8%** 의 개선을 보여 전반적인 로딩 속도가 향상되었습니다.

  코드 스플리팅은 웹 사이트 성능 최적화에 중요한 역할을 합니다. 초기 로딩 시간을 단축하여 사용자가 페이지에 빠르게 접근할 수 있도록 하고, 네트워크 트래픽을 효율적으로 관리하여 불필요한 리소스 로딩을 방지합니다. 이는 곧 사용자 경험의 향상으로 이어집니다.

  모듈 번들러와 프레임워크의 기능을 적극 활용하여 코드 스플리팅을 구현하면 더욱 빠르고 효율적인 서비스를 개발할 수 있습니다. 예를 들어, Webpack의 동적 import나 React의 `React.lazy`와 `Suspense`를 사용하여 필요한 시점에만 코드를 로드할 수 있습니다.

<br/>

## 결론

번들 사이즈 최적화는 웹 성능을 향상시키는 데 필수적인 요소입니다. 번들 크기를 줄이면 초기 로딩 시간이 단축되어 사용자에게 더 빠른 서비스를 제공할 수 있습니다. 이는 사용자 이탈률 감소와 전환율 상승으로 이어져 비즈니스 측면에도 긍정적인 영향을 미칩니다.

지금까지 Webpack의 `production` 모드 설정을 통해 기본적인 코드 압축과 최적화를 적용하였고, `mini-css-extract-plugin`과 `css-minimizer-webpack-plugin`을 사용하여 CSS 파일도 효율적으로 관리할 수 있었습니다. 또한, `Lazy Loading`을 도입하여 필요한 코드만 필요한 시점에 로드하는 `코드 스플리팅`을 적용함으로써 번들 사이즈를 더욱 줄일 수 있었습니다.

번들 사이즈 최적화는 단순히 파일 크기를 줄이는 것을 넘어 사용자 경험과 비즈니스 성과에 직접적인 영향을 끼칩니다. 번들 사이즈 최적화를 통해 웹 성능을 향상하여 더 빠르고 효율적인 웹 서비스를 제공한다면, 사용자 만족도와 전환율을 높일 수 있을 것입니다.

<br/>

### 참고 자료
- 유동균, 프론트엔드 성능 최적화 가이드, 인사이트, 2022.
- [How Loading Time Affects Your Bottom Line](https://neilpatel.com/blog/loading-time/)
- [How Web and Mobile Performance Optimize Conversion and User Experience](https://www.akamai.com/resources/white-paper/how-web-and-mobile-performance-optimize-conversion-and-user-experience)
- [webpack-bundle-analyzer npm package](https://www.npmjs.com/package/webpack-bundle-analyzer)
- [Webpack document - mode](https://webpack.js.org/configuration/mode/)
- [Webpack document - TerserWebpackPlugin](https://webpack.js.org/plugins/terser-webpack-plugin/)
- [Webpack document - code splitting](https://webpack.kr/guides/code-splitting/)
- [Lighthouse document](https://developer.chrome.com/docs/lighthouse/overview?hl=ko)
- [React document - lazy](https://react.dev/reference/react/lazy)
