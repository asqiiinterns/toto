<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Jeu de tir</title>

<style>
    body {
        margin: 0;
        background: #111;
        color: white;
        font-family: Arial;
        text-align: center;
        overflow: hidden;
    }

    h1 {
        margin: 10px;
    }

    #score {
        font-size: 22px;
        margin-bottom: 5px;
    }

    canvas {
        background: #222;
        border: 3px solid white;
        display: block;
        margin: auto;
    }

    #info {
        margin-top: 10px;
        font-size: 16px;
    }
</style>
</head>

<body>

<h1>🎯 Jeu de tir</h1>
<div id="score">Score : 0</div>

<canvas id="game" width="700" height="500"></canvas>

<div id="info">
    ← → pour bouger | ESPACE pour tirer
</div>

<script>

const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

let score = 0;
let gameOver = false;

const player = {
    x: 325,
    y: 450,
    width: 50,
    height: 30,
    speed: 7
};

let bullets = [];
let targets = [];

let keys = {};

document.addEventListener("keydown", function(e) {
    keys[e.key] = true;

    if (e.code === "Space") {
        shoot();
        e.preventDefault();
    }
});

document.addEventListener("keyup", function(e) {
    keys[e.key] = false;
});

function shoot() {

    bullets.push({
        x: player.x + player.width / 2 - 3,
        y: player.y,
        width: 6,
        height: 15,
        speed: 9
    });
}

function createTarget() {

    targets.push({
        x: Math.random() * (canvas.width - 40),
        y: -40,
        width: 40,
        height: 40,
        speed: 2 + Math.random() * 2
    });
}

setInterval(createTarget, 900);

function update() {

    if (gameOver) return;

    // Déplacement
    if (keys["ArrowLeft"]) {
        player.x -= player.speed;
    }

    if (keys["ArrowRight"]) {
        player.x += player.speed;
    }

    // Empêcher de sortir de l'écran
    if (player.x < 0) {
        player.x = 0;
    }

    if (player.x + player.width > canvas.width) {
        player.x = canvas.width - player.width;
    }

    // Déplacer les balles
    for (let i = bullets.length - 1; i >= 0; i--) {

        bullets[i].y -= bullets[i].speed;

        if (bullets[i].y < 0) {
            bullets.splice(i, 1);
        }
    }

    // Déplacer les cibles
    for (let i = targets.length - 1; i >= 0; i--) {

        targets[i].y += targets[i].speed;

        // Si une cible atteint le joueur
        if (collision(player, targets[i])) {
            gameOver = true;
        }

        if (targets[i].y > canvas.height) {
            targets.splice(i, 1);
        }
    }

    // Collision balle / cible
    for (let i = bullets.length - 1; i >= 0; i--) {

        for (let j = targets.length - 1; j >= 0; j--) {

            if (collision(bullets[i], targets[j])) {

                bullets.splice(i, 1);
                targets.splice(j, 1);

                score++;

                document.getElementById("score").textContent =
                    "Score : " + score;

                break;
            }
        }
    }
}

function collision(a, b) {

    return (
        a.x < b.x + b.width &&
        a.x + a.width > b.x &&
        a.y < b.y + b.height &&
        a.y + a.height > b.y
    );
}

function draw() {

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Joueur
    ctx.fillStyle = "lime";
    ctx.fillRect(
        player.x,
        player.y,
        player.width,
        player.height
    );

    // Canon
    ctx.fillStyle = "white";
    ctx.fillRect(
        player.x + 22,
        player.y - 15,
        6,
        15
    );

    // Balles
    ctx.fillStyle = "yellow";

    bullets.forEach(function(bullet) {
        ctx.fillRect(
            bullet.x,
            bullet.y,
            bullet.width,
            bullet.height
        );
    });

    // Cibles
    targets.forEach(function(target) {

        ctx.fillStyle = "red";

        ctx.beginPath();
        ctx.arc(
            target.x + 20,
            target.y + 20,
            20,
            0,
            Math.PI * 2
        );
        ctx.fill();

        // Centre de la cible
        ctx.fillStyle = "white";

        ctx.beginPath();
        ctx.arc(
            target.x + 20,
            target.y + 20,
            8,
            0,
            Math.PI * 2
        );
        ctx.fill();
    });

    // Game Over
    if (gameOver) {

        ctx.fillStyle = "rgba(0,0,0,0.75)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = "white";
        ctx.textAlign = "center";

        ctx.font = "50px Arial";
        ctx.fillText(
            "GAME OVER",
            canvas.width / 2,
            220
        );

        ctx.font = "25px Arial";
        ctx.fillText(
            "Score : " + score,
            canvas.width / 2,
            270
        );

        ctx.font = "18px Arial";
        ctx.fillText(
            "Recharge la page pour rejouer",
            canvas.width / 2,
            320
        );
    }
}

function gameLoop() {

    update();
    draw();

    requestAnimationFrame(gameLoop);
}

gameLoop();

</script>

</body>
</html>
