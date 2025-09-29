<!DOCTYPE html>
<html lang="ko">
<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>강릉, 가뭄 극복 게임</title>
   <script src="https://cdn.tailwindcss.com"></script>
   <style>
       @import url('https://fonts.googleapis.com/css2?family=Jua&display=swap');
      
       body {
           font-family: 'Jua', sans-serif;
           background-color: #f7f3e9;
           display: flex;
           justify-content: center;
           align-items: center;
           height: 100vh;
           margin: 0;
           flex-direction: column;
           overflow: hidden;
       }


       .game-container {
           width: 100%;
           max-width: 600px;
           height: 80vh;
           max-height: 800px;
           position: relative;
           background-color: #a0c2e6;
           border: 8px solid #6b4d3f;
           border-radius: 16px;
           box-shadow: 0 10px 20px rgba(0,0,0,0.2);
           overflow: hidden;
           display: flex;
           flex-direction: column;
       }


       .game-header {
           background-color: #e6b8a0;
           color: #6b4d3f;
           text-align: center;
           padding: 1rem;
           font-size: 1.5rem;
           border-bottom: 4px solid #6b4d3f;
       }


       #gameCanvas {
           flex-grow: 1;
           background-color: #f0f8ff;
           background-image: linear-gradient(to bottom, #a0c2e6, #f0f8ff);
       }
      
       .ui-container {
           position: absolute;
           top: 20px;
           right: 20px;
           display: flex;
           flex-direction: column;
           gap: 1rem;
           text-align: right;
           color: #6b4d3f;
       }


       .gauge-container {
           width: 100px;
           height: 200px;
           background-color: #fff;
           border: 4px solid #6b4d3f;
           border-radius: 12px;
           overflow: hidden;
           box-shadow: inset 0 0 5px rgba(0,0,0,0.2);
           position: relative;
       }


       .gauge-fill {
           width: 100%;
           background-color: #3b82f6;
           transition: height 0.3s ease-out;
           position: absolute;
           bottom: 0;
       }
      
       .gauge-label {
           position: absolute;
           top: 50%;
           left: 50%;
           transform: translate(-50%, -50%);
           font-size: 1.25rem;
           color: #fff;
           text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
           z-index: 10;
       }


       .info-box {
           background-color: #e6b8a0;
           border: 4px solid #6b4d3f;
           border-radius: 12px;
           padding: 0.5rem 1rem;
           font-size: 1.25rem;
       }


       .message-box {
           position: absolute;
           top: 50%;
           left: 50%;
           transform: translate(-50%, -50%);
           background-color: rgba(255, 255, 255, 0.9);
           border: 4px solid #6b4d3f;
           border-radius: 12px;
           padding: 2rem;
           text-align: center;
           box-shadow: 0 10px 20px rgba(0,0,0,0.2);
           z-index: 20;
           display: none;
           flex-direction: column;
           gap: 1rem;
       }


       .message-box button {
           background-color: #6b4d3f;
           color: #fff;
           padding: 0.75rem 1.5rem;
           border: none;
           border-radius: 8px;
           cursor: pointer;
           font-size: 1rem;
           transition: background-color 0.2s;
       }


       .message-box button:hover {
           background-color: #a0c2e6;
       }


       .message-box p {
           font-size: 1.5rem;
           color: #6b4d3f;
       }
   </style>
