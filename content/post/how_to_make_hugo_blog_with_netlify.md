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
    "iOS_developing_mac.png"
]
draft = true
+++

Hugo + Netlify로 블로그 만드는 방법을 소개합니다.
<!--more-->
# Lena-Chamna
Hugo + Netlify
[hugo-theme-chunky-poster 테마](https://github.com/puresyntax71/hugo-theme-chunky-poster)를 사용했습니다.

* 참고로 커스텀하는 과정에서 themes/hugo-theme-chunky-poster 하위에 있는 모든 파일들은 참고만 하고 수정하지는 않습니다. (제가 많이 삽질한 부분이라...)

## Ready

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

## hugo 블로그 시작하기
[hugo quick start](https://gohugo.io/getting-started/quick-start/)에 자세한 설명이 나와있습니다
### 새로운 사이트 만들기
```
hugo new site <블로그 레파지토리 이름>
```
### 테마 추가하기
```
cd <블로그 레파지토리 이름>
git init
git submodule add git@github.com:puresyntax71/hugo-theme-chunky-poster.git 
또는 
git submodule add https://github.com/puresyntax71/hugo-theme-chunky-poster.git
```




## 커스텀하기
### 각 디렉토리에서 할일
#### content  
  * (포스트에 들어가는 md 파일 및 이미지 등을 관리)  
포스팅할 md 파일을 `contents > post` 에 등록합니다.  
(⭐️ posts나 다른 폴더 이름으로 명명한다면 배포 후 테마가 적용되지 않을 수 있다.)
이미지를 추가하고 싶다면 images 폴더를 추가한 후(`content/images`) 이곳에 이미지를 추가하면 됩니다.  

그리고 아래 내용을 가진 `content/images/index.md` 를 생성합니다.  
``` html
--- 
headless: true 
---
```
그리고 포스트 md 파일에 아래와 같이 images 라인을 추가한다.

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
그러면 헤더 이미지를 볼 수 있다.

## archetypes
`hugo new posts/제목.md` 로 생성한 md 파일이 posts 폴더에 생성될 때 archetypes에서 정의한 기본 템플릿으로 생성된다.
Front Matter라고 불리며 페이지의 메타 데이터를 정의하는 곳이다.
``` html
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
---
```
## layouts
페이지의 HTML 템플릿을 커스텀할 수 있는 파일이 저장되어있다.
데모 사이트에서 봤던 템플릿을 참고 하고 싶다면 themes/chunky-poster/layouts 를 참고하면 된다.
홈페이지 레이아웃: `themes/chunky-poster/layouts/index.html`
포스트 페이지 레이아웃: `themes/chunky-poster/layouts/post/single.html`

이 폴더에 있는 레이아웃이 theme > layouts 폴더에 있는 레이아웃보다 우선하지만 theme > layouts에 레이아웃은 기본적으로 사용된다.
자세한 템플릿 관련 내용은 [hugo - templates](https://gohugo.io/templates/)를 참고

## resources
휴고에 의해 생성된 리소스들이 저장되어있는 폴더

## static
static 파일을 저장하는 폴더 (사람들이 다운로드할 파일을 저장할 수 있는 파일 등)
images 폴더를 만든 후 homepage-image.jpg 로 이미지를 저장하면 홈화면에 이미지를 설정할 수 있다.

## themes
모든 테마가 저장되어있다. example site 폴더를 참고하여 블로그를 꾸미면 된다.

🏗 Config
데모 사이트를 참고하여서 config를 작성하고 싶다면 `themes/chunky-poster/exampleSite` 에 있는 config.toml를 참고하면 된다.

특히 여기서
```
[params]
   author = "Hugo Authors"
   description = "Lorem ipsum dolor sit amet."
   homepageImage = "/images/homepage-image.jpg"
```

author에 작성자의 이름을, description에 설명을 쓰면 홈 화면에 적용되어 보여진다.

## utterances 댓글 설정
hugo-theme-chunky-poster에서는 commento와 Disqus를 지원하고 있다.
utternaces를 사용하기 위해서는 `themes/hugo-theme-chunky-poster/layouts/_default/single.html`를 참고하여  
layouts 폴더 하위에 `_default` 폴더를 생성한 후 `single.html`에 utterances html 코드를 추가한다. 위치는 content 아래에 위치하도록 정해줬다.
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

## 임시저장
content/posts/ 에 저장한 md 파일에서
```
---
title: "My First Post"
date: 2020-09-14T02:07:08+09:00
draft: false
---
```
draft가 true로 설정되어있으면 임시저장의 개념으로 배포했을 때 해당 문서는 글로 올라가지 않는다.
false로 설정해야 배포시 포스팅을 할 수 있다.


## Netlify로 배포하기
[Netlify](https://app.netlify.com/)에 가입하여서 진행하면 된다.
준비 사항은 github 레파지토리 (로컬에 만들어놓은 블로그 컨텐츠가 있는 레파지토리)를 미리 만들어놓고 Netlify와 연결하면 된다.

빌드 명령어: `hugo`
배포 디렉토리: `public`

배포 전 로컬 서버에서 확인하고 싶다면
```
hugo server 
또는 
hugo server -D
```
로 확인할 수 있다.


터미널에 변경 사항을 커밋한 후 
```
git push origin matser
```
로 변경사항을 원격 저장소로 푸시하면 Netlify 에서 자동으로 빌드와 배포를 시작한다.

Netlify 홈페이지에서 <br>
<img src="https://i.imgur.com/PWl6A92.png" style="width:50%"><br>
<img src="https://i.imgur.com/ldyI3CP.png" style="width:50%"><br>

배포 상태를 확인할 수 있고 Building 중인 디플로이를 선택하면 배포 과정 로그를 확인할 수 있다.
<img src="https://i.imgur.com/XgmflFT.png" style="width:50%"><br>






