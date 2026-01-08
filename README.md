<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Speed Quiz Protected</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <style>
        .perspective-1000 { perspective: 1000px; }
        .preserve-3d { transform-style: preserve-3d; }
        .backface-hidden { backface-visibility: hidden; }
        .rotate-y-180 { transform: rotateY(180deg); }
        @font-face {
            font-family: 'Pretendard';
            src: url('https://fastly.jsdelivr.net/gh/Project-Noonnu/noonfonts_2107@1.1/Pretendard-Regular.woff') format('woff');
        }
        body { 
            font-family: 'Pretendard', sans-serif; 
            -webkit-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
            user-select: none;
            background-color: #f8fafc;
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect } = React;

        // --- 접속 비밀번호 설정 ---
        const SECRET_PASSWORD = "1234"; 

        const QUIZ_DATA = {
            basic_verbs: [
                { day: 1, source: "대표", kr: "반려동물 키우시나요?", en: "Do you have any pets?" },
                { day: 1, source: "교재1", kr: "내 조카는 거북이를 키운다.", en: "My nephew has a turtle." },
                { day: 1, source: "교재1", kr: "나는 시카고에 사는 친구들이 있다.", en: "I have some friends living in Chicago." },
                { day: 1, source: "교재1", kr: "남동생과 나는 우애가 돈독하다.", en: "My brother and I have a strong bond." },
                { day: 1, source: "교재1", kr: "내 생각은 달랐다.", en: "I had a different idea." },
                { day: 1, source: "교재1", kr: "친구들과 술래잡기하면서 놀던 즐거운 추억이 있다.", en: "I have some fond memories of playing tag with my friends." },
                { day: 1, source: "교재2", kr: "프로젝트가 끝나서 좋죠?", en: "Are you glad the project is over?" },
                { day: 1, source: "교재2", kr: "솔직히 시원섭섭해요. 일을 끝내서 홀가분하긴 한데, 팀이 그리울 것 같아요.", en: "Honestly, I have mixed feelings. I’m relieved we finished the work, but I’ll miss the team." },
                { day: 1, source: "교재2", kr: "음대 대신 의대를 선택한 거 맞지?", en: "You chose med school over music, right?" },
                { day: 1, source: "교재2", kr: "응, 힘든 결정이었어.", en: "Yeah, it was a tough decision." },
                { day: 1, source: "교재2", kr: "음악을 계속하지 않은 걸 후회해?", en: "Do you have any regrets about not pursuing music?" },
                { day: 1, source: "교재2", kr: "조금은 그렇지만, 되도록 생각 안 하려고 해", en: "A few, but I try not to think about it too much." },
                { day: 1, source: "교재3", kr: "그녀는 공포 영화보다는 영어 강의를 택할 겁니다.", en: "She would choose an English lecture over any horror movie." },
                { day: 1, source: "교재3", kr: "어떤 사람들은 가구를 살 때 가격보다 품질을 중시한다.", en: "Some people choose quality over price when buying furniture." },
                { day: 1, source: "교재3", kr: "어떻게 내가 아닌 그 사람을 선택할 수가 있어!", en: "I can’t believe you chose him over me!" },
                { day: 1, source: "교재3", kr: "장기적인 목표보다 눈앞의 즐거움을 선택하는 사람들이 있다.", en: "Some people choose short-term pleasure over long-term goals." },
                { day: 2, source: "대표", kr: "방금 간식을 먹었더니, 배가 별로 안 고파요.", en: "I just had a snack, so I’m not that hungry." },
                { day: 2, source: "교재1", kr: "나는 아침으로 주로 시리얼을 먹는다.", en: "I usually have cereal for breakfast." },
                { day: 2, source: "교재1", kr: "저는 스테이크 먹을게요.", en: "I’ll have the steak, please." },
                { day: 2, source: "교재1", kr: "오늘 점심으로 아무것도 안 먹었어요.", en: "I didn’t have anything for lunch today." },
                { day: 2, source: "교재1", kr: "출발하기 전에 뭐라도 먹어야 해.", en: "You should have something before you leave." },
                { day: 2, source: "교재1", kr: "피자 먹으면서 영화 보자.", en: "Let’s have some pizza and watch a movie." },
                { day: 2, source: "교재2", kr: "수업 전에 뭐 좀 먹었어?", en: "Did you have anything before class?" },
                { day: 2, source: "교재2", kr: "그냥 바나나 하나 먹었어. 시간이 없었어.", en: "Just a banana. I was in a rush." },
                { day: 2, source: "교재2", kr: "점심은 주로 몇 시에 먹어요?", en: "What time do you usually have lunch?" },
                { day: 2, source: "교재2", kr: "그렇게 안 바쁘면 대략 12시쯤에요.", en: "Around noon, if I’m not too busy." },
                { day: 2, source: "교재2", kr: "오, 저도 그런데! 오늘 저랑 뭐 간단히 먹을래요?", en: "Oh, me too! Want to grab something together today?" },
                { day: 2, source: "교재2", kr: "좋지요! 한국 음식이라면 좋아요.", en: "Sure! I could go for some Korean food." },
                { day: 2, source: "교재3", kr: "어제 공원에서 널 봤어.", en: "I saw you at the park yesterday." },
                { day: 2, source: "교재3", kr: "어제 공원에서 널 지켜봤어.", en: "I watched you at the park yesterday." },
                { day: 2, source: "교재3", kr: "술집에 있는 TV에서 경기가 나오던데, 집중해서 보지는 않았어.", en: "I saw the game on the TV at the bar, but I didn’t watch it." },
                { day: 2, source: "교재3", kr: "그 영화를 대형 화면으로 봤어.", en: "I saw the movie on the big screen." },
                { day: 2, source: "교재3", kr: "어젯밤에 넷플릭스 다큐멘터리를 보고 있는데 엄마한테 전화가 왔다.", en: "My mom called while I was watching a Netflix documentary last night." },
                { day: 3, source: "대표", kr: "어제 아침에는 별일이 다 있었어요.", en: "I had a weird morning yesterday." },
                { day: 3, source: "교재1", kr: "우리는 지난주 일본에서 정말 좋은 시간을 보냈어요.", en: "We had a great time in Japan last week." },
                { day: 3, source: "교재1", kr: "나는 힘든 유년 시절을 보냈다.", en: "I had a rough childhood." },
                { day: 3, source: "교재1", kr: "(줌 회의에서) 제가 접속 상태가 안 좋은 것 같습니다.", en: "I think I have a bad connection." },
                { day: 3, source: "교재1", kr: "(친한 친구를 언급하며) 저희는 만나면 늘 대화가 즐겁습니다.", en: "We always have such good conversations." },
                { day: 3, source: "교재1", kr: "그는 비행기를 오래 타서, 잠을 좀 더 자야 해요.", en: "He had a long flight, so he needs some extra sleep." },
                { day: 3, source: "교재2", kr: "여전히 예술 학교에 진학할 계획이니?", en: "Are you still planning to go to art school?" },
                { day: 3, source: "교재2", kr: "잘 모르겠어. 전공 때문에 엄마랑 사이가 틀어졌거든.", en: "I’m not sure. My mom and I had a falling-out over my major." },
                { day: 3, source: "교재2", kr: "안녕, 최근에 잘 안 보이더라.", en: "Hey, I haven’t seen you around lately." },
                { day: 3, source: "교재2", kr: "응∙∙∙ 한 주 동안 마음이 좀 힘들었거든.", en: "I know... I had a rough week, emotionally." },
                { day: 3, source: "교재2", kr: "무슨 일인지 이야기해 줄래?", en: "Want to talk about it?" },
                { day: 3, source: "교재2", kr: "음, 다음에. 우선 머리 좀 식힐 시간이 필요해서.", en: "Maybe later. I just need some time to clear my head." },
                { day: 3, source: "교재3", kr: "내가 어렸을 때, 우리 이웃이 나무에서 떨어졌어요.", en: "My neighbor fell out of a tree when I was young." },
                { day: 3, source: "교재3", kr: "아무래도 버스에서 내리다 핸드폰이 주머니에서 빠진 것 같아.", en: "I think my phone fell out of my pocket while I was getting off the bus." },
                { day: 3, source: "교재3", kr: "그거 못 들었어? 그 두 사람 사이가 완전히 틀어졌대.", en: "Didn’t you hear? Those two had a major falling-out after Christmas." },
                { day: 3, source: "교재3", kr: "제 동업자와 저는 돈 문제로 사이가 멀어졌어요.", en: "My business partner and I had a falling-out over money." },
                { day: 4, source: "대표", kr: "삼성이 오늘 신제품 출시 행사를 열어요.", en: "Samsung is having a new-product launch today." },
                { day: 4, source: "교재1", kr: "제가 이번 주말에 집들이를 합니다.", en: "I’m having a housewarming party this weekend." },
                { day: 4, source: "교재1", kr: "백화점들이 이번 주에 일제히 세일을 한다.", en: "The department stores are all having sales this week." },
                { day: 4, source: "교재1", kr: "우리는 6월에 코엑스에서 채용 박람회를 열었다.", en: "We had a job fair at COEX in June." },
                { day: 4, source: "교재1", kr: "이번 주말에 모임이 있으니 오고 싶으면 오렴.", en: "We’re having a get-together this weekend if you want to join." },
                { day: 4, source: "교재1", kr: "그 제과점에서 오늘 원 플러스 원 행사를 한다.", en: "The bakery has a buy-one-get-one deal today." },
                { day: 4, source: "교재2", kr: "여행용 가방을 새로 사야 해.", en: "I need to buy some new luggage." },
                { day: 4, source: "교재2", kr: "아, 코스트코에서 지금 샘소나이트 여행 가방 세일 중인데.", en: "Oh, Costco is having a sale on Samsonite luggage right now." },
                { day: 4, source: "교재2", kr: "준호 씨, 브라질 생활 어때요?", en: "How is life in Brazil treating you, Junho?" },
                { day: 4, source: "교재2", kr: "쉽지는 않지만 나아지고 있습니다.", en: "It’s not easy, but it’s getting better." },
                { day: 4, source: "교재2", kr: "지금쯤이면 친구도 사귀었겠어요?", en: "Do you have any friends yet?" },
                { day: 4, source: "교재2", kr: "네, 아파트 단지에 한국인 가정이 몇 있는데 매주 금요일마다 바비큐 파티를 해요.", en: "Yeah, a couple of Korean families live in my apartment building and we have barbeques every Friday." },
                { day: 4, source: "교재3", kr: "퇴근은 한 거지?", en: "Did you leave work yet?" },
                { day: 4, source: "교재3", kr: "엄마한테 전화한 거지?", en: "Did you call your mom yet?" },
                { day: 4, source: "교재3", kr: "저녁은 먹은 거지?", en: "Did you eat dinner yet?" },
                { day: 4, source: "교재3", kr: "집안일은 다 한 거지?", en: "Did you finish your chores yet?" },
                { day: 4, source: "교재3", kr: "당신 이미 집 나선 거지?", en: "Did you leave yet?" },
                { day: 4, source: "교재3", kr: "지갑을 두고 왔네. 아래쪽으로 던져 줄 수 있을까?", en: "I forgot my wallet. Could you throw it down to me, please?" },
                { day: 5, source: "대표", kr: "오후에는 집안일을 좀 해야 해요.", en: "I have some chores to do in the afternoon." },
                { day: 5, source: "교재1", kr: "할 일이 산더미같이 쌓여 있다.", en: "I have tons of work to do." },
                { day: 5, source: "교재1", kr: "답장해야 할 이메일이 몇 개 있다.", en: "I have a few emails to reply to." },
                { day: 5, source: "교재1", kr: "오늘 오후에 몇 가지 볼일이 좀 있다.", en: "I have some errands to run this afternoon." },
                { day: 5, source: "교재1", kr: "금요일까지 프로젝트 하나를 마무리해야 한다.", en: "We have a project to finish by Friday." },
                { day: 5, source: "교재1", kr: "크리스마스 쇼핑을 좀 해야 한다.", en: "I have some Christmas shopping to do." },
                { day: 5, source: "교재2", kr: "안녕하세요, 제리. 우리 방금 프로젝트 진행 상황에 대해 이야기하고 있었어요.", en: "Good morning, Jerry. We were just talking about the project updates." },
                { day: 5, source: "교재2", kr: "네, 늦어서 죄송해요. 뭐 좀 처리할 게 있었어요.", en: "Yes, sorry I’m late. I had something to take care of." },
                { day: 5, source: "교재2", kr: "오늘 바빠? 점심 먹자.", en: "Are you busy today? Let’s have lunch." },
                { day: 5, source: "교재2", kr: "중고나라에서 바지 다섯 개를 팔아서, 택배 부칠 게 많아.", en: "I just sold five pairs of pants, so I have a lot of packages to ship." },
                { day: 5, source: "교재3", kr: "이번 주말에 친구 고양이를 돌봐 주기로 했어.", en: "I’m taking care of my friend’s cat this weekend." },
                { day: 5, source: "교재3", kr: "저희 아빠는 건강을 좀 더 신경 쓰셔야 해요.", en: "My dad needs to take better care of his health." },
                { day: 5, source: "교재3", kr: "휴가 가기 전에 처리해야 할 일이 좀 있어.", en: "I have some work to take care of before I start my vacation." },
                { day: 5, source: "교재3", kr: "저희 엄마는 주로 오후 시간에 정원을 손질하십니다.", en: "My mom usually takes care of her garden in the afternoons." },
                { day: 5, source: "교재3", kr: "회사에서 모든 경비를 다 대 줬어요.", en: "The company took care of everything." },
            ],
            conversation: [
                { day: 1, source: "교재1", kr: "저는 재택근무 체질이 아니에요. 늘 딴짓하게 되거든요", en: "Working from home isn’t for me. I always get distracted." },
                { day: 2, source: "대표", kr: "하루빨리 새 집으로 이사 가고 싶어요.", en: "I can’t wait to move into the new house." },
                { day: 3, source: "대표", kr: "죄송한데 조금 짧게 해 주시겠어요?", en: "Do you mind keeping it a bit short?" },
                { day: 4, source: "대표", kr: "물가가 올라도 너무 올라요.", en: "Everything is getting super expensive." },
                { day: 5, source: "대표", kr: "중고차 같은 중고 물품 사는 거 어떻게 생각하세요?", en: "How do you feel about buying used items?" },
            ],
            phrasal_verbs: [
                { day: 1, source: "교재1", kr: "뭔가 앞뒤가 안 맞잖아.", en: "Something doesn’t add up." },
                { day: 2, source: "교재1", kr: "바람이 워낙 강해서 눈이 다 날아가 버렸더군.", en: "The wind was so strong that it blew the snow away." },
                { day: 3, source: "교재1", kr: "몇 년을 매일 썼더니 컴퓨터가 결국 고장이 났다.", en: "The computer finally broke down after using it daily for years." },
                { day: 4, source: "교재1", kr: "여기 신호가 끊겨. 나이트클럽이거든.", en: "My signal is breaking up down here. I’m in a nightclub." },
                { day: 5, source: "교재1", kr: "다음 달에 파리로 여행 가는데, 프랑스어 복습 좀 해야겠어.", en: "I am travelling to Paris next month, so I think I need to brush up on my French." },
            ]
        };

        const SOURCE_LABELS = { 
            "대표": "대표예제", 
            "교재1": "Model examples", 
            "교재2": "Small Talk", 
            "교재3": "Cases in point", 
            "교재4": "Further studies" 
        };
        const CURRICULUM_LABELS = { 
            basic_verbs: "기본동사100", 
            conversation: "영어회화100", 
            phrasal_verbs: "구동사100" 
        };

        function App() {
            // 초기값을 엄격하게 false로 설정
            const [isUnlocked, setIsUnlocked] = useState(false);
            const [inputCode, setInputCode] = useState('');
            const [tab, setTab] = useState('basic_verbs');
            const [view, setView] = useState('main'); 
            const [difficulty, setDifficulty] = useState(null); 
            const [currentCards, setCurrentCards] = useState([]);
            const [currentIndex, setCurrentIndex] = useState(0);
            const [isFlipped, setIsFlipped] = useState(false);

            // 보안 설정
            useEffect(() => {
                const handleContextMenu = (e) => e.preventDefault();
                const handleKeyDown = (e) => {
                    if ((e.ctrlKey || e.metaKey) && (e.key === 'c' || e.key === 'u' || e.key === 's')) {
                        e.preventDefault();
                    }
                };
                window.addEventListener('contextmenu', handleContextMenu);
                window.addEventListener('keydown', handleKeyDown);
                return () => {
                    window.removeEventListener('contextmenu', handleContextMenu);
                    window.removeEventListener('keydown', handleKeyDown);
                };
            }, []);

            const checkCode = () => {
                if (inputCode === SECRET_PASSWORD) {
                    setIsUnlocked(true);
                } else {
                    alert("비밀번호가 틀렸습니다.");
                    setInputCode('');
                }
            };

            const startQuiz = (diff) => {
                let pool = QUIZ_DATA[tab] || [];
                if (diff === 'mild') {
                    pool = pool.filter(item => item.source === "대표" || item.source === "교재1");
                }
                const shuffled = [...pool].sort(() => 0.5 - Math.random());
                setCurrentCards(shuffled.slice(0, 10));
                setCurrentIndex(0);
                setIsFlipped(false);
                setDifficulty(diff);
                setView('quiz');
            };

            // 잠겨 있을 때 (비밀번호 화면)
            if (!isUnlocked) {
                return (
                    <div className="min-h-screen bg-slate-900 flex items-center justify-center p-6">
                        <div className="bg-white p-8 rounded-[2rem] shadow-2xl max-w-sm w-full text-center">
                            <div className="text-5xl mb-6">🔒</div>
                            <h2 className="text-2xl font-black text-slate-800 mb-2">Protected Content</h2>
                            <p className="text-slate-500 mb-8 text-sm leading-relaxed">
                                이 학습 도구는 저작권 보호를 위해<br/>
                                <span className="font-bold text-blue-600">접속 비밀번호</span>가 필요합니다.
                            </p>
                            <input 
                                type="tel" 
                                value={inputCode}
                                onChange={(e) => setInputCode(e.target.value)}
                                onKeyPress={(e) => e.key === 'Enter' && checkCode()}
                                placeholder="Passcode"
                                className="w-full p-4 bg-slate-100 rounded-2xl mb-4 text-center text-2xl font-black tracking-widest border-2 border-transparent focus:border-blue-500 outline-none transition-all"
                            />
                            <button 
                                onClick={checkCode}
                                className="w-full bg-blue-600 text-white py-4 rounded-2xl font-bold shadow-lg hover:bg-blue-700 active:scale-95 transition-all mb-4"
                            >
                                Unlock Now
                            </button>
                            <p className="text-[10px] text-slate-400">© 2024 Phrasal Verb Speed Quiz</p>
                        </div>
                    </div>
                );
            }

            // 잠금 해제 후 메인 화면
            if (view === 'main') return (
                <div className="min-h-screen p-8 max-w-md mx-auto text-center flex flex-col justify-center">
                    <h1 className="text-4xl font-black text-blue-700 mb-10 tracking-tighter">SPEED QUIZ</h1>
                    <div className="grid gap-4">
                        {Object.keys(CURRICULUM_LABELS).map(key => (
                            <button key={key} onClick={() => { setTab(key); setView('selection'); }}
                                className="bg-white p-6 rounded-2xl border-2 border-slate-100 shadow-sm hover:border-blue-400 hover:shadow-md transition-all text-xl font-bold text-slate-700">
                                {CURRICULUM_LABELS[key]}
                            </button>
                        ))}
                    </div>
                </div>
            );

            // 난이도 선택 화면
            if (view === 'selection') return (
                <div className="min-h-screen p-8 max-w-md mx-auto text-center flex flex-col justify-center">
                    <h2 className="text-2xl font-black mb-8">Week 1 (Day 1-5)</h2>
                    <div className="grid gap-4">
                        <button onClick={() => startQuiz('mild')} className="bg-green-50 border-2 border-green-200 p-8 rounded-3xl text-left hover:bg-green-100 transition-all">
                            <h3 className="text-xl font-bold text-green-800">순한맛 (Mild)</h3>
                            <p className="text-green-600">대표 & Model examples</p>
                        </button>
                        <button onClick={() => startQuiz('spicy')} className="bg-red-50 border-2 border-red-200 p-8 rounded-3xl text-left hover:bg-red-100 transition-all">
                            <h3 className="text-xl font-bold text-red-800">매운맛 (Spicy)</h3>
                            <p className="text-red-600">모든 예제 중 랜덤 10개</p>
                        </button>
                        <button onClick={() => setView('main')} className="mt-8 text-slate-400 font-bold underline">Go Back</button>
                    </div>
                </div>
            );

            // 퀴즈 화면
            if (view === 'quiz' && currentCards.length > 0) return (
                <div className="min-h-screen p-4 max-w-md mx-auto flex flex-col items-center justify-center">
                    <div className="w-full flex justify-between mb-4 font-bold text-slate-400">
                        <span>{currentIndex + 1} / {currentCards.length}</span>
                        <button onClick={() => setView('main')} className="text-blue-500">Exit</button>
                    </div>
                    <div className="relative w-full aspect-[4/3] perspective-1000 cursor-pointer" onClick={() => setIsFlipped(!isFlipped)}>
                        <div className={`relative w-full h-full transition-all duration-500 preserve-3d ${isFlipped ? 'rotate-y-180' : ''}`}>
                            <div className="absolute inset-0 bg-white rounded-3xl shadow-xl flex items-center justify-center p-8 text-center border-b-8 border-slate-200 backface-hidden">
                                <p className="text-2xl font-bold text-slate-800 leading-tight">{currentCards[currentIndex]?.kr}</p>
                            </div>
                            <div className="absolute inset-0 bg-blue-600 rounded-3xl shadow-xl flex flex-col items-center justify-center p-8 text-center rotate-y-180 border-b-8 border-blue-800 backface-hidden">
                                <div className="absolute top-6 flex gap-2">
                                    <span className="bg-blue-500 text-white px-3 py-1 rounded-full text-xs font-bold">Day {currentCards[currentIndex]?.day}</span>
                                    <span className="bg-yellow-400 text-blue-900 px-3 py-1 rounded-full text-xs font-black">{SOURCE_LABELS[currentCards[currentIndex]?.source] || "Example"}</span>
                                </div>
                                <p className="text-2xl font-black text-white leading-snug">{currentCards[currentIndex]?.en}</p>
                            </div>
                        </div>
                    </div>
                    <div className="flex gap-4 mt-10 w-full">
                        <button onClick={(e) => { e.stopPropagation(); setIsFlipped(false); if(currentIndex > 0) setCurrentIndex(currentIndex - 1); }}
                            className="flex-1 bg-white py-5 rounded-2xl font-bold border border-slate-200 shadow-sm active:bg-slate-50">Prev</button>
                        <button onClick={(e) => { 
                            e.stopPropagation(); 
                            if (currentIndex < currentCards.length - 1) {
                                setIsFlipped(false);
                                setTimeout(() => setCurrentIndex(currentIndex + 1), 200);
                            } else setView('result');
                        }}
                            className={`flex-1 text-white py-5 rounded-2xl font-bold shadow-lg active:scale-95 transition-all ${difficulty === 'mild' ? 'bg-green-600' : 'bg-red-600'}`}>
                            {currentIndex === currentCards.length - 1 ? 'Finish' : 'Next'}
                        </button>
                    </div>
                </div>
            );

            // 결과 화면
            if (view === 'result') return (
                <div className="min-h-screen p-10 text-center max-w-md mx-auto flex flex-col justify-center">
                    <div className="text-6xl mb-6">🏆</div>
                    <h2 className="text-4xl font-black text-slate-800 mb-4">Complete!</h2>
                    <p className="text-slate-500 mb-10">오늘의 10문장 학습을 완료했습니다.</p>
                    <button onClick={() => setView('main')} className="w-full bg-blue-600 text-white py-5 rounded-2xl font-bold text-xl shadow-lg hover:bg-blue-700 transition-all">Back to Menu</button>
                </div>
            );

            return null;
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
