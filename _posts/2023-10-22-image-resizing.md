---
title: '이미지 업로드 성능 최적화: 브라우저 내 이미지 리사이징'
date: 2023-10-22 12:21:00 +0800
categories: [Project]
tags: [성능 최적화]
---

![](https://velog.velcdn.com/images/saseungg/post/62f809cf-cd9e-418d-aa09-92a8d1c0b0eb/image.gif)

`<input type="file">`을 이용해서 이미지를 업로드하고 createObjectUrl을 이용해서 preview 화면을 보여주는 모습이다.

mutiple 속성을 추가해서 여러장의 이미지를 업로드할 수 있도록 구현했는데 화면에 렌더링하는 데 로딩이 걸리는 문제가 있었다. 또한 전부 로딩될 때까지 화면이 blocking 되고 있었다.

## 이미지 최적화?
이미지가 화면에 표시되기까지 시간이 걸리는 이유가 무엇일까? 원인은 업로드되는 이미지가 원본 이미지 사이즈 그대로 저장되서 화면에 표시되기 때문이다.

이때부터 난 이미지 최적화에 대해서 알아보기 시작했다.
이미지를 최적화하는 방법은 여러가지이다.

이미지 최적화에 대해서 찾아봤을 때 나오는 방법들은 아래와 같다.

1. 이미지 CDN : 파라미터를 이용해 이미지 사이즈를 조절하는 방법
	-> 이럴경우는 서버에서 api를 이용해 이미지를 받아올 경우 사용하는 방법인데 지금의 경우에는 클라이언트 측에서 이미지 사이즈를 조절해야하므로 패스!

2. lazy loading : 지정한 뷰포트에 도달했을 때 처리하는 방법
  -> 업로드 즉시 전부 표시해야하고 일단 나같은 경우에는 뷰포트가 없다. lazy loading은 주로 무한 스크롤 때 사용하면 좋다.
  
3. 포맷 변환 : 이미지는 PNG보단 JPEG가 더 용량이 가볍고 그 둘보단 WebP 형식이 용량이 가볍다.
  
4. 이미지 리사이징 : 업로드 하는 파일의 용량을 줄인다.

나는 여기서 이미지 리사이징 방법을 사용하기로 했다.


## 이미지 업로드 구현

먼저 간단한 예제를 활용해서 이미지 사이즈를 줄여보겠다.
input으로 이미지를 업로드해서 preview를 보여주는 로직이다.

fileReader로도 사용해보고 싶어서 createObjectUrl이 아닌 fileReader로 구현해봤다. 둘의 차이점도 있는데 나중에 정리해서 글을 올려볼까 한다. 

```jsx
import './App.css';
import { useState } from 'react';

const App = () => {
  const [image, setImages] = useState({
    file: '',
    previewUrl: '',
  });

  const handleImageChange = (e) => {
    const file = e.target.files[0];
    const reader = new FileReader();

    reader.readAsDataURL(file);
    reader.onload = () => {
      setImages({
        file: file,
        previewUrl: reader.result,
      });

      console.log('원본 파일 크기: ' + file.size / 1000 + 'KB');
    };
  };

  return (
    <div className="layout">
      <h1>이미지 업로드</h1>
      <input type="file" accept="image/*" onChange={handleImageChange} />
      {image.previewUrl && (
        <img src={image.previewUrl} alt="이미지 프리뷰" width={400} height={400} />
      )}
    </div>
  );
};

export default App;
```

![](https://velog.velcdn.com/images/saseungg/post/866022da-0702-46e7-bd51-84c58003a75f/image.png)

![](https://velog.velcdn.com/images/saseungg/post/3efb8c64-1ca4-46e0-8b5b-1415c14ed651/image.png)

콘솔을 확인해보면 원본 사이즈는 1569KB이고,
네트워크 창을 확인해서 이미지 파일을 더블 클릭해보면 원본 사이즈 그대로 다운이 되는 것을 볼 수 있다.

우리가 화면에서 보여줄 이미지 사이즈는 가로,세로 400px이다. 원본 사이즈를 그대로 다운 받을 필요는 없다.

지금부터 우리가 할 건 browser-image-compression을 이용해서 이미지 사이즈를 조절할 것이다.

## 이미지 리사이징
```jsx
import './App.css';
import { useState } from 'react';
import imageCompression from 'browser-image-compression';

const App = () => {
  const [image, setImage] = useState({
    file: '',
    previewUrl: '',
  });

  const handleImageChange = async (e) => {
    const originalFile = e.target.files[0];

    // 압축 설정
    const options = {
      maxWidthOrHeight: 800, // 최대 너비 또는 높이 (픽셀)
    };

    const compressedFile = await imageCompression(originalFile, options);

    console.log('원본 파일 크기: ' + originalFile.size / 1000 + 'KB');
    console.log('압축된 파일 크기: ' + compressedFile.size / 1000 + 'KB');

    const reader = new FileReader();
    reader.readAsDataURL(compressedFile); // 압축된 파일로 변경

    reader.onload = () => {
      setImage({
        file: compressedFile,
        previewUrl: reader.result, // 압축된 이미지로 변경
      });
    };
  };

  return (
    <div className="layout">
      <h1>이미지 업로드 및 압축</h1>
      <input type="file" accept="image/*" onChange={handleImageChange} />
      {image.previewUrl && (
        <img src={image.previewUrl} alt="이미지 프리뷰" width={400} height={400} />
      )}
    </div>
  );
};

export default App;

```
![](https://velog.velcdn.com/images/saseungg/post/6aa05182-53c1-4e4b-b0f8-520f1b3ecb18/image.png)
![](https://velog.velcdn.com/images/saseungg/post/09b21354-86f4-44c5-ac3e-978895b4a0c4/image.png)

이미지를 다운받는 시간이 32초에서 5초로 줄었으며, 이미지 사이즈도 확연히 줄어들었다. 

Preview에서는 이 차이가 크게 영향이 없어보일지라도 업로드하는 이미지들이 쌓여 저장된 이미지들을 api를 이용해서 불러올 때 리사이징하고 압축된 파일들이 렌더링 속도에 큰 영향을 미칠 것이다.

이렇게 이미지 리사이징를 통해 웹 애플리케이션의 성능을 향상시킬 수 있다.