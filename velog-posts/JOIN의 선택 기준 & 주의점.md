<h3 id="1-이-개념을-공부하게-된-배경">1. 이 개념을 공부하게 된 배경</h3>
<p>여러 테이블을 조인하는 상황에서 <strong>&quot;어떤 JOIN을 써야 하는지&quot;</strong> 판단 기준이 애매했다.<br />그래서 <strong>어떤 조건일 때 어떤 JOIN을 써야 하는지</strong> 기준을 정리해 보려고 했다.</p>
<hr />
<h3 id="2-join-종류-요약">2. JOIN 종류 요약</h3>
<table>
<thead>
<tr>
<th>JOIN 종류</th>
<th>기준 테이블</th>
<th>조회 대상</th>
<th>NULL 포함</th>
<th>주요 목적</th>
</tr>
</thead>
<tbody><tr>
<td>INNER JOIN</td>
<td>양쪽</td>
<td>양쪽 모두 일치</td>
<td>❌</td>
<td>공통 데이터 조회</td>
</tr>
<tr>
<td>LEFT JOIN</td>
<td>왼쪽</td>
<td>왼쪽 전체 + 오른쪽 일치</td>
<td>✅</td>
<td>왼쪽 테이블 보존</td>
</tr>
<tr>
<td>RIGHT JOIN</td>
<td>오른쪽</td>
<td>오른쪽 전체 + 왼쪽 일치</td>
<td>✅</td>
<td>오른쪽 테이블 보존</td>
</tr>
<tr>
<td>FULL OUTER JOIN</td>
<td>양쪽</td>
<td>모든 조합</td>
<td>✅</td>
<td>전체 병합 (MySQL 미지원)</td>
</tr>
<tr>
<td>SELF JOIN</td>
<td>단일</td>
<td>같은 테이블 간 관계</td>
<td>✅</td>
<td>계층 구조 조회</td>
</tr>
<tr>
<td>CROSS JOIN</td>
<td>없음</td>
<td>모든 조합</td>
<td>❌</td>
<td>경우의 수 생성</td>
</tr>
<tr>
<td>SEMI / ANTI JOIN</td>
<td>주로 단일 기준</td>
<td>존재 여부만 확인</td>
<td>❌ or ✅</td>
<td>필터링 (EXISTS / NOT EXISTS)</td>
</tr>
</tbody></table>
<hr />
<h3 id="3-실무-상황별-join-선택-기준">3. 실무 상황별 JOIN 선택 기준</h3>
<table>
<thead>
<tr>
<th>상황 예시</th>
<th>추천 JOIN</th>
<th>이유</th>
</tr>
</thead>
<tbody><tr>
<td>주문한 사용자만 조회</td>
<td>INNER JOIN</td>
<td>양쪽 모두 데이터 존재</td>
</tr>
<tr>
<td>모든 사용자 + 주문 유무 조회</td>
<td>LEFT JOIN</td>
<td>필수 테이블 유지</td>
</tr>
<tr>
<td>카테고리는 모두 보되, 상품이 있을 수도 없을 수도</td>
<td>RIGHT JOIN</td>
<td>오른쪽(카테고리) 기준</td>
</tr>
<tr>
<td>연결되지 않은 직원·부서도 모두 조회</td>
<td>FULL OUTER JOIN</td>
<td>전체 병합 필요 (MySQL: UNION)</td>
</tr>
<tr>
<td>직원-상사 구조 조회</td>
<td>SELF JOIN</td>
<td>테이블 내 관계</td>
</tr>
<tr>
<td>조건 만족하는 항목 존재 여부만 확인</td>
<td>SEMI JOIN</td>
<td>EXISTS 사용</td>
</tr>
<tr>
<td>조건을 만족하지 않는 항목 조회</td>
<td>ANTI JOIN</td>
<td>NOT EXISTS 또는 LEFT + IS NULL</td>
</tr>
</tbody></table>
<hr />
<h3 id="4-실무에서-자주-하는-실수--주의점">4. 실무에서 자주 하는 실수 &amp; 주의점</h3>
<h4 id="✅-실수-1-left-join을-써놓고-where-절에서-오른쪽-테이블-필드를-필터링">✅ 실수 1: LEFT JOIN을 써놓고 WHERE 절에서 오른쪽 테이블 필드를 필터링</h4>
<pre><code class="language-sql">SELECT u.id, u.name, o.id AS order_id
FROM user u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.status = 'DONE';</code></pre>
<p><em>문제점:</em><br />→ <code>o.status</code>가 NULL인 경우 필터링에서 제외되어, 결과적으로 <strong>INNER JOIN과 동일한 결과</strong>가 나옴.</p>
<p><em>의도:</em><br />→ 모든 사용자 + 주문이 있다면 상태가 'DONE'인 주문만 보고 싶었을 것.</p>
<p><em>수정 방법:</em>  </p>
<pre><code class="language-sql">-- 방법 1: JOIN 조건 안에 필터 조건 포함
LEFT JOIN orders o ON u.id = o.user_id AND o.status = 'DONE'

