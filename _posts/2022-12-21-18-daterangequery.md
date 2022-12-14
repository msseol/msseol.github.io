---
layout: post
title:  "두 날짜 사이의 겹치는 기간 쿼리로 쉽게 구현하기"
date:   2022-12-21 00:01:01
categories: DATABASE
tags: query mysql database db mariadb mssql
cover: sql.png
---

보통 <span class="text-danger">이벤트</span>나 <span class="text-danger">예약 시스템</span> 등을 구현할 때 중복한 기간에 포함되지 않게 구현해야 하는 경우가 종종 있었다. 예를 들어 **10월1일 ~ 10월10일**까지 등록되어 있는 이벤트가 있다면,
**1일~10일** 사이를 포함하는 기간은 이벤트 **시작/종료로 포함될 수 없게** 하는 그런 기능을 말한다.   
   
예전에 구현할 때는 별다른 생각 없이 하나하나 조건을 다 만들어서 생성했었는데, **한 줄**로 해당 쿼리를 해결할 수 있는 것을 알게 되어서 메모를 저장했었다.

---

**기존에 사용한 조건들**

1. start ~ end 안에 :start, :end가 포함
2. start ~ end 안에 :start이 포함
3. start ~ end 안에 :end가 포함
4. start ~ end 앞에 :start, :end가 존재
5. start ~ end 뒤에 :start, :end가 존재

하나 하나 쿼리로 구현하면 아주 귀찮고 실수로라도 하나 빼먹으면 부득이하게 기능 상의 구멍이 생기기도 한다..
   
이런 귀찮음은 아래의 쿼리로 **한번에 해결이 가능하다.(!!)**

```sql
SELECT *
FROM `table` 
WHERE `start` < :end AND `end` > :start  # 동일한 날짜도 포함시키려면 부등호만 바꾸면 된다.
```

쿼리가 의심되긴 하지만 실제로 돌려보면 아주 간단하게 테이블 내의 기간이 포함되는 항목들을 찾아낼 수 있는 것을 볼 수 있다.    
**~~뻘짓 이제 끝...~~**
