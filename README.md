<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>English Master Speed Quiz</title>
    <style>
        /* 보안 및 저작권 보호 UI */
        body { -webkit-user-select: none; user-select: none; font-family: 'Pretendard', sans-serif; background: #f0f4f8; margin: 0; display: flex; flex-direction: column; align-items: center; min-height: 100vh; }
        :root { --primary: #3498db; --mild: #2ecc71; --spicy: #e74c3c; --dark: #2c3e50; }
        
        /* 암호화 레이어 */
        #lock-screen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: var(--dark); display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 9999; color: white; }
        #pw-input { padding: 15px; border-radius: 8px; border: none; margin-top: 20px; width: 180px; font-size: 20px; text-align: center; letter-spacing: 5px; }

        /* 메인 UI */
        .tabs { display: flex; width: 100%; background: var(--dark); position: sticky; top: 0; z-index: 100; }
        .tab-btn { flex: 1; background: none; border: none; color: white; padding: 15px 0; cursor: pointer; font-size: 14px; opacity: 0.6; }
        .tab-btn.active { opacity: 1; border-bottom: 4px solid var(--primary); font-weight: bold; }

        .container { max-width: 500px; width: 90%; margin: 30px auto; text-align: center; }
        .panel { background: white; padding: 30px; border-radius: 20px; box-shadow: 0 10px 30px rgba(0,0,0,0.08); }
        .week-btn { width: 100%; padding: 20px; background: white; border: 2px solid var(--primary); color: var(--primary); border-radius: 15px; font-weight: bold; font-size: 18px; cursor: pointer; margin-bottom: 15px; }
        
        .mode-select { display: none; gap: 10px; margin-top: 10px; }
        .mode-btn { flex: 1; padding: 15px; border: none; border-radius: 10px; color: white; font-weight: bold; cursor: pointer; }
        .btn-mild { background: var(--mild); }
        .btn-spicy { background: var(--spicy); }

        /* 퀴즈 카드 */
        .quiz-area { display: none; width: 100%; }
        .card { width: 100%; height: 350px; perspective: 1000px; cursor: pointer; }
        .card-inner { position: relative; width: 100%; height: 100%; transition: transform 0.6s; transform-style: preserve-3d; }
        .card.flipped .card-inner { transform: rotateY(180deg); }
        .card-front, .card-back { position: absolute; width: 100%; height: 100%; backface-visibility: hidden; display: flex; flex-direction: column; justify-content: center; align-items: center; border-radius: 25px; padding: 30px; box-sizing: border-box; box-shadow: 0 5px 15px rgba(0,0,0,0.05); }
        .card-front { background: white; border: 1px solid #eee; }
        .card-back { background: var(--primary); color: white; transform: rotateY(180deg); }
        
        .txt-kr { font-size: 1.3rem; line-height: 1.6; word-break: keep-all; color: var(--dark); }
        .txt-en { font-size: 1.4rem; font-weight: bold; line-height: 1.4; }
        .info-box { margin-top: 20px; font-size: 0.85rem; background: rgba(0,0,0,0.1); padding: 5px 15px; border-radius: 20px; }

        .controls { display: flex; gap: 15px; margin-top: 25px; }
        .nav-btn { flex: 1; padding: 15px; border-radius: 12px; border: 1px solid #ddd; background: white; font-weight: bold; cursor: pointer; }
        .hidden { display: none; }
    </style>
</head>
<body oncontextmenu="return false">

    <div id="lock-screen">
        <h2>🔒 ACCESS RESTRICTED</h2>
        <p>비밀번호 4321 입력 후 Enter</p>
        <input type="password" id="pw-input" maxlength="4" onkeypress="handleLogin(event)">
    </div>

    <div id="app" class="hidden" style="width: 100%;">
        <div class="tabs">
            <button class="tab-btn active" id="t-기본동사" onclick="tab('기본동사')">기본동사100</button>
            <button class="tab-btn" id="t-영어회화" onclick="tab('영어회화')">영어회화100</button>
            <button class="tab-btn" id="t-구동사" onclick="tab('구동사')">구동사100</button>
        </div>

        <div class="container" id="menu-view">
            <div class="panel">
                <h2 id="curr-name" style="margin-bottom: 25px;">기본동사100</h2>
                <button class="week-btn" onclick="toggleMode()">Week 1 (Day 1 - Day 5)</button>
                
                <div id="mode-select" class="mode-select">
                    <button class="mode-btn btn-mild" onclick="start('mild')">순한맛<br>(대표/교재1)</button>
                    <button class="mode-btn btn-spicy" onclick="start('spicy')">매운맛<br>(전체 랜덤)</button>
                </div>
            </div>
        </div>

        <div class="container quiz-area" id="quiz-view">
            <div id="prog" style="margin-bottom: 10px; font-weight: bold; color: var(--primary);"></div>
            <div class="card" id="q-card" onclick="this.classList.toggle('flipped')">
                <div class="card-inner">
                    <div class="card-front"><div class="txt-kr" id="k-txt"></div></div>
                    <div class="card-back">
                        <div class="txt-en" id="e-txt"></div>
                        <div class="info-box" id="i-txt"></div>
                    </div>
                </div>
            </div>
            <div class="controls">
                <button class="nav-btn" onclick="move(-1)">이전</button>
                <button class="nav-btn" onclick="move(1)">다음</button>
            </div>
            <button class="nav-btn" style="margin-top:20px; width:100%; border:none; background:none; color:#999;" onclick="home()">목록으로 돌아가기</button>
        </div>
    </div>

    <script>
        const labelMap = { "대표": "대표예제", "교재1": "Model examples", "교재2": "Small Talk", "교재3": "Cases in point", "교재4": "Further studies" };

        // 1. 제공된 예문만으로 구성된 데이터베이스
        const db = {
            "기본동사": [
                {d:"Day001", s:"대표", k:"반려동물 키우시나요?", e:"Do you have any pets?"},
                {d:"Day001", s:"교재1", k:"내 조카는 거북이를 키운다.", e:"My nephew has a turtle."},
                {d:"Day001", s:"교재1", k:"나는 시카고에 사는 친구들이 있다.", e:"I have some friends living in Chicago."},
                {d:"Day001", s:"교재1", k:"남동생과 나는 우애가 돈독하다.", e:"My brother and I have a strong bond."},
                {d:"Day001", s:"교재1", k:"내 생각은 달랐다.", e:"I had a different idea."},
                {d:"Day001", s:"교재1", k:"친구들과 술래잡기하면서 놀던 즐거운 추억이 있다.", e:"I have some fond memories of playing tag with my friends."},
                {d:"Day001", s:"교재2", k:"프로젝트가 끝나서 좋죠?", e:"Are you glad the project is over?"},
                {d:"Day001", s:"교재2", k:"솔직히 시원섭섭해요.", e:"Honestly, I have mixed feelings."},
                {d:"Day001", s:"교재2", k:"음대 대신 의대를 선택한 거 맞지?", e:"You chose med school over music, right?"},
                {d:"Day001", s:"교재2", k:"응, 힘든 결정이었어.", e:"Yeah, it was a tough decision."},
                {d:"Day001", s:"교재2", k:"음악을 계속하지 않은 걸 후회해?", e:"Do you have any regrets about not pursuing music?"},
                {d:"Day001", s:"교재2", k:"조금은 그렇지만, 되도록 생각 안 하려고 해", e:"A few, but I try not to think about it too much."},
                {d:"Day001", s:"교재3", k:"그녀는 공포 영화보다는 영어 강의를 택할 겁니다.", e:"She would choose an English lecture over any horror movie."},
                {d:"Day001", s:"교재3", k:"어떤 사람들은 가구를 살 때 가격보다 품질을 중시한다.", e:"Some people choose quality over price when buying furniture."},
                {d:"Day002", s:"대표", k:"방금 간식을 먹었더니, 배가 별로 안 고파요.", e:"I just had a snack, so I’m not that hungry."},
                {d:"Day002", s:"교재1", k:"나는 아침으로 주로 시리얼을 먹는다.", e:"I usually have cereal for breakfast."},
                {d:"Day002", s:"교재1", k:"저는 스테이크 먹을게요.", e:"I’ll have the steak, please."},
                {d:"Day002", s:"교재1", k:"오늘 점심으로 아무것도 안 먹었어요.", e:"I didn’t have anything for lunch today."},
                {d:"Day002", s:"교재2", k:"수업 전에 뭐 좀 먹었어?", e:"Did you have anything before class?"},
                {d:"Day002", s:"교재2", k:"그냥 바나나 하나 먹었어. 시간이 없었어.", e:"Just a banana. I was in a rush."},
                {d:"Day002", s:"교재3", k:"어제 공원에서 널 봤어.", e:"I saw you at the park yesterday."},
                {d:"Day003", s:"대표", k:"어제 아침에는 별일이 다 있었어요.", e:"I had a weird morning yesterday."},
                {d:"Day003", s:"교재1", k:"우리는 지난주 일본에서 정말 좋은 시간을 보냈어요.", e:"We had a great time in Japan last week."},
                {d:"Day003", s:"교재2", k:"여전히 예술 학교에 진학할 계획이니?", e:"Are you still planning to go to art school?"},
                {d:"Day004", s:"대표", k:"삼성이 오늘 신제품 출시 행사를 열어요.", e:"Samsung is having a new-product launch today."},
                {d:"Day005", s:"대표", k:"오후에는 집안일을 좀 해야 해요.", e:"I have some chores to do in the afternoon."},
                {d:"Day005", s:"교재3", k:"회사에서 모든 경비를 다 대 줬어요.", e:"The company took care of everything."}
            ],
            "영어회화": [
                {d:"Day001", s:"교재1", k:"저는 재택근무 체질이 아니에요.", e:"Working from home isn’t for me."},
                {d:"Day001", s:"대표", k:"재택근무는 저랑 안 맞아요.", e:"Working from home isn’t for me."},
                {d:"Day001", s:"교재2", k:"내가 살 게.", e:"My treat!"},
                {d:"Day002", s:"대표", k:"하루빨리 새 집으로 이사 가고 싶어요.", e:"I can’t wait to move into the new house."},
                {d:"Day002", s:"교재1", k:"다음 에피소드는 어떤 내용일지 궁금해 미치겠어.", e:"I can’t wait to see what the next episode will bring."},
                {d:"Day003", s:"대표", k:"죄송한데 조금 짧게 해 주시겠어요?", e:"Do you mind keeping it a bit short?"},
                {d:"Day003", s:"교재1", k:"오는 길에 커피 좀 사다 줄 수 있나요?", e:"Do you mind grabbing me some coffee on your way?"},
                {d:"Day004", s:"대표", k:"물가가 올라도 너무 올라요.", e:"Everything is getting super expensive."},
                {d:"Day004", s:"교재1", k:"그 여자분 키 엄청 커요.", e:"She is super tall."},
                {d:"Day005", s:"대표", k:"중고 물품 사는 거 어떻게 생각하세요?", e:"How do you feel about buying second-hand items?"},
                {d:"Day005", s:"교재1", k:"교회에 가 보는 게 어때요?", e:"How do you feel about going to church?"},
                {d:"Day005", s:"교재4", k:"모든 게 완전 비싸네.", e:"Everything is totally overpriced."}
            ],
            "구동사": [
                {d:"Day001", s:"교재1", k:"뭔가 앞뒤가 안 맞잖아.", e:"Something doesn’t add up."},
                {d:"Day001", s:"교재1", k:"5 더하기 3은 뭘까?", e:"What’s five plus three?"},
                {d:"Day002", s:"교재1", k:"바람이 워낙 강해서 눈이 다 날아가 버렸더군.", e:"The wind was so strong that it blew the snow away."},
                {d:"Day002", s:"교재1", k:"정말 놀랐어요!", e:"You really blew me away!"},
                {d:"Day003", s:"교재1", k:"제 차가 고속 도로에서 고장이 났습니다.", e:"My car broke down on the highway."},
                {d:"Day003", s:"교재1", k:"결국 감정적으로 무너졌고 사무실에서 울었어요.", e:"I finally broke down and cried in my office."},
                {d:"Day004", s:"교재1", k:"너 Susie랑 헤어졌다는 게 사실이야?", e:"Is it true you broke up with Susie?"},
                {d:"Day004", s:"교재1", k:"여기 신호가 끊겨.", e:"My signal is breaking up down here."},
                {d:"Day005", s:"교재1", k:"프랑스어 복습 좀 해야겠어.", e:"I think I need to brush up on my French."},
                {d:"Day005", s:"교재1", k:"지난 수업에서 배운 내용을 복습해 보겠습니다.", e:"I’d like us to review what we learned last class."},
                {d:"Day005", s:"교재3", k:"최근에 헤어졌다.", e:"After dating for 10 years, we recently broke up."}
            ]
        };

        let currentCurr = "기본동사";
        let quizList = [];
        let curIdx = 0;

        function handleLogin(e) {
            if(e.key === 'Enter') {
                if(e.target.value === "4321") {
                    document.getElementById('lock-screen').style.display = 'none';
                    document.getElementById('app').classList.remove('hidden');
                } else { alert("Access Denied"); e.target.value=""; }
            }
        }

        function tab(name) {
            currentCurr = name;
            document.getElementById('curr-name').innerText = name + "100";
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.toggle('active', b.id === 't-'+name));
            home();
        }

        function toggleMode() {
            const ms = document.getElementById('mode-select');
            ms.style.display = (ms.style.display === 'flex') ? 'none' : 'flex';
        }

        function start(mode) {
            let pool = [...db[currentCurr]];
            if(mode === 'mild') {
                pool = pool.filter(v => v.s === '대표' || v.s === '교재1');
            }
            // 전체 랜덤 셔플 후 10개 추출
            quizList = pool.sort(() => Math.random() - 0.5).slice(0, 10);
            
            if(quizList.length < 1) { alert("문항이 부족합니다."); return; }
            
            curIdx = 0;
            render();
            document.getElementById('menu-view').classList.add('hidden');
            document.getElementById('quiz-view').style.display = 'block';
        }

        function render() {
            document.getElementById('q-card').classList.remove('flipped');
            const item = quizList[curIdx];
            document.getElementById('prog').innerText = `${curIdx + 1} / ${quizList.length}`;
            document.getElementById('k-txt').innerText = item.k;
            document.getElementById('e-txt').innerText = item.e;
            document.getElementById('i-txt').innerText = `${item.d} | ${labelMap[item.s] || item.s}`;
        }

        function move(dir) {
            if(curIdx + dir >= 0 && curIdx + dir < quizList.length) {
                curIdx += dir; render();
            }
        }

        function home() {
            document.getElementById('menu-view').classList.remove('hidden');
            document.getElementById('quiz-view').style.display = 'none';
            document.getElementById('mode-select').style.display = 'none';
        }
    </script>
</body>
</html>
