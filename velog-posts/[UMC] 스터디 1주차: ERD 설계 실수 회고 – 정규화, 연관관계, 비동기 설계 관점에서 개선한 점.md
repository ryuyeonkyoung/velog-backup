<p><strong>ERD - 초안</strong>
<img alt="ERD 초안" src="https://velog.velcdn.com/images/rykjjang/post/9afd2261-8eba-4284-aef9-fe3cc7c21694/image.png" /></p>
<p><strong>ERD - 피드백 후</strong>
<img alt="ERD 수정 후" src="https://velog.velcdn.com/images/rykjjang/post/6c44d544-199b-48af-9644-2d257dc9c4cf/image.png" /></p>
<h3 id="1-실습-개요">1. 실습 개요</h3>
<p>미션 자료의 공개는 불가하지만, 완성본과 함께 ERD 설계 중 발생한 실수와 그에 대한 <strong>구조적 개선 방식</strong>을 정리했다.
정규화 부족, 과도한 연관관계 설정, 동기 처리 방식 등에서 실무적 관점이 부족했던 점을 돌아보며 <strong>데이터 정합성과 설계 유지보수성</strong>을 개선하는 방향으로 수정했다.</p>
<hr />
<h3 id="2-적용-개념-요약-키워드-중심">2. 적용 개념 요약 (키워드 중심)</h3>
<ul>
<li>정규화 (1NF, 2NF, 3NF: 이행적 종속 제거)</li>
<li>다대다 테이블 분리 (중간 테이블 사용)</li>
<li>지역 정보 정합성 vs 실시간 조회 성능</li>
<li>알람 데이터의 비동기 처리 구조 (메시지 큐 + 캐시)</li>
<li>일관된 네이밍 규칙 적용</li>
<li>약관 및 요구사항 누락 방지</li>
</ul>
<hr />
<h3 id="3-주요-변경-사항">3. 주요 변경 사항</h3>
<h4 id="💡1-favorite_category-→-member_prefer--food_category로-정규화">💡1) favorite_category → member_prefer + food_category로 정규화</h4>
<ul>
<li>숫자로 카테고리를 저장하면 <strong>유지보수성과 가독성 모두 떨어짐</strong>  </li>
<li><code>member</code> 테이블에서 <code>category_num</code>은 이행적 종속 → 제3정규형(3NF) 위반  <ul>
<li><code>user_id</code> → <code>category_num</code>, </li>
<li><code>category_num</code> → 실제 카테고리 의미(예: &quot;한식&quot;))</li>
</ul>
</li>
<li><strong>중간 테이블(member_prefer)</strong> + <strong>카테고리 마스터 테이블(food_category)</strong> 구조로 개선</li>
</ul>
<h4 id="💡-2-region-연관관계-최소화">💡 2) region 연관관계 최소화</h4>
<ul>
<li>모든 테이블에 <code>region_id</code>를 연결하면 <strong>데이터 정합성이 무너질 수 있음</strong> <ul>
<li><code>user</code>와 <code>order</code>가 각각 다른 <code>region_id</code>를 가질 경우 데이터 충돌 발생 가능</li>
</ul>
</li>
<li>실무에서는 JOIN 최적화 &lt; <strong>정합성 보장</strong></li>
<li><code>store</code> 테이블에만 <code>region</code> 연결하고, 그 외는 간접적으로 참조하도록 설계</li>
</ul>
<h4 id="💡-3-필드-관련-리팩토링">💡 3) 필드 관련 리팩토링</h4>
<p>① 상태 관련 필드 통일 및 enum 적용
is_completed, is_open처럼 의미가 겹치는 boolean 필드를 status 하나로 통일
boolean → enum으로 자료형 변경
→ 2주차 과제에서 쿼리 작성 중 completed의 의미를 오해해 진행 상태를 반대로 구현했던 경험이 계기
→ 이후 status 필드로 변경하고 enum을 적용하니 &quot;진행중&quot;처럼 상태를 직관적으로 파악할 수 있어 쿼리 작성이 수월해졌음
→ 1주차 피드백에서도 이 부분이 지적되어 구조 개선 필요성을 인지했음</p>
<p>② region 테이블의 address 컬럼 중복 제거
store나 user 등의 테이블에 이미 주소가 존재하는 경우
region 테이블에 별도로 address를 둘 필요가 없어 중복 컬럼 제거</p>
<p>③ 소셜 로그인 고려: social_type 필드 추가
추후 소셜 로그인 기능을 대비해 member 테이블에 social_type 필드 추가
로그인 방식(GOOGLE, NAVER 등)을 저장함으로써 로그인 흐름 확장 가능</p>
<ol>
<li>상태 관련 필드들을 <code>status</code>로 통일 (<code>is_completed</code>, <code>is_open</code>) 및 enum으로 자료구조 변경
2주차 과제에서  쿼리문을 만들다 보니 completed의 의미가 헷갈려서 진행중과 진행 완료인 쿼리를 반대로 만들었었다.(<del>나만 헷갈린 거일수도 있지만...</del>) 마침 1주차 피드백에서도 completed의 자료형에 대한 부분을 들어서, 기존의 boolen 자료형에서 enum 자료형으로 바꾸고 필드명을 status로 바꿨다. 다시 쿼리를 작성해보니 '진행중'으로 명확하게 보여서 의미를 파악하기 편했다.</li>
<li><code>region</code> 테이블에서 <code>address</code> 컬럼 중복 → 삭제</li>
<li>소셜 로그인을 고려하여 <code>member</code> 테이블에 <code>social_type</code> 컬럼 추가</li>
</ol>
<h4 id="💡-4-알람-데이터는-비동기-처리-고려">💡 4) 알람 데이터는 비동기 처리 고려</h4>
<ul>
<li>알람을 RDB에 1:N 관계로 계속 저장하면, 데이터가 급격히 누적되며 성능에 부담을 줄 수 있다.
→ 메시지 큐(MQ)로 비동기 처리하고, 자주 조회되는 데이터는 Redis 캐시로 분리해 읽기 효율을 높이는 구조로 설계할 수 있다.</li>
<li>중요 알림만 최소한으로 alarm_log 테이블에 저장했다.</li>
</ul>
<h4 id="💡-5-네이밍-규칙-통일">💡 5) 네이밍 규칙 통일</h4>
<ul>
<li>테이블은 단수형</li>
<li>테이블의 사용처가 한정되어있다면 명시하기 (<code>photo</code> → <code>review_image</code>)</li>
<li>컬럼은 <code>is_completed</code> 대신 <code>completed</code>처럼 명확한 의미로</li>
<li>JPA에서 user entity를 만들 때는 조심하기! (이미 있던 기능하고 이름이 겹침)</li>
</ul>
<h4 id="💡-6-약관-동의-테이블-누락-보완">💡 6) 약관 동의 테이블 누락 보완</h4>
<ul>
<li>요구사항 분석 단계에서 약관 관리 기능이 빠졌다.</li>
<li><strong>계약성 데이터가 필요</strong>한 이유 : 사용자 서비스 운영에 <strong>법적 책임</strong>이 생길 수 있다.</li>
</ul>
<hr />
<h3 id="4-느낀-점--회고">4. 느낀 점 / 회고</h3>
<p>ERD 설계는 <strong>요구사항 분석 → 정규화 → 연관관계 최적화 → 실무적 구조 판단</strong>까지 종합적인 사고가 필요한 작업이라는 걸 체감했다.</p>
<p>특히 데이터 정합성, 비동기 처리, 실시간 조회 성능 등은 실무에서 <strong>의사결정이 필요한 지점</strong>이라 단순한 교과서적 설계만으로는 부족하다는 것도 느꼈다.</p>
<p>예를 들어, 해당 글에는 담지 않았지만 <strong>설계 도중 region 테이블을 아예 삭제했던 적이 있다.</strong>
부 API를 사용해 세부 주소를 지역 이름으로 변환할 수 있다는 점을 발견하고,
이 기능을 활용하면 region 테이블을 없애고 JOIN 횟수를 줄여 성능을 개선할 수 있다고 생각해서였다.</p>
<p>하지만 이후 해당 API가 일정 사용량 이 넘으면 비용을 청구한다는 점을 알게 되었고, 이로 인해 다시 region 테이블을 원복하게 되었다.</p>
<p>이처럼 여러 방면에서 서비스 운영을 염두에 둔 판단이 필요하다는 점을 배울 수 있었다.</p>