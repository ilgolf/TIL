# PostCSS

## PostCss란?

PostCSS는 CSS를 후처리하는 도구. Javascript 기반의 플러그인 아키텍처를 사용하여 CSS 코드를 변환, 분석, 최적화 할 수 있다. PostCSS는 단독으로는 아무 기능이 없다.
하지만 다양한 플러그인을 통해 강력한 기능을 제공할 수 있다.

## 주요 기능

1. CSS 전처리 및 후처리: SCSS, LESS와 유사한 기능을 제공하여 CSS 코드를 더 쉽게 작성하고 유지보수 할 수 있게 한다.

2. CSS 모듈화: CSS파일을 모듈화 하여 코드의 재사용성을 높이고, 전역 네임스페이스 오염을 방지할 수 있다.

3. 자동 벤더 프리픽스 추가: `Autoprefixer` 같은 플러그인을 사용하여 다양한 브라우저 호환성을 위해 CSS에 자동으로 벤더 프리픽스를 추가

4. 미니피케이션 및 최적화: CSS코드를 압축하고 최적화하여 파일 크기를 줄이고 로딩 속도를 향상 시킨다.

5. 미래 CSS문법 사용: 현재 브라우저에서 지원하지 않는 최신 CSS 문법을 사용할 수 있게 변환해 준다.

## 사용법

1. PostCSS 및 플러그인 설치 

```bash
npm install postcss postcss-cli autoprefixer
```

2. PostCSS 설정 파일 작성

```javascript
module.exports = {
    plugins: [
        require('autoprefixer')
    ]
}
```

3. CSS처리: CLI 또는 빌드도구(ex. webpack, Gulp)를 사용하여 CSS파일을 처리한다.

```bash
npx postcss input.css -o output.css
```

## tailwind와 함께 사용 사례

Tailwind는 PostCSS와 같이 사용되는 경우가 많다. Tailwind CSS는 유틸리티 기반의 CSS 프레임워크이며 CSS 클래스들을 미리 정의해 두어 빠르게 스타일링이 가능하다. 

1. TailWind CSS 및 PostCSS 설치

```bash
npm install tailwindcss postcss postcss-cli autoprefixer
```

2. Tailwind CSS 설정 파일 생성

```bash
npx tailwindcss init
```

3. PostCSS 설정 파일 작성  및 CSS 파일 생성 

postcss.config.js

```javascript
module.exports = {
  plugins: [
    require('tailwindcss'),
    require('autoprefixer'),
  ],
};
```

main.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

4. 빌드 스크립트 추가

```json
"scripts": {
  "build:css": "postcss styles.css -o output.css"
}
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="output.css" rel="stylesheet">
    <title>Document</title>
</head>
<body>
    <div class="bg-blue-500 text-white p-4">
        Hello, Tailwind CSS with PostCSS!
    </div>
</body>
</html>
```

`npm run build:css` 로 빌드 후 실행하면 Tailwind로 쉽게 유틸리티 클래스들을 사용하여 스타일링 할 수 있다.