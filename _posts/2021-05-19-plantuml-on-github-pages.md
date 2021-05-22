---
layout: post
title: GitHub Pages에서 PlantUML 렌더링해서 게시물에 삽입하기
tags: [plantuml, github pages]
author: ilwoong
---

PlantUml은 마크다운 기반의 UML 편집기입니다. 블로그 글 작성시에는 마인드 맵을 사용할 일이 종종 생기는데, 이미지로 저장해서 올리는 경우 수정이 필요할 때마다 이미지를 다시 올려야 하는 번거로움이 있습니다. 코드 자체를 렌더링 하는 방법을 찾아서 다음과 같이 기록으로 남깁니다.

### PlantUML 문법에 맞추어 소스 파일 작성하기

```
@startmindmap

* Root Node
** First Level Node
** First Level Node
*** Second Level Node
**** Third Level Node

@endmindmap
```

저 같은 경우는 블로그를 위한 repo 밑에 plantuml 폴더를 하나 만들고 그 안에 example.pu 라는 이름으로 위의 내용을 저장하였습니다.

### PlantUML 렌더 엔진을 이용하여 렌더링하기

소스파일을 PlantUML proxy에 입력으로 넣으면 결과를 렌더링해서 리턴해줍니다. 마크다운 문서의 원하는 위치에 아래처럼 링크를 입력하면 렌더링한 결과를 바로 넣을 수 있습니다.

```markdown
![diagram-name](http://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/ilwoong/ilwoong.github.io/master/plantuml/example.pu)
```

![diagram-name](http://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/ilwoong/ilwoong.github.io/master/plantuml/example.pu)


### 참고자료
- <https://github.com/jonashackt/plantuml-markdown>{:target="_blank"}