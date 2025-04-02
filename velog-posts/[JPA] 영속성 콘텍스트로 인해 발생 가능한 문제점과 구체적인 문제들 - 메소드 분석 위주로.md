<blockquote>
<p>JpaRepository 메서드는 내부적으로 어떻게 동작할까?
 -&gt; 실무에서 발생할 수 있는 문제와 해결 전략까지 정리</p>
</blockquote>
<h3 id="📌-개념을-공부하게-된-배경">📌 개념을 공부하게 된 배경</h3>
<p>처음 JPA를 접했을 때는 편하게 CRUD를 구현하는 게 주요 목표였기 때문에,<br />Spring Data JPA의 <code>JpaRepository</code>를 상속받아 <code>save()</code>, <code>findById()</code>, <code>delete()</code> 같은 메서드를 별다른 고민 없이 사용했었다.</p>
<p>하지만 <strong>영속성 컨텍스트</strong> 개념을 배우면서, 
내가 사용한 메서드들이 <strong>예상과 다른 동작이 발생하거나 성능 문제가 생길 수도 있다는 사실</strong>을 알게 되었다.</p>
<p>그래서 <code>JpaRepository</code>의 주요 메서드들이</p>
<ol>
<li>내부적으로 어떻게 동작하고,  </li>
<li>그로 인해 어떤 문제가 발생할 수 있으며,  </li>
<li>그걸 예방하기 위해 어떤 방식으로 코드를 짜야 할지 위주로 정리해 보려고 한다.</li>
</ol>
<hr />
<h3 id="1-jpa-메서드-jparepository-메서드-동작-정리">1. JPA 메서드, JpaRepository 메서드 동작 정리</h3>
<h4 id="1-jpa-메서드-정리">(1) JPA 메서드 정리</h4>
<p>우선, JpaRepository 내부에서 쓰이는 JPA 메서드들에 대해 정리해보자.</p>
<table>
<thead>
<tr>
<th>JPA 메서드</th>
<th>영속성 컨텍스트에서의 역할</th>
<th>동작 설명</th>
<th>발생 가능 문제</th>
</tr>
</thead>
<tbody><tr>
<td><code>persist(entity)</code></td>
<td>객체를 영속성 컨텍스트에 등록</td>
<td>1차 캐시에 저장됨, flush 시 DB insert</td>
<td>이미 영속 상태면 예외 (<code>EntityExistsException</code>)</td>
</tr>
<tr>
<td><code>merge(entity)</code></td>
<td>비영속 객체 병합 (새 영속 객체 생성)</td>
<td>DB에서 ID로 조회 후 값 복사 → 새로운 객체 반환</td>
<td>데이터 충돌, 원하지 않는 덮어쓰기</td>
</tr>
<tr>
<td><code>remove(entity)</code></td>
<td>컨텍스트에서 제거 + flush 시 delete</td>
<td>영속 상태일 때만 사용 가능</td>
<td>준영속이면 예외 (<code>IllegalArgumentException</code>)</td>
</tr>
<tr>
<td><code>find(Class, id)</code></td>
<td>1차 캐시 or DB에서 조회 → 영속 등록</td>
<td>없으면 DB에서 select → 영속 상태로 등록됨</td>
<td>없음 (정상 작동 시 안전)</td>
</tr>
<tr>
<td><code>flush()</code></td>
<td>컨텍스트의 변경사항 DB 반영</td>
<td>insert, update, delete 쿼리 발생</td>
<td>타이밍 제어 실패 시 불완전한 상태 반영</td>
</tr>
<tr>
<td><code>clear()</code></td>
<td>1차 캐시 초기화 → 모든 엔티티 detach</td>
<td>모든 객체가 준영속 상태 됨</td>
<td>이후 변경감지 안 됨, Lazy 초기화 예외 발생 가능</td>
</tr>
</tbody></table>
<h4 id="2-jparepository-메서드별-내부-동작">(2) JpaRepository 메서드별 내부 동작</h4>
<p>앞서 살펴본 JPA의 핵심 메서드들은, Spring Data JPA의 <code>JpaRepository</code> 메서드 내부에서 사용된다.<br />각 JpaRepository 메서드는 어떤 JPA 메서드를 호출하고, 그로 인해 어떤 문제가 발생할 수 있는지 구체적으로 보자.</p>
<table>
<thead>
<tr>
<th>메서드</th>
<th>내부 호출</th>
<th>최종 JPA 메서드</th>
</tr>
</thead>
<tbody><tr>
<td><code>save()</code></td>
<td><code>isNew()</code> → <code>persist()</code> 또는 <code>merge()</code></td>
<td><code>em.persist()</code> 또는 <code>em.merge()</code></td>
</tr>
<tr>
<td><code>saveAll()</code></td>
<td><code>save()</code> 반복 호출</td>
<td>반복적으로 <code>persist()</code> 또는 <code>merge()</code></td>
</tr>
<tr>
<td><code>findById()</code></td>
<td>단건 조회</td>
<td><code>em.find()</code></td>
</tr>
<tr>
<td><code>findAll()</code></td>
<td>JPQL 조회</td>
<td><code>em.createQuery()</code></td>
</tr>
<tr>
<td><code>delete()</code></td>
<td><code>merge()</code> 여부 확인 후 삭제</td>
<td><code>em.merge()</code> + <code>em.remove()</code></td>
</tr>
<tr>
<td><code>deleteById()</code></td>
<td><code>findById()</code>로 조회 후 삭제</td>
<td><code>em.find()</code> + <code>em.remove()</code></td>
</tr>
</tbody></table>
<h4 id="save">save()</h4>
<pre><code class="language-jpa">@Transactional
public &lt;S extends T&gt; S save(S entity) {
    if (entityInformation.isNew(entity)) {
        em.persist(entity); // JPA persist 호출
        return entity;
    } else {
        return em.merge(entity); // JPA merge 호출
    }
}
</code></pre>
<h4 id="saveall">saveAll()</h4>
<pre><code class="language-jpa">@Transactional
public &lt;S extends T&gt; List&lt;S&gt; saveAll(Iterable&lt;S&gt; entities) {
    List&lt;S&gt; result = new ArrayList&lt;&gt;();

    for (S entity : entities) {
        result.add(save(entity)); // save 반복 호출
    }

    return result;
}
</code></pre>
<h4 id="findid">findId()</h4>
<pre><code class="language-jpa">public Optional&lt;T&gt; findById(ID id) {
    return Optional.ofNullable(em.find(domainClass, id)); // JPA find 호출
}</code></pre>
<h4 id="findall">findAll()</h4>
<pre><code class="language-jpa">public List&lt;T&gt; findAll() {
    return em.createQuery(&quot;select e from &quot; + entityName + &quot; e&quot;, domainClass)
             .getResultList(); // JPQL 실행
}
</code></pre>
<h4 id="delete">delete()</h4>
<pre><code class="language-jpa">@Transactional
public void delete(T entity) {
    if (!em.contains(entity)) {
        entity = em.merge(entity); // 병합 후
    }

    em.remove(entity); // 삭제
}
</code></pre>
<h4 id="deleteid">deleteId()</h4>
<pre><code>@Transactional
public void deleteById(ID id) {
    T entity = findById(id).orElseThrow(...); // 조회
    delete(entity); // 위의 delete 호출
}</code></pre><hr />
<h3 id="2-⚠-발생할-수-있는-문제">2. ⚠ 발생할 수 있는 문제</h3>
<h4 id="1-jpa-메서드로-인해-발생할-수-있는-문제">(1) JPA 메서드로 인해 발생할 수 있는 문제</h4>
<table>
<thead>
<tr>
<th>문제 유형</th>
<th>설명</th>
<th>대표 원인 메서드</th>
</tr>
</thead>
<tbody><tr>
<td>잘못된 병합</td>
<td>기존 데이터 덮어쓰기, ID 충돌</td>
<td><code>merge()</code></td>
</tr>
<tr>
<td>변경감지 실패</td>
<td>영속 상태가 아니어서 update 안 됨</td>
<td><code>clear()</code> 이후, <code>merge()</code> 결과 무시</td>
</tr>
<tr>
<td>삭제 실패</td>
<td>삭제 대상이 영속 상태가 아님</td>
<td><code>remove()</code></td>
</tr>
<tr>
<td>OutOfMemory</td>
<td>너무 많은 엔티티가 컨텍스트에 남음</td>
<td><code>persist()</code> 반복, <code>flush()</code>/<code>clear()</code> 누락</td>
</tr>
<tr>
<td>Lazy 로딩 예외</td>
<td>clear 이후 프록시 초기화 시점에서 실패</td>
<td><code>clear()</code> 이후 지연로딩 접근 시</td>
</tr>
<tr>
<td>#### (2) 이로 인해 JPARepository에서 발생할 수 있는 문제</td>
<td></td>
<td></td>
</tr>
<tr>
<td>JpaRepository 메서드</td>
<td>내부적으로 호출하는 JPA 메서드</td>
<td>연결된 문제</td>
</tr>
<tr>
<td>-----------------------</td>
<td>----------------------------------------</td>
<td>----------------------------------------------------------------</td>
</tr>
<tr>
<td><code>save(entity)</code></td>
<td>ID 여부 → <code>persist()</code> or <code>merge()</code></td>
<td>예상과 다른 병합 (<code>merge()</code> 동작), 데이터 덮어쓰기</td>
</tr>
<tr>
<td><code>saveAll(entities)</code></td>
<td>반복적으로 <code>save()</code> 호출</td>
<td>OutOfMemory, 병합/신규 객체 혼합 위험</td>
</tr>
<tr>
<td><code>findById(id)</code></td>
<td><code>em.find()</code></td>
<td>없음 (단독으로 안전), 이후 수정 시 Dirty Checking 전제 필요</td>
</tr>
<tr>
<td><code>findAll()</code></td>
<td>JPQL → <code>em.createQuery()</code></td>
<td>Lazy 로딩으로 인한 N+1 문제 발생 가능</td>
</tr>
<tr>
<td><code>delete(entity)</code></td>
<td>(merge) + <code>remove()</code></td>
<td>준영속 상태일 경우 예외 발생</td>
</tr>
<tr>
<td><code>deleteById(id)</code></td>
<td><code>findById()</code> → <code>delete()</code></td>
<td>ID가 없는 경우 예외 or silent fail</td>
</tr>
<tr>
<td><code>flush()</code></td>
<td><code>em.flush()</code></td>
<td>시점 제어 안 하면 불완전한 커밋 가능</td>
</tr>
<tr>
<td><code>clear()</code> (직접 호출)</td>
<td><code>em.clear()</code></td>
<td>이후 Lazy 접근 시 <code>LazyInitializationException</code></td>
</tr>
</tbody></table>
<hr />
<h3 id="3-실전에서-겪는-문제--해결-전략-모음">3. 실전에서 겪는 문제 + 해결 전략 모음</h3>
<p>💥 문제 1: <code>save()</code>를 사용했는데 UPDATE가 안 됨</p>
<ul>
<li>JPA 내부 동작: <code>save()</code>가 <code>merge()</code> 호출</li>
<li>merge는 새 객체 생성 → 기존 객체와는 별개</li>
<li>실수로 기존 객체의 필드 변경이 반영 안 됨</li>
</ul>
<p>→ <strong>해결</strong>: <code>findById()</code>로 영속 객체 조회 후 직접 수정</p>
<p>💥 문제 2: 게시글 10만 건 저장 중 OutOfMemory</p>
<ul>
<li>내부 동작: <code>save()</code> or <code>saveAll()</code>로 <code>persist()</code> 반복</li>
<li>flush/clear 없이 1차 캐시에 객체 누적됨</li>
<li>GC 대상이 안 됨 → 메모리 터짐</li>
</ul>
<p>→ <strong>해결</strong>: 1000건 단위로 <code>flush()</code> + <code>clear()</code> 호출</p>
<p>💥 문제 3: Lazy 연관 객체 접근 시 예외 발생</p>
<ul>
<li>내부 동작: <code>findAll()</code> → 연관 객체는 <code>LAZY</code></li>
<li>이후 <code>@Transactional</code> 종료로 컨텍스트 clear</li>
<li>Controller 등에서 <code>엔티티.getX().getY()</code> 접근 시 <code>LazyInitializationException</code></li>
</ul>
<p>→ <strong>해결</strong>: 서비스 계층에서 연관 객체 접근 or fetch join 사용</p>
<hr />
<h3 id="회고">회고</h3>
<p>최대한 문제를 사전에 예방하기 위해 조사해봤지만, 아직 구체적인 트러블슈팅을 경험해보진 못했다.<br />다만 코드 리뷰나 실무 상황에서 비슷한 문제를 마주하게 된다면, 이번에 정리한 내용을 기준으로 보다 구조적으로 접근해볼 수 있을 것 같다.</p>
<p>SpringBoot나 JPA도 결국 내부 동작을 이해하고 구조를 파악해야 문제를 제대로 예방할 수 있다는 점은 변하지 않는 것 같다.<br />그래도 자주 발생하는 문제 유형들이 어느 정도 정형화되어 있어서, 미리 대비하고 학습하기가 예전보다 수월하다는 건 큰 장점이라고 느꼈다.
특히, 첫 프로젝트로 MyBatis와 JDBC를 사용하고, 로그인을 Servlet으로 구현해 본 입장에서는 지금이 훨씬 더 나은 것 같다...</p>
<hr />
<h3 id="💬-면접-준비">💬 면접 준비</h3>
<ul>
<li><code>save()</code>는 persist냐 merge냐 자동으로 분기됨 → 수정엔 직접 조회 후 수정이 더 안전  </li>
<li><code>merge()</code>는 상태 복구용이지, 추천 기본 방식은 아님  </li>
<li>면접에서 &quot;save() 내부 동작 아세요?&quot;, &quot;persist vs merge 차이 아세요?&quot; → 자주 나옴  </li>
<li>실무에서 &quot;왜 DB에 반영이 안 되죠?&quot; → 대부분은 상태 문제 (준영속 or 트랜잭션 누락)</li>
</ul>
<hr />
<h3 id="📚-references">📚 References</h3>
<ul>
<li><a href="https://www.yes24.com/Product/Goods/19040233">자바 ORM 표준 JPA 프로그래밍 (김영한 저)</a></li>
<li><a href="https://docs.spring.io/spring-data/jpa/reference/jpa/entity-persistence.html">공식 문서: Spring Data JPA Repository - Persisting Entities</a></li>
</ul>