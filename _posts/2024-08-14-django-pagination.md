---
title : DRF 페이지네이션 구조 정리
date : 2024-08-18 22:04:00 +0900
categories : [Backend, Django]
tags : [django]
---


# 개요
DRF에서 ViewSet을 사용하지 않지만, pagination 구조를 사용하기 위해 정리를 진행 해 보았다.


# ViewSet에서의 기본 사용 예시
DRF에서 페이지네이션은 다음과 같이 사용된다.

```python
def list(self, request, *args, **kwargs):
    queryset = self.filter_queryset(self.get_queryset())

    page = self.paginate_queryset(queryset)
    if page is not None:
        serializer = self.get_serializer(page, many=True)
        return self.get_paginated_response(serializer.data)

    serializer = self.get_serializer(queryset, many=True)
    return Response(serializer.data)
```

첫번째로 `paginate_queryset`이 쿼리셋에 페이지네이션을 적용한다.

두번째로 결과 반환 시 `get_paginated_response`를 사용한다.

## paginate_queryset
paginate_queryset의 구조는 간단하다.
```
self.paginator.paginate_queryset(queryset, self.request, view=self)
```
현재 view에 선언된 `paginator`의 메소드를 호출하는 것이다.

```python
# PageNumberPagination의 경우
paginator = self.django_paginator_class(queryset, page_size)
page_number = self.get_page_number(request, paginator)

try:
    self.page = paginator.page(page_number)
...

return list(self.page)
```
```python
def page(self, number):
    """Return a Page object for the given 1-based page number."""
    number = self.validate_number(number)
    bottom = (number - 1) * self.per_page
    top = bottom + self.per_page
    if top + self.orphans >= self.count:
        top = self.count
    return self._get_page(self.object_list[bottom:top], number, self)
```

view의 `paginator`에 데이터들을 초기화 시키고,  
실질적으로 queryset에 limit, offset이 지정되는 부분이다.

## get_paginated_response

get_paginated_response의 구조도 간단하다.

```python
self.paginator.get_paginated_response(data)
```

현재 view에 선언된 `paginator`의 메소드를 호출하는 것이다.

```python
# PageNumberPagination의 경우
def get_paginated_response(self, data):
    return Response(OrderedDict([
        ('count', self.page.paginator.count),
        ('next', self.get_next_link()),
        ('previous', self.get_previous_link()),
        ('results', data)
    ]))
```