- - - -
layout: post
title:  "[QueryDSL] like vs contains vs startswith"
subtitle:   "QueryDSL Where절 조건 검색문 비교"
date: 2021-03-22
categories: Jpa
tags: Query-Dsl
comments: true
- - - -

# Like vs Contains vs StartsWith
> 모두 QueryDSL에서 Where절의 키워드 검색을 위한 구문입니다.  

세 가지 모두 비슷하면서 다른 차이점이 각각 존재하며, 각 차이를 알아보기 전 아래와 같은 검색을 위한 메소드가 있다고 가정하겠습니다.

```
private final EntityManager em;
private final JPAQueryFactory queryFactory;

public List<Member> searchMemberWithKeyword(String keyword){
	return queryFactory
				.selectFrom(QMember.member)
				.where(QMember.member.name.like(keyword))
				.where(QMember.member.name.contains(keyword))
				.where(QMember.member.name.startsWith(keyword))
				.fetch();
}
```

각각의 차이점은 실제 JPA에서 쿼리가 발생할때 볼 수 있으며 차이점은 다음과 같습니다.
like(keyword) == like = keyword // 해당 키워드와 완벽히 일치하는 것만 검색
contains(keyword) == like = %keyword% // 해당 키워드를 포함하는 것을 검색 
startsWith(keyword) == like = keyword% // 해당 키워드로 시작하는 것을 검색

사소한 차이이지만 이 차이점으로 예상치 못한 검색결과가 발생하거나, 검색이 안되는등의 크리티컬한 문제가 발생할 수 있기 때문에, 정확히 잘 기억해두거나 애매할경우 정확히 확인 후 사용하는것을 권장합니다.