</head>
<body>
   <div class="game-container">
       <div class="game-header">강릉, 가뭄 극복!</div>
       <canvas id="gameCanvas"></canvas>
       <div class="ui-container">
           <div class="gauge-container">
               <div class="gauge-fill" id="waterGaugeFill"></div>
               <div class="gauge-label" id="gaugeLabel">0%</div>
           </div>
           <div class="info-box">
               놓친 물방울: <span id="missedDrops">0</span>
           </div>
       </div>
   </div>
  
   <div id="messageBox" class="message-box">
       <p id="messageText"></p>
       <button id="restartButton">다시 시작</button>
   </div>


   <script>
       const canvas = document.getElementById('gameCanvas');
       const ctx = canvas.getContext('2d');
       const waterGaugeFill = document.getElementById('waterGaugeFill');
       const gaugeLabel = document.getElementById('gaugeLabel');
       const missedDropsSpan = document.getElementById('missedDrops');
       const messageBox = document.getElementById('messageBox');
       const messageText = document.getElementById('messageText');
       const restartButton = document.getElementById('restartButton');


       let player;
       let raindrops = [];
       let score = 0;
       let missedDrops = 0;
       let isGameOver = false;
       let lastUpdateTime = 0;
       const targetGauge = 100;


       const playerImage = new Image();
       playerImage.src = 'https://placehold.co/100x100/6b4d3f/fff?text=주인공';
      
       const raindropImage = new Image();
       raindropImage.src = 'https://placehold.co/30x30/3b82f6/fff?text=💧';


       function resizeCanvas() {
           canvas.width = canvas.offsetWidth;
           canvas.height = canvas.offsetHeight;
       }
       window.addEventListener('resize', resizeCanvas);
       resizeCanvas();


       function init() {
           player = {
               x: canvas.width / 2,
               y: canvas.height - 80,
               width: 80,
               height: 80,
               speed: 5
           };
           raindrops = [];
           score = 0;
           missedDrops = 0;
           isGameOver = false;
           waterGaugeFill.style.height = '0%';
           gaugeLabel.textContent = '0%';
           missedDropsSpan.textContent = '0';
           messageBox.style.display = 'none';
       }


       function updateGauge() {
           const gaugeHeight = (score / targetGauge) * 100;
           waterGaugeFill.style.height = `${Math.min(gaugeHeight, 100)}%`;
           gaugeLabel.textContent = `${Math.min(score, targetGauge)}%`;
       }


       function draw() {
           ctx.clearRect(0, 0, canvas.width, canvas.height);


           // Draw player (bucket)
           ctx.fillStyle = '#9e8979';
           ctx.fillRect(player.x - player.width / 2, player.y - player.height / 2, player.width, player.height);
           ctx.fillStyle = '#6b4d3f';
           ctx.font = '24px Jua';
           ctx.textAlign = 'center';
           ctx.fillText('양동이', player.x, player.y + 10);


           // Draw raindrops
           raindrops.forEach(drop => {
               ctx.fillStyle = '#3b82f6';
               ctx.beginPath();
               ctx.arc(drop.x, drop.y, drop.radius, 0, Math.PI * 2);
               ctx.fill();
           });
       }


       function update(timestamp) {
           if (isGameOver) return;
          
           const deltaTime = timestamp - lastUpdateTime;
           lastUpdateTime = timestamp;


           // Update raindrops
           if (Math.random() < 0.02 * (1 + score / 200)) { // Spawn more raindrops as score increases
               const x = Math.random() * (canvas.width - 40) + 20;
               const speed = 2 + Math.random() * 2 + score / 50;
               raindrops.push({ x: x, y: 0, radius: 10, speed: speed });
           }


           raindrops.forEach((drop, index) => {
               drop.y += drop.speed;


               // Check for collision with player
               if (drop.y + drop.radius > player.y - player.height / 2 &&
                   drop.y - drop.radius < player.y + player.height / 2 &&
                   drop.x > player.x - player.width / 2 &&
                   drop.x < player.x + player.width / 2) {
                  
                   score += 1;
                   raindrops.splice(index, 1);
                   updateGauge();
               }


               // Check if missed
               if (drop.y > canvas.height) {
                   missedDrops += 1;
                   missedDropsSpan.textContent = missedDrops;
                   raindrops.splice(index, 1);
               }
           });


           // Game over conditions
           if (score >= targetGauge) {
               isGameOver = true;
               showMessage("가뭄 극복 성공! 당신 덕분에 강릉에 비가 내렸어요!");
           }
           if (missedDrops >= 20) {
               isGameOver = true;
               showMessage("아쉽네요... 놓친 물방울이 너무 많아 가뭄을 이겨내지 못했어요.");
           }


           draw();
           requestAnimationFrame(update);
       }


       function showMessage(text) {
           messageText.textContent = text;
           messageBox.style.display = 'flex';
       }


       // Input handling
       let keys = {};
       document.addEventListener('keydown', (e) => {
           keys[e.key] = true;
       });
       document.addEventListener('keyup', (e) => {
           keys[e.key] = false;
       });


       canvas.addEventListener('mousemove', (e) => {
           const rect = canvas.getBoundingClientRect();
           const mouseX = e.clientX - rect.left;
           player.x = mouseX;
       });
      
       canvas.addEventListener('touchmove', (e) => {
           const rect = canvas.getBoundingClientRect();
           const touchX = e.touches[0].clientX - rect.left;
           player.x = touchX;
       });


       function movePlayer() {
           if (keys['ArrowLeft'] || keys['a']) {
               player.x -= player.speed;
           }
           if (keys['ArrowRight'] || keys['d']) {
               player.x += player.speed;
           }
           // Clamp player position
           if (player.x < player.width / 2) {
               player.x = player.width / 2;
           }
           if (player.x > canvas.width - player.width / 2) {
               player.x = canvas.width - player.width / 2;
           }
       }
      
       function gameLoop(timestamp) {
           if (!isGameOver) {
               movePlayer();
               update(timestamp);
               requestAnimationFrame(gameLoop);
           }
       }


       restartButton.addEventListener('click', () => {
           init();
           requestAnimationFrame(gameLoop);
       });


       window.onload = function() {
           init();
           requestAnimationFrame(gameLoop);
       };
   </script>
</body>
</html>



