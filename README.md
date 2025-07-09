<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no" />
  <title>الهروب من الوحش</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Cairo&display=swap');
    body {
      margin: 0;
      font-family: 'Cairo', sans-serif;
      background: linear-gradient(#014d00, #00a400);
      color: white;
      text-align: center;
      direction: rtl;
      overflow: hidden;
      touch-action: none;
    }
    #start-screen {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      height: 100vh;
      background: linear-gradient(135deg, #0a3000 0%, #4caf50 100%);
      color: #fff;
      font-size: 2em;
      user-select: none;
      gap: 20px;
    }
    #start-screen h1 {
      font-size: 3em;
      text-shadow: 0 0 10px #0f0;
      margin: 0;
    }
    #start-screen button {
      font-size: 1.2em;
      padding: 15px 30px;
      border: none;
      border-radius: 15px;
      background-color: #00a400;
      color: white;
      cursor: pointer;
      box-shadow: 0 0 10px #00ff00;
      transition: background-color 0.3s;
    }
    #start-screen button:hover {
      background-color: #007700;
    }
    #game {
      position: relative;
      width: 100vw;
      height: 60vh;
      margin: auto;
      overflow: hidden;
      border: 2px solid white;
      background: linear-gradient(to top, #0a3000 0%, #4caf50 100%);
      display: none;
    }
    #rocket {
      position: absolute;
      bottom: 10%;
      left: 50%;
      transform: translateX(-50%);
      font-size: 40px;
    }
    .coin, .meteor, .monster, .fireball, .ball {
      position: absolute;
      font-size: 30px;
      width: 40px;
      height: 40px;
      text-align: center;
      user-select: none;
      pointer-events: none;
    }
    .coin::before { content: '🦴'; }
    .meteor::before { content: '🔥'; }
    .monster::before { content: '👹'; }
    .fireball::before { content: '🔥'; }
    .ball::before { content: '⚽'; font-size: 40px; }
    
    #controls {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 10px;
      margin: 10px auto;
    }
    .control-row {
      display: flex;
      justify-content: center;
      gap: 10px;
    }
    .arrow {
      font-size: 20px;
      padding: 10px;
      width: 50px;
      height: 50px;
      background-color: #fff;
      color: #014d00;
      border: 2px solid #00a400;
      border-radius: 50%;
      box-shadow: 0 0 5px #000;
      cursor: pointer;
      transition: background-color 0.2s;
      user-select: none;
    }
    .arrow:hover {
      background-color: #c0ffc0;
    }
    #win-screen, #game-over-screen {
      display: none;
      position: fixed;
      top: 0; left: 0;
      width: 100vw; height: 100vh;
      background: rgba(0,0,0,0.8);
      color: yellow;
      font-size: 2em;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      z-index: 10;
    }
    #developer {
      margin-top: 20px;
      font-size: 0.9em;
      color: #eee;
      user-select: none;
    }
    #highscore {
      font-size: 1em;
      margin-top: 5px;
    }
    #restartBtn, #gameOverRestartBtn {
      margin-top: 20px;
      padding: 10px 20px;
      font-size: 1em;
      background-color: #00a400;
      color: white;
      border: none;
      border-radius: 10px;
      cursor: pointer;
    }
  </style>
