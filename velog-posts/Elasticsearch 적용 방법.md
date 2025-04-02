<p>기존 프로젝트에서 CRUD 기능을 구현한 후, 사용자 경험을 더 개선하고 싶다는 욕심이 생겼다.
특히 검색 기능은 많은 웹 서비스에서 핵심 역할을 하기 때문에,
단순히 LIKE 쿼리를 사용하는 수준을 넘어 검색엔진을 활용한 확장 가능성 있는 구조를 직접 구현해보고자 했다.</p>
<p>이 글에서는 Elasticsearch를 도입한 이유와 기본 연동 과정을 정리하고,
다음 글에서는 실제 자동완성 기능 구현까지 이어서 설명할 예정이다.</p>
<h3 id="1-기존-방식의-한계">1. 기존 방식의 한계</h3>
<p>처음에는 MySQL에서 다음과 같이 검색 기능을 구현했다.</p>
<pre><code>SELECT * FROM posts WHERE title LIKE '%검색어%';</code></pre><p>이러한 방법에는 한계가 있다:</p>
<p><strong>한계</strong></p>
<ul>
<li>%검색어%는 인덱스를 사용할 수 없어 전체 테이블을 탐색해야 한다</li>
<li>대소문자, 띄어쓰기 등을 고려할 수 없다</li>
<li>검색어 자동완성 등 기능 확장이 어렵다</li>
<li>데이터가 많아지면 속도가 급격히 느려진다</li>
</ul>
<p>→ 이 문제를 해결하기 위해 Elasticsearch 를 사용할 수 있다.</p>
<h3 id="2-elasticsearch란">2. Elasticsearch란?</h3>
<p>Elasticsearch는 대용량 텍스트 기반 검색에 특화된 오픈소스 검색엔진이다.</p>
<p><strong>특징</strong></p>
<ul>
<li>문서 단위(JSON)로 데이터를 저장</li>
<li>역색인(inverted index) 구조를 사용해 빠른 검색 지원</li>
<li>다양한 자연어 분석기 제공 (n-gram, 형태소 분석 등)</li>
<li>match, term, bool, highlight 등 다양한 검색 기능 제공</li>
</ul>
<p><strong>흐름</strong></p>
<ol>
<li>사용자 검색어 입력</li>
<li>Elasticsearch에 검색 쿼리 요청</li>
<li>필터링된 문서 결과 반환</li>
</ol>
<h3 id="3-개발-환경-및-설정">3. 개발 환경 및 설정</h3>
<ul>
<li>Spring Boot: 3.2.0</li>
<li>Elasticsearch 버전: 7.17.10 (Docker로 로컬 실행)</li>
</ul>
<p>Gradle 의존성</p>
<pre><code>dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-elasticsearch'
}</code></pre><p>Docker 명령어 예시</p>
<pre><code>docker run -d -p 9200:9200 -e &quot;discovery.type=single-node&quot; elasticsearch:7.17.10</code></pre><h3 id="4-기본-검색-기능-구현-예시">4. 기본 검색 기능 구현 예시</h3>
<ol>
<li>Elasticsearch 전용 Document 정의<pre><code>@Document(indexName = &quot;articles&quot;)
public class ArticleDocument {
 @Id
 private String id;
 private String title;
 private String content;
}
</code></pre></li>
</ol>
<pre><code>2. ElasticsearchRepository 인터페이스 정의</code></pre><p>public interface ArticleSearchRepository extends ElasticsearchRepository&lt;ArticleDocument, String&gt; {
    List findByTitleContaining(String keyword);
}</p>
<pre><code>3. 비즈니스 로직 (검색 API 구현)</code></pre><p>@RestController
@RequestMapping(&quot;/search&quot;)
public class ArticleSearchController {</p>
<pre><code>private final ArticleSearchRepository repository;

public ArticleSearchController(ArticleSearchRepository repository) {
    this.repository = repository;
}

@GetMapping
public List&lt;ArticleDocument&gt; search(@RequestParam String q) {
    return repository.findByTitleContaining(q);
}</code></pre><p>}</p>
<p>```</p>