-- 방법 2: WHERE 조건에서 NULL 고려
WHERE o.status = 'DONE' OR o.id IS NULL</code></pre>
<hr />
<h4 id="✅-실수-2-join-순서에-따라-기준-테이블이-바뀌는-것-간과">✅ 실수 2: JOIN 순서에 따라 기준 테이블이 바뀌는 것 간과</h4>
<pre><code class="language-sql">SELECT d.name, e.name
FROM department d
RIGHT JOIN employee e ON e.dept_id = d.id;</code></pre>
<p><em>문제점:</em><br />→ employee를 기준으로 JOIN하므로, <strong>모든 직원은 포함되지만</strong> 부서 정보가 없는 경우 NULL로 나옴.</p>
<p><em>실무 판단 기준:</em><br />→ “<strong>누가 기준 테이블인가?</strong>”를 먼저 결정하고, 해당 방향으로 LEFT JOIN을 쓰는 게 명확함.</p>
<p><em>추천 방식:</em>  </p>
<pre><code class="language-sql">-- 부서를 기준으로 모든 직원 포함
SELECT d.name, e.name
FROM department d
LEFT JOIN employee e ON e.dept_id = d.id;</code></pre>
<hr />
<h4 id="✅-실수-3-조건-분기에서-null을-고려하지-않음">✅ 실수 3: 조건 분기에서 NULL을 고려하지 않음</h4>
<pre><code class="language-sql">WHERE o.status = 'DONE'</code></pre>
<p><em>문제점:</em><br />→ <code>o.status</code>가 NULL인 행은 필터링 대상이 아님.<br />→ LEFT JOIN 상황에서 주문이 없는 사용자 정보가 누락될 수 있음.</p>
<p><em>해결법:</em><br />→ 조건이 필요한 경우 <code>IS NULL</code>, <code>IS NOT NULL</code>을 명시적으로 고려할 것</p>
<hr />
<h4 id="✅-실수-4-self-join-시-alias-미사용">✅ 실수 4: SELF JOIN 시 alias 미사용</h4>
<pre><code class="language-sql">SELECT e.name, m.name
FROM employee e
JOIN employee m ON e.manager_id = m.id;</code></pre>
<p>→ 동일 테이블을 조인할 때 <strong>alias(별칭)</strong> 없으면 컬럼 충돌 또는 혼동 발생<br />→ 항상 <code>e</code>, <code>m</code>처럼 명확하게 구분할 것</p>
<hr />
<h3 id="5-📌-면접-포인트">5. 📌 면접 포인트</h3>
<table>
<thead>
<tr>
<th>질문</th>
<th>핵심 답변</th>
</tr>
</thead>
<tbody><tr>
<td>INNER vs LEFT JOIN?</td>
<td>LEFT는 기준 테이블 유지 + NULL 허용</td>
</tr>
<tr>
<td>LEFT JOIN 썼는데 결과 줄어드는 이유?</td>
<td>WHERE 절에서 오른쪽 조건 걸면 INNER처럼 동작</td>
</tr>
<tr>
<td>JOIN 성능 이슈는 왜 생기나요?</td>
<td>과도한 조인, 인덱스 미사용, 잘못된 조건 순서 등</td>
</tr>
<tr>
<td>JOIN 선택 기준은?</td>
<td>누가 필수 테이블인지 먼저 판단하는 게 핵심</td>
</tr>
</tbody></table>
<hr />
<h3 id="6-references">6. References</h3>
<ul>
<li><a href="https://www.w3schools.com/sql/sql_join.asp">W3Schools - SQL JOIN</a>  </li>
<li><a href="https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-JOIN">PostgreSQL - JOIN Types</a>  </li>
<li><a href="https://dev.mysql.com/doc/refman/8.0/en/join.html">MySQL 공식 문서 - JOIN</a>  </li>
</ul>