</head>
<body ontouchstart="startTouch(event)" ontouchmove="moveTouch(event)" ontouchend="endTouch()">
  <div id="start-screen">
    <h1>الهروب من الوحش</h1>
    <button onclick="startGame()">ابدأ اللعبة</button>
    <div id="developer">المطور: عبدالرحمن مجدي</div>
  </div>

  <div id="game">
    <div id="rocket">🐕</div>
  </div>
  <div id="score">العظام: 0</div>
  <div id="lives">الأرواح: ❤️❤️❤️❤️❤️</div>
  <div id="highscore">أعلى نتيجة: 0</div>
  <div id="controls">
    <div class="control-row">
      <button class="arrow" onclick="move('up')">⬆️</button>
    </div>
    <div class="control-row">
      <button class="arrow" onclick="move('right')">➡️</button>
      <button class="arrow" onclick="move('down')">⬇️</button>
      <button class="arrow" onclick="move('left')">⬅️</button>
    </div>
  </div>

  <div id="win-screen">
    🎉 تهانينا! لقد ربحت اللعبة! 🎉<br />
    🐶 لقد وجدت الكرة الضائعة!<br />
    <button id="restartBtn" onclick="restartGame()">🔁 إعادة اللعب</button>
  </div>

  <div id="game-over-screen">
    انتهت الأرواح!<br />
    العب من جديد لتصل إلى هدفك.<br />
    <button id="gameOverRestartBtn" onclick="restartGame()">🔁 إعادة اللعب</button>
  </div>

  <audio id="coinSound" src="https://cdn.pixabay.com/audio/2022/03/15/audio_1a7fa4905e.mp3"></audio>
  <audio id="hitSound" src="https://cdn.pixabay.com/audio/2022/03/15/audio_6c409e8dd2.mp3"></audio>
  <audio id="monsterSound" src="https://cdn.pixabay.com/audio/2022/03/15/audio_558fa5ad0b.mp3"></audio>
  <audio id="winSound" src="https://cdn.pixabay.com/audio/2022/03/15/audio_3b86e13961.mp3"></audio>

  <script>
    const rocket = document.getElementById("rocket");
    const game = document.getElementById("game");
    const scoreDisplay = document.getElementById("score");
    const livesDisplay = document.getElementById("lives");
    const coinSound = document.getElementById("coinSound");
    const hitSound = document.getElementById("hitSound");
    const monsterSound = document.getElementById("monsterSound");
    const winScreen = document.getElementById("win-screen");
    const winSound = document.getElementById("winSound");
    const highscoreDisplay = document.getElementById("highscore");
    const gameOverScreen = document.getElementById("game-over-screen");

    let rocketX = 50;
    let rocketY = 10;
    let coins = 0;
    let lives = 5;
    let rocketSpeed = 6;
    let meteorInterval;
    let meteorIntervalTime = 3000;
    let ballVisible = false;
    let ballEl = null;
    let highscore = localStorage.getItem("highscore") || 0;
    highscoreDisplay.innerText = `أعلى نتيجة: ${highscore}`;

    function updateRocketPosition() {
      rocket.style.left = rocketX + "%";
      rocket.style.bottom = rocketY + "%";
    }

    function updateLivesDisplay() {
      livesDisplay.innerHTML = 'الأرواح: ' + '❤️'.repeat(lives);
    }

    function updateHighscore() {
      if (coins > highscore) {
        highscore = coins;
        localStorage.setItem("highscore", highscore);
        highscoreDisplay.innerText = `أعلى نتيجة: ${highscore}`;
      }
    }

    function checkCollision(item) {
      const rocketRect = rocket.getBoundingClientRect();
      const itemRect = item.getBoundingClientRect();
      return (
        rocketRect.left < itemRect.right &&
        rocketRect.right > itemRect.left &&
        rocketRect.top < itemRect.bottom &&
        rocketRect.bottom > itemRect.top
      );
    }

    function spawnItem(type) {
      const item = document.createElement("div");
      item.classList.add(type);
      item.style.left = Math.random() * 90 + "%";
      item.style.top = "0px";
      game.appendChild(item);

      const interval = setInterval(() => {
        let top = parseInt(item.style.top);
        item.style.top = (top + 4) + "px";

        if (checkCollision(item)) {
          if (type === "coin") {
            coinSound.play();
            coins++;
            updateHighscore();
            scoreDisplay.innerText = "العظام: " + coins;
            item.remove();
            clearInterval(interval);

            if (coins % 10 === 0) {
              meteorIntervalTime = Math.max(1000, meteorIntervalTime - 300);
              clearInterval(meteorInterval);
              meteorInterval = setInterval(() => spawnItem("meteor"), meteorIntervalTime);
              rocketSpeed += 1;
            }
            if (coins % 30 === 0) spawnMonster();
            if (coins % 25 === 0 && lives < 10) {
              lives++;
              updateLivesDisplay();
            }
            if (coins >= 5000 && !ballVisible) {
              showLostBall();
            }
          } else {
            hitSound.play();
            lives--;
            updateLivesDisplay();
            item.remove();
            clearInterval(interval);
            if (lives <= 0) {
              gameOver();
            }
          }
        }

        if (top > game.offsetHeight) {
          item.remove();
          clearInterval(interval);
        }
      }, 30);
    }

    function showLostBall() {
      ballEl = document.createElement("div");
      ballEl.classList.add("ball");
      ballEl.style.left = Math.random() * 90 + "%";
      ballEl.style.top = "0px";
      game.appendChild(ballEl);
      ballVisible = true;

      const interval = setInterval(() => {
        let top = parseInt(ballEl.style.top);
        ballEl.style.top = (top + 3) + "px";

        if (checkCollision(ballEl)) {
          winSound.play();
          winScreen.style.display = "flex";
          clearInterval(interval);
        }

        if (top > game.offsetHeight) {
          ballEl.remove();
          clearInterval(interval);
          ballVisible = false;
          setTimeout(() => {
            if (!winScreen.style.display.includes("flex")) {
              showLostBall();
            }
          }, 10000);
        }
      }, 30);
    }

    function spawnMonster() {
      const monster = document.createElement("div");
      monster.className = "monster";
      monster.style.left = "50%";
      monster.style.top = "0%";
      monster.style.fontSize = "150px"; // تكبير الحجم
      game.appendChild(monster);
      monsterSound.play();

      for (let i = 0; i < 5; i++) { // 5 وحوش صغيرة بدلاً من 10
        setTimeout(() => spawnMiniMonster(monster), i * 300);
      }
      setTimeout(() => monster.remove(), 6000);
    }

    function spawnMiniMonster(monster) {
      const mini = document.createElement("div");
      mini.className = "monster";
      mini.style.fontSize = "40px";
      mini.style.left = monster.offsetLeft + Math.random() * 100 - 50 + "px";
      mini.style.top = monster.offsetTop + 100 + "px";
      game.appendChild(mini);

      let direction = Math.random() > 0.5 ? 1 : -1; // حركة يمين أو يسار عشوائية
      let speedX = 1 + Math.random() * 1.5; // سرعة أفقية متغيرة

      const interval = setInterval(() => {
        let top = parseInt(mini.style.top);
        let left = parseInt(mini.style.left);

        // تحديث الموقع رأسياً وأفقياً
        mini.style.top = (top + 3) + "px";
        mini.style.left = (left + direction * speedX) + "px";

        // إذا وصل يمين أو يسار الحافة يعكس الاتجاه
        if (left <= 0) direction = 1;
        if (left >= game.offsetWidth - mini.offsetWidth) direction = -1;

        // التحقق من التصادم
        if (checkCollision(mini)) {
          hitSound.play();
          lives--;
          updateLivesDisplay();
          mini.remove();
          clearInterval(interval);
          if (lives <= 0) {
            gameOver();
          }
        }

        if (top > game.offsetHeight) {
          mini.remove();
          clearInterval(interval);
        }
      }, 30);
    }

    function gameOver() {
      game.style.display = "none";
      gameOverScreen.style.display = "flex";
    }

    function restartGame() {
      location.reload();
    }

    function startGame() {
      document.getElementById("start-screen").style.display = "none";
      gameOverScreen.style.display = "none";
      winScreen.style.display = "none";
      game.style.display = "block";
      rocketX = 50;
      rocketY = 10;
      coins = 0;
      lives = 5;
      rocketSpeed = 6;
      meteorIntervalTime = 3000;
      updateRocketPosition();
      updateLivesDisplay();
      scoreDisplay.innerText = "العظام: 0";
      setInterval(() => spawnItem("coin"), 2000);
      meteorInterval = setInterval(() => spawnItem("meteor"), meteorIntervalTime);
    }

    function move(direction) {
      if (direction === "left" && rocketX > 0) rocketX -= rocketSpeed;
      if (direction === "right" && rocketX < 95) rocketX += rocketSpeed;
      if (direction === "up" && rocketY < 90) rocketY += rocketSpeed;
      if (direction === "down" && rocketY > 0) rocketY -= rocketSpeed;
      updateRocketPosition();
    }

    window.addEventListener("keydown", (e) => {
      if (e.key === "ArrowLeft") move("left");
      if (e.key === "ArrowRight") move("right");
      if (e.key === "ArrowUp") move("up");
      if (e.key === "ArrowDown") move("down");
    });

    let touchStartX = null;
    let touchStartY = null;

    function startTouch(evt) {
      const touch = evt.touches[0];
      touchStartX = touch.clientX;
      touchStartY = touch.clientY;
    }

    function moveTouch(evt) {
      if (!touchStartX || !touchStartY) return;
      let touchEndX = evt.touches[0].clientX;
      let touchEndY = evt.touches[0].clientY;
      let diffX = touchEndX - touchStartX;
      let diffY = touchEndY - touchStartY;

      if (Math.abs(diffX) > Math.abs(diffY)) {
        if (diffX > 20) move("right");
        else if (diffX < -20) move("left");
      } else {
        if (diffY < -20) move("up");
        else if (diffY > 20) move("down");
      }
      endTouch();
    }

    function endTouch() {
      touchStartX = null;
      touchStartY = null;
    }
  </script>
</body>
</html>
