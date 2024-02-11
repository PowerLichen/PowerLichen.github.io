---
title : Django의 쿼리 최적화 - 집합의 활용
date : 2024-02-11 20:45:00 +0900
categories : [Backend, Django]
tags : [django]
---

# 개요
몇달 간 새로운 기능을 작업하고, 기존에 구현되어 있던 코드의 최적화에 시간을 쏟았다.

주어진 여러 문제 중에서, 재미있는 로직을 최적화 한 과정이 있어 소개해보려고 한다.

# 집합 속성을 통한 contains 처리 문제
프론트엔드에서 하나의 태그를 여러 게시물을 추가하는 작업을 하게 되었다.

프론트 엔드에서 주는 데이터는 다음과 같았다.

```
{
    ...
    "posts": [22345, 42345, 23688, 12333, ...]
    ...
}
```
이전에 태그가 달려있던 `post`에서는 태그를 삭제하고,  
받은 `posts`에 해당하는 `post`에만 태그를 추가해야 한다.

## 기존 코드
post 테이블의 tags 필드는 `|` 구분자로 구분되어 있었기 때문에, 다음과 같이 구현되어 있었다.

```python
tag_regex = f"|{tag}|"

post_no_list = data["posts"]

...
post_list_add = post.objects.filter(id__in=post_no_list).exclude(tags__contains=tag_regex)
...
post_list_delete = post.objects.filter(tags__contains=tag_regex).exclude(id__in=post_no_list)
...
```

위 코드에서는 LIKE가 쓰이는 쿼리(Django ORM의 contains)가 두개 발생하여, 느려지는 문제가 있었다.

## 개선 코드
집합 구조의 차집합을 생각하면, 한번의 Like 쿼리로 손쉽게 변경할 데이터 목록을 가져올 수 있다.

그래서 다음과 같이 코드를 개선하였다.

```python
tag_regex = f"|{tag}|"

new_post_list = set(data["posts"])
old_post_list = set(
    post.objects
    .filter(tags__contains=tag_regex)
    .values_list("id", flat=True)
)
post_list_add = post.objects.filter(
    id__in=(new_post_list - old_post_list)
)
post_list_delete = post.objects.filter(
    id__in=(old_post_list - new_post_list)
)
```





