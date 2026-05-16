[te_iubesc_heart_horizontal.html](https://github.com/user-attachments/files/27852329/te_iubesc_heart_horizontal.html)
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: #0a0005;
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
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
const W = 580, H = 540;
canvas.width = W;
canvas.height = H;

const text = "te iubesc ";
const TOTAL_CHARS = 120;

function heartX(t) { return 16 * Math.pow(Math.sin(t), 3); }
function heartY(t) { return -(13*Math.cos(t) - 5*Math.cos(2*t) - 2*Math.cos(3*t) - Math.cos(4*t)); }

const steps = 1200;
const points = [];
for (let i = 0; i <= steps; i++) {
  const t = (i / steps) * Math.PI * 2;
  points.push({ x: heartX(t), y: heartY(t) });
}

const totalLen = points.reduce((acc, p, i) => {
  if (i === 0) return 0;
  return acc + Math.hypot(p.x - points[i-1].x, p.y - points[i-1].y);
}, 0);

const evenPoints = [];
const spacing = totalLen / TOTAL_CHARS;
let accumulated = 0, pIdx = 0;
for (let i = 0; i < TOTAL_CHARS; i++) {
  const target = i * spacing;
  while (pIdx < points.length - 1 && accumulated < target) {
    accumulated += Math.hypot(points[pIdx+1].x - points[pIdx].x, points[pIdx+1].y - points[pIdx].y);
    pIdx++;
  }
  evenPoints.push({ ...points[pIdx] });
}

const cx = W / 2, cy = H / 2 - 10;
const scale = 17;
let frame = 0;

function draw() {
  ctx.clearRect(0, 0, W, H);
  ctx.fillStyle = '#0a0005';
  ctx.fillRect(0, 0, W, H);

  const pulse = 0.93 + 0.07 * Math.sin(frame * 0.04);
  const glowPulse = 0.5 + 0.5 * Math.abs(Math.sin(frame * 0.04));
  const sc = scale * pulse;

  // faint heart fill glow
  ctx.save();
  ctx.translate(cx, cy);
  ctx.scale(sc, sc);
  const hp = new Path2D();
  for (let i = 0; i <= 300; i++) {
    const t = (i / 300) * Math.PI * 2;
    i === 0 ? hp.moveTo(heartX(t), heartY(t)) : hp.lineTo(heartX(t), heartY(t));
  }
  hp.closePath();
  ctx.shadowColor = `rgba(220,40,80,${glowPulse * 0.35})`;
  ctx.shadowBlur = 40;
  ctx.fillStyle = 'rgba(180,20,60,0.06)';
  ctx.fill(hp);
  ctx.restore();

  const offset = (frame * 0.5) % TOTAL_CHARS;

  for (let i = 0; i < TOTAL_CHARS; i++) {
    const idx = (i + Math.floor(offset)) % TOTAL_CHARS;
    const frac = offset - Math.floor(offset);
    const idxNext = (idx + 1) % TOTAL_CHARS;

    const px = evenPoints[idx].x + (evenPoints[idxNext].x - evenPoints[idx].x) * frac;
    const py = evenPoints[idx].y + (evenPoints[idxNext].y - evenPoints[idx].y) * frac;

    const screenX = cx + px * sc;
    const screenY = cy + py * sc;

    const ch = text[i % text.length];
    const posInLoop = i / TOTAL_CHARS;
    const hue = 340 + 25 * Math.sin(posInLoop * Math.PI * 4 + frame * 0.025);
    const sat = 85 + 15 * Math.sin(posInLoop * Math.PI * 2);
    const light = 62 + 18 * Math.sin(posInLoop * Math.PI * 3 + frame * 0.03);
    const alpha = 0.85 + 0.15 * Math.sin(posInLoop * Math.PI * 5 + frame * 0.04);

    ctx.save();
    ctx.translate(screenX, screenY);
    // NO rotation — litere orizontale

    const fontSize = 13;
    ctx.font = `700 ${fontSize}px Georgia, serif`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';

    ctx.shadowColor = `hsla(${hue}, 90%, 70%, 0.7)`;
    ctx.shadowBlur = 8;
    ctx.fillStyle = `hsla(${hue}, ${sat}%, ${light}%, ${alpha})`;
    ctx.fillText(ch, 0, 0);

    ctx.shadowBlur = 0;
    ctx.fillStyle = `hsla(${hue}, ${sat}%, ${light + 10}%, ${alpha * 0.5})`;
    ctx.fillText(ch, 0, 0);

    ctx.restore();
  }

  // center glow
  const gradient = ctx.createRadialGradient(cx, cy - 20, 20, cx, cy - 20, 240);
  gradient.addColorStop(0, `rgba(220,20,80,${0.05 * glowPulse})`);
  gradient.addColorStop(1, 'rgba(0,0,0,0)');
  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, W, H);

  frame++;
  requestAnimationFrame(draw);
}

draw();
</script>
</body>
</html>
