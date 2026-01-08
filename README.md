<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Private English Quiz</title>
    <style>
        /* 저작권 보호를 위한 드래그 및 선택 방지 */
        body { -webkit-user-select: none; -moz-user-select: none; -ms-user-select: none; user-select: none; font-family: 'Pretendard', -apple-system, sans-serif; background: #f0f2f5; margin: 0; display: flex; flex-direction: column; align-items: center; min-height: 100vh; }
        :root { --primary: #3498db; --mild: #2ecc71; --spicy: #e74c3c; --dark: #2c3e50; }

        /* 보안 화면 */
        #lock-screen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: var(--dark); display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 2000; color: white; text-align: center; }
        #password-input { padding: 15px; border-radius: 8px; border: none; margin-top: 20px; width: 250px; font-size: 16px; text-align: center; }

        /* 상단 탭 */
        .tabs { display: flex; width: 100%; background: var(--dark); position: sticky; top: 0; z-index: 100; box-shadow: 0 2px 5px rgba(0,0,0,0.2); }
        .tab-btn { flex: 1; background: none; border: none; color: white; padding: 15px 0; cursor: pointer; font-size: 14px; opacity: 0.6; transition: 0.3s; }
        .tab-btn.active { opacity: 1; border-bottom: 4px solid var(--primary); font-weight: bold; }

        /* 컨테이너 */
        .container { max-width: 500px; width: 90%; margin: 30px auto; text-align: center; }
        .week-card { background: white; padding: 30px; border-radius: 20px; box-shadow: 0 10px 20px rgba(0,0,0,0.05); }
        .mode-btn { width: 100%; padding: 18px; border: none; border-radius: 12px; color: white; font-weight: bold; cursor: pointer; margin: 10px 0; font-size: 16px; transition: transform 0.2s; }
        .mode-btn:active { transform: scale(0.98); }
        .btn-mild { background: var(--mild); }
        .btn-spicy { background: var(--spicy); }

        /* 퀴즈 카드 */
        .quiz-area { display: none; width: 100%; }
        .card { width: 100%; height: 320px; perspective: 1000px; cursor: pointer; margin-top: 10px; }
        .card-inner { position: relative; width: 100%; height: 100%; transition: transform 0.6s; transform-style: preserve-3d; }
        .card.flipped .card-inner { transform: rotateY(180deg); }
        .card-front, .card-back { position: absolute; width: 100%; height: 100%; backface-visibility: hidden; display: flex; flex-direction: column; justify-content: center; align-items: center; border-radius: 25px; padding: 25px; box-sizing: border-box; }
        .card-front { background: white; border: 2px solid #eef; }
        .card-back { background: var(--primary); color: white; transform: rotateY(180deg); }
        
        .kr-text { font-size: 1.4rem; line-height: 1.5; word-break: keep-all; color: var(--dark); }
        .en-text { font-size: 1.5rem; font-weight: bold; line-height: 1.4; }
        .source-info { margin-top: 20px; font-size: 0.85rem; background: rgba(0,0,0,0.15); padding: 6px 12px; border-radius: 20px; }

        .controls { display: flex; gap: 15px; margin-top: 25px; width: 100%; }
        .nav-btn { flex: 1; padding: 15px; border-radius: 10px; border: 1px solid #ddd; background: white; font-weight: bold; cursor: pointer; }
        .hidden { display: none; }
    </style>
</head>
<body oncontextmenu="return false" onselectstart="return false">

    <div id="lock-screen">
        <h2>STUDY PRIVATE AREA</h2>
        <p>인증이 필요한 페이지입니다.</p>
        <input type="password" id="password-input" placeholder="Password" onkeypress="handlePw(event)">
    </div>

    <div id="app" class="hidden" style="width: 100%;">
        <div class="tabs">
            <button class="tab-btn active" onclick="changeCurr('기본동사')">기본동사100</button>
            <button class="tab-btn" onclick="changeCurr('영어회화')">영어회화100</button>
            <button class="tab-btn" onclick="changeCurr('구동사')">구동사100</button>
        </div>

        <div class="container" id="menu-view">
            <div class="week-card">
                <h3 id="curr-display" style="color: var(--dark);">기본동사100</h3>
                <p style="color: #666;">Week 1 (Day 1 - 5)</p>
                <hr style="border: 0.5px solid #eee; margin: 20px 0;">
                <button class="mode-btn btn-mild" onclick="initQuiz('mild')">순한맛 (대표/교재1)</button>
                <button class="mode-btn btn-spicy" onclick="initQuiz('spicy')">매운맛 (전체 랜덤)</button>
            </div>
        </div>

        <div class="container quiz-area" id="quiz-view">
            <div id="prog-bar" style="margin-bottom: 10px; font-weight: bold; color: var(--primary);"></div>
            <div class="card" id="quiz-card">
                <div class="card-inner">
                    <div class="card-front"><div class="kr-text" id="k-txt"></div></div>
                    <div class="card-back">
                        <div class="en-text" id="e-txt"></div>
                        <div class="source-info" id="s-info"></div>
                    </div>
                </div>
            </div>
            <div class="controls">
                <button class="nav-btn" onclick="move(-1)">이전</button>
                <button class="nav-btn" onclick="move(1)">다음</button>
            </div>
            <button class="nav-btn" style="margin-top:15px; width:100%; color: #999;" onclick="goHome()">메뉴로 돌아가기</button>
        </div>
    </div>

    <script>
        // 저작권 보호: 개발자 도구 방지
        document.onkeydown = (e) => {
            if (e.keyCode == 123 || (e.ctrlKey && e.shiftKey && e.keyCode == 73)) return false;
        };

        const sourceLabels = { "대표": "대표예제", "교재1": "Model examples", "교재2": "Small Talk", "교재3": "Cases in point", "교재4": "Further studies" };

        // 데이터 통합 관리
        const db = {
            "기본동사": [
                {d:"Day001", s:"대표", k:"반려동물 키우시나요?", e:"Do you have any pets?"},
                {d:"Day001", s:"교재1", k:"내 조카는 거북이를 키운다.", e:"My nephew has a turtle."},
                {d:"Day001", s:"교재2", k:"솔직히 시원섭섭해요.", e:"Honestly, I have mixed feelings."},
                {d:"Day002", s:"대표", k:"방금 간식을 먹었더니, 배가 별로 안 고파요.", e:"I just had a snack, so I’m not that hungry."},
                // ... 나머지 데이터 삽입
            ],
            "영어회화": [
                {d:"Day001", s:"대표", k:"재택근무는 저랑 안 맞아요.", e:"Working from home isn’t for me."},
                {d:"Day003", s:"교재2", k:"미안한데, 오는 길에 커피 좀 사다 줄 수 있나요?", e:"Do you mind grabbing me some coffee on your way?"},
                // ... 나머지 데이터 삽입
            ],
            "구동사": [
                {d:"Day001", s:"교재1", k:"뭔가 앞뒤가 안 맞잖아.", e:"Something doesn’t add up."},
                {d:"Day005", s:"교재1", k:"프랑스어 복습 좀 해야겠어.", e:"I think I need to brush up on my French."},
                // ... 나머지 데이터 삽입
            ]
        };

        let curr = "기본동사";
        let pool = [];
        let idx = 0;

        function handlePw(e) {
            if(e.key === 'Enter') {
                if(e.target.value === "study123") { // 비밀번호 설정
                    document.getElementById('lock-screen').style.display = 'none';
                    document.getElementById('app').classList.remove('hidden');
                } else { alert("Wrong Password"); }
            }
        }

        function changeCurr(name) {
            curr = name;
            document.getElementById('curr-display').innerText = name + "100";
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.toggle('active', b.innerText.includes(name)));
            goHome();
        }

        function initQuiz(mode) {
            let data = db[curr];
            if(mode === 'mild') data = data.filter(v => v.s === '대표' || v.s === '교재1');
            
            // 1. 랜덤 셔플 2. 10개 추출
            pool = data.sort(() => Math.random() - 0.5).slice(0, 10);
            if(pool.length === 0) return alert("문항이 부족합니다.");
            
            idx = 0;
            render();
            document.getElementById('menu-view').classList.add('hidden');
            document.getElementById('quiz-view').style.display = 'block';
        }

        function render() {
            const card = document.getElementById('quiz-card');
            card.classList.remove('flipped');
            const q = pool[idx];
            document.getElementById('prog-bar').innerText = `${idx + 1} / ${pool.length}`;
            document.getElementById('k-txt').innerText = q.k;
            document.getElementById('e-txt').innerText = q.e;
            document.getElementById('s-info').innerText = `${q.d} | ${sourceLabels[q.s] || q.s}`;
        }

        function move(step) {
            if(idx + step >= 0 && idx + step < pool.length) {
                idx += step;
                render();
            }
        }

        function goHome() {
            document.getElementById('menu-view').classList.remove('hidden');
            document.getElementById('quiz-view').style.display = 'none';
        }

        document.getElementById('quiz-card').onclick = function() { this.classList.toggle('flipped'); };
    </script>
</body>
</html>
