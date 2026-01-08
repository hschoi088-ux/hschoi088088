<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Private English Quiz</title>
    <style>
        /* 저작권 보호 및 UI 설정 */
        body { -webkit-user-select: none; -moz-user-select: none; -ms-user-select: none; user-select: none; font-family: 'Pretendard', sans-serif; background: #f4f7f9; margin: 0; display: flex; flex-direction: column; align-items: center; min-height: 100vh; }
        :root { --primary: #4a90e2; --mild: #2ecc71; --spicy: #e74c3c; --dark: #2c3e50; }

        /* 잠금 화면 */
        #lock-screen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: var(--dark); display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 9999; color: white; text-align: center; }
        #pw-input { padding: 15px; border-radius: 8px; border: 2px solid #555; margin-top: 20px; width: 200px; font-size: 20px; text-align: center; background: white; color: black; }

        /* 메인 레이아웃 */
        .tabs { display: flex; width: 100%; background: var(--dark); position: sticky; top: 0; z-index: 100; }
        .tab-btn { flex: 1; background: none; border: none; color: white; padding: 15px 0; cursor: pointer; font-size: 14px; opacity: 0.6; }
        .tab-btn.active { opacity: 1; border-bottom: 4px solid var(--primary); font-weight: bold; }

        .container { max-width: 500px; width: 90%; margin: 30px auto; text-align: center; }
        .panel { background: white; padding: 30px; border-radius: 20px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); }
        .mode-btn { width: 100%; padding: 18px; border: none; border-radius: 12px; color: white; font-weight: bold; cursor: pointer; margin: 10px 0; font-size: 16px; }
        .btn-mild { background: var(--mild); }
        .btn-spicy { background: var(--spicy); }

        /* 퀴즈 카드 */
        .quiz-area { display: none; width: 100%; }
        .card { width: 100%; height: 320px; perspective: 1000px; cursor: pointer; }
        .card-inner { position: relative; width: 100%; height: 100%; transition: transform 0.6s; transform-style: preserve-3d; }
        .card.flipped .card-inner { transform: rotateY(180deg); }
        .card-front, .card-back { position: absolute; width: 100%; height: 100%; backface-visibility: hidden; display: flex; flex-direction: column; justify-content: center; align-items: center; border-radius: 25px; padding: 25px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); box-sizing: border-box; }
        .card-front { background: white; border: 2px solid #eee; color: var(--dark); }
        .card-back { background: var(--primary); color: white; transform: rotateY(180deg); }
        
        .txt-kr { font-size: 1.3rem; line-height: 1.6; word-break: keep-all; }
        .txt-en { font-size: 1.4rem; font-weight: bold; line-height: 1.4; }
        .info-tag { margin-top: 20px; font-size: 0.8rem; background: rgba(0,0,0,0.1); padding: 5px 12px; border-radius: 15px; }

        .controls { display: flex; gap: 10px; margin-top: 25px; }
        .nav-btn { flex: 1; padding: 15px; border-radius: 10px; border: 1px solid #ddd; background: white; font-weight: bold; cursor: pointer; }
        .hidden { display: none; }
    </style>
</head>
<body oncontextmenu="return false">

    <div id="lock-screen">
        <h2>🔒 학습자 인증</h2>
        <p>비밀번호 4자리를 입력하고 엔터를 누르세요.</p>
        <input type="password" id="pw-input" maxlength="4" placeholder="****" onkeypress="handlePassword(event)">
    </div>

    <div id="app" class="hidden" style="width: 100%;">
        <div class="tabs">
            <button class="tab-btn active" id="tab-기본동사" onclick="setCurr('기본동사')">기본동사100</button>
            <button class="tab-btn" id="tab-영어회화" onclick="setCurr('영어회화')">영어회화100</button>
            <button class="tab-btn" id="tab-구동사" onclick="setCurr('구동사')">구동사100</button>
        </div>

        <div class="container" id="menu-view">
            <div class="panel">
                <h2 id="display-title">기본동사100</h2>
                <p style="color: #666;">Week 1 (Day 1 - 5)</p>
                <button class="mode-btn btn-mild" onclick="startQuiz('mild')">순한맛 (대표/교재1)</button>
                <button class="mode-btn btn-spicy" onclick="startQuiz('spicy')">매운맛 (전체 랜덤)</button>
            </div>
        </div>

        <div class="container quiz-area" id="quiz-view">
            <div id="prog" style="margin-bottom: 10px; font-weight: bold; color: var(--primary);"></div>
            <div class="card" id="q-card">
                <div class="card-inner">
                    <div class="card-front"><div class="txt-kr" id="k-display"></div></div>
                    <div class="card-back">
                        <div class="txt-en" id="e-display"></div>
                        <div class="info-tag" id="i-display"></div>
                    </div>
                </div>
            </div>
            <div class="controls">
                <button class="nav-btn" onclick="nav(-1)">이전 문제</button>
                <button class="nav-btn" onclick="nav(1)">다음 문제</button>
            </div>
            <button class="nav-btn" style="margin-top:20px; width:100%; border:none; color:#999;" onclick="goHome()">종료하고 메뉴로</button>
        </div>
    </div>

    <script>
        // 저작권 보호: 우클릭 및 개발자도구 차단
        document.onkeydown = (e) => { if(e.keyCode == 123 || (e.ctrlKey && e.shiftKey && e.keyCode == 73)) return false; };

        const labels = { "대표": "대표예제", "교재1": "Model examples", "교재2": "Small Talk", "교재3": "Cases in point", "교재4": "Further studies" };

        // --- 데이터 시작 (여기에 예문을 계속 추가하세요) ---
        const database = {
            "기본동사": [
                {d:"Day001", s:"대표", k:"반려동물 키우시나요?", e:"Do you have any pets?"},
                {d:"Day001", s:"교재1", k:"내 조카는 거북이를 키운다.", e:"My nephew has a turtle."},
                {d:"Day001", s:"교재1", k:"나는 시카고에 사는 친구들이 있다.", e:"I have some friends living in Chicago."},
                {d:"Day001", s:"교재1", k:"남동생과 나는 우애가 돈독하다.", e:"My brother and I have a strong bond."},
                {d:"Day001", s:"교재1", k:"내 생각은 달랐다.", e:"I had a different idea."},
                {d:"Day001", s:"교재1", k:"친구들과 술래잡기하면서 놀던 즐거운 추억이 있다.", e:"I have some fond memories of playing tag with my friends."},
                {d:"Day001", s:"교재2", k:"프로젝트가 끝나서 좋죠?", e:"Are you glad the project is over?"},
                {d:"Day001", s:"교재2", k:"솔직히 시원섭섭해요.", e:"Honestly, I have mixed feelings."},
                {d:"Day002", s:"대표", k:"방금 간식을 먹었더니, 배가 별로 안 고파요.", e:"I just had a snack, so I’m not that hungry."}
            ],
            "영어회화": [
                {d:"Day001", s:"대표", k:"재택근무는 저랑 안 맞아요.", e:"Working from home isn’t for me."},
                {d:"Day001", s:"교재1", k:"소개팅은 저랑 안 맞아요.", e:"Going on blind dates isn’t for me."}
            ],
            "구동사": [
                {d:"Day001", s:"교재1", k:"뭔가 앞뒤가 안 맞잖아.", e:"Something doesn’t add up."},
                {d:"Day001", s:"교재1", k:"자, 이 숫자들을 더해 보자.", e:"Let’s add up these numbers now."}
            ]
        };

        let currentCategory = "기본동사";
        let quizList = [];
        let currentIdx = 0;

        // 비밀번호 확인 함수
        function handlePassword(e) {
            if(e.key === 'Enter') {
                const input = document.getElementById('pw-input').value;
                if(input === "4321") {
                    document.getElementById('lock-screen').style.display = 'none';
                    document.getElementById('app').classList.remove('hidden');
                } else {
                    alert("비밀번호가 틀렸습니다.");
                    document.getElementById('pw-input').value = "";
                }
            }
        }

        function setCurr(name) {
            currentCategory = name;
            document.getElementById('display-title').innerText = name + "100";
            document.querySelectorAll('.tab-btn').forEach(b => {
                b.classList.remove('active');
                if(b.id === 'tab-' + name) b.classList.add('active');
            });
            goHome();
        }

        function startQuiz(mode) {
            let dataPool = [...database[currentCategory]];
            
            if(mode === 'mild') {
                dataPool = dataPool.filter(x => x.s === '대표' || x.s === '교재1');
            }
            
            // 랜덤으로 섞기
            quizList = dataPool.sort(() => Math.random() - 0.5);
            
            // 최대 10개만 선택 (데이터가 적으면 적은 대로 나옴)
            quizList = quizList.slice(0, 10);
            
            if(quizList.length === 0) {
                alert("해당 조건의 문장이 없습니다. 데이터를 확인해주세요.");
                return;
            }
            
            currentIdx = 0;
            showQuiz();
            document.getElementById('menu-view').classList.add('hidden');
            document.getElementById('quiz-view').style.display = 'block';
        }

        function showQuiz() {
            const card = document.getElementById('q-card');
            card.classList.remove('flipped');
            
            const item = quizList[currentIdx];
            document.getElementById('prog').innerText = `QUESTION ${currentIdx + 1} / ${quizList.length}`;
            document.getElementById('k-display').innerText = item.k;
            document.getElementById('e-display').innerText = item.e;
            document.getElementById('i-display').innerText = `${item.d} | ${labels[item.s] || item.s}`;
        }

        function nav(dir) {
            if(currentIdx + dir >= 0 && currentIdx + dir < quizList.length) {
                currentIdx += dir;
                showQuiz();
            }
        }

        function goHome() {
            document.getElementById('menu-view').classList.remove('hidden');
            document.getElementById('quiz-view').style.display = 'none';
        }

        document.getElementById('q-card').onclick = function() {
            this.classList.toggle('flipped');
        };
    </script>
</body>
</html>
