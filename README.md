[nastea_heart_mobile.html](https://github.com/user-attachments/files/27852702/nastea_heart_mobile.html)
<!DOCTYPE html>
<html lang="ro">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Nastea ❤️</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  html, body {
    background: #080010;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100vh;
    overflow: hidden;
  }
  canvas { display: block; }
</style>
</head>
<body>
<canvas id="c"></canvas>
<script>
const canvas = document.getElementById('c');
const ctx = canvas.getContext('2d');

// Responsive: se potriveste pe orice telefon
const size = Math.min(window.innerWidth, window.innerHeight) * 0.96;
canvas.width = size;
canvas.height = size;

const W = size, H = size;
const cx = W / 2;
const cy = H / 2 + size * 0.02;

const SCALE = size * 0.031;
const FONT_SIZE = Math.round(size * 0.042);
const NAME_FONT = Math.round(size * 0.13);
const TOTAL_CHARS = 85;
const text = "te iubesc ";

function heartX(t) { return 16 * Math.pow(Math.sin(t), 3); }
function heartY(t) { return -(13*Math.cos(t) - 5*Math.cos(2*t) - 2*Math.cos(3*t) - Math.cos(4*t)); }

// Build evenly spaced points along heart path
const steps = 3000;
const raw = [];
for (let i = 0; i < steps; i++) {
  const t = (i / steps) * Math.PI * 2;
  raw.push({ x: heartX(t), y: heartY(t) });
}

let totalLen = 0;
for (let i = 1; i < raw.length; i++)
  totalLen += Math.hypot(raw[i].x - raw[i-1].x, raw[i].y - raw[i-1].y);

const sp = totalLen / TOTAL_CHARS;
const pts = [];
let acc = 0, pi = 0;
for (let i = 0; i < TOTAL_CHARS; i++) {
  const target = i * sp;
  while (pi < raw.length - 1 && acc < target) {
    acc += Math.hypot(raw[pi+1].x - raw[pi].x, raw[pi+1].y - raw[pi].y);
    pi++;
  }
  pts.push({ ...raw[Math.min(pi, raw.length - 1)] });
}

let frame = 0;

function draw() {
  ctx.fillStyle = '#080010';
  ctx.fillRect(0, 0, W, H);

  const pulse = 0.97 + 0.03 * Math.sin(frame * 0.025);
  const glowPulse = 0.5 + 0.5 * Math.abs(Math.sin(frame * 0.025));
  const sc = SCALE * pulse;

  const offset = (frame * 0.14) % TOTAL_CHARS;
  const iOff = Math.floor(offset);
  const frac = offset - iOff;

  // --- Literele conturului ---
  ctx.font = `900 ${FONT_SIZE}px Georgia, serif`;
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';

  for (let i = 0; i < TOTAL_CHARS; i++) {
    const a = (i + iOff) % TOTAL_CHARS;
    const b = (a + 1) % TOTAL_CHARS;
    const px = pts[a].x + (pts[b].x - pts[a].x) * frac;
    const py = pts[a].y + (pts[b].y - pts[a].y) * frac;
    const sx = cx + px * sc;
    const sy = cy + py * sc;

    const ch = text[i % text.length];
    const t = i / TOTAL_CHARS;
    const hue = 330 + 28 * Math.sin(t * Math.PI * 3 + frame * 0.012);
    const light = 60 + 14 * Math.sin(t * Math.PI * 5 + frame * 0.018);

    ctx.save();
    ctx.translate(sx, sy);
    ctx.shadowColor = `hsl(${hue}, 100%, 70%)`;
    ctx.shadowBlur = 16;
    ctx.fillStyle = `hsl(${hue}, 95%, ${light}%)`;
    ctx.fillText(ch, 0, 0);
    ctx.shadowBlur = 5;
    ctx.fillStyle = `hsla(${hue}, 80%, 92%, 0.5)`;
    ctx.fillText(ch, 0, 0);
    ctx.restore();
  }

  // --- Numele "Nastea" în centru ---
  const namePulse = 0.93 + 0.07 * Math.sin(frame * 0.04);
  const nameGlow = 0.65 + 0.35 * Math.abs(Math.sin(frame * 0.04));
  const nfs = Math.round(NAME_FONT * namePulse);

  ctx.save();
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.font = `700 ${nfs}px Georgia, serif`;

  // Glow exterior
  ctx.shadowColor = `rgba(255, 100, 160, ${nameGlow})`;
  ctx.shadowBlur = 35;
  ctx.fillStyle = `hsl(340, 100%, 76%)`;
  ctx.fillText('Nastea', cx, cy - size * 0.01);

  // Strat luminos interior
  ctx.shadowBlur = 12;
  ctx.fillStyle = `hsla(345, 90%, 90%, 0.65)`;
  ctx.fillText('Nastea', cx, cy - size * 0.01);

  ctx.restore();

  // Glow ambient
  const g = ctx.createRadialGradient(cx, cy - size*0.04, size*0.04, cx, cy, size*0.46);
  g.addColorStop(0, `rgba(200, 20, 80, ${0.08 * glowPulse})`);
  g.addColorStop(1, 'rgba(0,0,0,0)');
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, W, H);

  frame++;
  requestAnimationFrame(draw);
}

draw();

// Resize pe rotire ecran
window.addEventListener('resize', () => {
  const s = Math.min(window.innerWidth, window.innerHeight) * 0.96;
  canvas.width = s;
  canvas.height = s;
});
</script>
</body>
</html>
