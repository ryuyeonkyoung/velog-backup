<h3 id="실습-개요">실습 개요</h3>
<p>Offset 페이징은 첫 페이지에서는 무난하지만, 정렬 기준이 복합적인 경우 정확도가 떨어지고, 페이지가 뒤로 갈수록 성능 저하가 발생하는 단점이 있다. 이에 따라 Cursor 기반 페이징으로 SQL을 개선하여 더 안정적이고 효율적인 페이징 방식을 적용한다.</p>
<hr />
<h3 id="offset-기반-페이징-구조와-한계">Offset 기반 페이징 구조와 한계</h3>
<h4 id="✅-기존-offset-방식">✅ 기존 Offset 방식</h4>
<pre><code class="language-sql">SELECT m.*, mm.*, s.*
FROM mission AS m
JOIN member_mission AS mm ON m.id = mm.mission_id
JOIN store AS s ON m.store_id = s.id
WHERE mm.member_id = {member_id} AND mm.completed = true
ORDER BY mm.updated_at DESC
LIMIT 10 OFFSET ({페이지 번호} - 1) * 10;</code></pre>
<ul>
<li>장점  <ul>
<li>단순한 구조, 직관적인 구현  </li>
</ul>
</li>
<li>단점  <ul>
<li>중간 삽입/삭제 시 중복 또는 누락 발생  </li>
<li>OFFSET이 클수록 성능 저하  </li>
<li>복합 정렬 시 정확한 순서 보장 어려움  </li>
</ul>
</li>
</ul>
<hr />
<h3 id="cursor-기반-페이징-구조와-개선-포인트">Cursor 기반 페이징 구조와 개선 포인트</h3>
<h4 id="✅-개선된-cursor-방식">✅ 개선된 Cursor 방식</h4>
<pre><code class="language-sql">SELECT
  mm.mission_id,
  mm.completed,
  mm.updated_at,
  m.min_spend_money,
  m.reward_points,
  s.name AS store_name,
  CONCAT(
    LPAD(m.reward_points, 10, '0'),
    LPAD(UNIX_TIMESTAMP(mm.created_at), 10, '0')
  ) AS cursor_value
FROM mission AS m
JOIN member_mission AS mm ON m.id = mm.mission_id
JOIN store AS s ON m.store_id = s.id
WHERE mm.member_id = {member_id}
  AND mm.completed = true
  AND CONCAT(
        LPAD(m.reward_points, 10, '0'),
        LPAD(UNIX_TIMESTAMP(mm.created_at), 10, '0')
      ) &lt;
      (
        SELECT CONCAT(
                 LPAD(m_sub.reward_points, 10, '0'),
                 LPAD(UNIX_TIMESTAMP(mm_sub.created_at), 10, '0')
               )
        FROM mission AS m_sub
        JOIN member_mission AS mm_sub ON m_sub.id = mm_sub.mission_id
        WHERE mm_sub.id = {마지막_미션_ID}
      )
