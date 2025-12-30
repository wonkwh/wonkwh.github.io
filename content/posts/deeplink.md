+++
title = "Deep Links And universal Links On iOS"
date = 2020-03-22

[taxonomies]
tags = ["iOS"]
+++

Deep Links, Universal Links, Custom URI scheme등의 개념및 용어  branch.io 등 서드파티 서비스등을 정리 

<!-- more -->

# 1. Deep Link 란?
 - content의 주소를 포함하고있는 link
 - 대부분의 web link는 deep link
예)
 - https://www.vingle.net/posts/2816649
    - 요건 deepLink , 특정 세부 컨텐츠를 폼함
 - https://vingle.net/
    - 요건 deep link가 아님 걍 main homapage addredd

# 2. DeepLink와 mobile App

## 2.1 문제점
 - web link는 native mobile app에 적용되지 않음.
 - 모바일 폰으로 https://vingle.net/ 요링크를 클릭하면 native vingle app이 설치되어 있든 설치되어 있지 않든 웹 브라우져 연결 
 - 앱 개발사 입장에서는 모바일에서는 웹보다 앱을 사용하게 하고싶음. 

## 2.2 The Deep link solution
 - 위의 웹링크에 문제점을 해결한 것이 Mobile App deep links (간단히  “deep links”)
 - **우리가 흔히 deep link라 하는것이 이것**
 - 위 예시로 (https://www.vingle.net/posts/2816649)  게시물은 친구에게 보내주면 친구의 폰 에 빙글앱이 깔려있으면 앱이 열리면서 앱 내에서 해당게시물로 이동. 

# 3. 예전 deep links
## ios 9 이전버전
 - https:  로 시작하는 web address와 다르게 특정앱마다 Custom URI schemes 존재  
    - 예) twitter:// 
 - 앱이 설치되어 있으면 앱이 열림 
   - ![gif1](https://branch.io/img/what-is-deep-linking/traditional-1.gif)
 - 앱이 설치되어 있지 않으면 웹브라우져에서도 열지를 못하고 fail 
   - ![gif2](https://branch.io/img/what-is-deep-linking/traditional-2.gif)

# 4. Deferred deep links 
 - 위 문제점을 해결하기 위해 사용
 - 사용자가 앱이 설치되어 있지않을 때  
 - deep link를 open 시  App Store 혹은 Play Store 의 해당 앱 으로 이동
    - 사용자가 처음 설치후 해당 링크의 “deferred *(지연된)”  컨텐츠로 바로 이동

## 4.1 Contextual deep links
 - 위 지연된 deep links의 기능은 모두 가진다
 - 그외 추가적인 정보를 가지고 있다. 
   - 사용자가 무슨 사이트에서 click했는지 
   - 누가 공유한 링크를 타고 open했는지 
   - 그외 등등
 - 이런 추가적인 정보를 가지고 있으면 app 개발자가 이 정보를 가지고 추가적인 기능을 제공할 수 있다. 
    - 예) 이벤트 링크를 타고온 사용자에게 쿠폰을 제공한다. 

# 5. Custom URI Schemes
 - web 이 아닌 app을 위한 “*Private internet*”
 - myapp://path/to/content 요런 형태
## 5.1 장점
 - 쉽게 설정 가능하다.
 - 대부분의 앱들이 최소 하나는 가지고 있다.
## 5.2 단점
 - 앱이 설치되어야지만 이 “private internet” 에 대응할 수 있다. 
## 5.3 Solution
 - http:// 로 시작하는 기존web link를 포함하는 URI schemes 을 이용한다. 
 - javascript code를 포함한 링크로 웹브라우져를 한번 거쳐 앱이 설치되어 있으면 그앱으로 , 앱이 설치되어 있지 않으면 앱스토어, 플레이스토어 URL로 redirect 시킨다. 
 - 안드로이드에서는 이 방식이 아직 잘 작동한다. 
 - iOS에서는 이런 방식의 deep link를 2015년에 막아버렸다.
    - 그리고 대안으로  `Universal Links` 를 소개

# 6. iOS Universal links
 - Custom URI 를 쓰지않고 일반적인 web link와 같다 
 - domain이 등록되어 있는 앱이 설치되어 있는지 OS에서 판단하여 앱이 설치되어 있으면 웹 브라우져를 열지 않고 바로 앱이 열린다. 
 - domain에 해당되는 앱이 없다면 사파리(웹브라우져)에서 해당 address 를 로딩
 - AppStore로 redirect도 OS에서 판단해 웹에서의 추가 잡업 없이 safari app에서 배너 형태로 보여준다.
    - Ex) Airbnb app
     
    ![airbnb](https://i.stack.imgur.com/SbL0y.png)
    - Custom Banner를 보여주려면 간단한 스크립트를 웹 프론트엔드에서 추가해주어야 한다.


 - 맨위에 deepLink 예에서 
    - https://www.vingle.net/posts/2816649 클릭시 웹브라우져 거치지 않고 앱이 설치되어 있으면 바로 앱이 열린고 해당 게시물로 바로 이동한다. 
    - 빙글앱이 ios univaral link 를 이미 적용했기 때문..
## 단점 
 - ios device에서만 사용가능하다.

# 7. Android App Links
 - 위 ios Universal links와 같은 기능을 하는것이 android app links
 - 그런데 안드로이드는 위 Custom URI scheme을 계속 쓸수 있기 때문에 두가지중에 선택가능

# 8. Branch.io solution
 - ios univertail link, android app link 모두 아직 완전하지 않다. 
 - 앱링크는 안드로이드 플랫폼 만든 앱에서만 동작하고, 구글 이외에 앱에서는 정상적으로 동작하지 않습니다. 유니버셜 링크 역시 애플 디바이스에서만  정상적으로 동작하기 때문에 디바이스간 공유가 되지 않는다. 
 - 위처럼 os, platform 별로, 그리고 버전별로 deep link구현방식이 다르게 때문에 이를 통합하여 Deferred deep link, Contextual deep links 를 쉽게 구현하기 위한 3rd party solution 
 - 빙글앱에서 사용



## 참조
- https://branch.io/what-is-deep-linking/
- https://www.adjust.com/glossary/deep-linking/
- [StackOverflow universal-links-on-ios](https://stackoverflow.com/questions/35522618/universal-links-on-ios-vs-deep-links-url-schemes)
