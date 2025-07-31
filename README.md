<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Coffee & Tea: Compare and Contrast Quiz</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chosen Palette: Warm Neutrals (Amber, Stone, Terracotta) -->
    <!-- Application Structure Plan: A single-page, interactive quiz. It presents one sentence at a time with clickable answer buttons from a word bank. This structure focuses user attention, provides immediate feedback (correct/incorrect), and uses gamification (scoring, progress) to make the learning process more engaging and effective than a static text document. The flow is linear: question -> answer -> feedback -> next, ending with a final score and a restart option. -->
    <!-- Visualization & Content Choices: Report Info: 15 sentences and 15 compare/contrast phrases. Goal: Active learning and testing. Viz/Presentation: Sentences are displayed in a <p> tag. The word bank is a grid of <button> elements. Interaction: User clicks a button. JS checks the answer, provides instant visual feedback (color change), and controls progress. Justification: This interactive format is superior to a static list for language learning, as it provides active recall practice and immediate reinforcement. Library/Method: Vanilla JS for logic, Tailwind CSS for styling. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .quiz-container {
            transition: opacity 0.5s ease-in-out;
        }
        .feedback-correct {
            color: #16a34a; /* green-600 */
        }
        .feedback-incorrect {
            color: #dc2626; /* red-600 */
        }
        .btn-disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }
    </style>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap" rel="stylesheet">
