<p><img alt="" src="https://velog.velcdn.com/images/rykjjang/post/dcd92b1c-486a-42fb-98d3-79b1bfd8b790/image.png" /></p>
<h1 id="📌-트러블슈팅-javaioioexception-javaniofilenosuchfileexception-해결-과정">📌 [트러블슈팅] java.io.IOException: java.nio.file.NoSuchFileException 해결 과정</h1>
<p>POST 요청을 처리하는 과정에서 <strong>파일이 첨부되지 않았음에도 파일 저장 로직이 실행</strong>되면서 오류가 발생했다.<br />파일을 첨부하지 않았는데도 <code>boardFile</code> 리스트가 비어있지 않은 이유를 분석하고 해결 과정을 정리한다.</p>
<hr />
<h2 id="🛠-오류-상황">🛠 오류 상황</h2>
<h3 id="1️⃣-오류-발생">1️⃣ 오류 발생</h3>
<p><code>/save</code> API에 <code>POST</code> 요청을 보냈을 때, <strong>파일 첨부 여부와 관계없이 저장 로직이 실행됨</strong>  </p>
<ul>
<li><strong>오류 메시지:</strong><pre><code>plaintext
java.io.IOException: java.nio.file.NoSuchFileException: C:\springboot_img\1724750433697</code></pre>발생 원인:</li>
<li>파일을 첨부하지 않았는데도 boardFile 리스트가 비어있지 않음</li>
<li>file_attached 필드가 1로 저장됨
boardFile.getBoardFile().isEmpty() 값이 false를 반환</li>
</ul>
<h2 id="🚀-해결-과정">🚀 해결 과정</h2>
<h3 id="1️⃣-dto-출력-및-데이터-확인">1️⃣ DTO 출력 및 데이터 확인</h3>
<p>오류 원인을 찾기 위해 BoardDTO 데이터를 출력하여 확인</p>
<pre><code>boardDTO = BoardDTO(id=null, boardWriter=ㅎㅎ, boardPass=ㅎㅎ, boardTitle=ㅎㅎ, boardContents=ㅎㅎ, boardHits=0,
boardCreatedTime=null, boardUpdatedTime=null,
boardFile=[org.springframework.web.multipart.support.StandardMultipartHttpServletRequest$StandardMultipartFile@49f581d0],
originalFileName=null, storedFileName=null, fileAttached=0)</code></pre><p>🔍 특이점:</p>
<p>boardFile 리스트가 존재하지만, originalFileName 값이 null
fileAttached 필드는 0으로 설정되었음에도 boardFile이 비어있지 않음</p>
<h3 id="2️⃣-multipartfile의-특징-확인">2️⃣ MultipartFile의 특징 확인</h3>
<p>boardFile의 데이터 유형을 확인한 결과,
StandardMultipartHttpServletRequest$StandardMultipartFile 객체가 포함됨
해당 객체가 리스트에 존재하지만, 파일 이름(originalFileName)이 null인 상태
3️⃣ 문제 발생 코드 분석
파일 첨부 여부를 확인하는 기존 코드:</p>
<pre><code>if (boardDTO.getBoardFile().isEmpty()) {
    System.out.println(&quot;파일 첨부 없음&quot;);
    BoardEntity boardEntity = BoardEntity.toSaveEntity(boardDTO);
    boardRepository.save(boardEntity);
} else {
    System.out.println(&quot;파일 첨부 있음&quot;);
    // 파일 저장 로직 실행
}</code></pre><p>🔍 boardFile.isEmpty()가 false를 반환하여, 실제 파일이 첨부되지 않았음에도 파일 저장 로직이 실행됨</p>
<h3 id="4️⃣-해결-방법">4️⃣ 해결 방법</h3>
<p>📌 POST 요청에서 boardFile 리스트의 첫 번째 요소가 비어있는 경우 리스트를 초기화</p>
<pre><code>@PostMapping(&quot;/save&quot;)
public String save(BoardDTO boardDTO) throws IOException {
    if (!boardDTO.getBoardFile().isEmpty() &amp;&amp;
        boardDTO.getBoardFile().get(0).getOriginalFilename().isEmpty()) {
        boardDTO.setBoardFile(Collections.emptyList());
    }
    boardService.save(boardDTO);
    return &quot;redirect:/&quot;;
}</code></pre>