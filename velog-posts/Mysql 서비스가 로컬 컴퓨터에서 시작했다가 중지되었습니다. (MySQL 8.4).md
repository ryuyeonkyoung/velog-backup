<h1 id="설명">설명</h1>
<p>스프링부트 프로젝트를 다시 진행하려고 그동안 짰던 코드를 실행해 보았는데 종료코드 1과 함게 실행되지 않았다.</p>
<p>서버가 꺼졌나 싶어서 (노트북이 발열이 심해서 매번 서버를 수동으로 껐다 키면서 사용하고 있다) 서비스에 들어가서 시작 버튼을 눌렀는데
<img alt="오류창" src="https://velog.velcdn.com/images/rykjjang/post/fe855ee0-63ea-40b1-98ac-ab785da29e62/image.png" />
다음과 같은 오류가 떴다.</p>
<h1 id="해결방법🎈">해결방법🎈</h1>
<p><a href="https://flexiblecode.tistory.com/76">Mysql 서비스가 로컬 컴퓨터에서 시작했다가 중지되었습니다. 오류해결방법</a>
다음 게시글을 참고하였다. 전체적으로는 같은 과정을 겪었으나 내가 막혔었던 몇 가지 부분에 대한 설명을 추가하려고 한다. 부디 같은 오류를 겪으시는 분들은 나처럼 다른 길에서 헤메지 않으셨으면 한다. 이유는 모르겠으나 gpt한테 물으면 주로 겪는 오류는 안알려줘서 먼 길을 돌아가게 된다...</p>
<h2 id="1-관리자-모드로-cmd-실행">1. 관리자 모드로 cmd 실행</h2>
<p>cmd를 검색한 다음 &quot;관리자 권한으로 실행&quot;을 클릭한다.
cmd창에 관리자:명령 프롬프트라고 떠야 한다.
<img alt="" src="https://velog.velcdn.com/images/rykjjang/post/39c6cb58-01b7-4f3e-ad35-e692dd5fa016/image.png" /></p>
<h2 id="2-net-start-mysql의-서비스-이름">2. net start [MySQL의 서비스 이름]</h2>
<p>시스템에서 MySQL을 찾은 뒤 속성에 들어가면 서비스 이름을 알 수 있다. 나의 경우는 MySQL8.4.1이다. (표시 이름과 같다.)
<img alt="" src="https://velog.velcdn.com/images/rykjjang/post/6e1fd91f-2350-49e9-a431-fd5b336bd984/image.png" />
<img alt="" src="https://velog.velcdn.com/images/rykjjang/post/95d7c872-6257-4d4a-b349-7ed9befe3ef3/image.png" /></p>
<h2 id="3-cd-mysql의-bin파일-경로">3. cd &quot;[MySQL의 bin파일 경로]&quot;</h2>
<p>밑에 실행 파일 경로에서 bin파일의 경로를 알 수 있다.
나의 경우는 &quot;C:\Program Files\MySQL\MySQL Server 8.4\bin&quot;이다.
cmd창에 [파일경로]&gt; 로 뜨게 되면 성공이다.
<img alt="" src="https://velog.velcdn.com/images/rykjjang/post/e6d4a742-03c0-44be-abe6-b849af1bcf5b/image.png" /></p>
<h2 id="4-mysqld---initialize">4. mysqld --initialize</h2>
<h2 id="5-net-start-mysql의-서비스-이름">5. net start [MySQL의 서비스 이름]</h2>
<h2 id="6-45번으로-해결이-안되면">6. 4,5번으로 해결이 안되면</h2>
<p>cd로 이동한 상태에서
mysqld --remove [MySQL의 서비스 이름]
mysqld --install [MySQL의 서비스 이름]
을 시도해 볼 수도 있다.
나는 4,5까지 거쳤을 때는 err파일에서 ERROR가 사라졌지만 여전히 실행은 되지 않았었다. 그래서 6까지 한 후에야 실행이 되었다.</p>
<h1 id="추가-에러-코드">추가) 에러 코드</h1>
<p>[PC이름].err파일을 보면 에러코드를 확인할 수 있다.</p>