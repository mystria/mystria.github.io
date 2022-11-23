---
layout: post
title:  "GitHub Pages 블로그에 Markdown 으로 다이어그램 넣기"
date:   2022-11-21 23:00:00 +0900
categories: Jekyll GitHub Markdown
comments: true
---

# GitHub Pages 블로그에 Markdown 으로 다이어그램 넣기
그림 없이 블로그를 운영한 지 수년째... 너무 불편해서 [PlantUML](https://plantuml.com/ko/) 같은 방법은 없는지 찾아보았다.

## Markdown 으로 다이어그램 그리는 방법

IDE 에서 PlantUML 플러그인을 통해 코드로 다이어그램을 그려본 적이 있을 것이다. 이와 비슷한 Markdown(이하 MD) 용 도구가 Mermaid 이다.  
- 공식 사이트의 세부 가이드: https://mermaid-js.github.io/mermaid/#/flowchart
- 여기에서 실습해보자. https://mermaid.live

이 Mermaid 의 JavaScript 도구를 사용하여 MD 에 다이어그램을 넣는 방법을 사용하자.  
- Mermaid 공식 문서: https://mermaid-js.github.io/mermaid/#/

## GitHub Pages 에 적용하기

GitHub Pages 의 MD 는 Jekyll 이라는 정적 페이지 생성도구를 통해 HTML 로 만들어진다. 설치 및 사용 방법은 검색하면 잘 나온다.
- Pages 의 Jekyll 설정 방법: https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll

[GitHub 공식 문서](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-diagrams)
를 읽어보면 MD 문서에 다이어그램을 적용할 수 있다고 되어 있다.

하지만 동작하지 않는다!  
우선 이유를 간단히 말하자면 GitHub Pages 는 GitHub 이 아니기 때문이다.

## 문제점

검색에서 발견한 한글 블로그 중 2021년 1월 19일에 작성된 글 "[github page에서 Mermaid를 사용하여 diagram 그리기](https://frhyme.github.io/mermaid/Embedding_mermaid_in_github_page/)"에서,  
"Pages 는 Mermaid 가 적용되지 않으므로 HTML 요소를 사용해서 처리해야 한다"고 하고 있다. 이는 mermaid-js 가 `.mermaid` 를 인식해 브라우저에서 렌더링 하는 방식이다.
~~~ html
<div class="mermaid"> 
  flowchart LR
    A --> B
</div>
~~~

하지만, 2022년 2월 14일 GitHub 공식 블로그에서 GitHub 에서 MD 에 자동으로 Mermaid 를 적용한다는 소식을 확인했다.  
"Include diagrams in your Markdown files ..."  
https://github.blog/2022-02-14-include-diagrams-markdown-files-mermaid/  
참조: [한글 번역](https://sangseophwang.tistory.com/108)  

그러나 확인 결과 GitHub Pages 는 보안 문제 등으로 대상이 아니라고 한다.  
https://github.com/community/community/discussions/13761  

> Collaborator said: GitHub Pages was not in scope of this announcement since it lives outside of github.com.
>

실제로 아직(2022년 11월 기준) https://pages.github.com/versions/ 에는 jekyll-mermaid 를 지원하지 않고 있다.

## 대안

역시 우리의 친구 StackOverflow 에서 Mermaid 를 GitHub Pages 에 적용할 꼼수를 찾을 수 있었다.  
- https://stackoverflow.com/a/53893998/8350542  

> 주의 : 이 방법은 훗날 GitHub Pages 에도 Mermaid 가 적용된다면 충돌이 있을 수 있음
>

MD 로 아래와 같이 작성하면,
~~~~ 
~~~ mermaid
flowchart LR
	A --> B
~~~
~~~~

GitHub Pages 의 Jekyll 이 아래와 같이 렌더링 한다.
~~~ html
<pre><code class="language-mermaid">flowchart LR
    A --&gt; B
</code></pre>
~~~
이 부분을 위에 언급했던 **'HTML 요소로 처리하는 방법'**으로 해결 할 수 있는 것이다!  

실제로 Mermaid 공식문서에서 가이드 하는 방식이다.  
- https://mermaid-js.github.io/mermaid/#/n00b-gettingStarted?id=_3-calling-the-javascript-api

그렇다면 이것을 어떻게 우리의 GitHub Pages 에 적용할 것인가?  

GitHub Pages 에서 렌더링하는 페이지의 구조는 각자 다른 모양이겠지만,  
대략 post.html 아래에 head.html | header.html | footer.html 의 문서들로 구성되어 있다.  

우선 CDN 에서 mermaid.min.js 파일을 받도록 해야 하는데, 아래 코드를 head.html 에 넣어두면 페이지를 띄우자마자 해당 js 번들을 받을 것이다.  
~~~ html
<script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>
~~~

그리고 본문 출력 후, 아래와 같이 initialize() 해줘야 하는데, 이 부분은 post.html 의 제일 하단에 넣어둬야 한다.

~~~ html
<script>
mermaid.initialize({startOnLoad:true});
window.mermaid.init(undefined, document.querySelectorAll('.language-mermaid'));
</script>
~~~

HTML 문서가 완성 된 후 `<code class="language-mermaid">`의 영역에 다이어그램을 렌더링 할 수 있도록 `.language-mermaid` 를 대상으로 지정해 주는 것이다.  

아래와 같이 config 를 전달해 줄 수도 있다.
~~~ javascript
var config = {
    startOnLoad:true,
    theme: 'forest',
    flowchart:{
            useMaxWidth:false,
            htmlLabels:true
        }
};
mermaid.initialize(config);
~~~

해당 구문이 다이어그램도 없는 문서에서도 매번 수행되는게 부담된다면 아래와 같이 if 문으로 두르는 것도 가능하다.
{% raw %}
~~~ text
{% if page.mermaid %}
...
{% endif %}
~~~
{% endraw %}

이제 GitHub Pages 에 mermaid 코드 블럭으로 다이어그램이 그릴 수 있게 되었다.  
참고로 다이어그램에 특수 문자를 쓰고 싶을 때는 따옴표(")를 사용하자.  

~~~ mermaid
flowchart LR
    id1("이제...") --> id2["잘 된다!"]
~~~

