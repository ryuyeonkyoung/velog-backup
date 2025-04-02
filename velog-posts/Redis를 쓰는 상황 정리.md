<p>UMC 동아리에서 스터디를 하던 중, 이런 질문을 받았다.  </p>
<blockquote>
<p>&quot;회원가입이나 로그인 관련 정보를 RDBMS에 저장하는게 좋을까요 Redis에 저장하는게 좋을까요?&quot;</p>
</blockquote>
<p>나는 <strong>Redis가 빠른 조회가 가능한 NoSQL이라는 것만 알고 있었지</strong>, 정확히 어떤 상황에서 사용하는지는 몰라서 아무 말도 하지 못했다. </p>
<p>다른 팀원분들은 이미 한 기수 이상 활동한 시니어 분들이셔서 그런지, 의견을 거침없이 말하셨는데, 그래서 나는 무슨 말인지 잘 이해하지 못한 채 고개만 끄덕이고 있었다.</p>
<p>그래서 집에 돌아와 Redis를 처음부터 다시 정리해보기로 했다.
<strong>Redis는 어떤 역할을 하는지, 어떤 상황에서 꼭 필요한지</strong> 찾아보고 정리한 내용을 아래에 기록해 둔다.</p>
<h3 id="redis란">Redis란?</h3>
<p>Redis는 String, Set, Sorted Set, Hash, List 등의 다양한 타입을 빠르게 저장하고 조회할 수 있는 NOSQL이다. 일반적인 RDBMS(예: MySQL)과 달리, 데이터를 디스크가 아닌 메모리(RAM)에 저장하므로 읽기/쓰기 속도가 매우 빠르다.</p>
<h3 id="redis를-사용해야-하는-판단기준">Redis를 사용해야 하는 판단기준</h3>
<ol>
<li><p>자주 읽고 쓰는 데이터인가?<br />ex) 캐싱</p>
</li>
<li><p>DB에 굳이 안 넣어도 되는 일시적인 데이터인가?<br />ex) 인증 코드, 세션 등</p>
</li>
<li><p>빠르게 접근해야 하거나 TTL(Time To Livd, 인증만료)이 필요한가?  </p>
</li>
</ol>
<h3 id="실무에서-무조건-redis만-쓰는-경우">실무에서 무조건 Redis만 쓰는 경우</h3>
<p>→ 세션, 인증코드, 로그인 제한, (실시간)채팅/알림, 캐시</p>
<table>
<thead>
<tr>
<th>사용 케이스</th>
<th>이유</th>
</tr>
</thead>
<tbody><tr>
<td>🔑 세션 저장 (Spring Session 등)</td>
<td>빠른 접근 + TTL로 자동 만료 관리</td>
</tr>
<tr>
<td>🔑 이메일 인증코드 / 비밀번호 찾기 토큰</td>
<td>짧은 시간 동안만 유효하므로 DB 저장 불필요</td>
</tr>
<tr>
<td>⛔ 로그인 시도 제한 (IP 차단)</td>
<td>TTL 설정 + 빠른 누적 카운트 처리</td>
</tr>
<tr>
<td>📢 실시간 랭킹 / 트렌드</td>
<td>정렬과 업데이트가 빠름 (Sorted Set 구조 활용)</td>
</tr>
<tr>
<td>📩 채팅 / 알림 (pub/sub)</td>
<td>실시간 메시지 처리에 적합</td>
</tr>
<tr>
<td>📦 캐시 (게시글, 설정값 등)</td>
<td>자주 조회되지만 자주 바뀌지 않는 값 캐싱</td>
</tr>
</tbody></table>
<h3 id="redis-사용이-적절한-경우">Redis 사용이 적절한 경우</h3>
<table>
<thead>
<tr>
<th>기능</th>
<th>Redis 사용 이유</th>
<th>사용 예시 / 상세 설명</th>
</tr>
</thead>
<tbody><tr>
<td>로그인 (세션 관리)</td>
<td>빠른 속도 + 만료 기능</td>
<td>세션 기반 로그인에서 TTL 설정 가능</td>
</tr>
<tr>
<td>회원가입 시 이메일 인증</td>
<td>일시적 정보 + TTL</td>
<td>인증코드는 몇 분만 유효하면 되므로 DB 저장 불필요</td>
</tr>
<tr>
<td>비밀번호 찾기/변경 링크</td>
<td>일시적 토큰 저장</td>
<td>몇 분~몇 시간만 유효하므로 Redis 적합</td>
</tr>
<tr>
<td>중복 체크 (아이디, 닉네임)</td>
<td>자주 조회됨</td>
<td>인기 닉네임은 Redis로 캐싱</td>
</tr>
<tr>
<td>접속자 수 집계</td>
<td>빠른 쓰기 + TTL 초기화</td>
<td>Redis에 누적 후 주기적으로 DB 반영</td>
</tr>
<tr>
<td>게시글/상품 조회수</td>
<td>빠른 증가 처리</td>
<td>실시간 증가 후 주기적으로 DB 저장</td>
</tr>
<tr>
<td>좋아요/찜 기능</td>
<td>중복 체크 + 빠름</td>
<td>Set 구조로 이미 좋아요 여부 확인</td>
</tr>
<tr>
<td>공지사항 or 설정값</td>
<td>캐싱</td>
<td>자주 조회되지만 자주 바뀌지 않는 데이터</td>
</tr>
<tr>
<td>로그인 시도 제한</td>
<td>일정 횟수 초과 차단</td>
<td>횟수 저장 + 제한 시간 설정</td>
</tr>
<tr>
<td>API rate limiting</td>
<td>빠른 카운팅</td>
<td>초당/분당 요청 수 제한</td>
</tr>
<tr>
<td>실시간 랭킹, 추천 리스트</td>
<td>정렬 + 빠른 읽기</td>
<td>Sorted Set으로 인기 리스트 구현</td>
</tr>
<tr>
<td>채팅, 실시간 알림</td>
<td>빠른 처리 + pub/sub</td>
<td>실시간 메시지 전송</td>
</tr>
</tbody></table>
<h3 id="반대로-redis-사용을-권장하지-않는-경우">반대로 Redis 사용을 권장하지 않는 경우</h3>
<table>
<thead>
<tr>
<th>상황</th>
<th>이유</th>
</tr>
</thead>
<tbody><tr>
<td>중요한 유저 정보 저장</td>
<td>메모리 기반이라 데이터 유실 가능성 있음</td>
</tr>
<tr>
<td>영구적인 데이터 저장</td>
<td>휘발성 구조라 RDB 병행 필요</td>
</tr>
</tbody></table>
<h3 id="references">References</h3>
<ul>
<li><a href="https://ittrue.tistory.com/317?utm_source=chatgpt.com">[Redis] 레디스란 무엇인가? - 특징, 장단점, 사용 사례</a> </li>
<li>[자료구조별 명령어 모음 - [REDIS] 📚 자료구조 명령어 종류 &amp; 활용 사례 💯 총정리]
(<a href="https://inpa.tistory.com/entry/REDIS-%F0%9F%93%9A-%EB%8D%B0%EC%9D%B4%ED%84%B0-%ED%83%80%EC%9E%85Collection-%EC%A2%85%EB%A5%98-%EC%A0%95%EB%A6%AC?utm_source=chatgpt.com">https://inpa.tistory.com/entry/REDIS-%F0%9F%93%9A-%EB%8D%B0%EC%9D%B4%ED%84%B0-%ED%83%80%EC%9E%85Collection-%EC%A2%85%EB%A5%98-%EC%A0%95%EB%A6%AC?utm_source=chatgpt.com</a>)</li>
</ul>