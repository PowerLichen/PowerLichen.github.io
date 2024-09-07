---
title : 인스타그램 API와 저장 최적화
date : 2024-06-22 21:07:00 +0900
categories : [Backend, Django]
tags : [django]
---

# 개요
인스타그램 API를 사용해서 어떤 기능을 만들기로 했다.

인스타그램에서 작성한 게시물들을 받아와 저장해두고,  
추후에 서비스에서 게시글을 작성할 때 상단에 미리보기 처럼 나타낼 수 있는 형태이다.

다만 문제가 되는것은 인스타그램 미디어의 이미지, 동영상 부분이었다.

미래에 사용할 거라고 확신할 수 없는 데이터를 저장하는 것은 자원 낭비라고 생각했기에, 갱신 시스템으로 구현을 해 보았다.


# 인스타그램 게시물 저장 및 갱신 시스템
인스타그램 API를 통해 불러온 미디어는 인스타그램 CDN을 통해 제공되며, 각 미디어는 유효기간을 가진다.

링크의 param에 존재하는 `oe`라는 필드가 해당 값이다.

이번 구현에서는 인스타그램 API 미디어에서 데이터를 불러올 때 미디어의 유효일자를 항상 계산했다.

```python
for item in res["data"]:
    media_url = item.get('media_url', '')
    cdn_query = urlparse(media_url).query
    cdn_query = parse_qs(media_query)
    
    expired_time_epoch = int(
        media_query['oe'][0], 16)

    expired_time = (
        datetime.datetime
        .fromtimestamp(expired_time_epoch)
        .replace(tzinfo=datetime.timezone.utc)
    )
```

cdn의 쿼리 파라미터인 `oe`는 epoch time의 16진수 문자열이다.

이를 변환하는 기능을 통해, 만료시간을 항상 계산했다.


전체 데이터 중 만료시간이 가장 빠른 날짜를 저장한다.  
그리고 사용자가 인스타그램 목록을 불러올 때 만료시간이 지났다면 재갱신하도록 구현하였다.

이렇게 되면 우리 서비스의 자원을 소모하지 않으면서도,  
인스타그램 데이터의 목록을 항상 보여줄 수 있었다.

