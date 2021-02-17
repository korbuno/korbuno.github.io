---
layout: post
title: background-size
date: 2021-02-17 20:10:24 +0900
category: CSS
---
# 들어가기 전에
이 글은 [출처](https://css-tricks.com/almanac/properties/b/background-size/) 에서 읽은 글을 손번역하며 주관적인 생각도 덧붙여서 쓴 글이다.

## background-size란

CSS에서 가장 유용하지만 복잡한 background 속성 중 하나이다. 다양한 변수와 문법이 존재하며 사용방법도 상이하다.

### 간단한 예제

```css
html {
	background: url(greatimage.jpg);
	background-size: 300px 100px;
}
```

### Keywords

기본 속성으로는 **auto**가 적용되며, **cover**와 **contain**을 선택할 수 있다.

- cover   
	browser에게 이미지를 항상 영역에 맞춰서 포함시키도록 한다.   
	이미지가 해당 영역에 맞춰서 늘어나거나 줄어들 수도 있다.
	범위만 맞추는 용도이기 때문이다.

- contain
	항상 전체 이미지를 보여준다. 작은 여백이 남더라도 이미지를 전부 포함하도록 한다.

- auto
	default 설정으로, browser가 자동으로 실질적인 사이즈 비율을 조절한다.

### 4가지 표현방식

#### One Value

```css
html {
	background-size: 400px;
}
```

width를 400px로 설정하며, height는 auto로 지정된다.

### Two Values

```css
html {
	background-size: 400px 600px;
}
```

첫번째가 width, 두번째가 height을 표현하는 흔히 알고있는 방법이다.

### Multiple Images

여러 이미지를 합성하여 지정할 수도 있다.

```css
html {
	background: url(buno.jpg), url(kor.jpg);
	background-size: 300px 100px, cover;
}
```

여러 이미지를 선언하면, 선언한대로 순서가 적용된다.

<p>
	<div style="width: 100%; height: 1024px;">
		<iframe allowfullscreen="true" allowpaymentrequest="true" allowtransparency="true" class="cp_embed_iframe " frameborder="0" height="268" width="100%" name="cp_embed_1" scrolling="no" src="https://codepen.io/css-tricks/embed/NPMgem?height=268&amp;theme-id=1&amp;slug-hash=NPMgem&amp;default-tab=result&amp;user=css-tricks&amp;name=cp_embed_1" style="width: 100%; overflow: hidden; display: block; height: 100%;" title="CodePen Embed" loading="lazy" id="cp_embed_NPMgem"></iframe>
	</div>
</p>