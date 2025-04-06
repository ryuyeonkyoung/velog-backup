<h3 id="공부하게-된-이유"><strong>공부하게 된 이유</strong></h3>
<p>UMC 과제중에 Soft Delete를 어떤 HTTP 메소드로 구현해야 하는지 고민해 보는 과제가 있었는데, 이번 워크북을 통해 HTTP method들을 추가로 공부한 나는 당연히 PATCH일 것이라 생각했다. 하지만 의외로 실무에서는 DELETE를 많이 사용하는 것 같았고, 그 이유가 궁금해서 더 알아보고 정리해보았다.</p>
<hr />
<h3 id="soft-delete란"><strong>Soft Delete란?</strong></h3>
<p>→ <strong>데이터를 실제로 삭제하지 않고, 삭제된 것처럼 표시만 하는 방식</strong></p>
<ul>
<li>방법: 테이블에 <code>deleted</code>, <code>is_deleted</code>, <code>status</code> 같은 컬럼을 두고 값을 변경</li>
<li>장점: 데이터 복구 가능, 감사 로그 유지</li>
<li>단점: 쿼리가 복잡해지고, 삭제된 데이터가 계속 누적됨</li>
</ul>
<hr />
<h3 id="그럼-어떤-http-method를-써야-할까"><strong>그럼 어떤 HTTP Method를 써야 할까?</strong></h3>
<h4 id="✅-patch">✅ PATCH</h4>
<ul>
<li>REST 원칙만 보면, <strong>일부 상태만 바꾸는 거라서 PATCH가 맞는 것처럼 보임</strong></li>
<li>예: <code>is_deleted</code> 값을 false → true로 변경</li>
</ul>
<h4 id="✅-delete">✅ DELETE</h4>
<ul>
<li>하지만 사용자는 “해당 데이터를 삭제하고 싶다”는 의도를 갖고 있기 때문에 의미론적으로는 DELETE가 더 자연스러움</li>
<li>실제로 실무에서는 DELETE를 더 많이 쓰는 것 같았다</li>
</ul>
<p>→ 즉,<br /><strong>REST 원칙을 엄격히 따르려면 PATCH</strong>,  
<strong>사용자 의도를 표현하려면 DELETE</strong><br />(실제로는 DELETE가 더 흔하게 사용됨)</p>
<hr />
<blockquote>
<p>💡 근데 DELETE를 쓰면 진짜 다 지워지는 거 아닌가?</p>
</blockquote>
<p>그렇지 않다. HTTP Method는 <strong>의미 전달용</strong>일 뿐, 실제 동작을 강제하지는 않는다.</p>
<p>서버가 DELETE 요청을 받아도<br /><em>실제로 삭제할지</em>,  
<em>단순히 상태만 바꿀지</em>는 서버가 결정할 수 있다.</p>
<p><a href="https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.5">RFC7231, 4.3.5 DELETE</a> 의 부분을 보면 다음과 같이 시작한다.</p>
<pre><code>4.3.5.  DELETE

   The DELETE method requests that the origin server remove the
   association between the target resource and its current
   functionality.  In effect, this method is similar to the rm command
   in UNIX: it expresses a deletion operation on the URI mapping of the
   origin server rather than an expectation that the previously
   associated information be deleted.

   If the target resource has one or more current representations, they
   might or might not be destroyed by the origin server, and the
   associated storage might or might not be reclaimed, depending
   entirely on the nature of the resource and its implementation by the
   origin server (which are beyond the scope of this specification).
   ...</code></pre><p>한국어로 번역하면 다음과 같다.</p>
<pre><code>DELETE 메서드는 원 서버에게 **대상 리소스와 현재 기능(역할) 간의 연결을 제거하라고 요청**하는 것이다.
이 메서드는 유닉스의 rm 명령어와 유사하다. &quot;이전에 연결되어 있던 정보가 실제로 삭제될 것&quot;이라는 기대가 아니라 &quot;서버가 관리하는 URI 매핑에서 그 리소스를 제거하겠다는 의미&quot;로 이해되어야 한다.

