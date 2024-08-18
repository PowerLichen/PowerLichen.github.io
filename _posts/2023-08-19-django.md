---
title : Django 템플릿에서의 inclusion_tag 사용
date : 2023-08-19 12:24:00 +0900
categories : [Backend, Django]
tags : [django]
---
{% raw %}

# 개요
Django 템플릿에서는 동적인 서브 템플릿을 다른 템플릿에 삽입할 수 있다.

이번에 해결해야 했던 문제는, 템플릿마다 서로 다른 폰트가 적용될 수 있도록 지정하는 것이었다.

상세 조건은 다음과 같았다.
- 서비스 전체에서 지원하는 폰트가 5-6가지가 있다.
- 각 템플릿에서는 font_name이라는 정보 데이터를 가진다.
- 위 font_name에 일치하는 폰트만 스크립트에 포함되어야 한다.

이럴 때 쓸 수 있는 것이 `Django`의 `inclusion_tag` 기능이다.

`inclusion_tag`는 템플릿에서 다른 템플릿을 렌더링하여 일부 데이터를 표시하는 기능이다.  
이 템플릿은 항상 표시되지만 메인 템플릿에서 보낸 데이터에 따라 달라지며,  
그 페이지의 데이터를 기반으로 맞는 정보만을 출력해줄 수 있다.

# Django 레퍼런스의 예시

레퍼런스 문서에서는 간단한 예시로 목록 출력을 든다.

템플릿마다 쿼리셋을 받아, 모델의 context를 출력하고 싶다고 가정해보자.

```
<ul>
  <li>First choice</li>
  <li>Second choice</li>
  <li>Third choice</li>
</ul>
```

이 부분은 `inclusion_tag`를 통해 다음과 같이 나타낼 수 있다.

```html
<!-- in html template code -->
{% show_results poll %}
```
```
@register.inclusion_tag("results.html")
def show_results(poll):
    choices = poll.choice_set.all()
    return {"choices": choices}
```

그리고 이를 구현해주는 함수가 바로 다음 함수이다. 이때, 반환값은 간단한 `dict` 자료형으로 반환한다.  
이때 등록을 위해 `@register` 어노테이션을 같이 포함하도록 한다.


# 전역 폰트 설정의 구현
위 정보를 바탕으로 템플릿 최상단에 폰트 추가 템플릿을 로드 해 보았다.

```python
# app/templatetags/custom_tags.py
...
register = template.Library()

@register.inclusion_tag("font.html")
def apply_font(font_name):
    font_urls = {
        'Pretendard': None,
        'Gothic': '',
        ...
    }

    data = {
        "font_name": None,
        "font_link": None
    }

    if font_name in font_urls:
        data["font_name"] = font_name
        data["font_urls"] = font_urls[font_name]
    
    return data
```

필요한 폰트만 적용하여 속도를 빠르게 하기 위해서, 필요한 웹폰트만 로딩하도록 작성하였다.  
이 때, 전역적으로 미리 로딩된 폰트가 있다면 `font_urls`에 `None` 값을 주면서 쓸데없는 코드 중복은 제거했다.

다음으로 메인 템플릿과 렌더링 할 템플릿을 보자
```html
<!-- 메인 템플릿 -->
{% load custom_tags %}
{% apply_font font_name %}

<!-- font.html -->
{% if font_name %}
<style>
    {% if font_link %}@import url({{font_link}});{% endif %}
    * {
        font_family: "{{font_name}}, sans-serif";
    }
</style>
{% endif %}
```

메인 템플릿에서는 태그 파일을 load 를 통해 불러온다.
그리고 내부에 선언했던 `inclusion_tag`인 `apply_font`를 선언하고, `font_name` 값을 넘겨주었다.

반환된 데이터의 `key`에 font_name이 있다면 style 태그가 추가된다.
그리고 `font_link`도 있다면 해당 링크로 웹폰트를 `import` 할 수 있도록 작성하였다.


# 굳이 argument를 넘겨줘야 할까?
이 태그를 사용할 모든 템플릿에서 쓰이는 인수 이름(여기서는 `font_name`)이 같다면,  
메인 템플릿이 전달받은 인자를 그대로 쓸 수 있는 방법도 있다.

`inclusion_tag`의 `takes_context` 값을 True 변경하는 방법이다.

이외에서 python 문법을 그대로 사용 가능하여, `args`, `kwargs`도 동일하게 사용 가능하니  
입맛에 맞춰 여러 형태의 템플릿을 만들 수 있다.

{% endraw %}
