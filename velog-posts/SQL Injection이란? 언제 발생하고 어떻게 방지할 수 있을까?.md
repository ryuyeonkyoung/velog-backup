<p>웹 개발을 하다 보면 꼭 알아야 할 보안 취약점 중 하나가 <strong>SQL Injection(SQL 인젝션)</strong>이다.
이 글에서는 SQL Injection이 무엇인지, 어떤 상황에서 발생하는지, 그리고 어떻게 방지할 수 있는지를 정리해본다.</p>
<h2 id="1-sql-injection이란">1. SQL Injection이란?</h2>
<blockquote>
<p>사용자가 입력한 값이 SQL 쿼리에 그대로 삽입되어, 의도하지 않은 SQL 문이 실행되는 보안 취약점</p>
</blockquote>
<p>공격자는 SQL Injection을 통해 아래와 같은 악의적인 작업이 가능하다.</p>
<ul>
<li>데이터 조회 (권한 없는 정보 노출)</li>
<li>데이터 조작, 삭제</li>
<li>관리자 권한 탈취</li>
<li>전체 테이블 조회 등</li>
</ul>
<h2 id="2-언제-발생할까">2. 언제 발생할까?</h2>
<p><strong>조건</strong></p>
<ol>
<li>SQL 쿼리를 문자열로 직접 조합하고,</li>
<li>사용자 입력값을 그대로 삽입했을 때</li>
</ol>
<p><strong>공격 받는 과정</strong></p>
<pre><code>String query = &quot;SELECT * FROM users WHERE username = 'yeon' AND password = '1234'&quot;;</code></pre><p>다음은 아이디와 비밀번호를 입력하여 로그인할 때 사용되는 쿼리이다.
이때 만약 사용자가 정상적인 id 대신 &quot;admin' --&quot;을 입력한다면?</p>
<pre><code>SELECT * FROM users WHERE username = 'yeon' --' AND password = '1234'</code></pre><p>→ -- 이후는 SQL 주석이 되므로, 비밀번호 조건을 무시된다. 따라서 아이디만 알고 있으면 해당 계정에 접속 가능하게 된다.
또 나의 정보만 조회하는 쿼리문에서, &quot;' OR 1=1 --&quot;를 삽입하여 모든 사용자의 정보를 가져오는 쿼리문으로 바꿀 수도 있다.</p>
<h2 id="3-어떻게-막을-수-있을까">3. 어떻게 막을 수 있을까?</h2>
<h3 id="1-jdbc-바인딩">1) JDBC (바인딩)</h3>
<p>JDBC에서는 <strong>바인딩 처리</strong>를 통해 이를 예방할 수 있다.</p>
<pre><code>String query = &quot;SELECT * FROM users WHERE username = ?&quot;; // 바인딩
PreparedStatement stmt = conn.prepareStatement(query);
stmt.setString(1, username);</code></pre><p>? 위치에 사용자의 입력값이 바인딩 처리되어 SQL 문과 분리되기 때문에, 입력값은 명령어로 인식되지 않고 안전하게 처리된다.</p>
<h3 id="2-orm-preparedstatement">2) ORM (PreparedStatement)</h3>
<p>JPA, Hibernate, Querydsl 등의 대부분의 ORM 도구는 내부적으로 PreparedStatement를 사용한다. 따라서 기본적으로 SQL Injection에 대비되어 있다. 이들은 직접 쿼리를 문자열로 조합하지 않기 때문에 Injection에 강하다.</p>
<h3 id="3-mybatis-">3) MyBatis (#{})</h3>
<p>MyBatis도 #{}를 사용하면 PreparedStatement처럼 작동하여 SQL Injection을 방지할 수 있다.</p>
<pre><code>SELECT * FROM users WHERE username = #{username} -- 안전 O</code></pre><p>반면, ${}를 사용하면 문자열 치환이 가능해져 위험할 수 있다.</p>
<pre><code>SELECT * FROM ${columnName}  -- 위험 ❌</code></pre><h3 id="4-입력값-유효성-검사---보조적">4) 입력값 유효성 검사 - 보조적</h3>
<p>위처럼 PreparedStatement로 막는 것이 근본적인 해결책이지만,
입력값에 대한 유효성 검사도 보조적으로 반드시 수행해야 한다.</p>
<p><strong>방법</strong></p>
<ul>
<li>이메일, 아이디, 숫자 등은 형식에 맞는지 미리 검증</li>
<li>SQL 예약어나 특수문자(', ;, -- 등)가 포함되어 있는지 필터링</li>
</ul>
<p><em>예시코드</em></p>
<pre><code>public class InputValidator {

    // 이메일 유효성 검사
    public static boolean isValidEmail(String email) {
        return email != null &amp;&amp; email.matches(&quot;^[\\w.-]+@[\\w.-]+\\.[a-zA-Z]{2,6}$&quot;);
    }

    // 아이디 유효성 검사: 영문자+숫자 조합, 4~20자
    public static boolean isValidUsername(String username) {
        return username != null &amp;&amp; username.matches(&quot;^[a-zA-Z0-9]{4,20}$&quot;);
    }
}
</code></pre><pre><code>public class SqlInjectionFilter {

    // SQL Injection에 자주 사용되는 문자 또는 키워드
    private static final String[] DANGEROUS_PATTERNS = {
        &quot;'&quot;, &quot;\&quot;&quot;, &quot;--&quot;, &quot;;&quot;, &quot;/*&quot;, &quot;*/&quot;, &quot;xp_&quot;, &quot; or &quot;, &quot; and &quot;, &quot;=&quot;
    };

    // 입력값에 위험한 패턴이 포함되어 있는지 검사
    public static boolean containsSqlInjectionRisk(String input) {
        if (input == null) return false;

        String lowerInput = input.toLowerCase();
        for (String pattern : DANGEROUS_PATTERNS) {
            if (lowerInput.contains(pattern)) {
                return true;
            }
        }
        return false;
    }
}
</code></pre><h2 id="4-jpa-더-자세히-살펴보기">4. JPA 더 자세히 살펴보기</h2>
<p>JPA는 내부적으로 SQL을 자동 생성하고, 쿼리 실행 시 PreparedStatement 방식을 사용하므로 기본적으로 안전하다.</p>
<p><strong>예시 1 (단순 조회)</strong>
이 코드에서는 findByUsername(&quot;admin' OR '1'='1&quot;) 같은 입력을 넣어도,
JPA는 이를 문자열 그대로 바인딩하여 Injection 공격이 통하지 않는다.</p>
<pre><code>public interface UserRepository extends JpaRepository&lt;User, Long&gt; {
    Optional&lt;User&gt; findByUsername(String username);
}</code></pre><p><strong>예시2 (@Query 사용 - 주의)</strong></p>
<p>❗ 하지만 아래처럼 문자열을 직접 조합하면 위험할 수 있다.
다음 예시는 2에서 간략하게 언급한 예시이다.</p>
<pre><code>@Query(&quot;SELECT u FROM User u WHERE u.username = '&quot; + username + &quot;'&quot;) // ❌ 절대 금지</code></pre><p>만약 사용자가 username = &quot;' OR '1'='1&quot;을 입력하면, 쿼리는 다음과 같이 변형된다.</p>
<pre><code>SELECT u FROM User u WHERE u.username = '' OR '1'='1'</code></pre><p>→ 항상 참(1=1)이 되는 조건으로, 모든 사용자의 정보를 가져오거나 로그인 우회가 가능하다.</p>
<h2 id="6-spring-boot--querydsl-예제">6. Spring Boot + Querydsl 예제</h2>
<p>Querydsl은 동적 쿼리를 제공하며, 이를 통해 SQL Injection을 예방할 수 있다.</p>
<p><strong>예시 (Querydsl에서의 동적 쿼리 사용)</strong></p>
<pre><code>public List&lt;User&gt; searchUsers(String username, String email) {
    QUser user = QUser.user;

    return queryFactory.selectFrom(user)
        .where(
            username != null ? user.username.eq(username) : null,
            email != null ? user.email.eq(email) : null
        )
        .fetch();
}</code></pre><p>user.username.eq(username)는 내부적으로 PreparedStatement를 사용한다. 따라서 문자열이 직접 SQL로 삽입되지 않고, 이를 통해 Injection을 방지할 수 있다.</p>
<p><strong>Querydsl에서 주의할 점</strong>
Querydsl을 사용할 때도, 아래와 같이 직접 문자열을 조합하거나 JPQL을 직접 실행하면 주의해야 한다.</p>
<pre><code>String rawQuery = &quot;SELECT * FROM user WHERE name = '&quot; + input + &quot;'&quot;;</code></pre><p><del>그런데 이럴 거면 굳이 Querydsl을 쓸 필요가 없다</del></p>
<h2 id="정리">정리</h2>
<table>
<thead>
<tr>
<th>기술</th>
<th>SQL Injection 방지 방법</th>
</tr>
</thead>
<tbody><tr>
<td>JDBC</td>
<td><code>PreparedStatement</code>로 바인딩 처리</td>
</tr>
<tr>
<td>JPA</td>
<td><code>@Query(:param)</code> 또는 메서드 쿼리 사용</td>
</tr>
<tr>
<td>MyBatis</td>
<td><code>#{}</code> 바인딩 필수, <code>${}</code> 사용 지양</td>
</tr>
<tr>
<td>Querydsl</td>
<td><code>.eq()</code>, <code>.lt()</code> 등 메서드 바인딩</td>
</tr>
<tr>
<td>보조</td>
<td>유효성 검사 및 입력값 필터링</td>
</tr>
</tbody></table>
<h2 id="references">References</h2>
<ul>
<li><p><a href="https://mybatis.org/mybatis-3/sqlmap-xml.html#Dynamic_SQL">MyBatis 공식 문서 - 동적 SQL 처리</a></p>
</li>
<li><p><a href="https://owasp.org/www-community/attacks/SQL_Injection">OWASP SQL Injection 설명</a></p>
</li>
<li><p><a href="https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html">OWASP Cheat Sheet - SQL Injection Prevention</a></p>
</li>
<li><p>['SQL 인젝션(SQL Injection)' 웹 취약점 점검 및 조치 방법]
(<a href="https://blog.naver.com/softwidesec/223490809621">https://blog.naver.com/softwidesec/223490809621</a>)</p>
</li>
</ul>