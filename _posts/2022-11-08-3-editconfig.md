---
layout: post
title:  ".editorconfig 예시"
date:   2022-11-08 23:00:05
categories: ETC
tags: edit
---

<i class="fa-regular fa-circle-check" style="margin-right:0.7rem"></i>*포맷팅을 통일해주는 .editorconfig 설정 예시*

---

팀과 같이 개발할 때 각자의 포매팅 규칙이 다르면 매번 auto formatting 과정이 틀어지게 된다.    
프로젝트 초기부터 각종 컨벤션같은 스타일을 설정할때 팀과 상의해서 옵션을 정해놓고 시작하는 편이 좋다.   

이럴땐 **.editorconfig** 를 프로젝트 루트에 넣어놓은 뒤, IDE에서 지원하는 여러가지 형태로 포매팅 될 수 있도록 하면, 어떤 환경에서도 서로 다른 IDE라도 동일한 코드 포매팅을 유지시킬 수 있다.

---



**.editorconfig**
```yml
root = true
 
[*]
charset = utf-8
end_of_line = lf
indent_style = space
insert_final_newline = true
tab_width = 4
trim_trailing_whitespace = true
 
[*.{css,less,scss}]
indent_size = 2
 
[*.go]
indent_style = tab
 
[*.{groovy,gradle}]
indent_size = 4
 
[*.html]
indent_size = 2
 
[*.java]
indent_size = 4
 
[*.{js,json}]
indent_size = 2
 
[*.{kt,kts}]
indent_size = 4
 
[*.md]
trim_trailing_whitespace = false
 
[*.py]
indent_size = 4
 
[*.rb]
indent_size = 2
 
[*.{scala,sc}]
indent_size = 2
 
[*.xml]
indent_size = 2
 
[*.{yaml,yml}]
indent_size = 2
```

---

*각자 사용하는 IDE에서 editorconfig 플러그인을 설치하면 정상 적용된다*

<a href="/assets/images/2.png" data-lightbox="falcon9-large" data-title="Visual Studio Code 플러그인 예시">
  <img src="/assets/images/2.png" title="Visual Studio Code 플러그인 예시">
</a>

~~쥐를 찾으면 된다~~