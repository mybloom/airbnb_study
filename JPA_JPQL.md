# JPQL

## JPQL에서 AUTO ORDER BY
- 문제 : JPQL에서 ORDER BY를 명시하지 않았는데 원하지 않은 컬럼으로 자꾸 ORDER BY가 된다. 
- 원인 : 엔티티에 @OrderBy 애노테이션을 명시할 수 있는데 , 엔티티에 명시되어 있었다. 



> 공식문서 
- https://docs.oracle.com/cd/E11035_01/kodo41/full/html/ejb3_langref.html#ejb3_langref_orderby
