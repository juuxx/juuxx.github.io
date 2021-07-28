---
title: JPA nativeQuery 작성 방법
date: 2021-07-28 11:58:47 +07:00
modified:
tags: [jpa, java, nativeQuery]
description:
image:
---

> 상황

1.  pagination 구현해야함
2.  entity가 아닌 DTO 로 받아야함
3.  union all 사용

**→ `@Query` 사용해서 구현**

## 페이지네이션 사용

```java
    @Query(
      value = "SELECT * FROM Users ORDER BY id",
      countQuery = "SELECT count(*) FROM Users",
      nativeQuery = true)
    Page<User> findAllUsersWithPagination(Pageable pageable);
```

parameter에 pageable을 넣으면 알아서 offset, limit 설정해줌

※ 2.0.4 이전의 SpringData JPA 버전 → `\n-- #pageable\n` ← 이거 추가

단, 이후 버전인데 이걸 추가하면 실행이 안됨.. 주석이라 상관 없을 줄 알고 추가했는데.. paging 처리가 안됐었음 ..

```java
    @Query(
      value = "SELECT * FROM Users ORDER BY id **\n-- #pageable\n",**
      countQuery = "SELECT count(*) FROM Users",
      nativeQuery = true)
    Page<User> findAllUsersWithPagination(Pageable pageable);
```

## `@Query`에서 Projection 사용하기

Entity가 아닌 VO로 데이터를 받고 싶을 때 에러가 났음.

그래서 `select new 패키지경로.vo이름( a.col1, a.col2 ) from table a` 이렇게 받아봤는데 안됐음. 그래서 더 찾아보다가..

[Spring JPA native query with Projection gives "ConverterNotFoundException"](https://stackoverflow.com/questions/49500309/spring-jpa-native-query-with-projection-gives-converternotfoundexception)

vo 를 interface로 만들어서 받으라는 얘기가 있어서 하단의 형식으로 변경(Interface-based Projections)

![](https://images.velog.io/images/juuxmee/post/67462950-d0e9-4ad8-b4cc-83659fb7cf89/image.png)

[Spring Data JPA - Reference Documentation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#projections)

## rownum 사용 시 ":" 사용

Named Parameter 사용 시, `@Param("paramName")String paramName` 파라미터에 이렇게 선언을 하고 쿼리에서 `:paramName` 이렇게 쓰는데, `@rownum := @rownum + 1` rownum 사용 시 : 이게 들어가서 쿼리 인식을 못함.

⇒ `@rownum \\:= @rownum + 1`
⇒ `:` 앞에 `\\`를 써야함

## 타임리프에서 데이터 가져올 때

```html
<td class="tx_cnt" th:text="${list?.getNo()}"></td>
<!-- NO -->
<td class="tx_cnt" th:text="${list?.getGubun()}"></td>
<!-- 구분 -->
```