<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Matrix Particle Heart Animation</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body, html {
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #050508;
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'Courier New', Courier, monospace;
        }
        canvas {
            display: block;
            position: absolute;
            top: 0;
            left: 0;
        }
    </style>
</head>
<body>

<canvas id="canvas"></canvas>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

let width, height;

function initCanvas() {
    width = canvas.width = window.innerWidth;
    height = canvas.height = window.innerHeight;
}
initCanvas();
window.addEventListener('resize', initCanvas);

// --- MATRIX RAIN EFFECT ---
const characters = 'I LOVE YOU SAYANG 1234567890♥';
const fontSize = 14;
let columns = Math.floor(width / fontSize);
let drops = Array(columns).fill(1);

function drawMatrix() {
    ctx.fillStyle = 'rgba(5, 5, 8, 0.15)';
    ctx.fillRect(0, 0, width, height);

    ctx.fillStyle = 'rgba(255, 182, 193, 0.4)';
    ctx.font = fontSize + 'px monospace';

    for (let i = 0; i < drops.length; i++) {
        const text = characters.charAt(Math.floor(Math.random() * characters.length));
        ctx.fillText(text, i * fontSize, drops[i] * fontSize);

        if (drops[i] * fontSize > height && Math.random() > 0.985) {
            drops[i] = 0;
        }
        drops[i] += 0.6;
    }
}

// --- PARTICLE SYSTEM ---
class Particle {
    constructor(x, y) {
        this.x = Math.random() * width;
        this.y = Math.random() * height;
        this.destX = x;
        this.destY = y;
        this.vx = (Math.random() - 0.5) * 4;
        this.vy = (Math.random() - 0.5) * 4;
        this.accX = 0;
        this.accY = 0;
        this.friction = 0.92;
        this.active = true;
        this.color = '#ffffff';
        this.size = Math.random() * 2 + 1;
    }

    update() {
        if (!this.active) return;
        this.accX = (this.destX - this.x) / 300;
        this.accY = (this.destY - this.y) / 300;
        this.vx += this.accX;
        this.vy += this.accY;
        this.vx *= this.friction;
        this.vy *= this.friction;
        this.x += this.vx;
        this.y += this.vy;
    }

    draw() {
        if (!this.active) return;
        ctx.fillStyle = this.color;
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
        ctx.fill();
    }
}

// --- RIPPLE EFFECT FOR HEART ---
class RippleParticle {
    constructor(x, y) {
        this.x = x;
        this.y = y;
        this.size = Math.random() * 2 + 1;
        this.angle = Math.random() * Math.PI * 2;
        this.speed = Math.random() * 1 + 0.2;
        this.opacity = 1;
        this.active = true;
    }

    update() {
        this.x += Math.cos(this.angle) * this.speed;
        this.y += Math.sin(this.angle) * this.speed;
        this.opacity -= 0.008;
        if (this.opacity <= 0) this.active = false;
    }

    draw() {
        ctx.fillStyle = `rgba(255, 105, 180, ${this.opacity})`;
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
        ctx.fill();
    }
}

let particles = [];
let rippleParticles = [];
let state = "COUNTDOWN";
let textOpacity = 0;

function getTextPoints(text) {
    const offCanvas = document.createElement('canvas');
    const offCtx = offCanvas.getContext('2d');
    offCanvas.width = width;
    offCanvas.height = height;

    let fontSizeCalc = Math.min(width / 3, 200);
    offCtx.font = `bold ${fontSizeCalc}px sans-serif`;
    offCtx.textAlign = 'center';
    offCtx.textBaseline = 'middle';
    offCtx.fillStyle = 'white';
    offCtx.fillText(text, width / 2, height / 2);

    const imgData = offCtx.getImageData(0, 0, width, height);
    const points = [];
    const step = 6;

    for (let y = 0; y < height; y += step) {
        for (let x = 0; x < width; x += step) {
            const alpha = imgData.data[(y * width + x) * 4 + 3];
            if (alpha > 128) {
                points.push({ x, y });
            }
        }
    }
    return points;
}

