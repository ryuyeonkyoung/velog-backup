<h1 id="📌-트러블슈팅-mysql-오류-해결-후-gradle-프로젝트-모듈-오류-발생">📌 [트러블슈팅] MySQL 오류 해결 후, Gradle 프로젝트 모듈 오류 발생</h1>
<p>이전 오류를 해결한 후, 프로젝트를 실행했더니 예상치 못한 모듈 오류가 발생했다.
패키지 import가 전부 오류가 나서 처음에는 데이터베이스 문제를 의심했지만, 다른 Spring Boot 프로젝트는 정상적으로 실행됨.
즉, DB 문제가 아니라 프로젝트 설정 문제일 가능성이 높아 하나씩 점검하며 해결 과정을 정리했다.</p>
<h2 id="🛠-오류-상황">🛠 오류 상황</h2>
<h3 id="1️⃣-패키지-import-오류-발생">1️⃣ 패키지 import 오류 발생</h3>
<p>MySQL 관련 오류를 해결한 후 프로젝트를 실행했는데, import한 모든 패키지가 오류 발생
다른 Spring Boot 프로젝트는 정상적으로 실행됨 → DB 문제는 아님
프로젝트 설정 문제로 추정</p>
<h2 id="🚀-해결-과정">🚀 해결 과정</h2>
<h3 id="1️⃣-파일---캐시-무효화-invalidate-caches--restart">1️⃣ 파일 - 캐시 무효화 (Invalidate Caches &amp; Restart)</h3>
<p>IDE 캐시 문제일 가능성이 있어 무효화 후 재시작
하지만 여전히 동일한 오류 발생</p>
<h3 id="2️⃣-gradle-clean--build-실행">2️⃣ gradle clean &amp; build 실행</h3>
<pre><code>gradle clean
gradle build</code></pre><p>실행 후, 모듈 관련 오류 메시지가 나타남
저번에 발생했던 오류와 비슷한 유형</p>
<h3 id="3️⃣-settingsgradle-확인">3️⃣ settings.gradle 확인</h3>
<pre><code>rootProject.name = '[프로젝트명]'</code></pre><p>rootProject.name의 대소문자가 맞지 않으면 오류가 발생할 수 있음
하지만 확인해보니 이미 일치하는 상태 → 다른 원인으로 추정</p>
<h3 id="4️⃣-프로젝트-구조---모듈-설정-확인-ctrl--alt--shift--s">4️⃣ 프로젝트 구조 - 모듈 설정 확인 (Ctrl + Alt + Shift + S)</h3>
<p>IntelliJ에서 프로젝트 구조 (Ctrl + Alt + Shift + S) → Modules 설정 확인
rootProject.name과 모듈 이름이 다를 경우 오류 발생 가능
모듈 이름을 settings.gradle과 동일하게 변경 후 정상 작동</p>
<p>📌 정리
✔ settings.gradle에서 rootProject.name만 맞추는 것이 아니라, 모듈 이름도 동일하게 설정해야 오류가 발생하지 않는다.
✔ 같은 유형의 오류를 해결하는 속도가 점점 빨라지고 있음. (이전에는 1주일, 이번에는 1시간)
✔ 앞으로 유사한 오류 발생 시, IDE 설정과 Gradle 모듈을 우선적으로 확인할 것.</p>