<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Speed Quiz Protected</title>
    <!-- CSS & Libraries -->
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
            background-color: #0f172a; /* 초기 배경을 어둡게 설정하여 로딩 시 눈부심 방지 */
        }
        #root:empty::before {
            content: "잠시만 기다려 주세요...";
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            color: #94a3b8;
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect } = React;

        // --- 접속 비밀번호 설정 (원하는 것으로 수정 가능) ---
        const SECRET_PASSWORD = "1234"; 

        const QUIZ_DATA = {
            basic_verbs: [
                { day: 1, source: "대표", kr: "반려동물 키우시나요?", en: "Do you have any pets?" },
                { day: 1, source: "교재1", kr: "내 조카는 거북이를 키운다.", en: "My nephew has a turtle." },
                { day: 1, source: "교재1", kr: "나는 시카고에 사는 친구들이 있다.", en: "I have some friends living in Chicago." },
                { day: 1, source: "교재2", kr: "프로젝트가 끝나서 좋죠?", en: "Are you glad the project is over?" },
                { day: 2, source: "대표", kr: "방금 간식을 먹었더니, 배가 별로 안 고파요.", en: "I just had a snack, so I’m not that hungry." },
                { day: 3, source: "대표", kr: "어제 아침에는 별일이 다 있었어요.", en: "I had a weird morning yesterday." },
                { day: 4, source: "대표", kr: "삼성이 오늘 신제품 출시 행사를 열어요.", en: "Samsung is having a new-product launch today." },
                { day: 5, source: "대표", kr: "오후에는 집안일을 좀 해야 해요.", en: "I have some chores to do in the afternoon." },
                { day: 5, source: "교재3", kr: "회사에서 모든 경비를 다 대 줬어요.", en: "The company took care of everything." }
            ],
            conversation: [
                { day: 1, source: "교재1", kr: "저는 재택근무 체질이 아니에요.", en: "Working from home isn’t for me." },
                { day: 2, source: "대표", kr: "하루빨리 새 집으로 이사 가고 싶어요.", en: "I can’t wait to move into the new house." },
                { day: 3, source: "대표", kr: "죄송한데 조금 짧게 해 주시겠어요?", en: "Do you mind keeping it a bit short?" },
                { day: 4, source: "대표", kr: "물가가 올라도 너무 올라요.", en: "Everything is getting super expensive." },
                { day: 5, source: "대표", kr: "중고차 같은 중고 물품 사는 거 어떻게 생각하세요?", en: "How do you feel about buying used items?" }
            ],
            phrasal_verbs: [
                { day: 1, source: "교재1", kr: "뭔가 앞뒤가 안 맞잖아.", en: "Something doesn’t add up." },
                { day: 2, source: "교재1", kr: "바람이 워낙 강해서 눈이 다 날아가 버렸더군.", en: "The wind was so strong that it blew the snow away." },
                { day: 3, source: "교재1", kr: "몇 년을 매일 썼더니 컴퓨터가 결국 고장이 났다.", en: "The computer finally broke down after using it daily for years." },
                { day: 4, source: "교재1", kr: "여기 신호가 끊겨. 나이트클럽이거든.", en: "My signal is breaking up down here. I’m in a nightclub." },
                { day: 5, source: "교재1", kr: "다음 달에 파리로 여행 가는데, 프랑스어 복습 좀 해야겠어.", en: "I am travelling to Paris next month, so I think I need to brush up on my French." }
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
            const [isUnlocked, setIsUnlocked] = useState(false);
            const [inputCode, setInputCode] = useState('');
            const [tab, setTab] = useState('basic_verbs');
            const [view, setView] = useState('main'); 
            const [difficulty, setDifficulty] = useState(null); 
            const [currentCards, setCurrentCards] = useState([]);
            const [currentIndex, setCurrentIndex] = useState(0);
            const [isFlipped, setIsFlipped] = useState(false);

            useEffect(() => {
                // 우클릭 금지 및 복사 방지
                const handleContextMenu = (e) => e.preventDefault();
                window.addEventListener('contextmenu', handleContextMenu);
                return () => window.removeEventListener('contextmenu', handleContextMenu);
            }, []);

            const checkCode = () => {
                if (inputCode === SECRET_PASSWORD) {
                    setIsUnlocked(true);
                } else {
                    alert("비밀번호가 올바르지 않습니다.");
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

            // 1. 잠금 화면 (비밀번호를 치기 전까지는 다른 화면이 전혀 보이지 않음)
            if (!isUnlocked) {
                return (
                    <div className="min-h-screen bg-slate-900 flex items-center justify-center p-6 transition-all duration-500">
                        <div className="bg-white p-8 rounded-[2rem] shadow-2xl max-w-sm w-full text-center">
                            <div className="text-5xl mb-6">🔒</div>
                            <h2 className="text-2xl font-black text-slate-800 mb-2">Access Code</h2>
                            <p className="text-slate-500 mb-8 text-sm leading-relaxed">
                                이 학습 도구는 저작권 보호를 위해<br/>
                                <span className="font-bold text-blue-600 font-mono">Passcode</span>가 필요합니다.
                            </p>
                            <input 
                                type="tel" 
                                value={inputCode}
                                onChange={(e) => setInputCode(e.target.value)}
                                onKeyPress={(e) => e.key === 'Enter' && checkCode()}
                                placeholder="숫자 4자리 입력"
                                className="w-full p-4 bg-slate-100 rounded-2xl mb-4 text-center text-2xl font-black tracking-widest border-2 border-transparent focus:border-blue-500 outline-none transition-all"
                                autoFocus
                            />
                            <button 
                                onClick={checkCode}
                                className="w-full bg-blue-600 text-white py-4 rounded-2xl font-bold shadow-lg hover:bg-blue-700 active:scale-95 transition-all mb-4"
                            >
                                Unlock
                            </button>
                            <p className="text-[10px] text-slate-400">© 원작자 예문 저작권 준수</p>
                        </div>
                    </div>
                );
            }

            // 2. 메인 선택 화면
            if (view === 'main') return (
                <div className="min-h-screen p-8 max-w-md mx-auto text-center flex flex-col justify-center bg-slate-50">
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

            // 3. 난이도 선택 화면
            if (view === 'selection') return (
                <div className="min-h-screen p-8 max-w-md mx-auto text-center flex flex-col justify-center bg-slate-50">
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
                        <button onClick={() => setView('main')} className="mt-8 text-slate-400 font-bold underline">돌아가기</button>
                    </div>
                </div>
            );

            // 4. 퀴즈 실행 화면
            if (view === 'quiz' && currentCards.length > 0) return (
                <div className="min-h-screen p-4 max-w-md mx-auto flex flex-col items-center justify-center bg-slate-50">
                    <div className="w-full flex justify-between mb-4 font-bold text-slate-400">
                        <span>{currentIndex + 1} / {currentCards.length}</span>
                        <button onClick={() => setView('main')} className="text-blue-500">종료</button>
                    </div>
                    <div className="relative w-full aspect-[4/3] perspective-1000 cursor-pointer" onClick={() => setIsFlipped(!isFlipped)}>
                        <div className={`relative w-full h-full transition-all duration-500 preserve-3d ${isFlipped ? 'rotate-y-180' : ''}`}>
                            {/* 앞면 (한국어) */}
                            <div className="absolute inset-0 bg-white rounded-3xl shadow-xl flex items-center justify-center p-8 text-center border-b-8 border-slate-200 backface-hidden">
                                <p className="text-2xl font-bold text-slate-800 leading-tight">{currentCards[currentIndex]?.kr}</p>
                            </div>
                            {/* 뒷면 (영어 + 출처) */}
                            <div className="absolute inset-0 bg-blue-600 rounded-3xl shadow-xl flex flex-col items-center justify-center p-8 text-center rotate-y-180 border-b-8 border-blue-800 backface-hidden">
                                <div className="absolute top-6 flex gap-2">
                                    <span className="bg-blue-500 text-white px-3 py-1 rounded-full text-xs font-bold">Day {currentCards[currentIndex]?.day}</span>
                                    <span className="bg-yellow-400 text-blue-900 px-3 py-1 rounded-full text-xs font-black">{SOURCE_LABELS[currentCards[currentIndex]?.source] || "예제"}</span>
                                </div>
                                <p className="text-2xl font-black text-white leading-snug">{currentCards[currentIndex]?.en}</p>
                            </div>
                        </div>
                    </div>
                    <div className="flex gap-4 mt-10 w-full">
                        <button onClick={(e) => { e.stopPropagation(); setIsFlipped(false); if(currentIndex > 0) setCurrentIndex(currentIndex - 1); }}
                            className="flex-1 bg-white py-5 rounded-2xl font-bold border border-slate-200 shadow-sm">이전</button>
                        <button onClick={(e) => { 
                            e.stopPropagation(); 
                            if (currentIndex < currentCards.length - 1) {
                                setIsFlipped(false);
                                setTimeout(() => setCurrentIndex(currentIndex + 1), 200);
                            } else setView('result');
                        }}
                            className={`flex-1 text-white py-5 rounded-2xl font-bold shadow-lg ${difficulty === 'mild' ? 'bg-green-600' : 'bg-red-600'}`}>
                            {currentIndex === currentCards.length - 1 ? '완료' : '다음'}
                        </button>
                    </div>
                </div>
            );

            // 5. 학습 결과 화면
            if (view === 'result') return (
                <div className="min-h-screen p-10 text-center max-w-md mx-auto flex flex-col justify-center bg-slate-50">
                    <div className="text-6xl mb-6">🏆</div>
                    <h2 className="text-4xl font-black text-slate-800 mb-4">학습 완료!</h2>
                    <p className="text-slate-500 mb-10">오늘의 10문장 학습을 모두 마쳤습니다.</p>
                    <button onClick={() => setView('main')} className="w-full bg-blue-600 text-white py-5 rounded-2xl font-bold text-xl shadow-lg hover:bg-blue-700 transition-all">메인으로</button>
                </div>
            );

            return null;
        }

        // React 18 렌더링 방식 (가장 안정적인 방식)
        window.addEventListener('DOMContentLoaded', () => {
            const rootElement = document.getElementById('root');
            if (rootElement) {
                const root = ReactDOM.createRoot(rootElement);
                root.render(<App />);
            }
        });
    </script>
</body>
</html>
