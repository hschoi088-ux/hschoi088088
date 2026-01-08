<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Private English Speed Quiz</title>
    <style>
        /* 저작권 보호: 드래그 및 우클릭 방지 */
        body { -webkit-user-select: none; -moz-user-select: none; -ms-user-select: none; user-select: none; font-family: 'Pretendard', sans-serif; background: #f4f7f9; margin: 0; display: flex; flex-direction: column; align-items: center; min-height: 100vh; }
        :root { --primary: #4a90e2; --mild: #2ecc71; --spicy: #e74c3c; --dark: #2c3e50; }

        /* 잠금 화면 */
        #lock-screen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: var(--dark); display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 2000; color: white; }
        #pw-input { padding: 15px; border-radius: 8px; border: none; margin-top: 20px; width: 220px; font-size: 18px; text-align: center; letter-spacing: 5px; }

        /* 상단 탭 */
        .tabs { display: flex; width: 100%; background: var(--dark); position: sticky; top: 0; z-index: 100; }
        .tab-btn { flex: 1; background: none; border: none; color: white; padding: 15px 0; cursor: pointer; font-size: 14px; opacity: 0.6; }
        .tab-btn.active { opacity: 1; border-bottom: 4px solid var(--primary); font-weight: bold; }

        /* 메인 영역 */
        .container { max-width: 500px; width: 90%; margin: 30px auto; text-align: center; }
        .card-panel { background: white; padding: 30px; border-radius: 20px; box-shadow: 0 10px 25px rgba(0,0,0,0.05); }
        .mode-btn { width: 100%; padding: 18px; border: none; border-radius: 12px; color: white; font-weight: bold; cursor: pointer; margin: 10px 0; font-size: 16px; }
        .btn-mild { background: var(--mild); }
        .btn-spicy { background: var(--spicy); }

        /* 퀴즈 카드 UI */
        .quiz-area { display: none; width: 100%; }
        .card { width: 100%; height: 320px; perspective: 1000px; cursor: pointer; }
        .card-inner { position: relative; width: 100%; height: 100%; transition: transform 0.6s; transform-style: preserve-3d; }
        .card.flipped .card-inner { transform: rotateY(180deg); }
        .card-front, .card-back { position: absolute; width: 100%; height: 100%; backface-visibility: hidden; display: flex; flex-direction: column; justify-content: center; align-items: center; border-radius: 25px; padding: 25px; box-shadow: 0 4px 15px rgba(0,0,0,0.08); }
        .card-front { background: white; border: 2px solid #eee; }
        .card-back { background: var(--primary); color: white; transform: rotateY(180deg); }
        
        .txt-kr { font-size: 1.35rem; line-height: 1.6; color: var(--dark); word-break: keep-all; }
        .txt-en { font-size: 1.45rem; font-weight: bold; line-height: 1.4; }
        .info-tag { margin-top: 20px; font-size: 0.8rem; background: rgba(0,0,0,0.1); padding: 5px 12px; border-radius: 15px; }

        .controls { display: flex; gap: 10px; margin-top: 25px; }
        .nav-btn { flex: 1; padding: 15px; border-radius: 10px; border: 1px solid #ddd; background: white; font-weight: bold; cursor: pointer; }
        .hidden { display: none; }
    </style>
</head>
<body oncontextmenu="return false">

    <div id="lock-screen">
        <h3>PRIVATE QUIZ ACCESS</h3>
        <input type="password" id="pw-input" placeholder="****" maxlength="4" onkeypress="checkPw(event)">
        <p style="font-size: 12px; margin-top: 10px; opacity: 0.5;">4-digit password + Enter</p>
    </div>

    <div id="app" class="hidden" style="width: 100%;">
        <div class="tabs">
            <button class="tab-btn active" onclick="setCurr('기본동사')">기본동사100</button>
            <button class="tab-btn" onclick="setCurr('영어회화')">영어회화100</button>
            <button class="tab-btn" onclick="setCurr('구동사')">구동사100</button>
        </div>

        <div class="container" id="menu-view">
            <div class="card-panel">
                <h2 id="display-title">기본동사100</h2>
                <p>Week 1 (Day 001 - Day 005)</p>
                <hr style="border: 0.5px solid #f0f0f0; margin: 25px 0;">
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
                <button class="nav-btn" onclick="nav(-1)">PREV</button>
                <button class="nav-btn" onclick="nav(1)">NEXT</button>
            </div>
            <button class="nav-btn" style="margin-top:15px; width:100%; border:none; color:#999;" onclick="goHome()">QUIT QUIZ</button>
        </div>
    </div>

    <script>
        // 개발자 도구 차단
        document.onkeydown = (e) => { if(e.keyCode == 123 || (e.ctrlKey && e.shiftKey && e.keyCode == 73)) return false; };

        const labels = { "대표": "대표예제", "교재1": "Model examples", "교재2": "Small Talk", "교재3": "Cases in point", "교재4": "Further studies" };

        // [데이터 섹션] - 주신 예문을 여기에 정확히 카테고리별로 넣으세요.
        const database = {
            "기본동사": [
                {d:"Day001", s:"대표", k:"반려동물 키우시나요?", e:"Do you have any pets?"},
                {d:"Day001", s:"교재1", k:"내 조카는 거북이를 키운다.", e:"My nephew has a turtle."},
                {d:"Day001", s:"교재1", k:"나는 시카고에 사는 친구들이 있다.", e:"I have some friends living in Chicago."},
                {d:"Day001", s:"교재2", k:"솔직히 시원섭섭해요.", e:"Honestly, I have mixed feelings."},
                // ... (중략: 보내주신 86개 문장을 위 형식으로 모두 채워주세요)
            ],
            "영어회화": [
                {d:"Day001", s:"대표", k:"재택근무는 저랑 안 맞아요.", e:"Working from home isn’t for me."},
                {d:"Day001", s:"교재1", k:"소개팅은 저랑 안 맞아요.", e:"Going on blind dates isn’t for me."},
                // ... (중략: 보내주신 80개 문장)
            ],
            "구동사": [
                {d:"Day001", s:"교재1", k:"뭔가 앞뒤가 안 맞잖아.", e:"Something doesn’t add up."},
                {d:"Day003", s:"교재1", k:"제 차가 고속 도로에서 고장이 났습니다.", e:"My car broke down on the highway."},
                // ... (중략: 보내주신 100여개 문장)
            ]
        };

        let currentCategory = "기본동사";
        let quizList = [];
        let currentIdx = 0;

        function checkPw(e) {
            if(e.key === 'Enter') {
                if(e.target.value === "4321") {
                    document.getElementById('lock-screen').style.display = 'none';
                    document.getElementById('app').classList.remove('hidden');
                } else { alert("Access Denied"); e.target.value = ""; }
            }
        }

        function setCurr(name) {
            currentCategory = name;
            document.getElementById('display-title').innerText = name + "100";
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.toggle('active', b.innerText.includes(name)));
            goHome();
        }

        function startQuiz(mode) {
            let srcData = [...database[currentCategory]];
            
            // 1. 모드 필터링
            if(mode === 'mild') {
                srcData = srcData.filter(x => x.s === '대표' || x.s === '교재1');
            }
            
            // 2. 전체 랜덤 셔플 후 딱 10개만 추출
            quizList = srcData.sort(() => Math.random() - 0.5).slice(0, 10);
            
            if(quizList.length < 1) { alert("문항 데이터가 부족합니다."); return; }
            
            currentIdx = 0;
            updateQuiz();
            document.getElementById('menu-view').classList.add('hidden');
            document.getElementById('quiz-view').style.display = 'block';
        }

        function updateQuiz() {
            const card = document.getElementById('q-card');
            card.classList.remove('flipped');
            
            const q = quizList[currentIdx];
            document.getElementById('prog').innerText = `QUESTION ${currentIdx + 1} / ${quizList.length}`;
            document.getElementById('k-display').innerText = q.k;
            document.getElementById('e-display').innerText = q.e;
            document.getElementById('i-display').innerText = `${q.d} | ${labels[q.s] || q.s}`;
        }

        function nav(step) {
            if(currentIdx + step >= 0 && currentIdx + step < quizList.length) {
                currentIdx += step;
                updateQuiz();
            }
        }

        function goHome() {
            document.getElementById('menu-view').classList.remove('hidden');
            document.getElementById('quiz-view').style.display = 'none';
        }

        document.getElementById('q-card').onclick = function() { this.classList.toggle('flipped'); };
    </script>
</body>
</html>
