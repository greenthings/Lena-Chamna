# Lena-Chamna
Hugo + Netlify
[chunky-poster 테마](https://github.com/puresyntax71/hugo-theme-chunky-poster)를 사용하였습니다

## content
포스팅할 md 파일을 `contents > posts` 에 등록한다.
이미지를 추가하고 싶다면 `content/images` 에 이미지를 추가하면 된다.
이미지를 사용할 md 파일을 열고 images 경로를 추가한다.
그리고 아래 내용을 가진 `content/images/index.md` 를 생성 한다
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
그르면 헤더 이미지를 볼 수 있다.

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

```
baseURL = "https://example.com"
title = "Hugo Themes"
copyright = "Copyright © 2008–2019, Steve Francia and the Hugo Authors; all rights reserved."
paginate = 2
languageCode = "en"
DefaultContentLanguage = "en"
enableInlineShortcodes = true
footnoteReturnLinkContents = "^"
googleAnalytics = "UA-XXXX"
DisqusShortname = ""
theme = "hugo-theme-chunky-poster"

[menu]
  [[menu.main]]
    identifier = "home"
    name = "Home"
    url = "/"
    weight = 10
  [[menu.main]]
    identifier = "about"
    name = "About"
    url = "/about/"
    weight = 0

[taxonomies]
category = "categories"
tag = "tags"
series = "series"
author = "authors"

[params]
  author = "Hugo Authors"
  description = "Lorem ipsum dolor sit amet."
  homepageImage = "/images/homepage-image.jpg"
  share = true
  showLanguageSwitcher = false

  # Custom CSS and JS. Relative to /static/css and /static/js respectively.

  customCSS = []
  customJS = []

  [params.social]
    rss = true
    email = "example@example.com"
    facebook = "https://facebook.com"
    twitter = "https://twitter.com"
    linkedin = "https://linkedin.com"
    stack-overflow = "https://stackoverflow.com"
    instagram = "https://stackoverflow.com"
    github = "https://github.com"
    weibo = "https://www.weibo.com"
    medium = "https://medium.com"
    pinterest = "https://pinterest.com"
    reddit = "https://reddit.com"
    gitlab = "https://gitlab.com"
    mastodon = "https://mastodon.social"
    keybase = "https://keybase.io/"

  [params.prismJS]
    enable = true
    theme = ""

  [params.commento]
    enable = true
    url = "https://commento.io"

[markup]
  [markup.highlight]
    codeFences = false

[services]
  [services.instagram]
    disableInlineCSS = true

  [services.twitter]
    disableInlineCSS = true
```


특히 여기서
```
[params]
   author = "Hugo Authors"
   description = "Lorem ipsum dolor sit amet."
   homepageImage = "/images/homepage-image.jpg"
```

author에 작성자의 이름을, description에 설명을 쓰면 홈 화면에 적용되어 보여진다.

## Netlify로 배포하기
[Netlify](https://app.netlify.com/)에 가입하여서 진행하면 된다.
준비 사항은 github 레파지토리 (로컬에 만들어놓은 블로그 컨텐츠가 있는 레파지토리)를 미리 만들어놓고 Netlify와 연결하면 된다.

빌드 명령어: `hugo`
배포 디렉토리: `public`
