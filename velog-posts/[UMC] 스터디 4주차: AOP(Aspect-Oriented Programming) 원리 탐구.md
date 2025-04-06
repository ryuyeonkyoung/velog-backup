<h3 id="aop란-무엇인가-왜-oop로-끝내지-않는걸까">AOP란 무엇인가? 왜 OOP로 끝내지 않는걸까?</h3>
<p>객체지향(OOP)은 클래스 단위로 책임을 나누는 구조지만,
실무에서는 여러 클래스에 <strong>공통으로 필요한 기능(트랜잭션, 로깅, 인증 등)이 반복되며 코드가 지저분해지는 문제</strong>가 있다.</p>
<p><strong>AOP(Aspect-Oriented Programming)</strong>는 <strong>공통 기능을 별도의 모듈로 분리해
핵심 비즈니스 로직과 독립적으로 관리할 수 있게 하는 구조 설계 기법</strong>이다.</p>
<blockquote>
<p>결과적으로 <strong>코드의 응집도는 높이고, 중복은 줄이며, 유지보수성을 향상</strong>시킬 수 있다.</p>
</blockquote>
<hr />
<h3 id="oop와-aop의-차이">OOP와 AOP의 차이</h3>
<table>
<thead>
<tr>
<th>항목</th>
<th>OOP (객체지향 프로그래밍)</th>
<th>AOP (관점지향 프로그래밍)</th>
</tr>
</thead>
<tbody><tr>
<td>모듈 단위</td>
<td>클래스 중심</td>
<td><strong>관심사(Aspect)</strong> 중심</td>
</tr>
<tr>
<td>주요 목적</td>
<td>책임 분리, 재사용성</td>
<td><strong>횡단 관심사(Cross-Cutting Concern)</strong> 분리</td>
</tr>
<tr>
<td>중복 제거 대상</td>
<td>도메인 기능 중심</td>
<td><strong>공통 로직(로깅, 트랜잭션)</strong></td>
</tr>
<tr>
<td>예시</td>
<td>회원가입 기능</td>
<td>가입 시 로깅, 트랜잭션 등 <strong>공통 처리</strong></td>
</tr>
</tbody></table>
<hr />
<h3 id="언제-aop를-써야-할까">언제 AOP를 써야 할까?</h3>
<p>앞서 설명했듯이 <strong>비즈니스 로직과 분리되는 공통 관심사</strong>들을 AOP로 추출한다. </p>
<ul>
<li>트랜잭션 처리 (<code>@Transactional</code>)</li>
<li>로깅 (메서드 실행 시간 측정 등)</li>
<li>인증/인가 검증 (필터보다 세밀한 처리 가능)</li>
<li>예외 처리 및 공통 응답 포맷 처리</li>
<li>리소스 해제 등 공통 사후 처리</li>
</ul>
<p>→ 핵심 로직에 영향을 주지 않는 <strong>부가 기능</strong>이면서, <strong>여러 계층(Service, Controller 등)에 중복되기 쉬운 경우</strong>가 AOP 대상이다.</p>
<hr />
<h3 id="aop-핵심-구성-요소">AOP 핵심 구성 요소</h3>
<p><strong>Advice</strong>
실행 시점과 공통 로직 정의 (예: <code>@Around</code>, <code>@Before</code>)</p>
<p><strong>JoinPoint</strong>
Advice가 적용될 수 있는 실행 지점 (ex. 메서드 호출)</p>
<p><strong>Pointcut</strong>
어떤 JoinPoint에 Advice를 적용할지 조건 지정 (ex: <code>execution(* com.service.*(..))</code>)</p>
<p><strong>Aspect</strong>
Pointcut + Advice를 하나의 모듈로 묶은 클래스 (<code>@Aspect</code>)</p>
<p><strong>Weaving</strong>
Aspect를 실제 코드에 적용하는 과정</p>
<hr />
<h3 id="spring-aop는-어떻게-동작할까">Spring AOP는 어떻게 동작할까?</h3>
<p>→ Spring AOP는 <strong>런타임 프록시 기반 AOP</strong>를 사용하며, 내부적으로 <strong>프록시 패턴(Proxy Pattern)</strong>을 따른다.</p>
<ol>
<li>AOP가 적용된 Bean은 <strong>프록시 객체</strong>로 감싸진다.
→ 이 때 프록시 객체로 감싸는 이유: 일반 객체는 매서드 호출을 중간데 가로챌 수 없지만, 프록시 객체는 가능하기 때문에</li>
<li>외부에서 메서드 호출 시, <strong>프록시가 먼저 실행되며 Advice를 수행</strong>한다.</li>
<li>이후 실제 비즈니스 로직 메서드를 호출한다.</li>
</ol>
<blockquote>
<p>⚠ 주의: <strong>자기 자신의 내부 메서드 호출에는 AOP가 적용되지 않는다.</strong><br />(프록시 객체를 우회하게 되기 때문 → 해결: 구조를 분리하거나 외부로 분리)</p>
</blockquote>
<pre><code class="language-java">@Around(&quot;execution(* com.example.service.*.*(..))&quot;)
public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
    long start = System.currentTimeMillis();
    Object result = joinPoint.proceed(); // 실제 메서드 실행
    long time = System.currentTimeMillis() - start;
    System.out.println(&quot;실행 시간: &quot; + time + &quot;ms&quot;);
    return result;
}</code></pre>
<hr />
<h3 id="런타임-위빙-vs-컴파일-타임-위빙">런타임 위빙 vs 컴파일 타임 위빙</h3>
<table>
<thead>
<tr>
<th>항목</th>
<th>런타임 위빙 (Spring AOP)</th>
<th>컴파일 타임 위빙 (AspectJ)</th>
</tr>
</thead>
<tbody><tr>
<td>시점</td>
<td>애플리케이션 실행 시</td>
<td>컴파일 시 바이트코드 조작</td>
</tr>
<tr>
<td>방식</td>
<td><strong>프록시 객체 생성</strong></td>
<td>바이트코드 직접 수정</td>
</tr>
<tr>
<td>장점</td>
<td>설정 간단, DI와 통합 쉬움</td>
<td>성능 최적화 가능</td>
</tr>
<tr>
<td>단점</td>
<td>내부 호출 적용 불가, final 메서드 제외</td>
<td>빌드/디버깅 복잡</td>
</tr>
<tr>
<td>실무 사용</td>
<td>✅ Spring AOP 사용 일반적</td>
<td>❌ 성능 민감한 경우만 사용</td>
</tr>
</tbody></table>
<hr />
<h3 id="자주-하는-실수-및-주의점">자주 하는 실수 및 주의점</h3>
<ul>
<li>AOP는 <strong>핵심 로직이 아닌 공통 부가 로직</strong>만 다뤄야 한다.
→ 로깅, 트랜잭션, 인증 등 도메인과 무관한 기능에만 적용해야 함</li>
<li>Pointcut 범위가 너무 넓으면, 의도하지 않은 클래스에 Advice가 적용돼<br /><strong>사이드 이펙트</strong>(예: 트랜잭션 예외, 불필요한 로그)가 발생할 수 있다.</li>
<li><strong>내부 메서드 호출이나 <code>final</code> 메서드/클래스에는 AOP가 적용되지 않는다.</strong> 
→ 이유: Spring AOP는 프록시 기반이기 때문에 프록시를 우회하거나 상속 불가한 구조에는 적용되지 않음</li>
<li>Advice 내부에 비즈니스 로직을 담으면 <strong>관심사 분리가 깨지고 테스트도 어려워지며 디버깅 비용이 증가</strong>한다.</li>
</ul>
<hr />
<h3 id="transactional">@Transactional</h3>
<p>Spring에서 <code>@Transactional</code>은 대표적인 AOP 기반 기능이다.<br />내부적으로 <code>@Transactional</code>도 AOP 프록시를 통해 트랜잭션 시작/커밋/롤백을 처리한다.</p>
<p>따라서 트랜잭션이 동작하지 않는다면, 내부 호출이 아닌지, 프록시 적용 여부를 반드시 확인해야 한다.</p>
<h4 id="적용-예시">적용 예시</h4>
<pre><code class="language-java">// 정상 적용 예시
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @Transactional
    public void placeOrder() {
        saveOrder();
        paymentService.pay(); // 다른 Bean이 호출 → 트랜잭션 적용됨
    }

    private void saveOrder() {
        // 트랜잭션 범위 내에서 실행됨
    }
}</code></pre>
<pre><code class="language-java">// ❌ 동작하지 않는 예시 (내부 호출)
@Service
public class MemberService {

