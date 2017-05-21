---
layout: post
title: 파이썬으로 SPARQL Endpoint 활용하기
categories: [sparql]
tags: [Python, Sparql, Semantic Web]
fullview: true
comments: true
---
# 파이썬으로 SPARQL Endpoint 활용하기 

SPARQL Endpoint는 DBpedia, Wikidata 같은 Triplestore에서 해당 Store에 대한 Sparql 질의를 쉽게 할수 있도록 하는 기능이다. 그 Sparql Endpoint에 접근함으로써 각 Store에 저장된 RDF 데이터를 자신의 원하는 형태로 불러와 활용이 가능하다. SPARQL Endpoint에 접근해 활용하는 것에는 몇 가지 방법이 있다.

### SPARQL 문 내의 "SERVICE" Keyword 

SERVICE 키워드는 SPARQL 1.1 버전에서 제공한 기능이다. SERVICE 키워드를 사용함으로써 사용자는 특정 데이터에 대해 (웹을 통해) 원격으로 접근 가능하다. 웹에 업로드 되어있는 RDF 파일에 접근하는 "FROM" 키워드에 비해, "SERVICE" 키워드는 웹의 데이터 파일이 아닌 웹에서 제공하는 SPARQL Endpoint 서비스에 접근하는 것이다.  "FROM" 키워드를 사용한 SPARQL 질의문 예시는 다음과 같다.

```SPARQL
# filename: ex166.rq

PREFIX dc: <http://purl.org/dc/elements/1.1/>

SELECT ?title
FROM <http://dig.csail.mit.edu/2008/webdav/timbl/foaf.rdf>
WHERE { ?s dc:title ?title .}
```

"SERVICE" 키워드를 활용한 SPARQL 질의문의 예시는 다음과 같다.

```SPARQL
# filename: ex167.rq

PREFIX cat:     <http://dbpedia.org/resource/Category:>
PREFIX skos:    <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs:    <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl:     <http://www.w3.org/2002/07/owl#>
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>

SELECT ?p ?o 
WHERE
{
  SERVICE <http://DBpedia.org/sparql>
  { SELECT ?p ?o 
    WHERE { <http://dbpedia.org/resource/Joseph_Hocking> ?p ?o . }
  }
}
```

위 SPARQL 문의 예시는 DBpedia의 SPARQL Endpoint에 접근한다. 

### Python 라이브러리를 사용한 SPARQL Endpoint 접근 - urllib 라이브러리

SPARQL 언어 내의 SERIVICE 키워드를 사용하는 것이 아닌, 타 프로그래밍 언어를 통해서도 SPARQL Endpoint에 접근할 수 있다. Python에 내장되어 있는 웹 크롤링 라이브러리인 urllib를 통해 Triplesotre의 Endpoint에 SPARQL 질의를 던질 수 있다. 다음 예시는 urllib를 통해 DBpedia의 Endpoint에 질의를 하는 것이다. DBpedia의 Endpoint는 URL상에서 "query"라는 parameter에 SPARQL 언어로 작성된 질의문을 담고있다. 질의문을 "query" 파라미터에 담기 전, SPARQL로 작성된 질의문을 URL에 덧붙일 때 잘 인식하도록 적절한 과정이 필요하다. 이는 Python 3에서는 ```urllib.parse``` 라이브러리로 가능하다.

```urllib``` 를 활용한 코드 예시는 다음과 같다.

``` python
import urllib

endpoint="http://dbpedia.org/sparql"
query="""SELECT ?s WHERE {
  ?s ?p ?o
} LIMIT 10"""
escapedQuery = urllib.parse.quote(query)
requestURL = endpoint + "?query=" + escapedQuery
result = urllib.request.urlopen(requestURL)
print(requestURL)
print(result.read())
```

```urllib``` 는 Python 내장 라이브러리이다. 별도의 설치 없이 해당 라이브러리를 사용 가능하다.  ```urllib.parse.quote()```을 통해 질의문의 내용을 URL에 적절하게 덧붙이도록 한다. 

### Python 라이브러리를 사용한 SPARQL Endpoint 접근 - SPARQLWrapper 라이브러리

urllib```를 활용한 이 코드의 결과는 XML 포맷의 데이터이다. 이 데이터를 활용하기 위해선 XML 포맷을 파싱하는 과정이 필요하다. 번거로울 수 있는 XML 파싱 과정을 생략하고 더 쉬운 방법으로 native 상태의 데이터를 사용하기 위해 다른 라이브러리를 사용할 수 있다. Python의 경우 [SPARQLWrapper](https://rdflib.github.io/sparqlwrapper/) 라이브러리가 있다.

```SPARQLWrapper```의 설치는 간단히 ```pip``` 명령어를 통해 가능하다.

```pip install SPARQLWrapper```

```SPARQLWrapper``` 라이브러리는 RDF 파싱 라이브러리인 [rdflib](https://pypi.python.org/pypi/rdflib) 설치가 우선 되어야 한다.

```SPARQLWrapper```를 활용한 코드 예시는 다음과 같다.

```python
from SPARQLWrapper import SPARQLWrapper, JSON

sparql = SPARQLWrapper("http://dbpedia.org/sparql")
sparql.setQuery("""
    PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
    SELECT ?label
    WHERE { <http://dbpedia.org/resource/Asturias> rdfs:label ?label }
""")
sparql.setReturnFormat(JSON)
results = sparql.query().convert()

for result in results["results"]["bindings"]:
    print(result["label"]["value"])
```

setReturnFormat은 Endpoint에 질의를 한 후, 받아온 데이터를 어느 포맷으로 할지 설정하는 것이다. 가능한 포맷으로는 JSON, XML, TURTLE, N3, RDF이 있다. 위 코드에선 JSON 포맷으로 데이터를 받아와 간단한 for loop으로 native 상태의 데이터를 가질수 있었다.

더 자세한 내용은 [SPARQLWrapper의 온라인 문서](https://rdflib.github.io/sparqlwrapper/doc/)를 참조 가능하다.

이러한 SPAQRL 라이브러리는 파이썬에만 있는 것은 아니다. JAVA, Perl 같은 타 프로그래밍 언어에도 이와 유사한 라이브러리가 존재한다. JAVA에는 ARQ 라이브러리, Perl에는 RDF-Query등이 있다. 이러한 라이브러리를 사용해 Semantic Web Data를 자신의 마음 것 활용할 수 있다.