</head>
<body class="bg-amber-50 text-stone-800 flex items-center justify-center min-h-screen p-4">

    <main class="w-full max-w-3xl mx-auto bg-white rounded-2xl shadow-lg p-6 sm:p-8 md:p-12">
        
        <div id="quiz-view">
            <header class="text-center mb-8">
                <h1 class="text-3xl sm:text-4xl font-bold text-amber-800">Compare & Contrast Quiz</h1>
                <p class="text-stone-600 mt-2">Choose the best word or phrase to complete the sentence.</p>
                <div class="w-full bg-stone-200 rounded-full h-2.5 mt-4">
                    <div id="progress-bar" class="bg-amber-600 h-2.5 rounded-full" style="width: 0%"></div>
                </div>
            </header>

            <div class="quiz-container min-h-[120px]">
                <p id="sentence-display" class="text-xl sm:text-2xl text-center leading-relaxed font-medium mb-8"></p>
            </div>

            <div id="word-bank" class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-5 gap-3 mb-6">
            </div>

            <div id="feedback-container" class="text-center min-h-[50px] flex items-center justify-center">
                <p id="feedback-text" class="text-lg font-semibold"></p>
                <button id="next-button" class="hidden bg-amber-600 text-white font-bold py-2 px-6 rounded-lg hover:bg-amber-700 transition-colors">
                    Next &rarr;
                </button>
            </div>
        </div>

        <div id="final-view" class="hidden text-center">
            <h2 class="text-3xl sm:text-4xl font-bold text-amber-800 mb-4">Quiz Complete!</h2>
            <p class="text-2xl text-stone-700 mb-2">Your Score:</p>
            <p id="final-score" class="text-5xl font-bold text-green-600 mb-8"></p>
            <p id="final-message" class="text-lg text-stone-600 mb-8"></p>
            <button id="restart-button" class="bg-amber-600 text-white font-bold py-3 px-8 rounded-lg hover:bg-amber-700 transition-colors text-lg">
                Try Again
            </button>
        </div>

    </main>

    <script>
        const questions = [
            { sentence: "______ coffee and tea contain caffeine, which helps people feel more awake.", answer: "Both" },
            { sentence: "Coffee is made from roasted beans; ______, tea comes from dried leaves.", answer: "on the other hand" },
            { sentence: "Tea is often part of peaceful ceremonies; ______, coffee is usually drunk quickly in busy cafes.", answer: "in contrast" },
            { sentence: "You can drink coffee hot or cold, and ______ you can enjoy tea hot or cold.", answer: "similarly" },
            { sentence: "______ tea, coffee beans are roasted before they are ground.", answer: "Unlike" },
            { sentence: "Coffee has a strong, bold taste; ______, many teas have a lighter, more floral flavor.", answer: "however" },
            { sentence: "Coffee is popular in Western countries; ______, tea is very popular in Asian countries.", answer: "likewise" },
            { sentence: "Making coffee involves grinding beans; ______, making tea involves steeping leaves.", answer: "conversely" },
            { sentence: "Some people like to add milk to their coffee, ______ others prefer to drink it black.", answer: "while" },
            { sentence: "Coffee can be quite bitter, ______ tea often has a milder taste.", answer: "whereas" },
            { sentence: "Coffee, ______ tea, is a drink enjoyed by billions of people worldwide.", answer: "together with" },
            { sentence: "______ to coffee, tea is known for its wide range of flavors and types.", answer: "In comparison" },
            { sentence: "______ many other popular drinks, coffee and tea both offer health benefits.", answer: "Similarly to" },
            { sentence: "Coffee and tea ______ each other in their main ingredient and how they are grown.", answer: "differ from" },
            { sentence: "______ coffee is often drunk alone, tea is frequently shared during social gatherings.", answer: "(another) while" }
        ];

        const wordBankPhrases = [
            "similarly", "on the other hand", "in contrast", "both", "unlike",
            "however", "likewise", "conversely", "whereas", "together with",
            "In comparison", "Similarly to", "while", "differ from", "(another) while"
        ];

        let currentQuestionIndex = 0;
        let score = 0;
        let availableWords = [...wordBankPhrases];

        const sentenceDisplay = document.getElementById('sentence-display');
        const wordBankContainer = document.getElementById('word-bank');
        const feedbackText = document.getElementById('feedback-text');
        const nextButton = document.getElementById('next-button');
        const progressBar = document.getElementById('progress-bar');
        
        const quizView = document.getElementById('quiz-view');
        const finalView = document.getElementById('final-view');
        const finalScoreDisplay = document.getElementById('final-score');
        const finalMessageDisplay = document.getElementById('final-message');
        const restartButton = document.getElementById('restart-button');

        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
        }

        function displayQuestion() {
            const question = questions[currentQuestionIndex];
            sentenceDisplay.textContent = question.sentence.replace('______', '___');
            
            wordBankContainer.innerHTML = '';
            feedbackText.textContent = '';
            nextButton.classList.add('hidden');
            
            shuffleArray(availableWords);
            
            availableWords.forEach(word => {
                const button = document.createElement('button');
                button.textContent = word;
                button.className = "bg-stone-100 text-stone-700 font-medium py-3 px-2 rounded-lg hover:bg-stone-200 transition-colors duration-200 text-sm sm:text-base";
                button.onclick = () => checkAnswer(word);
                wordBankContainer.appendChild(button);
            });

            const progressPercentage = (currentQuestionIndex / questions.length) * 100;
            progressBar.style.width = `${progressPercentage}%`;
        }

        function checkAnswer(chosenWord) {
            const correctAnswer = questions[currentQuestionIndex].answer;
            const buttons = wordBankContainer.querySelectorAll('button');

            buttons.forEach(button => {
                button.disabled = true;
                button.classList.add('btn-disabled');
            });

            if (chosenWord.toLowerCase() === correctAnswer.toLowerCase()) {
                score++;
                feedbackText.textContent = 'Correct!';
                feedbackText.className = 'feedback-correct text-lg font-semibold';
                sentenceDisplay.textContent = questions[currentQuestionIndex].sentence.replace('______', ` ${correctAnswer} `);
            } else {
                feedbackText.innerHTML = `Not quite. The correct answer is: <strong class="text-amber-700">${correctAnswer}</strong>`;
                feedbackText.className = 'feedback-incorrect text-lg font-semibold';
                sentenceDisplay.textContent = questions[currentQuestionIndex].sentence.replace('______', ` ${correctAnswer} `);
            }
            
            availableWords = availableWords.filter(w => w.toLowerCase() !== correctAnswer.toLowerCase());

            nextButton.classList.remove('hidden');
        }

        function nextQuestion() {
            currentQuestionIndex++;
            if (currentQuestionIndex < questions.length) {
                displayQuestion();
            } else {
                showFinalScreen();
            }
        }

        function showFinalScreen() {
            quizView.classList.add('hidden');
            finalView.classList.remove('hidden');
            
            finalScoreDisplay.textContent = `${score} / ${questions.length}`;
            
            let message = '';
            const percentage = (score / questions.length) * 100;
            if (percentage === 100) {
                message = "Perfect score! You're a master of comparison!";
            } else if (percentage >= 75) {
                message = "Excellent job! You have a great understanding of these phrases.";
            } else if (percentage >= 50) {
                message = "Good effort! A little more practice will make you an expert.";
            } else {
                message = "Keep practicing! Reviewing the answers will help a lot.";
            }
            finalMessageDisplay.textContent = message;
            progressBar.style.width = '100%';
        }

        function restartQuiz() {
            currentQuestionIndex = 0;
            score = 0;
            availableWords = [...wordBankPhrases];
            finalView.classList.add('hidden');
            quizView.classList.remove('hidden');
            displayQuestion();
        }

        nextButton.addEventListener('click', nextQuestion);
        restartButton.addEventListener('click', restartQuiz);

        displayQuestion();
    </script>
</body>
</html>
