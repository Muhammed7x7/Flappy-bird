# Flappy-bird
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Flappy Bird</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background-color: #70c5ce;
            font-family: Arial, sans-serif;
            touch-action: manipulation;
            -webkit-touch-callout: none;
            -webkit-user-select: none;
            -khtml-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
            user-select: none;
        }
        
        #gameCanvas {
            display: block;
            margin: 0 auto;
            background-color: #70c5ce;
        }
        
        #startScreen, #gameOverScreen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background-color: rgba(0, 0, 0, 0.5);
            color: white;
            font-size: 24px;
        }
        
        #gameOverScreen {
            display: none;
        }
        
        button {
            margin-top: 20px;
            padding: 10px 20px;
            font-size: 18px;
            background-color: #ff5722;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        
        #scoreDisplay {
            position: absolute;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 30px;
            color: white;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
            z-index: 10;
        }
    </style>
</head>
<body>
    <div id="scoreDisplay">0</div>
    <canvas id="gameCanvas"></canvas>
    
    <div id="startScreen">
        <h1>Flappy Bird</h1>
        <p>Ekrana dokunun veya tıklayın</p>
        <button id="startButton">Oyunu Başlat</button>
    </div>
    
    <div id="gameOverScreen">
        <h1>Oyun Bitti!</h1>
        <p id="finalScore">Skor: 0</p>
        <button id="restartButton">Tekrar Oyna</button>
    </div>

    <script>
        // Oyun değişkenleri
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startScreen = document.getElementById('startScreen');
        const gameOverScreen = document.getElementById('gameOverScreen');
        const startButton = document.getElementById('startButton');
        const restartButton = document.getElementById('restartButton');
        const scoreDisplay = document.getElementById('scoreDisplay');
        const finalScoreDisplay = document.getElementById('finalScore');
        
        // Canvas boyutlarını ayarla
        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        
        resizeCanvas();
        window.addEventListener('resize', resizeCanvas);
        
        // Oyun nesneleri
        let bird = {
            x: 100,
            y: canvas.height / 2,
            width: 40,
            height: 30,
            velocity: 0,
            gravity: 0.5,
            jump: -10,
            color: '#ffeb3b'
        };
        
        let pipes = [];
        let score = 0;
        let gameStarted = false;
        let gameOver = false;
        let animationId;
        let pipeGap = 200;
        let pipeWidth = 80;
        let pipeFrequency = 1500; // milisaniye
        let lastPipeTime = 0;
        
        // Boru oluştur
        function createPipe() {
            const minHeight = 50;
            const maxHeight = canvas.height - pipeGap - minHeight;
            const height = Math.floor(Math.random() * (maxHeight - minHeight + 1) + minHeight);
            
            pipes.push({
                x: canvas.width,
                y: 0,
                width: pipeWidth,
                height: height,
                passed: false
            });
            
            pipes.push({
                x: canvas.width,
                y: height + pipeGap,
                width: pipeWidth,
                height: canvas.height - height - pipeGap,
                passed: false
            });
        }
        
        // Kuşu çiz
        function drawBird() {
            ctx.fillStyle = bird.color;
            ctx.fillRect(bird.x, bird.y, bird.width, bird.height);
            
            // Göz
            ctx.fillStyle = 'black';
            ctx.beginPath();
            ctx.arc(bird.x + 30, bird.y + 10, 5, 0, Math.PI * 2);
            ctx.fill();
            
            // Gaga
            ctx.fillStyle = '#ff9800';
            ctx.beginPath();
            ctx.moveTo(bird.x + 40, bird.y + 15);
            ctx.lineTo(bird.x + 50, bird.y + 15);
            ctx.lineTo(bird.x + 40, bird.y + 20);
            ctx.fill();
        }
        
        // Boruları çiz
        function drawPipes() {
            ctx.fillStyle = '#4caf50';
            pipes.forEach(pipe => {
                ctx.fillRect(pipe.x, pipe.y, pipe.width, pipe.height);
            });
        }
        
        // Oyunu güncelle
        function update() {
            // Kuşu güncelle
            bird.velocity += bird.gravity;
            bird.y += bird.velocity;
            
            // Yer çarpması kontrolü
            if (bird.y + bird.height > canvas.height) {
                bird.y = canvas.height - bird.height;
                bird.velocity = 0;
                endGame();
            }
            
            // Tavan çarpması kontrolü
            if (bird.y < 0) {
                bird.y = 0;
                bird.velocity = 0;
            }
            
            // Boruları güncelle
            const currentTime = Date.now();
            if (currentTime - lastPipeTime > pipeFrequency && gameStarted && !gameOver) {
                createPipe();
                lastPipeTime = currentTime;
            }
            
            pipes.forEach((pipe, index) => {
                pipe.x -= 3;
                
                // Çarpışma kontrolü
                if (bird.x < pipe.x + pipe.width &&
                    bird.x + bird.width > pipe.x &&
                    bird.y < pipe.y + pipe.height &&
                    bird.y + bird.height > pipe.y) {
                    endGame();
                }
                
                // Skor artırma
                if (!pipe.passed && pipe.x + pipe.width < bird.x && index % 2 === 0) {
                    pipe.passed = true;
                    score++;
                    scoreDisplay.textContent = score;
                }
            });
            
            // Ekran dışına çıkan boruları kaldır
            pipes = pipes.filter(pipe => pipe.x + pipe.width > 0);
        }
        
        // Oyunu çiz
        function draw() {
            // Arka planı temizle
            ctx.fillStyle = '#70c5ce';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Boruları çiz
            drawPipes();
            
            // Kuşu çiz
            drawBird();
        }
        
        // Oyun döngüsü
        function gameLoop() {
            if (!gameOver) {
                update();
                draw();
                animationId = requestAnimationFrame(gameLoop);
            }
        }
        
        // Oyunu başlat
        function startGame() {
            startScreen.style.display = 'none';
            gameOverScreen.style.display = 'none';
            scoreDisplay.style.display = 'block';
            
            // Oyun durumunu sıfırla
            bird.y = canvas.height / 2;
            bird.velocity = 0;
            pipes = [];
            score = 0;
            scoreDisplay.textContent = '0';
            gameStarted = true;
            gameOver = false;
            
            // Oyun döngüsünü başlat
            gameLoop();
        }
        
        // Oyunu bitir
        function endGame() {
            gameOver = true;
            gameStarted = false;
            cancelAnimationFrame(animationId);
            
            gameOverScreen.style.display = 'flex';
            finalScoreDisplay.textContent = `Skor: ${score}`;
            scoreDisplay.style.display = 'none';
        }
        
        // Kontroller
        function handleJump() {
            if (!gameStarted) {
                startGame();
            } else if (!gameOver) {
                bird.velocity = bird.jump;
            }
        }
        
        // Mobil dokunma olayları
        canvas.addEventListener('touchstart', (e) => {
            e.preventDefault();
            handleJump();
        });
        
        // Mause tıklama olayı
        canvas.addEventListener('click', handleJump);
        
        // Başlatma butonu
        startButton.addEventListener('click', startGame);
        
        // Yeniden başlatma butonu
        restartButton.addEventListener('click', startGame);
        
        // Başlangıç ekranını göster
        startScreen.style.display = 'flex';
    </script>
</body>
</html>