function getHeartOutlinePoints(count = 220) {
    const points = [];
    const scale = Math.min(width, height) / 35;
    for (let i = 0; i < count; i++) {
        const t = (i / count) * Math.PI * 2;
        const x = 16 * Math.pow(Math.sin(t), 3);
        const y = -(13 * Math.cos(t) - 5 * Math.cos(2*t) - 2 * Math.cos(3*t) - Math.cos(4*t));
        
        points.push({
            pt: {
                x: width / 2 + x * scale,
                y: height / 2 + y * scale
            }
        });
    }
    return points;
}

function updateParticleTargets(text) {
    const points = getTextPoints(text);
    
    while (particles.length < points.length) {
        particles.push(new Particle(width / 2, height / 2));
    }

    for (let i = 0; i < particles.length; i++) {
        if (i < points.length) {
            particles[i].destX = points[i].x;
            particles[i].destY = points[i].y;
            particles[i].color = '#ffffff';
            particles[i].active = true;
        } else {
            particles[i].active = false;
        }
    }
}

// --- PERBAIKAN: Partikel teks dikosongkan sebelum bentuk hati dibentuk ---
function clearTextParticlesToHeart() {
    const heartPoints = getHeartOutlinePoints(220);
    
    // Sesuaikan jumlah partikel pas dengan jumlah titik bingkai hati
    particles = []; 
    for (let i = 0; i < heartPoints.length; i++) {
        let p = new Particle(width / 2, height / 2);
        p.destX = heartPoints[i].pt.x;
        p.destY = heartPoints[i].pt.y;
        p.color = '#ff69b4';
        p.active = true;
        particles.push(p);
    }
}

const delay = ms => new Promise(res => setTimeout(res, ms));

async function startParticleSequence() {
    // 1. Countdown (3, 2, 1)
    const countdown = ["3", "2", "1"];
    for (let text of countdown) {
        updateParticleTargets(text);
        await delay(2200);
    }

    // 2. Teks (You -> Are -> My -> Love)
    const finalWords = ["You", "Are", "My", "Love"];
    for (let word of finalWords) {
        updateParticleTargets(word);
        await delay(2000);
    }

    // 3. Bersihkan partikel kata "Love" & ganti menjadi bingkai Hati Pink
    clearTextParticlesToHeart();

    await delay(2000);
    state = "FINAL_HEART";
}

let rippleTimer = 0;
function createHeartRipple() {
    if (rippleTimer % 5 === 0) {
        const heartPoints = getHeartOutlinePoints(120);
        heartPoints.forEach(p => {
            rippleParticles.push(new RippleParticle(p.pt.x, p.pt.y));
        });
    }
    rippleTimer++;
}

function drawFinalText() {
    if (state === "FINAL_HEART") {
        if (textOpacity < 1) textOpacity += 0.008;
        
        let fontSizeCalc = Math.min(width / 18, 28);
        ctx.font = `bold ${fontSizeCalc}px sans-serif`;
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillStyle = `rgba(255, 255, 255, ${textOpacity})`;
        // Menampilkan hanya teks rapi di tengah hati
        ctx.fillText("I LOVE YOU SAYANG 💕", width / 2, height / 2);
    }
}

// --- MAIN ANIMATION LOOP ---
function draw() {
    drawMatrix();

    if (state === "COUNTDOWN") {
        particles.forEach(p => {
            p.update();
            p.draw();
        });
    } else if (state === "FINAL_HEART") {
        createHeartRipple();
        
        for (let i = rippleParticles.length - 1; i >= 0; i--) {
            rippleParticles[i].update();
            rippleParticles[i].draw();
            if (!rippleParticles[i].active) {
                rippleParticles.splice(i, 1);
            }
        }

        particles.forEach(p => {
            p.update();
            p.draw();
        });

        drawFinalText();
    }

    requestAnimationFrame(draw);
}

// Jalankan animasi
startParticleSequence();
draw();
</script>
</body>
</html>