    @Transactional
    public void updateMember() {
        this.logActivity(); // 내부 메서드 호출 → 프록시 우회 → 트랜잭션 미적용
    }

    @Transactional
    public void logActivity() {
        // 적용되지 않음
    }
}
</code></pre>
<hr />
<h3 id="정리-aop-적용-판단-기준">정리: AOP 적용 판단 기준</h3>
<table>
<thead>
<tr>
<th>항목</th>
<th>체크 포인트</th>
</tr>
</thead>
<tbody><tr>
<td>공통 관심사인가?</td>
<td>여러 레이어에서 반복되는가</td>
</tr>
<tr>
<td>핵심 로직과 분리 가능한가?</td>
<td>AOP 적용이 비즈니스 의미를 흐리지 않는가</td>
</tr>
<tr>
<td>설정 유연성</td>
<td>Pointcut 조건으로 적용 범위를 잘 제어할 수 있는가</td>
</tr>
<tr>
<td>성능 영향도</td>
<td>Advice에서 무거운 작업을 수행하는가</td>
</tr>
</tbody></table>
<hr />
<h3 id="references">References</h3>
<ul>
<li><a href="https://docs.spring.io/spring-framework/reference/core/aop.html">Spring 공식 문서: Aspect Oriented Programming with Spring</a></li>
</ul>