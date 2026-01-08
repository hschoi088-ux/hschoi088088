<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Speed Quiz</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <style>
        @font-face {
            font-family: 'Pretendard';
            src: url('https://fastly.jsdelivr.net/gh/Project-Noonnu/noonfonts_2107@1.1/Pretendard-Regular.woff') format('woff');
        }
        body { font-family: 'Pretendard', sans-serif; background-color: #f8fafc; margin: 0; padding: 0; }
        .perspective-1000 { perspective: 1000px; }
        .preserve-3d { transform-style: preserve-3d; }
        .backface-hidden { backface-visibility: hidden; }
        .rotate-y-180 { transform: rotateY(180deg); }
        /* 초기 잠금화면 스타일 */
        #login-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: #0f172a; display: flex; align-items: center; justify-content: center; z-index: 9999;
        }
    </style>
</head>
<body>
    <!-- 1. 비밀번호 잠금 화면 (React 로딩 전에도 보임) -->
    <div id="login-overlay">
        <div style="background: white; padding: 2rem; border-radius: 1.5rem; text-align: center; width: 90%; max-width: 350px;">
            <div style="font-size: 3rem; margin-bottom: 1rem;">🔒</div>
            <h2 style="font-size: 1.5rem; font-weight: 900; margin-bottom: 0.5rem;">Access Required</h2>
            <p style="color: #64748b; font-size: 0.875rem; margin-bottom: 1.5rem;">비밀번호를 입력하세요.</p>
            <input type="tel" id="pw-input" placeholder="Passcode" 
                style="width: 100%; padding: 1rem; background: #f1f5f9; border: none; border-radius: 1rem; text-align: center; font-size: 1.5rem; font-weight: 900; margin-bottom: 1rem; outline: none;">
            <button onclick="checkPassword()" 
                style="width: 100%; padding: 1rem; background: #2563eb; color: white; border: none; border-radius: 1rem; font-weight: 700; cursor: pointer;">접속하기</button>
        </div>
    </div>

    <!-- 2. 실제 앱이 그려질 곳 -->
    <div id="root"></div>

    <script>
        // 비밀번호 체크 함수 (순수 자바스크립트)
        function checkPassword() {
            const input = document.getElementById('pw-input').value;
            if (input === "1234") {
                document.getElementById('login-overlay').style.display = 'none';
            } else {
                alert("비밀번호가 틀렸습니다.");
                document.getElementById('pw-input').value = '';
            }
        }
    </script>

    <script type="text/babel">
        const { useState } = React;

        // 아이콘 SVG
        const ZapIcon = () => <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M13 2 3 14h9l-1 8 10-12h-9l1-8z"/></svg>;

        const QUIZ_DATA = {
            basic_verbs: [
                { day: 1, source: "대표", kr: "반려동물 키우시나요?", en: "Do you have any pets?" },
                { day: 1, source: "교재1", kr: "내 조카는 거북이를 키운다.", en: "My nephew has a turtle." },
                { day: 1, source: "교재2", kr: "프로젝트가 끝나서 좋죠?", en: "Are you glad the project is over?" },
                { day: 2, source: "대표", kr: "방금 간식을 먹었더니, 배가 별로 안 고파요.", en: "I just had a snack, so I’m not that hungry." },
                { day: 3, source: "대표", kr: "어제 아침에는 별일이 다 있었어요.", en: "I had a weird morning yesterday." },
                { day: 4, source: "대표", kr: "삼성이 오늘 신제품 출시 행사를 열어요.", en: "Samsung is having a new-product launch today." },
                { day: 5, source: "대표", kr: "오후에는 집안일을 좀 해야 해요.", en: "I have some chores to do in the afternoon." }
            ],
            conversation: [
                { day: 1, source: "교재1", kr: "저는 재택근무 체질이 아니에요.", en: "Working from home isn’t for me." },
                { day: 2, source: "대표", kr: "하루빨리 새 집으로 이사 가고 싶어요.", en: "I can’t wait to move into the new house." }
            ],
            phrasal_verbs: [
                { day: 1, source: "교재1", kr: "뭔가 앞뒤가 안 맞잖아.", en: "Something doesn’t add up." },
                { day: 2, source: "교재1", kr: "연기가 정말 인상적이었어.", en: "I was absolutely blown away." }
            ]
        };

        const SOURCE_LABELS = { 
            "대표": "대표예제", "교재1": "Model examples", "교재2": "Small Talk", "교재3": "Cases in point", "교재4": "Further studies" 
        };

        function App() {
            const [tab, setTab] = useState('basic_verbs');
            const [view, setView] = useState('main'); 
            const [currentCards, setCurrentCards] = useState([]);
            const [currentIndex, setCurrentIndex] = useState(0);
            const [isFlipped, setIsFlipped] = useState(false);
            const [difficulty, setDifficulty] = useState('spicy');

            const startQuiz = (diff) => {
                let pool = QUIZ_DATA[tab] || [];
                if (diff === 'mild') pool = pool.filter(i => i.source === "대표" || i.source === "교재1");
                const shuffled = [...pool].sort(() => 0.5 - Math.random()).slice(0, 10);
                setCurrentCards(shuffled);
                setCurrentIndex(0);
                setIsFlipped(false);
                setDifficulty(diff);
                setView('quiz');
            };

            if (view === 'main') return (
                <div className="p-8 max-w-md mx-auto text-center min-h-screen flex flex-col justify-center">
                    <h1 className="text-3xl font-black text-blue-700 mb-8 flex items-center justify-center gap-2">
                        <ZapIcon /> SPEED QUIZ
                    </h1>
                    <div className="grid gap-4">
                        <button onClick={() => { setTab('basic_verbs'); setView('selection'); }} className="bg-white p-6 rounded-2xl border-2 border-slate-100 shadow-sm font-bold text-xl">기본동사100</button>
                        <button onClick={() => { setTab('conversation'); setView('selection'); }} className="bg-white p-6 rounded-2xl border-2 border-slate-100 shadow-sm font-bold text-xl">영어회화100</button>
                        <button onClick={() => { setTab('phrasal_verbs'); setView('selection'); }} className="bg-white p-6 rounded-2xl border-2 border-slate-100 shadow-sm font-bold text-xl">구동사100</button>
                    </div>
                </div>
            );

            if (view === 'selection') return (
                <div className="p-8 max-w-md mx-auto text-center min-h-screen flex flex-col justify-center">
                    <h2 className="text-2xl font-black mb-8">난이도 선택</h2>
                    <div className="grid gap-4">
                        <button onClick={() => startQuiz('mild')} className="bg-green-50 border-2 border-green-200 p-8 rounded-3xl text-left">
                            <h3 className="font-bold text-green-800 text-lg">순한맛</h3>
                            <p className="text-green-600 text-sm">대표 & Model examples</p>
                        </button>
                        <button onClick={() => startQuiz('spicy')} className="bg-red-50 border-2 border-red-200 p-8 rounded-3xl text-left">
                            <h3 className="font-bold text-red-800 text-lg">매운맛</h3>
                            <p className="text-red-600 text-sm">랜덤 10문장</p>
                        </button>
                        <button onClick={() => setView('main')} className="mt-4 text-slate-400 font-bold underline">뒤로가기</button>
                    </div>
                </div>
            );

            if (view === 'quiz') return (
                <div className="p-4 max-w-md mx-auto flex flex-col items-center justify-center min-h-screen">
                    <div className="w-full flex justify-between mb-4 font-bold text-slate-400">
                        <span>{currentIndex + 1} / {currentCards.length}</span>
                        <button onClick={() => setView('main')}>그만하기</button>
                    </div>
                    <div className="relative w-full aspect-[4/3] perspective-1000 cursor-pointer" onClick={() => setIsFlipped(!isFlipped)}>
                        <div className={`relative w-full h-full transition-all duration-500 preserve-3d ${isFlipped ? 'rotate-y-180' : ''}`}>
                            <div className="absolute inset-0 bg-white rounded-3xl shadow-xl flex items-center justify-center p-8 text-center border-b-8 border-slate-200 backface-hidden">
                                <p className="text-xl font-bold text-slate-800">{currentCards[currentIndex]?.kr}</p>
                            </div>
                            <div className="absolute inset-0 bg-blue-600 rounded-3xl shadow-xl flex flex-col items-center justify-center p-8 text-center rotate-y-180 border-b-8 border-blue-800 backface-hidden">
                                <span className="bg-yellow-400 text-blue-900 px-2 py-1 rounded text-[10px] font-black mb-4">
                                    {SOURCE_LABELS[currentCards[currentIndex]?.source] || "예제"}
                                </span>
                                <p className="text-xl font-bold text-white">{currentCards[currentIndex]?.en}</p>
                            </div>
                        </div>
                    </div>
                    <div className="flex gap-4 mt-8 w-full">
                        <button onClick={(e) => { e.stopPropagation(); setIsFlipped(false); if(currentIndex > 0) setCurrentIndex(currentIndex - 1); }}
                            className="flex-1 bg-white py-4 rounded-xl font-bold border border-slate-200">이전</button>
                        <button onClick={(e) => { 
                            e.stopPropagation(); 
                            if (currentIndex < currentCards.length - 1) {
                                setIsFlipped(false);
                                setTimeout(() => setCurrentIndex(i => i + 1), 200);
                            } else setView('result');
                        }}
                            className={`flex-1 text-white py-4 rounded-xl font-bold shadow-lg ${difficulty === 'mild' ? 'bg-green-600' : 'bg-red-600'}`}>
                            {currentIndex === currentCards.length - 1 ? '완료' : '다음'}
                        </button>
                    </div>
                </div>
            );

            if (view === 'result') return (
                <div className="p-10 text-center min-h-screen flex flex-col justify-center">
                    <h2 className="text-3xl font-black mb-4">학습 완료!</h2>
                    <button onClick={() => setView('main')} className="bg-blue-600 text-white py-4 rounded-xl font-bold shadow-lg">메인으로</button>
                </div>
            );

            return null;
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
