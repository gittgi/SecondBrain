# XML

```table-of-contents
```

##  XML이란?

- XML (Extensible Markup Language)는 마크업[^1] 형태를 쓰는 데이터 교환 형식

## XML의 구성 

```XML
<?xml version="1.0" encoding="UTF-8"?> 
<movieList>
	<movie rank="1">
		<title>탑건 : 매버릭</title>
		<actor>톰 크루즈</actor>
	</movie>
	<movie rank="2">
		<title>해리포터</title>
		<actor>다니엘 레드클리프</actor>
	</movie>	 
</movieList>
```

1. 프롤로그 : 버전, 인코딩 정보
2. 루트요소 : 단 하나만 존재 (`movieList`)
3. 하위 요소들


## HTML과 XML 비교

- HTML의 용도는 데이터 표시, XML은 데이터를 저장 및 전송
- HTML에는 미리 정의된 태그 사용(`div`, `h1` 등), XML에서는 고유한 태그를 정의해서 사용가능
- HTML은 대소문자 구분 x, XML은 대소문자 구분 -> XML 구문 분석기에서 오류 발생

## JSON과 XML 비교

- JSON과 달리 XML은 닫는 태그가 지속 발생 -> JSON 보다 무거워짐
- Javascript Object로 변환하기 위해서 JSON보다 더 많은 노력 필요 (JSON.parse는 내장, XML은 외부라이브러리 필요)


## XML의 활용

- XML은 sitemap.xml 에 주로 사용
	- sitemap.xml은 서비스 내의 모든 페이지들을 리스트업한 데이터
	- 매우 큰 사이트나, 링크가 종속적이지 않은 페이지 등(장바구니 페이지 등)의 경우 검색엔진 크롤러가 일부 페이지를 누락할 가능성 -> sitemap.xml이 있다면 누락하는 페이지 없이 모든 페이지를 크롤링할 수 있도록 도움
```XML
<?xml version="1.0" encoding="UTF-8"?>  
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
	<url>
		<loc>http://www.example.com/foo.html</loc>
		<lastmod>2018-06-04</lastmod>
	</url> 
	<url>
		<loc>http://www.example.com/abc.html</loc>
		<lastmod>2018-06-04</lastmod>
	</url>
</urlset>
```

[^1]: 마크업 (markup)은 태그 등을 이용하여 문서나 데이터의 구조를 나타내는 방법, 속성 부여도 가능