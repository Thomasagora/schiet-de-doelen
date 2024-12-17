<!DOCTYPE html>
<html lang="nl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Schiet de Doelen</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background-color: #87CEEB;
            font-family: Arial, sans-serif;
        }
        h1 {
            position: absolute;
            top: 10px;
            left: 50%;
            transform: translateX(-50%);
            color: #fff;
            z-index: 10;
        }
        canvas {
            display: block;
            background-color: #333;
        }
    </style>
</head>
<body>
    <h1>Schiet de Doelen!</h1>
    <canvas id="gameCanvas"></canvas>
    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        const targets = [];
        const bullets = [];
        let score = 0;
        let gameOver = false;

        class Target {
            constructor(x, y, radius, speed) {
                this.x = x;
                this.y = y;
                this.radius = radius;
                this.speed = speed;
                this.color = "red";
            }

            move() {
                this.y += this.speed;
                if (this.y - this.radius > canvas.height) {
                    gameOver = true;
                }
            }

            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.closePath();
            }
        }

        class Bullet {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.speed = 5;
                this.width = 5;
                this.height = 10;
                this.color = "yellow";
            }

            move() {
                this.y -= this.speed;
            }

            draw() {
                ctx.fillStyle = this.color;
                ctx.fillRect(this.x, this.y, this.width, this.height);
            }
        }

        function spawnTarget() {
            const radius = 20 + Math.random() * 30;
            const x = Math.random() * (canvas.width - radius * 2) + radius;
            const speed = 1 + Math.random() * 3;
            targets.push(new Target(x, 0 - radius, radius, speed));
        }

        function updateGame() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Teken en beweeg doelen
            for (let i = targets.length - 1; i >= 0; i--) {
                const target = targets[i];
                target.move();
                target.draw();

                // Controleer of het doel het scherm verlaat
                if (target.y - target.radius > canvas.height) {
                    targets.splice(i, 1);
                    gameOver = true;
                }
            }

            // Teken en beweeg kogels
            for (let i = bullets.length - 1; i >= 0; i--) {
                const bullet = bullets[i];
                bullet.move();
                bullet.draw();

                // Controleer botsingen met doelen
                for (let j = targets.length - 1; j >= 0; j--) {
                    const target = targets[j];
                    const dist = Math.hypot(bullet.x - target.x, bullet.y - target.y);
                    if (dist < target.radius) {
                        bullets.splice(i, 1);
                        targets.splice(j, 1);
                        score++;
                        break;
                    }
                }

                // Verwijder kogels die uit beeld gaan
                if (bullet.y + bullet.height < 0) {
                    bullets.splice(i, 1);
                }
            }

            // Toon score
            ctx.fillStyle = "white";
            ctx.font = "20px Arial";
            ctx.fillText("Score: " + score, 10, 30);

            // Controleer game over
            if (gameOver) {
                ctx.fillStyle = "red";
                ctx.font = "40px Arial";
                ctx.fillText("Game Over! Score: " + score, canvas.width / 2 - 150, canvas.height / 2);
                return;
            }

            requestAnimationFrame(updateGame);
        }

        window.addEventListener("click", (e) => {
            if (!gameOver) {
                bullets.push(new Bullet(e.clientX, canvas.height - 20));
            }
        });

        setInterval(spawnTarget, 1000);
        updateGame();
    </script>
</body>
</html>
