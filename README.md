<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Private English Quiz</title>
    <style>
        /* 저작권 보호 UI */
        body { -webkit-user-select: none; user-select: none; font-family: 'Pretendard', sans-serif; background: #f0f4f8; margin: 0; display: flex; flex-direction: column; align-items: center; min-height: 100vh; }
        :root { --primary: #3498db; --mild: #2ecc71; --spicy: #e74c3c; --dark: #2c3e50; }
        
        /* 암호화 레이어 (수정됨: 비밀번호 노출 제거) */
        #lock-screen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: var(--dark); display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 9999; color: white; }
        #pw-input { padding: 15px; border-radius: 8px; border: none; margin-top: 20px; width: 180px; font-size: 20px; text-align: center; letter-spacing: 5px; outline: none; }

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
        <h2 style="letter-spacing: 2px;">ACCESS RESTRICTED</h2>
        <p style="opacity: 0.7; font-size: 0.9rem;">콘텐츠 보호를 위해 인증이 필요합니다.</p>
        <input type="password" id="pw-input" maxlength="4" placeholder="Password" onkeypress="handleLogin(event)">
        <p style="margin-top: 20px; font-size: 0.8rem; color: #888;">Enter 4-digit code</p>
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

        const db = {
            "기본동사": [
                {d:"Day001", s:"대표", k:"반려동물 키우시나요?", e:"Do you have any pets?"},
                {d:"Day001", s:"교재1", k:"내 조카는 거북이를 키운다.", e:"My nephew has a turtle."},
                {d:"Day001", s:"교재1", k:"나는 시카고에 사는 친구들이 있다.", e:"I have some friends living in Chicago."},
                {d:"Day001", s:"교재1", k:"남동생과 나는 우애가 돈독하다.", e:"My brother and I have a strong bond."},
                {d:"Day001", s:"교재1", k:"내 생각은 달랐다.", e:"I had a different idea."},
                {d:"Day001", s:"교재1", k:"친구들과 술래잡기하면서 놀던 즐거운 추억이 있다.", e:"I have some fond memories of playing tag with my friends."},
                {d:"Day001", s:"교재2", k:"프로젝트가 끝나서 좋죠?", e:"Are you glad the project is over?"},
                {d:"Day001", s:"교재2", k:"솔직히 시원섭섭해요. 일을 끝내서 홀가분하긴 한데, 팀이 그리울 것 같아요.", e:"Honestly, I have mixed feelings. I’m relieved we finished the work, but I’ll miss the team."},
                {d:"Day001", s:"교재2", k:"음대 대신 의대를 선택한 거 맞지?", e:"You chose med school over music, right?"},
                {d:"Day001", s:"교재2", k:"응, 힘든 결정이었어.", e:"Yeah, it was a tough decision."},
                {d:"Day001", s:"교재2", k:"음악을 계속하지 않은 걸 후회해?", e:"Do you have any regrets about not pursuing music?"},
                {d:"Day001", s:"교재2", k:"조금은 그렇지만, 되도록 생각 안 하려고 해", e:"A few, but I try not to think about it too much."},
                {d:"Day001", s:"교재3", k:"그녀는 공포 영화보다는 영어 강의를 택할 겁니다.", e:"She would choose an English lecture over any horror movie."},
                {d:"Day001", s:"교재3", k:"어떤 사람들은 가구를 살 때 가격보다 품질을 중시한다.", e:"Some people choose quality over price when buying furniture."},
                {d:"Day001", s:"교재3", k:"어떻게 내가 아닌 그 사람을 선택할 수가 있어!", e:"I can’t believe you chose him over me!"},
                {d:"Day001", s:"교재3", k:"장기적인 목표보다 눈앞의 즐거움을 선택하는 사람들이 있다.", e:"Some people choose short-term pleasure over long-term goals."},
                {d:"Day002", s:"대표", k:"방금 간식을 먹었더니, 배가 별로 안 고파요.", e:"I just had a snack, so I’m not that hungry."},
                {d:"Day002", s:"교재1", k:"나는 아침으로 주로 시리얼을 먹는다.", e:"I usually have cereal for breakfast."},
                {d:"Day002", s:"교재1", k:"저는 스테이크 먹을게요.", e:"I’ll have the steak, please."},
                {d:"Day002", s:"교재1", k:"오늘 점심으로 아무것도 안 먹었어요.", e:"I didn’t have anything for lunch today."},
                {d:"Day002", s:"교재1", k:"출발하기 전에 뭐라도 먹어야 해.", e:"You should have something before you leave."},
                {d:"Day002", s:"교재1", k:"피자 먹으면서 영화 보자.", e:"Let’s have some pizza and watch a movie."},
                {d:"Day002", s:"교재2", k:"수업 전에 뭐 좀 먹었어?", e:"Did you have anything before class?"},
                {d:"Day002", s:"교재2", k:"그냥 바나나 하나 먹었어. 시간이 없었어.", e:"Just a banana. I was in a rush."},
                {d:"Day002", s:"교재2", k:"점심은 주로 몇 시에 먹어요?", e:"What time do you usually have lunch?"},
                {d:"Day002", s:"교재2", k:"그렇게 안 바쁘면 대략 12시쯤에요.", e:"Around noon, if I’m not too busy."},
                {d:"Day002", s:"교재2", k:"오, 저도 그런데! 오늘 저랑 뭐 간단히 먹을래요?", e:"Oh, me too! Want to grab something together today?"},
                {d:"Day002", s:"교재2", k:"좋지요! 한국 음식이라면 좋아요.", e:"Sure! I could go for some Korean food."},
                {d:"Day002", s:"교재3", k:"어제 공원에서 널 봤어. (지나가다 우연히 보거나 목격했다는 의미)", e:"I saw you at the park yesterday."},
                {d:"Day002", s:"교재3", k:"어제 공원에서 널 지켜봤어. (나무 뒤에 숨어서 의도적으로 유심히 지켜봤다는 의미)", e:"I watched you at the park yesterday."},
                {d:"Day002", s:"교재3", k:"술집에 있는 TV에서 경기가 나오던데, 집중해서 보지는 않았어.", e:"I saw the game on the TV at the bar, but I didn’t watch it."},
                {d:"Day002", s:"교재3", k:"그 영화를 대형 화면으로 봤어. (대형 화면으로 관람한 경험을 강조)", e:"I saw the movie on the big screen."},
                {d:"Day002", s:"교재3", k:"어젯밤에 넷플릭스 다큐멘터리를 보고 있는데 엄마한테 전화가 왔다.", e:"My mom called while I was watching a Netflix documentary last night."},
                {d:"Day003", s:"대표", k:"어제 아침에는 별일이 다 있었어요.", e:"I had a weird morning yesterday."},
                {d:"Day003", s:"교재1", k:"우리는 지난주 일본에서 정말 좋은 시간을 보냈어요.", e:"We had a great time in Japan last week."},
                {d:"Day003", s:"교재1", k:"나는 힘든 유년 시절을 보냈다.", e:"I had a rough childhood."},
                {d:"Day003", s:"교재1", k:"제가 접속 상태가 안 좋은 것 같습니다.", e:"I think I have a bad connection."},
                {d:"Day003", s:"교재1", k:"저희는 만나면 늘 대화가 즐겁습니다.", e:"We always have such good conversations."},
                {d:"Day003", s:"교재1", k:"그는 비행기를 오래 타서, 잠을 좀 더 자야 해요.", e:"He had a long flight, so he needs some extra sleep."},
                {d:"Day003", s:"교재2", k:"여전히 예술 학교에 진학할 계획이니?", e:"Are you still planning to go to art school?"},
                {d:"Day003", s:"교재2", k:"잘 모르겠어. 전공 때문에 엄마랑 사이가 틀어졌거든.", e:"I’m not sure. My mom and I had a falling-out over my major."},
                {d:"Day003", s:"교재2", k:"안녕, 최근에 잘 안 보이더라.", e:"Hey, I haven’t seen you around lately."},
                {d:"Day003", s:"교재2", k:"응∙∙∙ 한 주 동안 마음이 좀 힘들었거든.", e:"I know... I had a rough week, emotionally."},
                {d:"Day003", s:"교재2", k:"무슨 일인지 이야기해 줄래?", e:"Want to talk about it?"},
                {d:"Day003", s:"교재2", k:"음, 다음에. 우선 머리 좀 식힐 시간이 필요해서.", e:"Maybe later. I just need some time to clear my head."},
                {d:"Day003", s:"교재3", k:"내가 어렸을 때, 우리 이웃이 나무에서 떨어졌어요.", e:"My neighbor fell out of a tree when I was young."},
                {d:"Day003", s:"교재3", k:"아무래도 버스에서 내리다 핸드폰이 주머니에서 빠진 것 같아.", e:"I think my phone fell out of my pocket while I was getting off the bus."},
                {d:"Day003", s:"교재3", k:"그거 못 들었어? 그 두 사람 크리스마스 이후로 사이가 완전히 틀어졌대.", e:"Didn’t you hear? Those two had a major falling-out after Christmas."},
                {d:"Day003", s:"교재3", k:"제 동업자와 저는 돈 문제로 사이가 멀어졌어요.", e:"My business partner and I had a falling-out over money."},
                {d:"Day004", s:"대표", k:"삼성이 오늘 신제품 출시 행사를 열어요.", e:"Samsung is having a new-product launch today."},
                {d:"Day004", s:"교재1", k:"제가 이번 주말에 집들이를 합니다.", e:"I’m having a housewarming party this weekend."},
                {d:"Day004", s:"교재1", k:"백화점들이 이번 주에 일제히 세일을 한다.", e:"The department stores are all having sales this week."},
                {d:"Day004", s:"교재1", k:"우리는 6월에 코엑스에서 채용 박람회를 열었다.", e:"We had a job fair at COEX in June."},
                {d:"Day004", s:"교재1", k:"이번 주말에 모임이 있으니 오고 싶으면 오렴.", e:"We’re having a get-together this weekend if you want to join."},
                {d:"Day004", s:"교재1", k:"그 제과점에서 오늘 원 플러스 원 행사를 한다.", e:"The bakery has a buy-one-get-one deal today."},
                {d:"Day004", s:"교재2", k:"여행용 가방을 새로 사야 해.", e:"I need to buy some new luggage."},
                {d:"Day004", s:"교재2", k:"아, 코스트코에서 지금 샘소나이트 여행 가방 세일 중인데.", e:"Oh, Costco is having a sale on Samsonite luggage right now."},
                {d:"Day004", s:"교재2", k:"준호 씨, 브라질 생활 어때요?", e:"How is life in Brazil treating you, Junho?"},
                {d:"Day004", s:"교재2", k:"쉽지는 않지만 나아지고 있습니다.", e:"It’s not easy, but it’s getting better."},
                {d:"Day004", s:"교재2", k:"지금쯤이면 친구도 사귀었겠어요?", e:"Do you have any friends yet?"},
                {d:"Day004", s:"교재2", k:"네, 아파트 단지에 한국인 가정이 몇 있는데 매주 금요일마다 바비큐 파티를 해요.", e:"Yeah, a couple of Korean families live in my apartment building and we have barbeques every Friday."},
                {d:"Day004", s:"교재3", k:"퇴근은 한 거지?", e:"Did you leave work yet?"},
                {d:"Day004", s:"교재3", k:"엄마한테 전화한 거지?", e:"Did you call your mom yet?"},
                {d:"Day004", s:"교재3", k:"저녁은 먹은 거지?", e:"Did you eat dinner yet?"},
                {d:"Day004", s:"교재3", k:"집안일은 다 한 거지?", e:"Did you finish your chores yet?"},
                {d:"Day004", s:"교재3", k:"당신 이미 집 나선 거지?", e:"Did you leave yet?"},
                {d:"Day004", s:"교재3", k:"아니, 왜?", e:"No, why?"},
                {d:"Day004", s:"교재3", k:"지갑을 두고 왔네. 아래쪽으로 던져 줄 수 있을까?", e:"I forgot my wallet. Could you throw it down to me, please?"},
                {d:"Day004", s:"교재3", k:"알았어.", e:"OK."},
                {d:"Day005", s:"대표", k:"오후에는 집안일을 좀 해야 해요.", e:"I have some chores to do in the afternoon."},
                {d:"Day005", s:"교재1", k:"할 일이 산더미같이 쌓여 있다.", e:"I have tons of work to do."},
                {d:"Day005", s:"교재1", k:"답장해야 할 이메일이 몇 개 있다.", e:"I have a few emails to reply to."},
                {d:"Day005", s:"교재1", k:"오늘 오후에 몇 가지 볼일이 좀 있다.", e:"I have some errands to run this afternoon."},
                {d:"Day005", s:"교재1", k:"금요일까지 프로젝트 하나를 마무리해야 한다.", e:"We have a project to finish by Friday."},
                {d:"Day005", s:"교재1", k:"크리스마스 (선물) 쇼핑을 좀 해야 한다.", e:"I have some Christmas shopping to do."},
                {d:"Day005", s:"교재2", k:"안녕하세요, 제리. 우리 방금 프로젝트 진행 상황에 대해 이야기하고 있었어요.", e:"Good morning, Jerry. We were just talking about the project updates."},
                {d:"Day005", s:"교재2", k:"네, 늦어서 죄송해요. 뭐 좀 처리할 게 있었어요.", e:"Yes, sorry I’m late. I had something to take care of."},
                {d:"Day005", s:"교재2", k:"오늘 바빠? 점심 먹자.", e:"Are you busy today? Let’s have lunch."},
                {d:"Day005", s:"교재2", k:"안 돼. 우체국 가는 길이야.", e:"I can’t. I’m on my way to the post office."},
                {d:"Day005", s:"교재2", k:"정말? 왜?", e:"Really? Why?"},
                {d:"Day005", s:"교재2", k:"중고나라에서 바지 다섯 개를 팔아서, 택배 부칠 게 많아.", e:"I just sold five pairs of pants on Joonggonara, so I have a lot of packages to ship."},
                {d:"Day005", s:"교재3", k:"이번 주말에 친구 고양이를 돌봐 주기로 했어.", e:"I’m taking care of my friend’s cat this weekend."},
                {d:"Day005", s:"교재3", k:"저희 아빠는 건강을 좀 더 신경 쓰셔야 해요.", e:"My dad needs to take better care of his health."},
                {d:"Day005", s:"교재3", k:"휴가 가기 전에 처리해야 할 일이 좀 있어.", e:"I have some work to take care of before I start my vacation."},
                {d:"Day005", s:"교재3", k:"저희 엄마는 주로 오후 시간에 정원을 손질하십니다.", e:"My mom usually takes care of her garden in the afternoons."},
                {d:"Day005", s:"교재3", k:"회사에서 모든 경비를 다 대 줬어요.", e:"The company took care of everything."}
            ],
            "영어회화": [
                {d:"Day001", s:"교재1", k:"저는 재택근무 체질이 아니에요. 늘 딴짓하게 되거든요", e:"Working from home isn’t for me. I always get distracted."},
                {d:"Day001", s:"교재1", k:"소개팅은 저랑 안 맞아요.", e:"Going on blind dates isn’t for me."},
                {d:"Day001", s:"교재1", k:"노트북은 저랑 좀 안 맞아요.", e:"Laptops aren’t really for me."},
                {d:"Day001", s:"교재1", k:"전기차는 좀 별로예요.", e:"Electric cars aren’t for me."},
                {d:"Day001", s:"교재1", k:"그런 남자는 나는 별로야.", e:"Guys like him aren’t really for me."},
                {d:"Day001", s:"교재2", k:"우리 나가서 맛난 회 먹을까? 내가 살 게.", e:"Why don’t we go out and get some nice sashimi? My treat!"},
                {d:"Day001", s:"교재2", k:"난 회를 별로 안 좋아해.", e:"Raw fish just isn’t for me."},
                {d:"Day001", s:"교재2", k:"미국 프로그램이 체질에 안 맞아요.", e:"American shows aren’t for me."},
                {d:"Day001", s:"교재2", k:"저는 가르치는 거랑 잘 안 맞아요.", e:"Teaching isn’t really for me."},
                {d:"Day001", s:"교재3", k:"나랑은 별로 안 맞더라고", e:"Turns out it’s not really for me."},
                {d:"Day001", s:"교재4", k:"학과장을 안 하는 게 너랑 맞는 거야", e:"Not being chair suits you."},
                {d:"Day001", s:"교재4", k:"혼자 일하는 건 나랑 안 맞는다는 걸 느꼈어.", e:"Working on my own doesn’t really suit me."},
                {d:"Day001", s:"교재4", k:"재택근무는 내 체질이 아니야.", e:"Working from home doesn’t really work for me."},
                {d:"Day001", s:"대표", k:"재택근무는 저랑 안 맞아요.", e:"Working from home isn’t for me."},
                {d:"Day002", s:"교재1", k:"다음 에피소드는 어떤 내용일지 궁금해 미치겠어.", e:"I can’t wait to see what the next episode will bring."},
                {d:"Day002", s:"교재1", k:"아내가 제 선물을 개봉할 때 어떤 표정일지 궁금해 죽겠습니다.", e:"I can’t wait to see the look on my wife’s face when she opens my gift."},
                {d:"Day002", s:"교재1", k:"이 프로젝트가 빨리 끝났으면 좋겠어요.", e:"I can’t wait to be done with this project."},
                {d:"Day002", s:"교재1", k:"여보, 어서 먹고 싶어.", e:"That dinner smells delicious, honey. I can’t wait."},
                {d:"Day002", s:"교재1", k:"이곳에서도 어서 개봉했으면 좋겠다.", e:"I can’t wait for it to come out here."},
                {d:"Day002", s:"교재2", k:"응! 어서 보고 싶어.", e:"Yes! I can’t wait to see it."},
                {d:"Day002", s:"교재2", k:"어서 끝내고 뭔가 다른 걸로 넘어가고 싶어요.", e:"I can’t wait to finish it and finally move on to something else."},
                {d:"Day002", s:"교재2", k:"내가 자기 주려고 이걸 만든 걸 알면 어떤 표정일까 궁금해 죽겠어.", e:"I can’t wait to see the look on her face."},
                {d:"Day002", s:"교재3", k:"신형 그랜저를 어서 보고 싶네요.", e:"I can’t wait to get a glimpse of the new Grandeur."},
                {d:"Day002", s:"교재4", k:"그 여성분 어서 만나 보고 싶어.", e:"I’m really anxious to meet her."},
                {d:"Day002", s:"교재4", k:"어서 집에 가서 선물을 개봉해 보고 싶다.", e:"I’m anxious to get home to open my presents."},
                {d:"Day002", s:"교재4", k:"하루빨리 함께 일하고 싶습니다.", e:"I look forward to working with you."},
                {d:"Day002", s:"교재4", k:"아이유도 나온다고 하니까 더 기대된다.", e:"I was looking forward to the concert, but now even more so, since I heard IU will be there."},
                {d:"Day002", s:"대표", k:"하루빨리 새 집으로 이사 가고 싶어요.", e:"I can’t wait to move into the new house."},
                {d:"Day003", s:"교재1", k:"제가 마지막 남은 피자 한 조각 먹어도 될까요?", e:"Do you mind if I finish off the last piece of pizza?"},
                {d:"Day003", s:"교재1", k:"오는 길에 커피 좀 사다 줄 수 있나요?", e:"Do you mind grabbing me some coffee on your way?"},
                {d:"Day003", s:"교재1", k:"짧게 해 주실 수 있을까요?", e:"Do you mind keeping it short?"},
                {d:"Day003", s:"교재1", k:"에어컨 좀 약하게 하면 안 될까요?", e:"Do you mind turning down the air-conditioning?"},
                {d:"Day003", s:"교재1", k:"개인적인 질문 하나 해도 될까요?", e:"Do you mind if I ask you a personal question?"},
                {d:"Day003", s:"교재2", k:"죄송한데, 회의를 금요일로 옮겨도 될까요?", e:"Do you mind if we move the meeting to Friday?"},
                {d:"Day003", s:"교재2", k:"시리얼 상자들 중 하나를 내려 줄 수 있을까요?", e:"Do you mind grabbing me one of those cereal boxes?"},
                {d:"Day003", s:"교재2", k:"제가 그쪽 사무실로 가도 상관없습니다.", e:"I don’t mind coming over to your office."},
                {d:"Day003", s:"교재3", k:"괜찮으시면 혹시 모르니까 이번에는 2시 30분에 시작해도 될는지요?", e:"If you don’t mind, could we start at 2:30 this time, just to be safe?"},
                {d:"Day003", s:"교재4", k:"미안하지만 좀 도와주실 수 있을까요?", e:"Would you mind giving me a hand?"},
                {d:"Day003", s:"교재4", k:"잠깐 나가서 간단히 뭐 좀 먹고 와도 될까요?", e:"Would you mind if I stepped out for a moment and grabbed a bite to eat?"},
                {d:"Day003", s:"대표", k:"죄송한데 조금 짧게 해 주시겠어요?", e:"Do you mind keeping it a bit short?"},
                {d:"Day004", s:"교재1", k:"그 여자분 키 엄청 커요.", e:"She is super tall."},
                {d:"Day004", s:"교재1", k:"그 사람이 무지 바쁘거나, 아니면 저에 대한 관심이 식고 있는 거겠죠.", e:"Either he has been super busy, or he is losing interest in me."},
                {d:"Day004", s:"교재1", k:"제가 요즘 이사 준비 때문에 엄청 바빴어요.", e:"I’ve been super busy with my upcoming move."},
                {d:"Day004", s:"교재1", k:"연세 있으신 분치고는 몸매가 너무 좋으시네요.", e:"Wow. You’re in super good shape for an old guy."},
                {d:"Day004", s:"교재1", k:"서울은 어디라도 다 너무 비싸. 근데 후암동은 상대적으로 저렴한 편이지", e:"All the neighborhoods in Seoul are super expensive, but Huam-dong is relatively cheap"},
                {d:"Day004", s:"교재2", k:"그러게. 요새 물가가 너무너무 비싸.", e:"Yeah. Everything is getting super expensive."},
                {d:"Day004", s:"교재2", k:"지금 제철이니 엄청 쌀 거야.", e:"They should be super cheap since they’re in-season."},
                {d:"Day004", s:"교재2", k:"가을이 점점 짧아지고는 있는데 올해는 엄청 길다.", e:"Autumn has been getting shorter, but this year, it’s been super long."},
                {d:"Day004", s:"교재3", k:"이번 침대 프레임 조립은 정말 쉽더군요.", e:"I’m normally really bad at following instructions, but this bed frame was super easy to put together."},
                {d:"Day004", s:"교재4", k:"그 삼겹살집은 맛은 괜찮은 편인데 가격이 상당히 비싸다.", e:"The pork belly place is pretty good, but it’s quite expensive."},
                {d:"Day004", s:"교재4", k:"당신은 영어를 상당히 잘하는군요.", e:"Your English is quite good, especially considering you’ve never lived abroad."},
                {d:"Day004", s:"대표", k:"물가가 올라도 너무 올라요.", e:"Everything is getting super expensive."},
                {d:"Day005", s:"교재1", k:"중매업체에 등록해 보는 게 어때요?", e:"How do you feel about signing up for a matchmaking service?"},
                {d:"Day005", s:"교재1", k:"교회에 가 보는 게 어때요?", e:"How do you feel about going to church?"},
                {d:"Day005", s:"교재1", k:"등산 모임에 가입해 보는 게 어떨까요?", e:"How do you feel about joining a hiking club?"},
                {d:"Day005", s:"교재1", k:"성형 수술 하는 거 어떻게 생각하세요?", e:"How do you feel about plastic surgery?"},
                {d:"Day005", s:"교재2", k:"그런 팀에 합류하는 기분이 어떠신가요?", e:"How do you feel about joining a team when the coach is your ex-teammate?"},
                {d:"Day005", s:"교재2", k:"저녁 먹고 우리 집에 가서 <컨저링> 볼까 하는데. 공포 영화 어때?", e:"After dinner, I was thinking we could go to my place and watch The Conjuring. How do you feel about horror movies?"},
                {d:"Day005", s:"교재3", k:"Frank 팀장님 밑에서 일하니까 어떤가요?", e:"How do you feel about working under Frank?"},
                {d:"Day005", s:"교재4", k:"모든 게 완전 비싸네.", e:"Everything is totally overpriced."},
                {d:"Day005", s:"교재4", k:"완전 깜박했어.", e:"It totally slipped my mind."},
                {d:"Day005", s:"교재4", k:"양복 입으니까 완전 딴 사람 같네.", e:"You look totally different in a suit."},
                {d:"Day005", s:"교재4", k:"완전 괜찮아.", e:"That’s totally fine."},
                {d:"Day005", s:"교재4", k:"괜히 고치느라 애쓰지 마. 완전히 고장이 났으니까.", e:"Don’t bother trying to fix it. It’s totally broken."},
                {d:"Day005", s:"교재4", k:"그 영화는 무조건 아이맥스로 봐야 해.", e:"You should totally go see the movie in IMAX."},
                {d:"Day005", s:"교재4", k:"난 초밥이 너무 땡겨.", e:"I could totally go for some sushi."},
                {d:"Day005", s:"대표", k:"중고차 같은 중고 물품 사는 거 어떻게 생각하세요?", e:"How do you feel about buying second-hand items?"}
            ],
            "구동사": [
                {d:"Day001", s:"교재1", k:"뭔가 앞뒤가 안 맞잖아.", e:"Something doesn’t add up."},
                {d:"Day001", s:"교재1", k:"자, 이 숫자들을 더해 보자. 5 더하기 3은 뭘까?", e:"Let’s add up these numbers now. What’s five plus three?"},
                {d:"Day001", s:"교재1", k:"월 10만 원도 쌓이면 4년 후에 거의 5백만 원이 된다.", e:"Just 100,000 won a month will add up to almost 5 million won in four years."},
                {d:"Day001", s:"교재1", k:"그의 이야기에는 앞뒤가 맞지 않는 것이 있어요.", e:"There’s something about his story that doesn’t add up."},
                {d:"Day001", s:"교재2", k:"한 달에 백 달러도 5년이면 6천 달러야.", e:"A hundred bucks a month will add up to $6,000 in five years."},
                {d:"Day001", s:"교재2", k:"지난 분기 매출 수치를 합해서 목표치와 비교해 주시겠어요?", e:"Can you add up the sales figures from last quarter and compare them to our targets?"},
                {d:"Day001", s:"교재2", k:"이건 말이 안 돼, Gina야.", e:"This doesn’t add up, Gina."},
                {d:"Day001", s:"교재3", k:"저를 믿어 보세요. 이게 쌓이면 정말 큽니다.", e:"Trust me, it really adds up."},
                {d:"Day002", s:"교재1", k:"바람이 워낙 강해서 눈이 다 날아가 버렸더군.", e:"The wind was so strong that it blew the snow away."},
                {d:"Day002", s:"교재1", k:"그냥 압축공기를 이용해서 먼지를 날려 버린답니다.", e:"I just use compressed air to blow the dust away."},
                {d:"Day002", s:"교재1", k:"자칫 날아갈 수도 있어!", e:"You might get blown away!"},
                {d:"Day002", s:"교재1", k:"이전 모델들과는 완전히 다르더라고.", e:"When I first saw their new tablet, I was blown away."},
                {d:"Day002", s:"교재1", k:"우와, 프레젠테이션 정말 유익했어요. 정말 놀랐어요!", e:"Wow, your presentation was so informative. You really blew me away!"},
                {d:"Day002", s:"교재1", k:"바람이 많이 부는 날씨는 불쾌하지만, 적어도 미세 먼지를 날려 버리긴 하지.", e:"Windy weather is unpleasant, but at least it blows away all the microdust."},
                {d:"Day002", s:"교재1", k:"모자가 강풍에 날아가 버렸다.", e:"My hat blew away in the strong wind."},
                {d:"Day002", s:"교재1", k:"제 동생이 영어를 너무 잘해서 정말 놀랐어요.", e:"I was really blown away by his English."},
                {d:"Day002", s:"교재2", k:"바람이 너무 세서 자동차도 날아갔나 보더라고.", e:"Apparently, the wind was so strong that it even blew away cars."},
                {d:"Day002", s:"교재2", k:"주연 배우의 연기가 정말 인상적이더라고.", e:"I was absolutely blown away by the main actor’s performance."},
                {d:"Day002", s:"교재2", k:"<런닝맨>에서 외국인들과 대화하는 거 봤는데 정말 대단하더라.", e:"I was completely blown away when I saw him chatting with foreigners on Running Man."},
                {d:"Day002", s:"교재3", k:"음식점에 가면 얼마나 올랐는지 못 느꼈어? 메뉴 볼 때마다 깜짝 놀라.", e:"Haven’t you noticed how high the prices have gotten at restaurants? I’m blown away every time I look at a menu."},
                {d:"Day003", s:"교재1", k:"몇 년을 매일 썼더니 컴퓨터가 결국 고장이 났다.", e:"The computer finally broke down after using it daily for years."},
                {d:"Day003", s:"교재1", k:"협상이 결렬된 것은 불가피했습니다.", e:"It was inevitable that the negotiation over working hours broke down."},
                {d:"Day003", s:"교재1", k:"제가 다시는 프로 선수로 뛸 수 없다고 했을 때 저는 무너졌습니다.", e:"When the doctor said I could never play professionally again, I broke down."},
                {d:"Day003", s:"교재1", k:"스케줄을 세부적으로 말씀드릴게요.", e:"Let me break down the schedule."},
                {d:"Day003", s:"교재1", k:"미안한데 다시 한번 자세히 설명해 주시겠어요?", e:"Would you mind going back and breaking those down?"},
                {d:"Day003", s:"교재1", k:"담배꽁초가 분해되는 데 18개월에서 10년이 걸리는 거 알았어?", e:"Did you know that cigarette butts take between 18 months and 10 years to break down?"},
                {d:"Day003", s:"교재1", k:"제 차가 고속 도로에서 고장이 났습니다.", e:"My car broke down on the highway."},
                {d:"Day003", s:"교재1", k:"돈 이야기가 나오면 이런 대화가 깨집니다.", e:"Those talks always break down once money comes up."},
                {d:"Day003", s:"교재1", k:"결국 감정적으로 무너졌고 사무실에서 울었어요.", e:"I finally broke down and cried in my office."},
                {d:"Day003", s:"교재1", k:"표현 하나하나를 살펴봤습니다.", e:"I broke it down into parts and looked at the phrases one by one."},
                {d:"Day003", s:"교재1", k:"동물성 단백질은 체내에서 분해되는 데 더 많은 에너지를 필요로 한다.", e:"Protein from meat requires more energy for your body to break down."},
                {d:"Day003", s:"교재2", k:"대학과 대학원을 다니면서 사용하던 컴퓨터가 결국 고장이 났거든요.", e:"My old computer finally broke down after using it through college and grad school."},
                {d:"Day003", s:"교재2", k:"정신적으로 완전히 무너져서 아무 말도 안 나왔어.", e:"I just broke down completely and couldn’t even get words out."},
                {d:"Day003", s:"교재2", k:"아파트 청약 제도를 자세히 좀 설명해 주시겠어요?", e:"Could you break it down for me?"},
                {d:"Day003", s:"교재3", k:"버려진 후에 분해되는 데 정말 오래 걸려.", e:"They take forever to break down once they’re thrown away."},
                {d:"Day003", s:"교재3", k:"나도 (빨대가) 눅눅해지기 전에 서둘러 음료를 마시게 되더라.", e:"I always end up rushing to finish my drink before that happens."},
                {d:"Day004", s:"교재1", k:"너 Susie랑 헤어졌다는 게 사실이야?", e:"Is it true you broke up with Susie?"},
                {d:"Day004", s:"교재1", k:"사실 Susie가 헤어지자고 해서 헤어진 거야.", e:"She broke up with me, actually."},
                {d:"Day004", s:"교재1", k:"구름이 서서히 걷히고 해가 더 밝아졌다.", e:"The clouds gradually broke up and the sun got brighter."},
                {d:"Day004", s:"교재1", k:"닭고기를 한 입 크기로 찢은 다음, 샐러드에 넣어 섞으세요.", e:"First, use two forks to break up the chicken into bite-size pieces."},
                {d:"Day004", s:"교재1", k:"여기 (지하라서) 신호가 끊겨.", e:"My signal is breaking up down here."},
                {d:"Day004", s:"교재1", k:"사람들은 매일 같이 헤어지잖아. 너무 힘들게 받아들이지 마!", e:"People break up every day. Don’t take it so hard!"},
                {d:"Day004", s:"교재1", k:"자, 얘들아. 3명씩 조를 나누어라.", e:"OK, class. I need you to break up into groups of three."},
                {d:"Day004", s:"교재1", k:"우리 아빠 말로는 오노 요코 때문에 비틀즈가 해체됐다고 한다.", e:"My dad says Yoko Ono broke up The Beatles."},
                {d:"Day004", s:"교재1", k:"업무 흐름이 지루해지지 않습니다.", e:"I like to break up my day with various kinds of tasks."},
                {d:"Day004", s:"교재1", k:"네 말이 끊겨서 들려. 엘리베이터 안인 거야?", e:"You are breaking up. Are you in an elevator?"},
                {d:"Day004", s:"교재2", k:"좀 더 짧은 클립으로 쪼개야 해.", e:"You need to break it up into smaller clips."},
                {d:"Day004", s:"교재2", k:"잠시만, 잘 안 들려. 신호가 계속 끊기네.", e:"Hold on, I can’t hear you. Your signal keeps breaking up."},
                {d:"Day004", s:"교재2", k:"밴드가 해산하는 건 미나 잘못이야. 너무 이기적이야.", e:"I heard Mina is going solo, so it’s her fault the band is breaking up."},
                {d:"Day004", s:"교재3", k:"관계가 흔들렸고 최근에 헤어지게 되었다.", e:"Their relationship eventually got rocky, and they recently broke up."},
                {d:"Day004", s:"교재3", k:"민호랑 헤어진 마당에 서울에 계속 있어야 할 이유가 없어.", e:"Now that Minho and I broke up, I don’t have any reason to stay in Seoul."},
                {d:"Day005", s:"교재1", k:"프랑스어 복습 좀 해야겠어.", e:"I am travelling to Paris next month, so I think I need to brush up on my French."},
                {d:"Day005", s:"교재1", k:"지난 수업에서 배운 내용을 복습해 보겠습니다.", e:"I’d like us to review what we learned last class."},
                {d:"Day005", s:"교재1", k:"지난 학기에 배운 내용을 다시 한번 복습해 보겠습니다.", e:"I’d like us to brush up on what we learned last semester."},
                {d:"Day005", s:"교재1", k:"빵 굽는 것을 연습하려면 요리책 읽어 봐야겠다.", e:"I’m going to read a cookbook so that I can brush up on my baking skills."},
                {d:"Day005", s:"교재1", k:"잊어버릴 만하면 (이 책에 담긴) 구동사를 복습하셔야 합니다.", e:"You should brush up on these phrasal verbs from time to time."},
                {d:"Day005", s:"교재1", k:"한동안 안 치던 기타도 연습하고 있어요.", e:"I’ve been brushing up on my guitar playing."},
                {d:"Day005", s:"교재2", k:"가기 전에 한국 역사 복습 좀 해야겠어.", e:"I want to brush up on my Korean history before we go."},
                {d:"Day005", s:"교재2", k:"발표 스킬을 좀 가다듬고 있어요.", e:"I’ve been brushing up on my presentation skills this week."},
                {d:"Day005", s:"교재2", k:"요리 연습을 좀 해야 해요. 안 만들어 본 지 10년도 넘었거든요.", e:"I need to brush up on my cooking skills."},
                {d:"Day005", s:"교재3", k:"고등학교 때 만난 여자 친구와 10년을 사귀었는데 최근에 헤어졌다.", e:"After dating for 10 years, we recently broke up."},
                {d:"Day005", s:"교재3", k:"작업 멘트를 연습해 보는 것이 내가 생각할 수 있는 전부였다.", e:"All I could think to do was brush up on some pickup lines."}
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
                } else { alert("비밀번호가 틀렸습니다."); e.target.value=""; }
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
            // 셔플 후 딱 10개만 추출
            quizList = pool.sort(() => Math.random() - 0.5).slice(0, 10);
            
            if(quizList.length < 1) { alert("문항 데이터가 부족합니다."); return; }
            
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
