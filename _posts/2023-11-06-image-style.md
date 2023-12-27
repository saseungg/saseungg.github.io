---
title: 개츠비 이미지 width값 조정
date: 2023-11-06 12:21:00 +0800
categories: [Blog]
tags: [gatsby]
---

개인 블로그를 만들면서 가장 하고 싶었던 건 첨부하는 이미지의 스타일을 입히는 거였다. 이미지에 스타일을 주느냐 안주느냐에 따라 느낌이 확 달라진다.
그림을 예시를 두고 차이를 비교해보자.

![](/assets/img/contents/image-style/border-example.jpeg){: w="500"}

왼쪽은 원본 이미지 그대로 붙여넣었고 오른쪽은 이미지에 radius와 border를 적용한 모습이다. 딱 봤을 때 오른쪽이 훨씬 깔끔하고 보기 편하다. 또한 블로그에 들어가는 모든 이미지들에게 통일성을 준다면 전체적인 스타일과 가독성도 올라간다.

이미지 스타일을 주기 위해 개츠비에 스타일을 적용하기 위한 방법을 찾아봤다.

## 삽질의 과정들

먼저 다양한 블로그들을 확인해봤다. 스타일이 있는 이미지들을 확인해보면 대부분 원본 이미지 자체에서 스타일을 넣은 상태로 png 형식으로 이미지를 첨부한 것을 확인할 수 있었다.

맥북에서는 스크린 캡쳐(cmd+shigt+4+spacebar)를 하면 자동으로 radius와 그림자가 들어가지만 전체 화면을 캡쳐해야만 적용됐기에 피그마에서 이미지 스타일을 적용한 후 png 파일로 넣어줬다.

개츠비에서 마크다운으로 이미지를 넣는 방법은 두가지이다. 마크다운 이미지 첨부 방식으로 넣는 방법과 HTML형식으로 넣는 방법.

이미지를 첨부하면 png 파일이 제대로 들어간다. 마크다운은 원래 사이즈 조절이 되지만 개츠비는 이미지를 webp형식으로 변환 후 url을 백그라운드로 넣기 때문에 width 값을 조절할 수 없다.

확인을 위해 아래와 같은 방법으로 테스트를 해보았다.

```js
// 마크다운
![](./images/design-system/text.png)

// img 태그
<img src="./images/design-system/text.png" width="300px" />

// url로 변환한 이미지를 첨부
<img src="https://github.com/saseungg/saseungg.github.io/assets/115215178/54b6ad74-82e7-4050-b197-33e42e79f100" width="300px" />
```

![](/assets/img/contents/image-style/example.gif)

로컬에 있는 이미지가 잘들어가긴 하지만 너비 조절이 되지 않고 png 형식이 바로 적용되지 않고 새로고침해야 적용되는 것을 볼 수 있다.

url로 변환한 이미지는 width 조절이 되지만 한가지 문제점이 있었다.

![](/assets/img/contents/image-style/image-zoom.gif)

적용한 플러그인 중에 gatsyb-image-zoom 이라는 플러그인을 사용 중인데 이 플러그인은 이미지를 클릭했을 때 모달형식으로 줌 한 이미지를 볼 수 있다.

url로 변환한 이미지에는 이미지를 확대할 수 없었다.

따라서 url도 img 태그를 사용하는 것도 해결책이 되지 않았다. 구글링을 하던 중 개츠비 repo에 이슈를 발견했다.

![](/assets/img/contents/image-style/issue.jpeg)

width 값을 조정할 수 있는 플러그인을 누가 만들었다!! 얄루!

기존에 있던 gatsby-remark-image를 삭제하고 해당 플러그인을 다운 받아서 적용해보자!

```shell
npm install @bonobolabs/gatsby-remark-images-custom-widths
```

```js
resolve: `gatsby-transformer-remark`,
  options: {
    plugins: [
      {
        resolve: '@bonobolabs/gatsby-remark-images-custom-widths',
        options: {
          maxWidth: 700,
          linkImagesToOriginal: false,
        },
      },
    // ... 기타 플러그인
    ]
  }
```

border와 raidus는 이미지 태그에 스타일을 추가로 덮어씌우는 걸 선택했다. remark-images 플러그인에 border와 radius를 options로 넣을 수 있으면 좋겠지만 할 수 없으니 태그에 스타일을 오버라이딩하는 방법을 선택했다. 처음부터 이미지 width값도 태그에 스타일을 지정하지 않은 이유는 이미지마다 내가 지정하고 싶은 width값이 전부 다르기 때문에 태그 자체에 고정된 width 값을 설정할 수 없었다.

```css
img {
  border: 1px solid #e0e0e0;
  border-radius: 0.6em;
}
```
`styles/post.scss` 파일에 아래와 같이 스타일을 적용해주었다.

현재 해당 블로그 스타일처럼 이미지에 스타일이 제대로 들어가는 것을 확인할 수 있다.

## References
- [@bonobolabs/gatsby-remark-images-custom-widths](https://www.gatsbyjs.com/plugins/@bonobolabs/gatsby-remark-images-custom-widths/)
- [@bonobolabs/gatsby-remark-images-custom-widths - npm package](https://snyk.io/advisor/npm-package/@bonobolabs/gatsby-remark-images-custom-widths?utm_medium=referral&utm_source=skypack&utm_campaign=snyk-widget)
- [Allow specifying image width via attribute in gatsby-remark-images · Issue #19439 · gatsbyjs/gatsby](https://github.com/gatsbyjs/gatsby/issues/19439)
- [Not working with gatsby · Issue #1 · rbeer/gatsby-remark-image-attributes](https://github.com/rbeer/gatsby-remark-image-attributes/issues/1)
- [Gatsbyjs: how to implement advanced image styling with gatsby-remark-images and markdown? - Stack Overflow](https://stackoverflow.com/questions/49857731/gatsbyjs-how-to-implement-advanced-image-styling-with-gatsby-remark-images-and
)