대상 리소스가 현재 하나 이상의 표현(예: JSON 응답)을 갖고 있다면, 그 표현들이 실제로 삭제될 수도 있고, 아닐 수도 있다. 이건 전적으로 서버 구현 방식에 따라 다르다.</code></pre><blockquote>
<p>따라서 DELETE는 리소스를 완전히 제거한다는 뜻이 아니라,<br /><strong>&quot;더 이상 해당 리소스를 제공하지 않겠다&quot;는 의미</strong>에 가깝다.<br />즉, Soft Delete를 PATCH가 아닌 DELETE로 처리해도 문제가 없다.</p>
</blockquote>
<hr />
<h3 id="결론">결론</h3>
<p>사용자에게 &quot;삭제 요청&quot;이라는 의미를 전달하고 싶을 때는 DELETE가 더 직관적이여서 자주 사용된다.
→ 결국 중요한 건, <strong>HTTP 메서드를 기술이 아니라 ‘의도를 전달하는 언어’로 이해하는 것</strong>인 것 같다. 기능적으로는 완전한 삭제가 아니더라도, <strong>DELETE를 사용하는 것은 REST API의 의미 전달 측면에서 충분히 RESTful할 수 있다.</strong></p>
<hr />
<h3 id="번외-진짜-rest-가짜-rest">번외. 진짜 REST? 가짜 REST?</h3>
<p>이 글을 작성하기 위해 자료를 찾아보다가 우리가 보편적으로 사용하고 있는게 실제로는 RESTful하지 않다는 이야기를 자주 접했다.</p>
<p>REST의 창시자인 Roy Fielding의 말을 정리해보면,</p>
<blockquote>
<p>Roy Fielding이 말하는 “진짜 REST”는 단순히 HTTP 메서드를 쓰는 게 아니라,
클라이언트가 서버로부터 상태 전이에 필요한 모든 정보를 받아야 하는 HATEOAS 구조를 요구한다.</p>
</blockquote>
<p>HATEOAS (Hypermedia As The Engine Of Application State) 는 서버가 자원과 함께 가능한 행동까지 설명함으로써,
클라이언트가 서버의 내부 URI 구조를 몰라도 동작할 수 있도록 돕는다.</p>
<p>구체적인 예시는 다음과 같다.</p>
<p>예1) 우리가 아는 REST API</p>
<pre><code class="language-json">{
  &quot;id&quot;: 1,
  &quot;name&quot;: &quot;철수&quot;,
  &quot;email&quot;: &quot;chulsoo@example.com&quot;
}</code></pre>
<p>예2) &quot;진짜&quot; REST API</p>
<pre><code class="language-json">{
  &quot;id&quot;: 1,
  &quot;name&quot;: &quot;철수&quot;,
  &quot;email&quot;: &quot;chulsoo@example.com&quot;,
  &quot;_links&quot;: {
    &quot;self&quot;: { &quot;href&quot;: &quot;/users/1&quot; },
    &quot;update&quot;: { &quot;href&quot;: &quot;/users/1&quot;, &quot;method&quot;: &quot;PUT&quot; },
    &quot;delete&quot;: { &quot;href&quot;: &quot;/users/1&quot;, &quot;method&quot;: &quot;DELETE&quot; }
  }
}
</code></pre>
<p>_links는 다음 행동에 대한 정보이다. 이것의 목적은 클라이언트가 서버에 대한 지식 없이도 정보를 처리할 수 있게 하기 위함이다. 이를 통해 클라이언트는 서버가 안내하는 대로만 행동해도 계속 다음 동작을 이어나갈 수 있다.</p>
<p>하지만 실무적으로는 전자와 같은 REST 스타일 API도 RESTful하다고 여겨지고, 협업하는 데에는 기존의 정보만 알고 있어도 크게 문제가 없을 것 같다. (아니라면 다시 포스팅하겠다...)</p>