<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>시험문제 만들기</title>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        /* 기본 스타일 */
        body {
            font-family: 'Roboto', sans-serif;
            background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            height: 100vh;
            overflow: hidden;
        }

        .pokemon-container {
            position: fixed;
            top: 0;
            bottom: 0;
            width: 120px; /* 너비를 조금 더 넓게 조정 */
            display: flex;
            flex-direction: column;
            justify-content: space-around;
            align-items: center;
            overflow-y: auto;
        }

        .pokemon-container.left {
            left: 0;
        }

        .pokemon-container.right {
            right: 0;
        }

        .pokemon-container img {
            width: 100px; /* 이미지 크기를 키움 */
            height: 100px; /* 이미지 크기를 키움 */
        }

        .container {
            max-width: 800px;
            width: 100%;
            margin: 20px;
            padding: 20px;
            background-color: #ffffff;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            border-radius: 10px;
            z-index: 1;
            overflow-y: auto;
            max-height: 90vh;
        }

        header {
            text-align: center;
            padding: 20px 0;
            background-color: #007bff;
            color: #fff;
            border-radius: 10px 10px 0 0;
        }

        header h1 {
            margin: 0;
            font-size: 24px;
            font-weight: 700;
        }

        main {
            padding: 20px;
        }

        .upload-section, .ocr-result-section, .text-input-section, .question-section {
            margin-bottom: 20px;
        }

        .btn {
            display: inline-block;
            padding: 10px 20px;
            font-size: 16px;
            color: #fff;
            background-color: #007bff;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }

        .btn:hover {
            background-color: #0056b3;
        }

        .dropdown, .input-field, textarea {
            width: 100%;
            padding: 10px;
            margin-top: 10px;
            border: 1px solid #ccc;
            border-radius: 5px;
            box-sizing: border-box;
        }

        .questions-section {
            margin-top: 20px;
        }

        .popup {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 20px;
            background-color: rgba(0, 0, 0, 0.8);
            color: #fff;
            font-size: 18px;
            border-radius: 5px;
            z-index: 1000;
        }

        /* 추가 스타일 */
        h2 {
            font-size: 20px;
            font-weight: 600;
            color: #333;
        }

        input, select, textarea {
            font-family: 'Roboto', sans-serif;
        }

        @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap');
    </style>
</head>
<body>
    <div class="pokemon-container left"></div>
    <div class="container">
        <header>
            <h1>시험문제 만들기</h1>
        </header>
        <main>
            <section class="upload-section">
                <button id="upload-btn" class="btn">PDF 업로드</button>
                <p id="upload-status"></p>
                <input type="file" id="pdf-file" accept="application/pdf" style="display: none;">
            </section>
            <section class="ocr-result-section">
                <h2>OCR 결과</h2>
                <textarea id="ocr-result" rows="5" cols="50" readonly></textarea>
            </section>
            <section class="text-input-section">
                <h2>텍스트 입력</h2>
                <textarea id="text-input" rows="5" cols="50" placeholder="텍스트를 입력하세요"></textarea>
            </section>
            <section class="question-section">
                <h2>문제 유형 선택</h2>
                <select id="question-type" class="dropdown">
                    <option value="ox">O/X 문제</option>
                    <option value="multiple-choice">객관식문제</option>
                    <option value="subjective">주관식문제</option>
                    <option value="mixed">혼합(o/x, 객관식, 주관식)</option>
                </select>
                <h2>문항수 입력</h2>
                <input type="number" id="question-count" min="1" placeholder="문항수를 입력하세요" class="input-field">
                <h2>언어 선택</h2>
                <select id="language" class="dropdown">
                    <option value="ko">한국어</option>
                    <option value="en">영어</option>
                </select>
                <button id="generate-btn" class="btn">문제 생성</button>
            </section>
            <section id="questions" class="questions-section">
                <h2>생성된 문제</h2>
                <textarea id="generated-questions" rows="5" cols="50" readonly></textarea>
            </section>
        </main>
    </div>
    <div class="pokemon-container right"></div>
    <div id="popup" class="popup">문제 생성 중입니다...</div>
    <script>
        document.getElementById('upload-btn').addEventListener('click', () => {
            document.getElementById('pdf-file').click();
        });

        document.getElementById('pdf-file').addEventListener('change', async (event) => {
            const file = event.target.files[0];
            if (file) {
                document.getElementById('upload-status').textContent = '파일 업로드 중입니다.';
                const formData = new FormData();
                formData.append('file', file);

                try {
                    const response = await fetch('/upload', {
                        method: 'POST',
                        body: formData
                    });
                    const data = await response.json();
                    document.getElementById('upload-status').textContent = '';
                    if (data.text) {
                        document.getElementById('ocr-result').value = data.text;
                    } else {
                        document.getElementById('ocr-result').value = 'OCR 결과를 가져오지 못했습니다.';
                    }
                } catch (error) {
                    console.error('Error:', error);
                    document.getElementById('upload-status').textContent = '파일 업로드 실패';
                }
            }
        });

        document.getElementById('generate-btn').addEventListener('click', async () => {
            const questionType = document.getElementById('question-type').value;
            const questionCount = document.getElementById('question-count').value;
            const textInput = document.getElementById('text-input').value;
            const language = document.getElementById('language').value;

            // 팝업 표시
            const popup = document.getElementById('popup');
            popup.style.display = 'block';

            try {
                const response = await fetch('/generate', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({ questionType, questionCount, textInput, language })
                });
                const data = await response.json();
                const generatedQuestions = document.getElementById('generated-questions');
                generatedQuestions.value = '';

                data.questions.forEach((question, index) => {
                    generatedQuestions.value += `${question}\n\n`;
                });

                // 팝업 숨기기
                popup.style.display = 'none';
            } catch (error) {
                console.error('Error:', error);
                // 팝업 숨기기
                popup.style.display = 'none';
            }
        });

        // 포켓몬 이미지를 가져와서 표시하는 함수
        async function fetchPokemon() {
            const leftContainer = document.querySelector('.pokemon-container.left');
            const rightContainer = document.querySelector('.pokemon-container.right');
            leftContainer.innerHTML = '';
            rightContainer.innerHTML = '';

            for (let i = 0; i < 5; i++) {
                const randomId = Math.floor(Math.random() * 898) + 1; // 포켓몬 ID는 1부터 898까지
                const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${randomId}`);
                const data = await response.json();
                const img = document.createElement('img');
                img.src = data.sprites.front_default;
                if (i % 2 === 0) {
                    leftContainer.appendChild(img);
                } else {
                    rightContainer.appendChild(img);
                }
            }
        }

        // 페이지 로드 시 포켓몬 이미지 가져오기
        window.onload = fetchPokemon;
    </script>
</body>
</html>
