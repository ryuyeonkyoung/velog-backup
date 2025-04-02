<h3 id="문제상황--revert--reset--이후-커밋-squash라고-이해하고-있었다">문제상황 : revert = reset + 이후 커밋 squash라고 이해하고 있었다</h3>
<ol>
<li>Spring Security로 로그인/로그아웃/회원가입 기능을 구현한 후, README를 수정하려고 했다.</li>
<li>그런데 계속 README를 자잘하게 수정하게 되어서 마지막에 커밋들을 하나로 뭉쳐서 정리하려고 했다.</li>
<li>내가 당시 <code>revert</code>의 기능을 잘못 이해하고 있었는데, 나는 이게 <code>reset</code>과 <code>squash</code>를 합쳐놓은 기능이라고 이해하고 있었다.</li>
<li>그래서 <code>revert</code>를 적용했다.</li>
<li>며칠 뒤 프로젝트를 이어나려고 했는데, <strong>그동안 내가 구현한 기능들이 다 사라져 있었다...!</strong>
<img alt="" src="https://velog.velcdn.com/images/rykjjang/post/30aae5b8-5345-4aee-99c2-c2045a4afcd3/image.png" /></li>
<li>구현한 기능들을 다 삭제한 채로 README만 수정한 상황이 되었다.</li>
</ol>
<hr />
<h3 id="해결-방안-cherry-pick으로-원하는-커밋만-골라-복구"><strong>해결 방안: cherry-pick으로 원하는 커밋만 골라 복구</strong></h3>
<p>단순히 README.md 파일만 따로 백업해두고 reset 후 다시 추가할 수도 있었지만, 이번 기회에 cherry-pick이라는 정석적인 Git 기능을 직접 사용해보고 싶었습니다.
평소에 궁금한 기능이기도 했고, 이런 상황은 협업 중에도 자주 발생할 수 있기 때문에, 이번 기회에 과정을 천천히 따라가며 제대로 익히고자 했습니다.</p>
<h4 id="원하는-커밋-화면">원하는 커밋 화면</h4>
<p>수정 전</p>
<pre><code>// reset 전 커밋 상태
E - README.md 최종 수정
D - revert: A에서 추가된 코드 제거
C - README.md 수정 3
B - README.md 수정 2
A - feat: 로그인/로그아웃/회원가입 구현</code></pre><p>수정 후</p>
<pre><code>// reset 후 + cherry-pick 적용 결과 (최신 → 과거 순)
F - cherry-pick: README.md 최종 수정
└─ A - feat: 로그인/로그아웃/회원가입 구현
</code></pre><h4 id="해결-방법">해결 방법</h4>
<ol>
<li>E 커밋의 해시를 복사해둔다.</li>
<li>A를 기준으로 reset한다.
→ 이 시점에서 A 이후 커밋들은 브랜치에서 사라진다. E도 함께 사라진다.<pre><code class="language-bash">git reset --hard A</code></pre>
</li>
<li>E를 cherry-pick으로 가져온다.
만약 reset을 해버려서 로그에서 커밋이 안 보이는 상황이라면, reflog에서 확인할 수 있다. reset 전에는 항상 git log 또는 git reflog로 커밋 해시를 복사해두는 습관을 들이는 게 좋다.<pre><code class="language-bash"># 최근 브랜치 이동 내역 확인 (삭제된 커밋도 포함)
git reflog
</code></pre>
</li>
</ol>
<p>git cherry-pick E</p>
<pre><code>4. 충돌이 발생한다면:
충돌하는 파일을 수정한 뒤 add → cherry-pick을 마무리한다.
```bash
git add README.md
git cherry-pick --continue</code></pre><ol start="5">
<li>push 시도 시 충돌 발생 → 선택지</li>
</ol>
<ul>
<li>병합(M) 또는 리베이스(R):
UI를 통해 push를 시도하면 아래와 같은 창에서 선택 가능
<img alt="" src="https://velog.velcdn.com/images/rykjjang/post/c4012232-404d-4eba-a8cd-a06653ef3539/image.png" /></li>
<li>둘 다 원하지 않는다면 → 터미널에서 강제 푸시:<pre><code class="language-bash">git push --force</code></pre>
</li>
</ul>
<p>if. cherry-pick을 중간에 취소하고 싶다면?</p>
<pre><code class="language-bash">git cherry-pick --abort</code></pre>
<p>결과
<img alt="" src="https://velog.velcdn.com/images/rykjjang/post/fecc0dca-93bd-42f2-a983-252d02293482/image.png" /></p>
<blockquote>
<p>✨ <strong>여기서 신기했던 점</strong>
reset으로 E 커밋을 삭제했는데,
그 커밋을 다시 cherry-pick으로 적용할 수 있다...!!처음엔 reset한 순간 이후 커밋은 다 날아가서 복구가 불가능할 줄 알았다.
하지만 git은 reflog라는 내부 로그를 통해
브랜치가 이동했던 모든 커밋을 잠시 기억하고 있다.</p>
</blockquote>
<hr />
<h3 id="깨달은-점-revert는-커밋-삭제가-아니라-반대-커밋-추가다"><strong>깨달은 점: revert는 '커밋 삭제'가 아니라 '반대 커밋 추가'다</strong></h3>
<table>
<thead>
<tr>
<th>내가 착각한 revert</th>
<th>실제 revert</th>
</tr>
</thead>
<tbody><tr>
<td>reset처럼 시점 복구 + 기록 유지</td>
<td>커밋 자체는 놔두고, <strong>지정 커밋의 반대 연산을 적용한 새 커밋 추가</strong></td>
</tr>
<tr>
<td>이후 변경사항은 사라짐</td>
<td>이후 변경사항은 그대로 유지됨</td>
</tr>
<tr>
<td>특정 시점으로 되돌아감</td>
<td>되돌리려는 커밋만 무효화</td>
</tr>
</tbody></table>
<p>추가로, 이후 커밋이 기능과 겹쳤다면 <code>git revert</code> 중 충돌(conflict)도 발생할 수 있었지만,<br />운 좋게도 이후 커밋이 README 수정뿐이라 충돌 없이 넘어갈 수 있었습니다.</p>
<hr />
<h3 id="정리-rebase-vs-revert-vs-reset">정리: rebase vs revert vs reset</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th><code>rebase</code></th>
<th><code>revert</code></th>
<th><code>reset</code></th>
</tr>
</thead>
<tbody><tr>
<td>목적</td>
<td>히스토리 정리 (순서/병합/삭제 등)</td>
<td>특정 커밋의 효과를 취소</td>
<td>커밋 자체를 과거 시점으로 <strong>되돌림</strong></td>
</tr>
<tr>
<td>작동 방식</td>
<td>커밋을 <strong>재작성</strong>하고 새로 push (force 가능성 있음)</td>
<td>되돌리는 <strong>새 커밋을 추가</strong></td>
<td>커밋 포인터를 <strong>이전 커밋으로 이동</strong> (옵션에 따라 워킹 디렉토리 영향)</td>
</tr>
<tr>
<td>사용 상황</td>
<td>커밋 메시지 정리, squash, 충돌 해결</td>
<td>실수한 커밋 되돌릴 때, 협업 중 오류 수정</td>
<td>최근 커밋 자체를 없애고 싶을 때 (혼자 작업 시 주로 사용)</td>
</tr>
<tr>
<td>히스토리 안전성</td>
<td>❌ 히스토리 바뀜 → force push 필요</td>
<td>✅ 히스토리 유지 안전</td>
<td>❌ 위험도 높음 (강제 push 필요, 기록 자체 삭제 가능)</td>
</tr>
<tr>
<td>복구 가능성</td>
<td>✅ 백업 브랜치 있으면 가능</td>
<td>✅ 언제든 복구 가능 (되돌린 커밋은 남음)</td>
<td>❌ 되돌린 커밋은 완전히 사라짐 (<code>--hard</code> 사용 시)</td>
</tr>
<tr>
<td>중간 커밋 제어</td>
<td>✅ 가능 (순서 수정, squash 등 자유롭게)</td>
<td>🔶 간접 가능 (여러 개 revert 가능하지만 순서 못 바꿈)</td>
<td>🔶 불가능 (단순 이전 시점으로 이동)</td>
</tr>
<tr>
<td>협업 상황 적합성</td>
<td>🔶 조심해야 함 (force push는 팀원과 충돌 가능)</td>
<td>✅ 안전하게 공유 가능</td>
<td>❌ 협업 중 절대 금지 (<code>reset</code> 후 force push는 위험함)</td>
</tr>
</tbody></table>