ORDER BY m.reward_points DESC, mm.created_at DESC
LIMIT 10;
</code></pre>
<ul>
<li>정렬 기준  <ul>
<li>1순위: <code>reward_points</code> (높을수록 우선)  </li>
<li>2순위: <code>created_at</code> (최신순)  </li>
</ul>
</li>
<li>cursor_value 생성  <ul>
<li><code>LPAD</code>로 자릿수 고정 → 문자열 정렬 안정성 확보  </li>
<li><code>UNIX_TIMESTAMP</code> 사용 → 문자열 연산보다 성능 우수</li>
</ul>
</li>
</ul>
<hr />
<h3 id="offset-vs-cursor-비교-요약">Offset vs Cursor 비교 요약</h3>
<table>
<thead>
<tr>
<th>항목</th>
<th>Offset 방식</th>
<th>Cursor 방식</th>
</tr>
</thead>
<tbody><tr>
<td>구현 난이도</td>
<td>쉬움</td>
<td>상대적으로 복잡</td>
</tr>
<tr>
<td>중복/누락 위험</td>
<td>있음</td>
<td>없음</td>
</tr>
<tr>
<td>성능 (대용량 기준)</td>
<td>느림 (OFFSET 증가)</td>
<td>빠름 (WHERE + cursor 필터)</td>
</tr>
<tr>
<td>정렬 기준 확장성</td>
<td>단일 기준에 적합</td>
<td>다중 기준도 안정적 처리 가능</td>
</tr>
<tr>
<td>인덱스 활용</td>
<td>어려움</td>
<td>조건 최적화 시 가능</td>
</tr>
</tbody></table>
<hr />
<h3 id="cursor-페이징에서의-성능-최적화">Cursor 페이징에서의 성능 최적화</h3>
<p>(1) <strong><code>DATE_FORMAT</code> → <code>UNIX_TIMESTAMP</code></strong></p>
<ul>
<li><code>DATE_FORMAT</code>은 사람이 읽기 쉬운 문자열 기반이지만, 계산과 비교 연산에서 느릴 수 있음.</li>
<li>반면 <code>UNIX_TIMESTAMP</code>는 <strong>숫자 기반 비교</strong>가 가능해 <strong>정렬과 비교 시 더 빠름</strong>.
→ 대용량 환경에서는 <code>UNIX_TIMESTAMP</code>가 더 실용적.</li>
</ul>
<p>(2) <strong><code>HAVING</code> → <code>WHERE</code></strong></p>
<ul>
<li><code>HAVING</code>은 원래 <code>GROUP BY</code> 이후에 집계 결과를 필터링할 때 사용하는 절로, <strong>인덱스를 타지 않음</strong>.</li>
<li><code>WHERE</code>는 쿼리 실행 초기에 동작하므로 <strong>인덱스 기반 필터링 가능 → 성능 우위</strong>.
→ 커서 비교는 반드시 <code>WHERE</code> 절로 처리해야 효율적.</li>
</ul>
<p>(3) <strong>문자열 기반 정렬 (CONCAT + LPAD)의 한계</strong></p>
<ul>
<li><p><code>CONCAT(LPAD(...))</code> 방식은 <strong>정렬 기준을 고정하는 데는 효과적</strong>이지만, 비교 연산은 결국 <strong>문자열 기반</strong>이라 성능이 100% 최적은 아님.</p>
</li>
<li><p>만약 클라이언트에 cursor 값을 전달할 필요가 없다면, 아래처럼 <strong>복합 정렬 필드 직접 비교</strong>가 더 빠를 수 있음.</p>
<pre><code class="language-sql">WHERE (m.reward_points, mm.created_at) &lt; (
SELECT m_sub.reward_points, mm_sub.created_at
...
)
ORDER BY m.reward_points DESC, mm.created_at DESC</code></pre>
</li>
</ul>
<p>(4) SELECT *는 모든 컬럼을 가져오며 I/O 비용을 증가의 원인이 된다.
특히 JOIN이 포함된 쿼리에서는 컬럼 수가 급격히 늘어나고, 중복 필드나 대용량 텍스트 컬럼까지 포함될 수 있어 성능 저하가 발생할 수 있다.
따라서 정확히 필요한 컬럼만 지정하여 가져와야 한다.</p>
<hr />
<h3 id="jpa-환경에서의-구현-방식">JPA 환경에서의 구현 방식</h3>
<p>→ Spring Data JPA의 PagingAndSortingRepository는  OFFSET 기반 페이징만 지원하므로, @Query를 사용해 Native Query로 Cursor 페이징을 구현</p>
<pre><code class="language-java">@Query(value = &quot;&quot;&quot;
  SELECT
    mm.mission_id,
    mm.completed,
    mm.updated_at,
    m.min_spend_money,
    m.reward_points,
    s.name AS store_name,
    CONCAT(
      LPAD(m.reward_points, 10, '0'),
      LPAD(UNIX_TIMESTAMP(mm.created_at), 10, '0')
    ) AS cursor_value
  FROM mission AS m
  JOIN member_mission AS mm ON m.id = mm.mission_id
  JOIN store AS s ON m.store_id = s.id
  WHERE mm.member_id = :memberId
    AND mm.completed = true
    AND CONCAT(
      LPAD(m.reward_points, 10, '0'),
      LPAD(UNIX_TIMESTAMP(mm.created_at), 10, '0')
    ) &lt; :cursorValue
  ORDER BY m.reward_points DESC, mm.created_at DESC
  LIMIT 10
&quot;&quot;&quot;, nativeQuery = true)
List&lt;MissionResponseDto&gt; findMissionsByCursor(
    @Param(&quot;memberId&quot;) Long memberId,
    @Param(&quot;cursorValue&quot;) String cursorValue
);
</code></pre>
<p>💬 왜 JPQL이 아닌 Native Query를 사용했는가?</p>
<ol>
<li><code>LPAD</code>, <code>UNIX_TIMESTAMP</code>, <code>CONCAT</code>은 MySQL 내장 함수이며, JPQL에서는 이를 지원하지 않음</li>
<li>문자열 정렬 기준(<code>cursor_value</code>)을 생성하고 비교하는 로직은 쿼리 구조가 복잡하기 때문에, Native Query가 가독성과 유지보수 측면에서 더 적합힘</li>
<li>WHERE 절 기반의 필터링 조건을 명확하게 작성할 수 있어, 인덱스를 활용한 성능 최적화가 가능해짐</li>
</ol>
<hr />
<h3 id="번외-왜-select절에서-정의한-cursor_value를-where절에서-다시-계산해야-할까-→-협업의-측면">번외) 왜 SELECT절에서 정의한 cursor_value를 WHERE절에서 다시 계산해야 할까? → 협업의 측면</h3>
<p>SELECT에서 정의된 <code>cursor_value</code>는 WHERE 절에서 직접 사용할 수 없다. 이는 SQL의 실행 순서상 <code>WHERE</code>이 <code>SELECT</code>보다 먼저 실행되기 때문
(반면, <code>HAVING</code> 절에서는 <code>SELECT</code> 결과를 사용할 수 있음)</p>
<pre><code class="language-sql">WHERE cursor_value &lt; {지금 보고 있는 게시물의 cursor_value}</code></pre>
<p>즉, <code>WHERE</code>에서는 <code>cursor_value</code>를 직접 사용할 수 없기 때문에 같은 계산식을 다시 풀어 써야 한다.
하지만 그럼에도 <code>SELECT</code> 절에 <code>cursor_value</code>를 정의하는 이유는, <strong>클라이언트에 해당 값을 전달하여 다음 페이지 요청 시 기준값으로 재사용하기 위함</strong>이다.</p>
<hr />
<h3 id="최종-요약">최종 요약</h3>
<ul>
<li>Offset은 단순하지만 대용량/정렬 기준 추가에 한계가 있음</li>
<li>Cursor는 클라이언트-서버 간 기준값 연동에 적합하고, WHERE 기반 필터링으로 성능 우수</li>
<li>정렬 기준이 명확하다면 CONCAT보다는 튜플 비교가 더 빠름 (단, 응답값 전달 필요 시는 CONCAT 유지)</li>
<li><code>SELECT *</code> 사용 지양, 복합 인덱스와 필요한 컬럼만 지정 필요<pre><code>                       ---</code></pre></li>
</ul>
<h3 id="references">References</h3>
<ul>
<li><a href="https://toss.tech/pagination">토스 기술 블로그 - Cursor 기반 페이징 도입기</a>  </li>
<li><a href="https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_date-format">공식 MySQL DATE_FORMAT 문서</a></li>
</ul>