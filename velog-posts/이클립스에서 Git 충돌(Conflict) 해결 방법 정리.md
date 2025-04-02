<h3 id="1-본인-소스를-commit하지-않고-그대로-pull-받는-경우-→-💡mark-as-merged">1. 본인 소스를 commit하지 않고 그대로 pull 받는 경우 → 💡<strong>Mark as Merged</strong></h3>
<ol>
<li>내 코드 변경 사항을 버리고, 다른 사람의 변경 사항만 반영할 때 사용</li>
<li>내 변경 사항이 중요하거나 반영되어야 한다면 <strong>절대 사용하지 말 것</strong></li>
</ol>
<hr />
<h3 id="2-본인-소스를-commit해야-하는-경우-→-💡merge-tool-또는-수동-해결">2. 본인 소스를 commit해야 하는 경우 → 💡<strong>Merge Tool 또는 수동 해결</strong></h3>
<h4 id="2-1-eclipse-gui-merge-tool-사용">2-1. Eclipse GUI (Merge Tool) 사용</h4>
<ol>
<li>충돌이 발생한 파일을 확인</li>
<li>해당 파일 우클릭 → <code>Team</code> → <code>Merge Tool</code></li>
<li><code>Select a merge mode</code>에서 비교<br />(첫 번째 ours 클릭해보면 차이 확인 가능)</li>
<li>내용 수정 후 <code>Ctrl + S</code>로 저장</li>
<li>Git Staging 탭에서 수정된 파일 <code>Stage changes</code></li>
<li>자동 생성된 메시지 확인 → <code>Commit and Push</code></li>
</ol>
<hr />
<h4 id="2-2-직접-충돌-해결-텍스트-수정-방식">2-2. 직접 충돌 해결 (텍스트 수정 방식)</h4>
<ol>
<li>충돌난 파일 클릭 → 에디터에서 아래와 같은 conflict 마크 확인</li>
</ol>
<pre><code class="language-plaintext">&lt;&lt;&lt;&lt;&lt;&lt;&lt; HEAD
내 로컬 변경 내용
=======
다른 사람의 원격 변경 내용
&gt;&gt;&gt;&gt;&gt;&gt;&gt; origin/main</code></pre>
<ol start="2">
<li>원하는 코드로 수정한 후 conflict 마크(<code>&lt;&lt;&lt;&lt;&lt;&lt;&lt;</code>, <code>=======</code>, <code>&gt;&gt;&gt;&gt;&gt;&gt;&gt;</code>) 제거</li>
<li><code>Ctrl + S</code>로 저장</li>
<li>Git Staging 탭에서 수정된 파일 <code>Stage changes</code></li>
<li>자동 생성된 메시지 확인 → <code>Commit and Push</code></li>
</ol>
<hr />
<h3 id="references">References</h3>
<ul>
<li><a href="https://tiqndjd12.tistory.com/67">https://tiqndjd12.tistory.com/67</a></li>
</ul>