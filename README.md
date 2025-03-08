shashushaun
shashushaun
Add files via upload
348dd36
 · 
21 minutes ago

Code

Blame
268 lines (250 loc) · 8.83 KB
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>トランプゲーム HIGH&LOW</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            background: linear-gradient(135deg, #1e3c72, #2a5298);
            color: #fff;
        }
        #game-area {
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 15px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
        }
        #card-container {
            position: relative;
            width: 180px;
            height: 260px;
            margin-bottom: 20px;
        }
        .card {
            position: relative;
            width: 100%;
            height: 100%;
            background: #fff;
            border-radius: 10px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
            transition: transform 0.3s ease;
            display: flex;
            flex-wrap: wrap;
            align-content: flex-start;
            justify-content: center;
            padding: 10px;
            box-sizing: border-box;
            overflow: hidden;
        }
        .card img {
            object-fit: contain;
            margin: 2px;
        }
        .card .rank {
            position: absolute;
            bottom: 10px;
            right: 10px;
            font-size: 24px;
            font-weight: bold;
            color: #000;
        }
        .card:hover {
            transform: scale(1.05);
        }
        #score {
            font-size: 28px;
            font-weight: bold;
            margin-bottom: 20px;
            text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
        }
        #buttons {
            display: flex;
            gap: 15px;
            margin-bottom: 20px;
        }
        .button {
            padding: 12px 25px;
            font-size: 18px;
            font-weight: bold;
            cursor: pointer;
            border: none;
            border-radius: 8px;
            background: #ff6f61;
            color: white;
            box-shadow: 0 3px 6px rgba(0, 0, 0, 0.2);
            transition: background 0.3s ease, transform 0.2s ease;
            min-width: 100px;
        }
        .button:hover:not(:disabled) {
            background: #ff8c7f;
            transform: translateY(-2px);
        }
        .button:disabled {
            background: #ccc;
            cursor: not-allowed;
            box-shadow: none;
        }
        #control-buttons {
            display: flex;
            gap: 15px;
        }
        #message {
            font-size: 24px;
            font-weight: bold;
            margin-top: 20px;
            text-align: center;
            text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
        }
    </style>
</head>
<body>
    <div id="game-area">
        <div id="card-container"></div>
        <div id="score">スコア: 0</div>
        <div id="buttons">
            <button id="high" class="button">HIGH</button>
            <button id="low" class="button">LOW</button>
        </div>
        <div id="control-buttons">
            <button id="reset" class="button">リセット</button>
        </div>
        <div id="message"></div>
    </div>
    <script>
        const cardContainer = document.getElementById('card-container');
        const scoreElement = document.getElementById('score');
        const highButton = document.getElementById('high');
        const lowButton = document.getElementById('low');
        const resetButton = document.getElementById('reset');
        const messageElement = document.getElementById('message');

        let deck = [];
        let currentCard = null;
        let score = 0;
        let isPlaying = false;

        function createDeck() {
            const suits = ['ハート', 'ダイヤ', 'クラブ', 'スペード'];
            const ranks = ['A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K'];
            deck = [];
            for (let suit of suits) {
                for (let rank of ranks) {
                    deck.push({ suit, rank });
                }
            }
            shuffleDeck();
        }

        function shuffleDeck() {
            for (let i = deck.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [deck[i], deck[j]] = [deck[j], deck[i]];
            }
        }

        function drawCard() {
            if (deck.length === 0) {
                createDeck();
            }
            return deck.pop();
        }

        function getCardValue(card) {
            const rankValues = { 'A': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9, '10': 10, 'J': 11, 'Q': 12, 'K': 13 };
            return rankValues[card.rank];
        }

        function displayCard(card) {
            cardContainer.innerHTML = '';
            const cardElement = document.createElement('div');
            cardElement.classList.add('card');

            const count = getCardValue(card);
            const cardWidth = cardContainer.clientWidth - 20;
            const cardHeight = cardContainer.clientHeight - 20;
            const totalMargin = 4;
            const cols = Math.ceil(Math.sqrt(count));
            const rows = Math.ceil(count / cols);
            const imgSize = Math.min(
                (cardWidth - (totalMargin * (cols - 1))) / cols,
                (cardHeight - (totalMargin * (rows - 1))) / rows
            );

            for (let i = 0; i < count; i++) {
                const img = document.createElement('img');
                img.src = 'https://pbs.twimg.com/profile_images/1890585622500503552/rOXm_mUw_400x400.jpg';
                img.alt = `${card.rank} of ${card.suit}`;
                img.style.width = `${imgSize}px`;
                img.style.height = `${imgSize}px`;
                cardElement.appendChild(img);
            }

            const rankElement = document.createElement('div');
            rankElement.classList.add('rank');
            rankElement.textContent = card.rank;
            cardElement.appendChild(rankElement);

            cardContainer.appendChild(cardElement);
        }

        function startGame() {
            createDeck();
            currentCard = drawCard();
            displayCard(currentCard);
            score = 0;
            updateScore();
            isPlaying = true;
            enableButtons();
            messageElement.textContent = '';
        }

        function resetGame() {
            startGame();
        }

        function updateScore() {
            scoreElement.textContent = `スコア: ${score}`;
            if (score >= 5) {
                messageElement.textContent = `ゲームクリア！しょーんもな𖤐`;
                isPlaying = false;
                disableButtons();
            }
        }

        function enableButtons() {
            highButton.disabled = false;
            lowButton.disabled = false;
            resetButton.disabled = false;
        }

        function disableButtons() {
            highButton.disabled = true;
            lowButton.disabled = true;
            resetButton.disabled = false;
        }

        function predict(prediction) {
            if (!isPlaying) return;
            const nextCard = drawCard();
            const currentValue = getCardValue(currentCard);
            const nextValue = getCardValue(nextCard);
            let correct = false;
            if (prediction === 'high' && nextValue > currentValue) {
                correct = true;
            } else if (prediction === 'low' && nextValue < currentValue) {
                correct = true;
            }
            if (correct) {
                score++;
                updateScore();
                currentCard = nextCard;
                displayCard(currentCard);
            } else {
                messageElement.textContent = `ゲームオーバー！スコア: ${score}`;
                isPlaying = false;
                disableButtons();
            }
        }

        highButton.addEventListener('click', () => predict('high'));
        lowButton.addEventListener('click', () => predict('low'));
        resetButton.addEventListener('click', resetGame);

        startGame();
    </script>
</body>
</html>
​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​
