<p>MySQL을 재설치하고 접속했더니 비밀번호가 변경되어있었다.
MySQL8 이상부터는 비밀번호가 랜덤한 값으로 주어지기 때문이다.
이를 확인하기 위해서는 err파일을 확인해야 한다.</p>
<h1 id="해결과정🎈">해결과정🎈</h1>
<ol>
<li>MySQL이 설치된 폴더에서 data파일에 들어간다.</li>
<li>[PC이름].err파일에 들어간다.</li>
<li>[Note]줄을 확인하면 비밀번호를 알 수 있다. (ctrl+f로 password를 검색해도 된다.)
<img alt="" src="https://velog.velcdn.com/images/rykjjang/post/d7dc3ff5-a918-4612-bc25-45c3a610a61b/image.png" /></li>
</ol>