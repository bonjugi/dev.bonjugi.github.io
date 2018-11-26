---
layout: post
title:  "Intelij resource 파일 수정시 자동 배포"
date:   2018-11-26
excerpt: "Intelij는 기본적으로 파일 수정시 자동 배포가 이루어지지 않습니다. 설정을 바꿔서 자동배포가 이뤄지도록 합니다."
tag:
- intelij
- hot deploy
comments: true
---

인텔리제이에 톰캣을 연동하여 사용하다보면 .jsp 는 물론 html css js 같은 정적 리소스들을 Run 중에 수정시, 수정내용이 자동 적용되지 않습니다.
따라서 위와 같은 정적페이지 및 jsp 내의 서블릿코드들을 자동으로 배포할수 있도록 설정하는 방법을 설명 하겠습니다.

### Run/debug Configuration 에서의 변경.

Edit Configuration 에 들어가서 `On frame deactivation` 의 설정을 `update classes and resources` 로 변경합니다.
>  On frame deactivation - 프레임 비활성화 시 라는 의미인데 정확히 어떤 시점인지 모르겠습니다.

- ![Image Alt 텍스트](/assets/img/upload/update-classes-and-resources1.png)




### Run window 에서의 변경
`Update resources on frame deactivation` 버튼을 눌러 기능 활성화 합니다.
- ![Image Alt 텍스트](/assets/img/upload/update-classes-and-resources2.png)


참고로 이 방법들 만으로는 maven 프로젝트의 java파일 까지 핫디플로이를 해주진 않는다고 합니다.
다음 포스트에서 java 파일도 핫디플로이 가 되게끔 설정을 변경 하도록 하겠습니다.

