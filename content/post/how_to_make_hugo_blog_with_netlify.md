+++
authors = [
    "Lena"
]
title = "hugo + netlify로 블로그 만들기"
date = 2020-09-14T02:07:08+09:00
description = "hugo + netlify로 블로그 만들기"
tags = [
    "hugo", "netlify" 
]
images = [
  "/images/hugo+netlify.png"
]
draft = false
+++

Hugo + Netlify로 블로그 만드는 방법을 소개합니다.
<!--more-->

[hugo-theme-chunky-poster 테마](https://github.com/puresyntax71/hugo-theme-chunky-poster)를 사용했습니다.


### <span style="color: #6666FF">🧐  Github.io를 사용하는 것보다 Netlify를 사용했을 때 편한 점은</span>
1. 일단 빌드 속도, 배포 속도가 빠릅니다.
2. 두 개의 레파지토리에 변경사항을 푸시하고 배포하는 번거로움이 없습니다.
3. 2번 과정의 수고로움을 덜기 위해 shell script를 작성하거나 github action을 사용하여 자동배포를 할 필요 없이 Netlify에서 빌드와 배포를 알아서 해줍니다. 
4. 배포 과정을 로그로 확인할 수 있으며 실패시 어디가 문제인지 명확히 위치를 알려줍니다.

<br>

### <span style="color: #6666FF">🧐  hugo를 선택한 이유는  </span>
1. jekyll + github.io로 블로그를 열었으나 빌드 및 배포 속도가 느리고 관리하기에 썩 편한 느낌을 받지 못했습니다.  
2. 그래서 이 보다 빠르고 관리가 편한 다른 방법을 찾다가 Hugo 와 Gatsby를 찾았습니다.  
3. 처음에는 Gatsby + Netlify로 블로그를 배포했지만 리액트를 다루거나 프론트에 대한 지식이 풍부한 상태가 아니여서 커스텀에 한계를 느꼈습니다.    
4. 그래서 hugo + github.io로 블로그를 새로 열었으나 여러번 실패했습니다.  처음에는 커스텀이 어려웠고 나중에는 배포가 되지 않아 404 페이지만 마주했죠...   
5. 그러다가 hugo + github.io로 배포까지 성공하였으나 블로그 컨텐츠가 들어있는 레파지토리와 github.io 레파지토리 두 번에 걸쳐 푸시하여 배포하는 점이 번거로웠습니다.  
6. 이 수고를 덜기 위해 shell script를 작성하여 자동배포를 했었지만 중간에 배포 실패시 원인을 찾는 것이 그리 쉽지는 않았습니다.  
7. 그러다가 이전에 Gatsby를 할 때 Netlify가 알아서 빌드/배포해주고 실패시 문제가 일어난 위치를 보기 쉽게 알려주어서 마지막에 결국 hugo + netlify 로 블로그를 열었습니다.  

<br>

### <span style="color: #6666FF">🧐  이런 분한테는 hugo + netlify 조합이 별로일 것입니다.  </span>
1. netlify로 배포하면 url endpoin가 netlify.app 또는 netlify.com이 됩니다. github.io 로 된 url을 갖고 싶으시다면 netlify를 추천하지 않습니다.  
2. hugo는 한국어로된 자료가 jekyll에 비해 많이 적습니다. 대신에 hugo 사이트에 정말 친절하게 나와있긴 합니다.  영어로 자료를 읽는데에 거부감이 있거나 어려움이 있으시다면 별로 일 수 있습니다.  

* 참고로 커스텀하는 과정에서 themes/hugo-theme-chunky-poster 하위에 있는 모든 파일들은 참고만 하고 수정하지는 않습니다. (제가 많이 삽질한 부분...)

<br>

### <span style="color: orange">🌱 준비</span>

1. hugo 설치
```
brew install hugo
```
2. go 설치
```
brew install go
```
3. [Netlify](https://app.netlify.com/) 가입  
4. [hugo theme](https://themes.gohugo.io/) 정하기 + 깃허브에서 HTTPS url이나 SSH key 복사해놓기

<br>

### <span style="color: orange">🌱 hugo 블로그 시작하기</span>


* 참고 자료를 먼저 적겠습니다. 아래 자료를 순서대로 본다면 더 이해하기 쉽습니다.<br><br><span style="color: red">⭐️⭐️[hugo quick start](https://gohugo.io/getting-started/quick-start/)에 자세한 설명이 나와있습니다.⭐️⭐️</span><br>  
<span style="color: red">⭐️⭐️[hugo + netlify로 블로그 만드는 방법 참고한 youtube](https://www.youtube.com/watch?v=gSiAcyTWU3c)를 보면 hugo와 netlify를 이용해서 배포하는 과정을 이해하기 쉽습니다. ⭐️⭐️</span><br><span style="color: red">⭐️⭐️[각 폴더의 역할 참고한 블로그](https://medium.com/@fgrgic/how-to-create-your-online-portfolio-the-fun-way-fb47ef0fb3c8)를 참고하면 블로그를 커스텀할 때 도움이 됩니다. ⭐️⭐️</span><br>

<br>


#### <span style="color: orange">👩🏻‍💻  새로운 사이트 만들기</span>
```
hugo new site <블로그 레파지토리 이름>
```

<br>

#### <span style="color: orange">👩🏻‍💻  테마 추가하기</span>

```
cd <블로그 레파지토리 이름>
git init
git submodule add git@github.com:puresyntax71/hugo-theme-chunky-poster.git 
또는 
git submodule add https://github.com/puresyntax71/hugo-theme-chunky-poster.git
```
> submodule?  
> [Git 도구 - 서브모듈](https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-%EC%84%9C%EB%B8%8C%EB%AA%A8%EB%93%88)  
> Git 저장소 안에 다른 Git 저장소를 디렉토리로 분리해 넣는 것이 서브모듈이다. 다른 독립된 Git 저장소를 Clone 해서 내 Git 저장소 안에 포함할 수 있으며 각 저장소의 커밋은 독립적으로 관리한다.
> 서브보듈을 추가하고 난 후 제대로 됐는지 확인하고 싶다면 [command + . + shift]를 눌러 숨겨진 파일 중 `.gitmodules` 파일이 있습니다.
> `.gitmodules` 파일은 서브디렉토리와 하위 프로젝트 URL의 매핑 정보를 담은 설정파일입니다. 이곳에서 추가한 아래와 같이 submodule을 확인할 수 있습니다.  

  ```ini
  [submodule "DbConnector"]
    path = DbConnector
    url = https://github.com/chaconinc/DbConnector
  ```

> 서브모듈 개수만큼 이 항목이 생긴다. 이 파일도 `.gitignore` 파일처럼 버전을 관리한다. 다른 파일처럼 Push 하고 Pull 한다. 이 프로젝트를 Clone 하는 사람은 `.gitmodules` 파일을 보고 어떤 서브모듈 프로젝트가 있는지 알 수 있다.
> 혹시 submodule을 추가하는 와중에 문제가 생겨서 submodule을 삭제하고 다시 진행하고 싶다면 [stackoverflow - git submodule add a git directory is founc locally issue](
https://stackoverflow.com/questions/20929336/git-submodule-add-a-git-directory-is-found-locally-issue)와 [iOS git submodule](https://zeddios.tistory.com/704)를 참고하여 진행하면 해결할 수 있습니다.
<br>

<br>

#### <span style="color: orange">👩🏻‍💻  커스텀하기</span>

1. **<블로그 레파지토리>에 저장한 내용**   
배포할 때 우선적으로 적용됩니다. html이나 화면을 위한 코드를 재정의하고 싶다면 `themes/hugo-theme-chunky-poster`에 있는 폴더 이름이나 파일이름을 똑같이 복사해서 가져온 후 안에 내용을 바꾸면 됩니다.  
2. **`themes/hugo-theme-chunky-poster`에 있는 파일들**   
hugo 테마에서 제공하는 기본적인 설정들이 저장되어있습니다.  <블로그 레파지토리>에서 재정의하지 않는다면 이 곳에 저장된 layouts이나 archetypes 이 적용됩니다.   
3. **`themes/hugo-theme-chunky-poster/exampleSite`에 있는 파일들**   
데모 사이트에서 봤던 내용들이 저장되어있습니다. md 파일 양식이나 다른 부분을 데모 사이트와 똑같이 하고 싶다면 이곳에 있는 파일들을 확인하면 됩니다.   

<br>

#### <span style="color: orange">👩🏻‍💻  각 디렉토리에서 할일  </span>
* `content` : 포스트에 들어가는 md 파일 및 이미지 등을 관리<br>
  <br>
    포스팅할 md 파일을 `contents > post` 에 등록합니다.  
  (⭐️  posts나 다른 폴더 이름으로 명명한다면 배포 후 테마가 적용되지 않을 수 있습니다.)

  이미지를 추가하고 싶다면 images 폴더를 추가한 후(`content/images`) 이곳에 이미지를 추가하면 됩니다.  <br>
  그리고 아래 내용을 가진 `content/images/index.md` 를 생성합니다.  

  ``` html
    --- 
  headless: true 
  ---
  ```
  <br>그리고 포스트 md 파일에 아래와 같이 images 라인을 추가합니다.

  ``` html
  ---
  title:
  "Project" date: 2020-03-04T09:20:30+01:00
  images: []
  categories: []
  tags: [] 
  authors: []
  images: ['/images/your-image.png']
  ---
  ```

* `archetypes`<br>
  `hugo new posts/제목.md` 로 생성한 md 파일이 `post` 폴더에 생성될 때 `archetypes`에서 정의한 기본 템플릿으로 생성됩니다. Front Matter라고 불리며 페이지의 메타 데이터를 정의하는 곳입니다.

  ``` html
  ---
   title: "{{ replace .Name "-" " " | title }}"
  date: {{ .Date }}
  draft: true
  ---
  ```

* `layouts`  
  페이지의 HTML 템플릿을 커스텀할 수 있는 파일이 저장되어있습니다.
  데모 사이트에서 봤던 템플릿을 참고 하고 싶다면 `themes/chunky-poster/layouts` 를 참고하면 됩니다.    <br>
  홈페이지 레이아웃: `themes/chunky-poster/layouts/index.html`  
  포스트 페이지 레이아웃: `themes/chunky-poster/layouts/post/single.html`  <br>
  <br>

  이 폴더에 있는 레이아웃이 `theme > layouts` 폴더에 있는 레이아웃보다 우선하지만 `themes > layouts`에 레이아웃은 기본적으로 사용됩니다.
  자세한 템플릿 관련 내용은 [hugo - templates](https://gohugo.io/templates/)를 참고.<br>

* `resources`  
휴고에 의해 생성된 리소스들이 저장되어있는 폴더입니다.  

* `static`  
static 파일을 저장하는 폴더 (사람들이 다운로드할 파일을 저장할 수 있는 파일 등)입니다.   
`images` 폴더를 만든 후 homepage-image.jpg 로 이미지를 저장하면 홈화면에 이미지를 설정할 수 있습니다.  

* `themes`  
모든 테마가 저장되어있습니다. `exampleSite` 폴더를 참고하여 블로그를 꾸미면 됩니다.  

* `Config`  
데모 사이트를 참고하여서 config를 작성하고 싶다면 `themes/chunky-poster/exampleSite` 에 있는 config.toml를 참고하면 됩니다.  

  특히 여기서
  ```
  [params]
     author = "Hugo Authors"
     description = "Lorem ipsum dolor sit amet."
     homepageImage = "/images/homepage-image.jpg"
  ```

  author에 작성자의 이름을, description에 설명을 쓰면 홈 화면에 적용되어 보여집니다.


#### <span style="color: orange">👩🏻‍💻  utterances 댓글 추가하기</span>

  hugo-theme-chunky-poster에서는 commento와 Disqus를 지원하고 있습니다. 하지만 저는 Github 이슈로 관리할 수 있고 마크다운을 지원하는 댓글 기능을 추가하고 싶어서 utterances로 선택했습니다.  
  utternaces를 사용하기 위해서는 `themes/hugo-theme-chunky-poster/layouts/_default/single.html`를 참고하여 `layouts` 폴더 하위에 `_default` 폴더를 생성한 후 `single.html`에 utterances html 코드를 추가합니다. 위치는 `content` 아래에 위치하도록 정해줬습니다.  

```html
{{ define "main" }}
<main class="content-page container pt-7 pb-5">
    <div class="row">
        <div class="col">
            <article>
                <div class="row justify-content-center">
                    <div class="col-lg-8">
                        <h2 class="mb-3">{{ .Title }}</h2>

                        <div class="content">
                            {{ .Content }}
                        </div>
                    </div>
                </div>
            </article>
            <script src="https://utteranc.es/client.js"
                    repo="dev-Lena/blog-comments"
                    issue-term="pathname"
                    label="✨💬✨"
                    theme="github-light"
                    crossorigin="anonymous"
                    async>
            </script>
        </div>
    </div>
</main>
{{ end }}
```

<br>

#### <span style="color: orange">👩🏻‍💻  임시저장</span>

`content/posts/` 에 저장한 md 파일에서
```
---
title: "My First Post"
date: 2020-09-14T02:07:08+09:00
draft: false
---
```
draft가 true로 설정되어있으면 임시저장의 개념으로 배포했을 때 해당 문서는 글로 올라가지 않습니다.
false로 설정해야 배포시 글이 보입니다.  

<br>


#### <span style="color: orange">👩🏻‍💻  Netlify로 배포하기</span>
[Netlify](https://app.netlify.com/start)에 가입하여서 진행하면 됩니다.
준비 사항은 github 레파지토리 (로컬에 만들어놓은 블로그 컨텐츠가 있는 레파지토리 - <블로그 레파지토리>)를 미리 만들어놓고 Netlify와 연결하면 됩니다. 
주의할 점: 2.Pick a repository 에서 모든 레파지토리를 선택하지 않고 특정 레파지토리와 연동되도록 설정해야합니다.  
사이트 생성이 완료된 직후에는 알파벳과 숫자가 섞인 임시 이름이 적혀있는데, 사이트 이름을 변경해서 사용할 수 있습니다. 마찬가지로 url도 설정할 수 있습니다.  

빌드 명령어: `hugo`  
배포 디렉토리: `public`  

* 배포 전 로컬 서버에서 확인하고 싶다면
  ```
  hugo server 
  또는 
  hugo server -D
  ```
  로 확인할 수 있습니다.


* 터미널에 변경 사항을 커밋한 후 
  ```
  git push origin matser
  ```
  로 변경사항을 원격 저장소로 푸시하면 Netlify 에서 자동으로 빌드와 배포를 시작합니다.

<br>

* Netlify 홈페이지에서 관리하기 <br><br>
배포 상태를 확인할 수 있고 Building 중인 디플로이를 선택하면 배포 과정 로그를 확인할 수 있습니다. <br> 
<img src="https://i.imgur.com/PWl6A92.png" style="width:50%"><br>
<img src="https://i.imgur.com/ldyI3CP.png" style="width:50%"><br><br>
로그를 통해 진행 과정을 확인할 수 있고 실패시 오류의 원인을 제시합니다.  

<img src="https://i.imgur.com/0E1QRkw.png" style="width:49%"><br>

#### <span style="color: orange">대표 이미지가 보이지 않을 때</span>

- [hugo-theme-chunky-poster](https://themes.gohugo.io/hugo-theme-chunky-poster/) 참고
  1. `content/images/index.md` 추가
```html
---
headless: true
---
```
  2. 포스트에 이미지 프로퍼티 
```html
---
images: ["/images/image.png"]
---
```
<br>

#### <span style="color: orange">내가 보려고 기록해놓는 명령어</span>>

* 새 글 추가
```terminal
hugo new post/fileName.md
```
* local:1313에서 보기
```terminal
hugo server
```
* 도움!
```terminal
hugo help
```