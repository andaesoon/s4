# s4
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>실시간 음성 통역</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font Awesome for icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    <!-- Google Fonts - Inter -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
        }
        .container {
            max-width: 900px;
        }
        .pulse-animation {
            animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }
        @keyframes pulse {
            0%, 100% {
                transform: scale(1);
            }
            50% {
                transform: scale(1.1);
            }
        }
        .mic-button:hover .mic-icon {
            color: #d1d5db;
        }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">

    <div class="container bg-white p-8 rounded-3xl shadow-2xl w-full flex flex-col items-center space-y-8">
        <h1 class="text-3xl font-bold text-gray-800 text-center">다국어 실시간 음성 통역</h1>

        <!-- 언어 선택 및 전환 버튼 -->
        <div class="flex flex-col md:flex-row items-center justify-center w-full space-y-4 md:space-y-0 md:space-x-4">
            <select id="sourceLang" class="p-3 bg-gray-100 rounded-xl text-gray-700 w-full md:w-1/3 text-lg focus:outline-none focus:ring-2 focus:ring-blue-500 transition duration-300">
                <option value="ko" selected>한국어</option>
                <option value="ru">러시아어</option>
                <option value="zh-CN">중국어</option>
            </select>
            <div class="flex items-center text-xl text-gray-400">
                <button id="swapLang" class="p-2 rounded-full hover:bg-gray-200 transition duration-300 focus:outline-none focus:ring-2 focus:ring-blue-500">
                    <i class="fas fa-arrows-alt-h"></i>
                </button>
            </div>
            <select id="targetLang" class="p-3 bg-gray-100 rounded-xl text-gray-700 w-full md:w-1/3 text-lg focus:outline-none focus:ring-2 focus:ring-blue-500 transition duration-300">
                <option value="ru">러시아어</option>
                <option value="zh-CN">중국어</option>
                <option value="ko" selected>한국어</option>
            </select>
        </div>

        <!-- 마이크 버튼 -->
        <div class="flex flex-col items-center space-y-4">
            <button id="micButton" class="mic-button relative w-28 h-28 rounded-full bg-blue-500 text-white shadow-lg flex items-center justify-center focus:outline-none focus:ring-4 focus:ring-blue-300 transition-all duration-300 transform active:scale-95">
                <i id="micIcon" class="fas fa-microphone text-4xl transition-all duration-300"></i>
                <span id="loadingSpinner" class="hidden absolute w-28 h-28 rounded-full border-4 border-white border-t-transparent animate-spin"></span>
            </button>
            
            <!-- 추가 기능 버튼들 -->
            <div class="flex space-x-4 mt-4">
                <button id="translateManual" class="px-4 py-2 bg-gray-200 text-gray-800 rounded-full font-medium hover:bg-gray-300 transition duration-300">
                    <i class="fas fa-arrow-right"></i> 텍스트 번역
                </button>
                <button id="clearButton" class="px-4 py-2 bg-gray-200 text-gray-800 rounded-full font-medium hover:bg-gray-300 transition duration-300">
                    <i class="fas fa-redo"></i> 초기화
                </button>
            </div>

            <!-- 대화 모드 토글 -->
            <div class="flex items-center mt-4">
                <input type="checkbox" id="conversationMode" class="form-checkbox h-5 w-5 text-blue-600 rounded">
                <label for="conversationMode" class="ml-2 text-sm text-gray-700 select-none">대화 모드 (자동 언어 전환)</label>
            </div>
        </div>

        <!-- 상태 메시지 -->
        <p id="statusMessage" class="text-sm text-gray-500 mt-4 text-center">
            마이크 버튼을 눌러 통역을 시작하세요.
        </p>

        <!-- 번역 결과 표시 영역 -->
        <div class="w-full space-y-6">
            <div class="bg-gray-100 p-6 rounded-2xl shadow-inner">
                <h2 class="text-xl font-medium text-gray-800 mb-2">음성 인식 결과</h2>
                <textarea id="sourceText" class="w-full bg-transparent text-gray-600 leading-relaxed min-h-[4rem] border-none focus:outline-none resize-none"></textarea>
            </div>
            <div class="bg-gray-100 p-6 rounded-2xl shadow-inner">
                <h2 class="text-xl font-medium text-gray-800 mb-2">번역 결과</h2>
                <p id="translatedText" class="text-gray-600 leading-relaxed min-h-[4rem]"></p>
            </div>
        </div>

    </div>

    <script>
        // API 키를 여기에 입력하세요. (Gemini API는 이 환경에서 자동 제공)
        const API_KEY = "";

        // UI 요소 가져오기
        const micButton = document.getElementById('micButton');
        const swapLangButton = document.getElementById('swapLang');
        const translateManualButton = document.getElementById('translateManual');
        const clearButton = document.getElementById('clearButton');
        const conversationModeCheckbox = document.getElementById('conversationMode');
        const micIcon = document.getElementById('micIcon');
        const statusMessage = document.getElementById('statusMessage');
        const sourceLangSelect = document.getElementById('sourceLang');
        const targetLangSelect = document.getElementById('targetLang');
        const sourceTextarea = document.getElementById('sourceText');
        const translatedTextP = document.getElementById('translatedText');

        // 음성 인식 객체 초기화 (브라우저 지원 확인)
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        let recognition = null;
        let isRecording = false;

        if (SpeechRecognition) {
            recognition = new SpeechRecognition();
            recognition.continuous = false; // 한 번의 발화만 인식
            recognition.interimResults = false; // 최종 결과만 반환
        } else {
            statusMessage.textContent = '죄송합니다. 이 브라우저에서는 음성 인식을 지원하지 않습니다.';
            micButton.disabled = true;
        }

        // --- 이벤트 리스너 ---
        micButton.addEventListener('click', () => {
            if (recognition) {
                if (isRecording) {
                    recognition.stop();
                    isRecording = false;
                    setButtonState('idle');
                } else {
                    startRecognition();
                }
            }
        });

        swapLangButton.addEventListener('click', () => {
            const temp = sourceLangSelect.value;
            sourceLangSelect.value = targetLangSelect.value;
            targetLangSelect.value = temp;
        });

        translateManualButton.addEventListener('click', () => {
            const textToTranslate = sourceTextarea.value;
            if (textToTranslate.trim() !== '') {
                translateAndSpeak(textToTranslate);
            } else {
                statusMessage.textContent = '번역할 텍스트를 입력하세요.';
            }
        });

        clearButton.addEventListener('click', () => {
            sourceTextarea.value = '';
            translatedTextP.textContent = '';
            statusMessage.textContent = '마이크 버튼을 눌러 통역을 시작하세요.';
        });

        // --- 기능 함수 ---
        function startRecognition() {
            const sourceLang = sourceLangSelect.value;
            recognition.lang = sourceLang;

            isRecording = true;
            setButtonState('recording');
            sourceTextarea.value = '듣고 있습니다...';
            translatedTextP.textContent = '';
            statusMessage.textContent = `${getLanguageName(sourceLang)} 음성을 인식하고 있습니다...`;

            recognition.start();

            recognition.onresult = (event) => {
                const speechResult = event.results[0][0].transcript;
                sourceTextarea.value = speechResult;
                statusMessage.textContent = '번역 중...';
                translateAndSpeak(speechResult);
            };

            recognition.onerror = (event) => {
                console.error(event.error);
                statusMessage.textContent = '음성 인식 오류가 발생했습니다. 다시 시도해 주세요.';
                isRecording = false;
                setButtonState('idle');
            };

            recognition.onend = () => {
                isRecording = false;
                setButtonState('idle');
            };
        }

        async function translateAndSpeak(text) {
            const sourceLangCode = sourceLangSelect.value;
            const targetLangCode = targetLangSelect.value;
            
            const sourceLangName = getLanguageName(sourceLangCode);
            const targetLangName = getLanguageName(targetLangCode);

            const prompt = `Translate the following text from ${sourceLangName} to ${targetLangName}: ${text}`;
            
            const payload = {
                contents: [{
                    parts: [{ text: prompt }]
                }],
            };

            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${API_KEY}`;

            try {
                if (!API_KEY) {
                    throw new Error("API 키가 설정되지 않았습니다. Gemini API는 이 환경에서 자동으로 제공됩니다. 잠시 후 다시 시도해 주세요.");
                }

                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                
                if (!response.ok) {
                    const error = await response.json();
                    throw new Error(`API 오류: ${JSON.stringify(error)}`);
                }

                const result = await response.json();
                const translatedText = result?.candidates?.[0]?.content?.parts?.[0]?.text;

                if (translatedText) {
                    translatedTextP.textContent = translatedText;
                    statusMessage.textContent = '통역 완료';
                    speakText(translatedText, targetLangCode);

                    // 대화 모드 활성화 시 언어 자동 전환
                    if (conversationModeCheckbox.checked) {
                        swapLanguages();
                        setTimeout(() => {
                            startRecognition();
                        }, 1000); // 1초 후 다음 통역 시작
                    }
                } else {
                    throw new Error("번역 결과를 가져올 수 없습니다.");
                }

            } catch (error) {
                console.error('번역 오류:', error);
                statusMessage.textContent = `번역 중 오류가 발생했습니다: ${error.message}`;
            }
        }

        function speakText(text, lang) {
            const utterance = new SpeechSynthesisUtterance(text);
            utterance.lang = lang;
            speechSynthesis.speak(utterance);
        }

        function setButtonState(state) {
            if (state === 'recording') {
                micButton.classList.add('pulse-animation');
                micButton.classList.remove('bg-blue-500');
                micButton.classList.add('bg-red-500');
                micIcon.classList.remove('fa-microphone');
                micIcon.classList.add('fa-stop-circle');
            } else {
                micButton.classList.remove('pulse-animation');
                micButton.classList.remove('bg-red-500');
                micButton.classList.add('bg-blue-500');
                micIcon.classList.remove('fa-stop-circle');
                micIcon.classList.add('fa-microphone');
            }
        }

        function swapLanguages() {
            const tempSource = sourceLangSelect.value;
            const tempTarget = targetLangSelect.value;
            sourceLangSelect.value = tempTarget;
            targetLangSelect.value = tempSource;
        }
        
        function getLanguageName(code) {
            switch(code) {
                case 'ko': return '한국어';
                case 'ru': return '러시아어';
                case 'zh-CN': return '중국어';
                default: return '';
            }
        }
    </script>

</body>
</html>
