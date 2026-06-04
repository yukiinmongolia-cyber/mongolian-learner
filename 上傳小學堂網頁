<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🌟 蒙古語小學堂 🌟</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800&family=Noto+Sans+TC:wght@400;500;700&display=swap');
        body {
            font-family: 'Nunito', 'Noto Sans TC', sans-serif;
            background-color: #FFF8F0; /* 可愛柔和的奶油底色 */
            margin: 0;
            padding: 0;
            -webkit-tap-highlight-color: transparent;
        }
        /* 自訂可愛的捲軸 */
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f1f1; 
            border-radius: 10px;
        }
        ::-webkit-scrollbar-thumb {
            background: #FFD166; 
            border-radius: 10px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #FFC033; 
        }
        .no-tap-highlight {
            -webkit-tap-highlight-color: transparent;
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect } = React;

        // 蒙古語解析 AI API 呼叫 (直接呼叫 Google Gemini API)
        const analyzeMongolianText = async (text, apiKey) => {
            if (!apiKey) {
                throw new Error("請先設定您個人的 Gemini API 金鑰唷 🔑");
            }
            
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${apiKey}`;

            const systemInstruction = `你是一個溫柔、簡約、可愛且專業的蒙古語教師。
請幫助學生分析這段蒙古語短文，並以 JSON 格式回傳。
要求：
1. full_translation: 提供整段文字的繁體中文翻譯。
2. words: 將短文拆解成單字陣列。包含：
   - original: 文章中的原始蒙古文單字。
   - lemma: 該單字的「原形」(Base form)。
   - translation: 繁體中文翻譯，必須「單行、簡潔」，不要有複雜的語言學符號。
   - is_grammar: 布林值，如果是文法標記、語尾助詞或格位，請設為 true。
3. grammar_notes: 陣列，提供這段文字的重要文法解析。請完整、詳實地列出文中出現的格位變化、動詞語尾、助詞或重要片語。
   - point: 文法要點名稱（例如：主格助詞、屬格詞尾 -ын 等）。
   - explanation: 清晰白話的繁體中文詳細說明。`;

            const schema = {
                type: "OBJECT",
                properties: {
                    full_translation: { type: "STRING" },
                    words: {
                        type: "ARRAY",
                        items: {
                            type: "OBJECT",
                            properties: {
                                original: { type: "STRING" },
                                lemma: { type: "STRING" },
                                translation: { type: "STRING" },
                                is_grammar: { type: "BOOLEAN" }
                            },
                            required: ["original", "lemma", "translation", "is_grammar"]
                        }
                    },
                    grammar_notes: {
                        type: "ARRAY",
                        items: {
                            type: "OBJECT",
                            properties: {
                                point: { type: "STRING" },
                                explanation: { type: "STRING" }
                            },
                            required: ["point", "explanation"]
                        }
                    }
                },
                required: ["full_translation", "words", "grammar_notes"]
            };

            const payload = {
                contents: [{ parts: [{ text: `請分析以下蒙古語短文：\n\n${text}` }] }],
                systemInstruction: { parts: [{ text: systemInstruction }] },
                generationConfig: {
                    responseMimeType: "application/json",
                    responseSchema: schema
                }
            };

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                
                if (!response.ok) {
                    const errData = await response.json();
                    throw new Error(errData.error?.message || "API 呼叫失敗，請確認 Key 是否正確");
                }

                const result = await response.json();
                if (result.candidates && result.candidates[0].content.parts[0].text) {
                    return JSON.parse(result.candidates[0].content.parts[0].text);
                } else {
                    throw new Error("無法解析回傳結果");
                }
            } catch (error) {
                console.error("API Error:", error);
                throw error;
            }
        };

        const App = () => {
            const [activeTab, setActiveTab] = useState('read'); // read, words, game
            const [inputText, setInputText] = useState("Сайн байна уу? Таны ажил сайн уу?");
            const [isAnalyzing, setIsAnalyzing] = useState(false);
            const [analysisResult, setAnalysisResult] = useState(null);
            const [selectedWord, setSelectedWord] = useState(null);
            
            // 狀態變數：皆從 localStorage 讀取/寫入
            const [apiKey, setApiKey] = useState("");
            const [tempApiKey, setTempApiKey] = useState(""); 
            const [savedWords, setSavedWords] = useState([]);

            // Modal 及通知狀態
            const [isKeyModalOpen, setIsKeyModalOpen] = useState(false);
            const [showKeyPassword, setShowKeyPassword] = useState(false);
            const [backupString, setBackupString] = useState("");
            const [importString, setImportString] = useState("");
            const [toast, setToast] = useState(null);

            const showToast = (msg) => {
                setToast(msg);
                setTimeout(() => setToast(null), 3000);
            };

            // 1. 初始化讀取 LocalStorage
            useEffect(() => {
                try {
                    const localKey = localStorage.getItem("mongolian_learner_api_key");
                    if (localKey) {
                        setApiKey(localKey);
                        setTempApiKey(localKey);
                    }
                    const localWords = localStorage.getItem("saved_mongolian_words");
                    if (localWords) {
                        setSavedWords(JSON.parse(localWords));
                    }
                } catch (e) {
                    console.error("無法存取 LocalStorage", e);
                }
            }, []);

            // 2. 儲存 API Key 至 LocalStorage
            const saveApiKeyToLocal = (key) => {
                try {
                    localStorage.setItem("mongolian_learner_api_key", key);
                    setApiKey(key);
                    setTempApiKey(key);
                } catch (e) {
                    showToast("瀏覽器不支援儲存金鑰 😢");
                }
            };

            // 3. 儲存單字至 LocalStorage
            const saveWordsToLocal = (newWords) => {
                try {
                    localStorage.setItem("saved_mongolian_words", JSON.stringify(newWords));
                    setSavedWords(newWords);
                } catch (e) {
                    showToast("瀏覽器不支援儲存單字 😢");
                }
            };

            // 一鍵產生備份轉移代碼 (方便使用者更換瀏覽器/設備時轉移)
            const generateBackupCode = () => {
                const backupObj = {
                    apiKey: apiKey,
                    savedWords: savedWords
                };
                try {
                    const code = btoa(encodeURIComponent(JSON.stringify(backupObj)));
                    setBackupString(code);
                    showToast("備份碼已產生！🔑");
                } catch (e) {
                    showToast("產生備份碼失敗 😢");
                }
            };

            // 一鍵匯入備份碼
            const handleImportBackup = () => {
                try {
                    if (!importString.trim()) {
                        showToast("請先貼上備份碼唷！🐾");
                        return;
                    }
                    const decoded = JSON.parse(decodeURIComponent(atob(importString.trim())));
                    if (decoded.apiKey) {
                        saveApiKeyToLocal(decoded.apiKey);
                    }
                    if (decoded.savedWords) {
                        saveWordsToLocal(decoded.savedWords);
                    }
                    showToast("💪 成功還原所有金鑰與單字！");
                    setImportString("");
                } catch (e) {
                    showToast("匯入失敗，請確認備份碼是否完整正確 😢");
                }
            };

            const handleAnalyze = async () => {
                if (!apiKey) {
                    showToast("請先設定您的 Gemini API 金鑰唷 🔑");
                    setIsKeyModalOpen(true);
                    return;
                }
                if (!inputText.trim()) {
                    showToast("請輸入蒙古語短文唷 🐾");
                    return;
                }
                setIsAnalyzing(true);
                setAnalysisResult(null);
                try {
                    const result = await analyzeMongolianText(inputText, apiKey);
                    setAnalysisResult(result);
                    showToast("分析完成！✨");
                } catch (error) {
                    showToast(`分析失敗：${error.message || "伺服器忙碌中"} 😢`);
                } finally {
                    setIsAnalyzing(false);
                }
            };

            const toggleSaveWord = (wordObj) => {
                const exists = savedWords.find(w => w.lemma === wordObj.lemma);
                let updated;
                if (exists) {
                    updated = savedWords.filter(w => w.lemma !== wordObj.lemma);
                    showToast("已取消收藏 💔");
                } else {
                    updated = [...savedWords, wordObj];
                    showToast("單字收藏成功！💖");
                }
                saveWordsToLocal(updated);
                setSelectedWord(null); 
            };

            const renderReadTab = () => (
                <div className="flex flex-col gap-4 animate-fade-in">
                    <div className="bg-white p-4 rounded-3xl shadow-sm border-2 border-amber-100">
                        <textarea
                            className="w-full h-24 p-3 border-none bg-orange-50 rounded-2xl focus:ring-4 focus:ring-amber-200 focus:outline-none resize-none text-gray-700"
                            placeholder="請在此貼上蒙古語短文..."
                            value={inputText}
                            onChange={(e) => setInputText(e.target.value)}
                        />
                        <button 
                            onClick={handleAnalyze}
                            disabled={isAnalyzing}
                            className={`mt-3 w-full py-3 rounded-2xl font-bold text-white transition-all shadow-md active:scale-95
                                ${isAnalyzing ? 'bg-gray-400' : 'bg-gradient-to-r from-amber-400 to-orange-400 hover:from-amber-500 hover:to-orange-500'}`}
                        >
                            {isAnalyzing ? "正在努力解析中... 🐾" : "開始解析 ✨"}
                        </button>
                    </div>

                    {analysisResult && (
                        <div className="flex flex-col gap-4 animate-fade-in">
                            {/* 整段翻譯 */}
                            <div className="bg-white p-5 rounded-3xl shadow-sm border-2 border-emerald-100 relative overflow-hidden">
                                <div className="absolute top-0 right-0 bg-emerald-100 text-emerald-700 px-3 py-1 rounded-bl-2xl font-bold text-sm">
                                    整段翻譯
                                </div>
                                <p className="text-gray-700 leading-relaxed mt-2">
                                    {analysisResult.full_translation}
                                </p>
                            </div>

                            {/* 互動式單字閱讀區 */}
                            <div className="bg-white p-6 rounded-3xl shadow-sm border-2 border-blue-100 text-xl leading-loose">
                                <div className="mb-2 text-sm text-blue-500 font-bold flex items-center gap-2">
                                    <span>👉 點擊單字查看解析，紅字代表文法標記</span>
                                </div>
                                <div className="flex flex-wrap gap-x-2 gap-y-1">
                                    {analysisResult.words.map((word, idx) => (
                                        <span 
                                            key={idx}
                                            onClick={() => setSelectedWord(word)}
                                            className={`cursor-pointer px-1 py-0.5 rounded-lg transition-colors hover:bg-amber-100 active:bg-amber-200
                                                ${word.is_grammar ? 'text-red-500 font-bold' : 'text-gray-800'}`}
                                        >
                                            {word.original}
                                        </span>
                                    ))}
                                </div>
                            </div>

                            {/* 文法解說區 */}
                            <div className="bg-orange-50/60 p-5 rounded-3xl border border-orange-100 flex flex-col gap-3">
                                <h3 className="font-bold text-orange-800 text-sm tracking-wider uppercase flex items-center gap-2">
                                    <span>💡</span> 蒙古語文法解析
                                </h3>
                                <div className="flex flex-col gap-3">
                                    {analysisResult.grammar_notes && analysisResult.grammar_notes.map((note, idx) => (
                                        <div key={idx} className="bg-white p-4 rounded-2xl border border-orange-100/50 shadow-sm flex flex-col gap-1.5 transition-all hover:shadow-md animate-fade-in">
                                            <div className="flex items-center gap-2">
                                                <span className="bg-orange-100 text-orange-800 text-xs font-bold px-2.5 py-1 rounded-xl">
                                                    {note.point || "文法重點"}
                                                </span>
                                            </div>
                                            <p className="text-gray-600 text-xs leading-relaxed pl-1">
                                                {note.explanation}
                                            </p>
                                        </div>
                                    ))}
                                </div>
                            </div>
                        </div>
                    )}
                </div>
            );

            const renderWordModal = () => {
                if (!selectedWord) return null;
                const isSaved = savedWords.some(w => w.lemma === selectedWord.lemma);

                return (
                    <div className="fixed inset-0 bg-black/40 backdrop-blur-sm flex items-center justify-center z-50 p-4 animate-fade-in" onClick={() => setSelectedWord(null)}>
                        <div className="bg-white rounded-[2rem] p-6 w-full max-w-sm shadow-2xl transform scale-100 transition-transform relative" onClick={e => e.stopPropagation()}>
                            <button onClick={() => setSelectedWord(null)} className="absolute top-4 right-4 w-8 h-8 flex items-center justify-center bg-gray-100 rounded-full text-gray-500 hover:bg-gray-200">
                                ✕
                            </button>
                            
                            <div className="text-center mb-6 mt-2">
                                <div className="text-sm text-gray-400 font-bold mb-1">文章中出現的字</div>
                                <div className={`text-2xl font-black mb-4 ${selectedWord.is_grammar ? 'text-red-500' : 'text-gray-800'}`}>
                                    {selectedWord.original}
                                </div>
                                
                                <div className="bg-orange-50 rounded-2xl p-4 border border-orange-100 shadow-inner">
                                    <div className="text-sm text-orange-800 font-bold mb-1 flex justify-center items-center gap-1">
                                        <span>🔍</span> 單字原形
                                    </div>
                                    <div className="text-xl font-bold text-gray-800 mb-3">{selectedWord.lemma}</div>
                                    
                                    <div className="text-sm text-blue-600 font-bold mb-1 flex justify-center items-center gap-1">
                                        <span>📝</span> 單行解析
                                    </div>
                                    <div className="text-lg text-gray-700">{selectedWord.translation}</div>
                                </div>
                            </div>
                            
                            <button 
                                onClick={() => toggleSaveWord(selectedWord)}
                                className={`w-full py-3 rounded-2xl font-bold transition-all shadow-sm active:scale-95 flex justify-center items-center gap-2
                                    ${isSaved ? 'bg-red-50 text-red-500 hover:bg-red-100 border border-red-200' : 'bg-gradient-to-r from-pink-400 to-rose-400 text-white hover:from-pink-500 hover:to-rose-500'}`}
                            >
                                {isSaved ? "💔 移除收藏" : "💖 收藏單字"}
                            </button>
                        </div>
                    </div>
                );
            };

            const renderWordsTab = () => (
                <div className="animate-fade-in pb-10">
                    <div className="flex justify-between items-center mb-4 px-2">
                        <h2 className="text-2xl font-bold text-gray-800">我的單字本 📚</h2>
                        <span className="text-xs bg-orange-100 text-orange-800 font-bold px-3 py-1 rounded-full">
                            共 {savedWords.length} 個字
                        </span>
                    </div>
                    {savedWords.length === 0 ? (
                        <div className="bg-white p-8 rounded-3xl shadow-sm text-center border-2 border-gray-100 mt-10">
                            <div className="text-5xl mb-4">🪹</div>
                            <p className="text-gray-500 font-bold">還沒有收藏任何單字唷！<br/>趕快去閱讀文章收集吧 ✨</p>
                        </div>
                    ) : (
                        <div className="grid grid-cols-1 gap-3">
                            {savedWords.map((word, idx) => (
                                <div key={idx} className="bg-white p-4 rounded-2xl shadow-sm border border-gray-100 flex justify-between items-center group">
                                    <div>
                                        <div className="text-lg font-bold text-gray-800">{word.lemma}</div>
                                        <div className="text-sm text-gray-500">{word.translation}</div>
                                    </div>
                                    <button 
                                        onClick={() => toggleSaveWord(word)}
                                        className="w-10 h-10 flex items-center justify-center bg-red-50 text-red-400 rounded-full hover:bg-red-100 hover:text-red-600 transition-colors"
                                    >
                                        ✕
                                    </button>
                                </div>
                            ))}
                        </div>
                    )}
                </div>
            );

            // 複習小遊戲相關狀態與邏輯
            const [gameState, setGameState] = useState('start'); 
            const [currentQuestion, setCurrentQuestion] = useState(null);
            const [options, setOptions] = useState([]);
            const [score, setScore] = useState(0);
            const [questionIndex, setQuestionIndex] = useState(0);
            const [feedback, setFeedback] = useState(null); 

            const shuffleArray = (array) => [...array].sort(() => Math.random() - 0.5);

            const startGame = () => {
                if (savedWords.length < 1) {
                    showToast("至少需要收藏 1 個單字才能玩遊戲唷！🌟");
                    return;
                }
                setScore(0);
                setQuestionIndex(0);
                nextQuestion(0, shuffleArray(savedWords));
                setGameState('playing');
            };

            const nextQuestion = (qIndex, wordsPool = savedWords) => {
                if (qIndex >= Math.min(10, wordsPool.length)) {
                    setGameState('result');
                    return;
                }
                
                const targetWord = wordsPool[qIndex];
                const dummyTranslations = ["蘋果", "美麗的", "家", "吃飯", "天空", "跑", "書", "好", "謝謝", "水", "太陽", "月亮"];
                let wrongOptions = wordsPool
                    .filter(w => w.lemma !== targetWord.lemma)
                    .map(w => w.translation);
                
                while (wrongOptions.length < 3) {
                    const randomDummy = dummyTranslations[Math.floor(Math.random() * dummyTranslations.length)];
                    if (!wrongOptions.includes(randomDummy) && randomDummy !== targetWord.translation) {
                        wrongOptions.push(randomDummy);
                    }
                }

                wrongOptions = shuffleArray(wrongOptions).slice(0, 3);
                const currentOptions = shuffleArray([targetWord.translation, ...wrongOptions]);

                setCurrentQuestion(targetWord);
                setOptions(currentOptions);
                setFeedback(null);
            };

            const handleAnswer = (selectedTranslation, idx) => {
                if (feedback) return; 
                
                const isCorrect = selectedTranslation === currentQuestion.translation;
                setFeedback({ isCorrect, selectedIdx: idx, correctTranslation: currentQuestion.translation });
                
                if (isCorrect) setScore(score + 1);

                setTimeout(() => {
                    setQuestionIndex(questionIndex + 1);
                    nextQuestion(questionIndex + 1);
                }, 1500); 
            };

            const renderGameTab = () => {
                if (gameState === 'start') {
                    return (
                        <div className="flex flex-col items-center justify-center pt-10 animate-fade-in text-center">
                            <div className="text-7xl mb-6">🎮</div>
                            <h2 className="text-2xl font-bold text-gray-800 mb-2">單字大挑戰</h2>
                            <p className="text-gray-500 mb-8 max-w-xs leading-relaxed">
                                從您的收藏中挑選單字進行選擇題測驗，看看您記得了多少！
                            </p>
                            <button 
                                onClick={startGame}
                                className="bg-gradient-to-r from-blue-400 to-indigo-400 text-white font-bold py-4 px-12 rounded-full text-lg shadow-lg hover:shadow-xl hover:scale-105 transition-all active:scale-95"
                            >
                                開始挑戰！🚀
                            </button>
                            <p className="text-sm text-gray-400 mt-4">目前已收藏 {savedWords.length} 個單字</p>
                        </div>
                    );
                }

                if (gameState === 'playing' && currentQuestion) {
                    return (
                        <div className="animate-fade-in flex flex-col items-center">
                            <div className="w-full flex justify-between items-center mb-6 px-2">
                                <div className="bg-white px-4 py-1.5 rounded-full text-sm font-bold text-gray-500 shadow-sm">
                                    題目 {questionIndex + 1} / {Math.min(10, savedWords.length)}
                                </div>
                                <div className="bg-amber-100 text-amber-600 px-4 py-1.5 rounded-full text-sm font-bold shadow-sm">
                                    ⭐ 分數: {score}
                                </div>
                            </div>

                            <div className="bg-white w-full rounded-3xl p-8 shadow-sm border-2 border-indigo-100 text-center mb-6 relative overflow-hidden">
                                <div className="text-sm text-indigo-400 font-bold mb-2">請問這個單字是什麼意思？</div>
                                <div className="text-4xl font-black text-gray-800 mt-2 tracking-wide">
                                    {currentQuestion.lemma}
                                </div>
                            </div>

                            <div className="w-full grid grid-cols-1 gap-3">
                                {options.map((opt, idx) => {
                                    let btnClass = "bg-white border-2 border-gray-100 hover:border-indigo-300 text-gray-700";
                                    
                                    if (feedback) {
                                        if (opt === currentQuestion.translation) {
                                            btnClass = "bg-green-100 border-green-500 text-green-700 shadow-inner"; 
                                        } else if (idx === feedback.selectedIdx) {
                                            btnClass = "bg-red-100 border-red-500 text-red-700 shadow-inner"; 
                                        } else {
                                            btnClass = "bg-gray-50 border-gray-100 text-gray-400 opacity-50"; 
                                        }
                                    }

                                    return (
                                        <button
                                            key={idx}
                                            onClick={() => handleAnswer(opt, idx)}
                                            className={`p-4 rounded-2xl font-bold text-lg transition-all shadow-sm ${btnClass} ${!feedback && 'active:scale-95'}`}
                                        >
                                            {opt}
                                        </button>
                                    );
                                })}
                            </div>

                            {feedback && (
                                <div className={`mt-6 font-bold text-lg animate-bounce ${feedback.isCorrect ? 'text-green-500' : 'text-red-500'}`}>
                                    {feedback.isCorrect ? '答對了！🎉' : '哎呀，答錯了！💦'}
                                </div>
                            )}
                        </div>
                    );
                }

                if (gameState === 'result') {
                    const maxScore = Math.min(10, savedWords.length);
                    return (
                        <div className="flex flex-col items-center justify-center pt-10 animate-fade-in text-center">
                            <div className="text-7xl mb-4">🏆</div>
                            <h2 className="text-3xl font-black text-gray-800 mb-2">挑戰結束！</h2>
                            <p className="text-xl text-gray-600 mb-8 font-bold">
                                您的得分：<span className="text-amber-500 text-3xl">{score}</span> / {maxScore}
                            </p>
                            <button 
                                onClick={() => setGameState('start')}
                                className="bg-white border-2 border-indigo-200 text-indigo-500 font-bold py-3 px-10 rounded-full text-lg shadow-sm hover:bg-indigo-50 active:scale-95 transition-all"
                            >
                                再玩一次 🔄
                            </button>
                        </div>
                    );
                }
            };

            return (
                <div className="max-w-xl mx-auto min-h-screen bg-gray-50 shadow-2xl relative flex flex-col overflow-hidden">
                    {/* Header */}
                    <div className="bg-white pt-10 pb-4 px-6 rounded-b-[2rem] shadow-sm flex items-center justify-between border-b-2 border-orange-100 z-10 relative">
                        <div className="text-xs text-emerald-600 bg-emerald-50 px-2.5 py-1.5 rounded-full font-bold shadow-inner">
                            🟢 本地儲存已啟用
                        </div>
                        <h1 className="text-2xl font-black text-amber-600 tracking-wider flex items-center gap-2">
                            <span>🏕️</span> 蒙古語小學堂
                        </h1>
                        <button 
                            onClick={() => {
                                setTempApiKey(apiKey);
                                setIsKeyModalOpen(true);
                            }}
                            className={`p-2 rounded-full transition-all active:scale-95 flex items-center justify-center
                                ${apiKey ? 'bg-emerald-50 text-emerald-600 hover:bg-emerald-100' : 'bg-rose-50 text-rose-500 hover:bg-rose-100 animate-pulse'}`}
                            title="設定與備份"
                        >
                            <span>⚙️</span>
                        </button>
                    </div>

                    {/* Toast 提示通知 */}
                    {toast && (
                        <div className="absolute top-24 left-1/2 transform -translate-x-1/2 z-50 animate-fade-in">
                            <div className="bg-gray-800/90 text-white px-5 py-2.5 rounded-full shadow-lg text-sm font-bold flex items-center gap-2">
                                {toast}
                            </div>
                        </div>
                    )}

                    {/* 主要內容區 */}
                    <div className="flex-1 overflow-y-auto p-5 pb-28">
                        {activeTab === 'read' && renderReadTab()}
                        {activeTab === 'words' && renderWordsTab()}
                        {activeTab === 'game' && renderGameTab()}
                    </div>

                    {/* 單字詳情 Modal */}
                    {renderWordModal()}

                    {/* 設定與備份管理 Modal */}
                    {isKeyModalOpen && (
                        <div className="fixed inset-0 bg-black/40 backdrop-blur-sm flex items-center justify-center z-50 p-4 animate-fade-in" onClick={() => setIsKeyModalOpen(false)}>
                            <div className="bg-white rounded-[2rem] p-6 w-full max-w-sm shadow-2xl transform scale-100 transition-transform relative max-h-[85vh] overflow-y-auto" onClick={e => e.stopPropagation()}>
                                <button onClick={() => setIsKeyModalOpen(false)} className="absolute top-4 right-4 w-8 h-8 flex items-center justify-center bg-gray-100 rounded-full text-gray-500 hover:bg-gray-200">
                                    ✕
                                </button>
                                
                                <div className="text-center mb-5 mt-2">
                                    <div className="text-3xl mb-1">⚙️</div>
                                    <h3 className="text-xl font-black text-gray-800 font-bold">學堂系統設定</h3>
                                </div>

                                {/* 區塊一：本機隱私聲明 */}
                                <div className="mb-4 p-3.5 bg-emerald-50 text-emerald-900 rounded-2xl border border-emerald-200 text-xs leading-relaxed">
                                    🔒 <b>100% 隱私與安全聲明：</b><br/>
                                    本程式完全運行在您的瀏覽器中。您輸入的 API 金鑰與收藏單字皆僅儲存於本機電腦 (localStorage)。我們絕不會收集、傳輸或洩漏您的任何隱私資料。
                                </div>

                                {/* 區塊二：Gemini 金鑰設定 */}
                                <div className="mb-5 p-4 bg-orange-50/50 rounded-2xl border border-orange-100">
                                    <h4 className="font-bold text-orange-800 text-sm mb-2 flex items-center gap-1">🔑 1. 設定您的 Gemini API 金鑰</h4>
                                    <div className="relative">
                                        <input 
                                            type={showKeyPassword ? "text" : "password"}
                                            className="w-full p-3 pr-10 border border-gray-200 bg-white rounded-xl focus:ring-2 focus:ring-amber-200 focus:outline-none text-xs text-gray-700"
                                            placeholder="請在此貼上您的 API Key"
                                            value={tempApiKey}
                                            onChange={(e) => setTempApiKey(e.target.value)}
                                        />
                                        <button 
                                            type="button"
                                            onClick={() => setShowKeyPassword(!showKeyPassword)}
                                            className="absolute inset-y-0 right-3 flex items-center text-gray-400"
                                        >
                                            {showKeyPassword ? "👁️" : "🙈"}
                                        </button>
                                    </div>
                                    <div className="mt-2 text-right">
                                        <a href="https://aistudio.google.com/" target="_blank" rel="noopener noreferrer" className="text-[10px] text-amber-600 font-bold underline">
                                            🚀 免費申請您專屬的 Google AI 金鑰
                                        </a>
                                    </div>
                                </div>

                                {/* 區塊三：手動備份碼管理 */}
                                <div className="mb-6 p-4 bg-blue-50/50 rounded-2xl border border-blue-100">
                                    <h4 className="font-bold text-blue-800 text-sm mb-2 flex items-center gap-1">📋 2. 手動代碼備份與跨設備還原</h4>
                                    
                                    <div className="flex gap-2 mb-3">
                                        <button 
                                            onClick={generateBackupCode}
                                            className="flex-1 py-1.5 bg-blue-500 text-white font-bold rounded-xl text-xs hover:bg-blue-600 active:scale-95 transition-all"
                                        >
                                            產生目前備份碼
                                        </button>
                                        <button 
                                            onClick={() => {
                                                // 使用標準 execCommand 確保 iframe 內剪貼簿運作相容性
                                                const el = document.createElement('textarea');
                                                el.value = backupString;
                                                document.body.appendChild(el);
                                                el.select();
                                                document.execCommand('copy');
                                                document.body.removeChild(el);
                                                showToast("備份碼已複製到剪貼簿 📋");
                                            }}
                                            disabled={!backupString}
                                            className={`flex-1 py-1.5 font-bold rounded-xl text-xs transition-all ${backupString ? 'bg-gray-800 text-white hover:bg-gray-900 active:scale-95' : 'bg-gray-200 text-gray-400'}`}
                                        >
                                            複製備份碼
                                        </button>
                                    </div>

                                    {backupString && (
                                        <textarea 
                                            readOnly
                                            className="w-full p-2 bg-white border border-gray-200 rounded-lg text-[9px] text-gray-500 select-all mb-4 focus:outline-none"
                                            value={backupString}
                                            rows="2"
                                        />
                                    )}

                                    <div className="border-t border-blue-100/50 pt-3">
                                        <input 
                                            type="text"
                                            className="w-full p-2 border border-gray-200 bg-white rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-200 text-xs text-gray-700 mb-2"
                                            placeholder="在此貼上舊的備份碼..."
                                            value={importString}
                                            onChange={(e) => setImportString(e.target.value)}
                                        />
                                        <button 
                                            onClick={handleImportBackup}
                                            className="w-full py-1.5 bg-gray-800 text-white font-bold rounded-xl text-xs hover:bg-gray-900 active:scale-95 transition-all"
                                        >
                                            確認匯入還原
                                        </button>
                                    </div>
                                </div>
                                
                                <button 
                                    onClick={() => {
                                        saveApiKeyToLocal(tempApiKey.trim());
                                        setIsKeyModalOpen(false);
                                        showToast("設定儲存成功！✨");
                                    }}
                                    className="w-full py-3 bg-gradient-to-r from-amber-400 to-orange-400 text-white rounded-2xl font-bold hover:from-amber-500 hover:to-orange-500 transition-all shadow-sm active:scale-95"
                                >
                                    完成設定 ✨
                                </button>
                            </div>
                        </div>
                    )}

                    {/* Bottom Navigation */}
                    <div className="absolute bottom-0 w-full bg-white rounded-t-[2.5rem] shadow-[0_-10px_40px_rgba(0,0,0,0.05)] border-t border-gray-100 flex justify-around items-center p-3 pb-8 z-40">
                        <button 
                            onClick={() => setActiveTab('read')}
                            className={`flex flex-col items-center gap-1 p-3 px-6 rounded-2xl transition-all duration-300 no-tap-highlight
                                ${activeTab === 'read' ? 'bg-orange-100 text-orange-600 scale-110' : 'text-gray-400 hover:bg-gray-50'}`}
                        >
                            <span className="text-2xl mb-1">📖</span>
                            <span className="text-xs font-bold">閱讀翻譯</span>
                        </button>
                        
                        <button 
                            onClick={() => setActiveTab('words')}
                            className={`flex flex-col items-center gap-1 p-3 px-6 rounded-2xl transition-all duration-300 no-tap-highlight relative
                                ${activeTab === 'words' ? 'bg-pink-100 text-pink-600 scale-110' : 'text-gray-400 hover:bg-gray-50'}`}
                        >
                            <span className="text-2xl mb-1">💖</span>
                            <span className="text-xs font-bold">收藏單字</span>
                            {savedWords.length > 0 && (
                                <span className="absolute top-2 right-4 bg-pink-500 text-white text-[10px] font-black px-1.5 py-0.5 rounded-full border-2 border-white">
                                    {savedWords.length}
                                </span>
                            )}
                        </button>

                        <button 
                            onClick={() => setActiveTab('game')}
                            className={`flex flex-col items-center gap-1 p-3 px-6 rounded-2xl transition-all duration-300 no-tap-highlight
                                ${activeTab === 'game' ? 'bg-blue-100 text-blue-600 scale-110' : 'text-gray-400 hover:bg-gray-50'}`}
                        >
                            <span className="text-2xl mb-1">🎮</span>
                            <span className="text-xs font-bold">遊戲複習</span>
                        </button>
                    </div>

                    {/* CSS 動畫 */}
                    <style dangerouslySetInnerHTML={{__html: `
                        @keyframes fadeIn {
                            from { opacity: 0; transform: translateY(10px); }
                            to { opacity: 1; transform: translateY(0); }
                        }
                        .animate-fade-in {
                            animation: fadeIn 0.4s ease-out forwards;
                        }
                    `}} />
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
