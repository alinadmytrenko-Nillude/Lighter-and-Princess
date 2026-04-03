<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Heart Particles</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { background: #000; overflow: hidden; }
  canvas { display: block; }
</style>
</head>
<body>
<canvas id="c"></canvas>
<script>
const canvas = document.getElementById('c');
const ctx    = canvas.getContext('2d');

let W = canvas.width  = window.innerWidth;
let H = canvas.height = window.innerHeight;

window.addEventListener('resize', () => {
  W = canvas.width  = window.innerWidth;
  H = canvas.height = window.innerHeight;
  init();
});

/* ── Heart parametric curve ─────────────────────────────── */
function heartX(t) { const s = Math.sin(t); return 16 * s * s * s; }
function heartY(t) {
  return 13*Math.cos(t) - 5*Math.cos(2*t) - 2*Math.cos(3*t) - Math.cos(4*t);
}

function rand(a, b) { return a + Math.random() * (b - a); }

/* ── Particles ───────────────────────────────────────────── */
/* KEY: every particle anchors ON the heart curve.
   Each has a random scatter direction and max distance.
   The global "breath" drives how far they move from their anchor.
   → contracted: all on curve  → bright ring
   → expanded:   scattered in/out → filled body + halo             */

const NUM = 15000;
let particles = [];

function init() {
  particles = [];

  /* Heart — slightly smaller, centered higher to avoid bottom gap */
  const scale = Math.min(W, H) * 0.025;
  const cx = W / 2;
  const cy = H / 2 - scale * 0.5;

  /* Pre-build heart Path2D for rejection sampling (interior fill) */
  const heartPath = new Path2D();
  for (let k = 0; k <= 300; k++) {
    const ang = (k / 300) * Math.PI * 2;
    const px  = cx + heartX(ang) * scale;
    const py  = cy - heartY(ang) * scale;
    k === 0 ? heartPath.moveTo(px, py) : heartPath.lineTo(px, py);
  }
  heartPath.closePath();

  /* Bounding box of the heart curve (in canvas px).
     heartX range: -16 … +16; heartY range: -17 … +12
     Canvas y = cy - heartY*scale, so bottom tip is at cy + 17*scale. */
  const bboxX0 = cx - 16 * scale;
  const bboxX1 = cx + 16 * scale;
  const bboxY0 = cy - 13 * scale;   /* top of heart  */
  const bboxY1 = cy + 18 * scale;   /* bottom tip + small margin */

  /* Helper: random interior point via rejection sampling.
     30% of calls bias toward the lower third of the heart to fill
     the narrow bottom triangle that uniform sampling misses.       */
  function randomInsideHeart(forceLower) {
    let rx, ry, attempts = 0;
    const lowerStart = bboxY0 + (bboxY1 - bboxY0) * 0.58;
    do {
      if (forceLower || Math.random() < 0.12) {
        rx = rand(bboxX0, bboxX1);
        ry = rand(lowerStart, bboxY1);
      } else {
        rx = rand(bboxX0, bboxX1);
        ry = rand(bboxY0, bboxY1);
      }
      attempts++;
    } while (!ctx.isPointInPath(heartPath, rx, ry) && attempts < 120);
    return { x: rx, y: ry };
  }

  for (let i = 0; i < NUM; i++) {
    const t = rand(0, Math.PI * 2);

    /* ── Particle group determines anchor and behavior ─ */
    const grp = Math.random();
    let ax, ay, maxDist, r, g, b, alpha;

    if (grp < 0.38) {
      /* GROUP A — INTERIOR FILL
         Anchor is a truly random point INSIDE the heart via rejection sampling.
         No spine / axis artifacts — uniform area distribution.            */
      const ip = randomInsideHeart();
      ax = ip.x;
      ay = ip.y;
      maxDist = rand(4, 28);
      r = Math.round(rand(185, 245));
      g = Math.round(rand(0, 35));
      b = Math.round(rand(60, 130));
      alpha = rand(0.60, 0.92);

    } else if (grp < 0.68) {
      /* GROUP B — OUTLINE RING
         Anchor ON the curve, small scatter → creates the bright ring. */
      ax = cx + heartX(t) * scale;
      ay = cy - heartY(t) * scale;
      maxDist = rand(2, 35);
      r = Math.round(rand(215, 255));
      g = Math.round(rand(0, 45));
      b = Math.round(rand(85, 160));
      alpha = rand(0.70, 0.98);

    } else if (grp < 0.90) {
      /* GROUP C — OUTER HALO
         Anchor ON the curve, large scatter outward → breathing cloud halo. */
      ax = cx + heartX(t) * scale;
      ay = cy - heartY(t) * scale;
      maxDist = rand(45, 100);
      r = Math.round(rand(200, 255));
      g = Math.round(rand(0, 40));
      b = Math.round(rand(75, 155));
      alpha = rand(0.20, 0.55);

    } else {
      /* GROUP D — WHITE SPARKLES
         Bright near-white particles scattered on ring & inside → shimmer/glow. */
      const useInside = Math.random() < 0.55;
      if (useInside) {
        const ip = randomInsideHeart();
        ax = ip.x; ay = ip.y;
      } else {
        ax = cx + heartX(t) * scale;
        ay = cy - heartY(t) * scale;
      }
      maxDist = rand(3, 22);
      r = 255;
      g = Math.round(rand(190, 255));
      b = Math.round(rand(200, 255));
      alpha = rand(0.55, 1.0);
    }

    /* scatter direction: random for interior/ring/sparkle, outward for halo */
    let sDirX, sDirY;
    if (grp >= 0.68 && grp < 0.90) {
      /* halo: bias direction AWAY from heart center */
      const toCenter = Math.atan2(cy - ay, cx - ax);
      const outAng = toCenter + Math.PI + rand(-1.1, 1.1);
      sDirX = Math.cos(outAng);
      sDirY = Math.sin(outAng);
    } else {
      const sAng = rand(0, Math.PI * 2);
      sDirX = Math.cos(sAng);
      sDirY = Math.sin(sAng);
    }

    const phaseOff    = rand(0, 0.18);
    const jitPhase    = rand(0, Math.PI * 2);
    const jitSpeed    = rand(1.8, 4.5);
    const jitAmp      = rand(0.8, 3.2);
    const isSparkle   = grp >= 0.90;
    /* sparkle flicker: fast random frequency so each one is independent */
    const sparkleFreq = isSparkle ? rand(3.5, 9.0) : 0;

    particles.push({
      ax, ay, sDirX, sDirY, maxDist,
      phaseOff, jitPhase, jitSpeed, jitAmp,
      size: isSparkle ? rand(1.0, 2.4) : rand(1.2, 3.2),
      alpha, r, g, b,
      isSparkle, sparkleFreq,
    });
  }

}

/* ── Render loop ─────────────────────────────────────────── */
const t0 = performance.now();

function draw(now) {
  const t = (now - t0) / 1000;

  /* Solid black — no trail, clean each frame */
  ctx.fillStyle = '#000';
  ctx.fillRect(0, 0, W, H);

  /* Subtle central glow — keep cy in sync with init() */
  const cx  = W / 2;
  const cy  = H / 2 - Math.min(W,H) * 0.025 * 0.5;
  const gR  = Math.min(W, H) * 0.25;
  const grd = ctx.createRadialGradient(cx, cy, 0, cx, cy, gR);
  grd.addColorStop(0,    'rgba(160,0,50,0.28)');
  grd.addColorStop(0.5,  'rgba(90,0,25,0.12)');
  grd.addColorStop(1,    'rgba(0,0,0,0)');
  ctx.fillStyle = grd;
  ctx.fillRect(cx - gR, cy - gR, gR * 2, gR * 2);

  /* ── Breathing cycle ───────────────────────────────────
     Start at MAXIMUM expansion, breathe slowly (~3.7s cycle).
     Range 0.45 → 1.0: heart never fully collapses to just an outline */
  const breathFreq = 0.27;
  const raw = 0.5 + 0.5 * Math.sin(t * breathFreq * Math.PI * 2 + Math.PI / 2);
  const smooth = raw * raw * (3 - 2 * raw);
  const globalBreathe = 0.45 + 0.55 * smooth;

  for (let i = 0; i < particles.length; i++) {
    const p = particles[i];

    /* tiny individual phase lag — nearly synchronous */
    const pRaw = 0.5 + 0.5 * Math.sin((t + p.phaseOff * 0.35) * breathFreq * Math.PI * 2 + Math.PI / 2);
    const pSmooth = pRaw * pRaw * (3 - 2 * pRaw);
    const pBreathe = 0.45 + 0.55 * pSmooth;

    const dist = p.maxDist * pBreathe;

    /* chaotic micro-jitter — makes the cloud feel organic */
    const jx = Math.sin(t * p.jitSpeed + p.jitPhase)            * p.jitAmp;
    const jy = Math.cos(t * p.jitSpeed * 0.8 + p.jitPhase + 1.3) * p.jitAmp;

    const px = p.ax + p.sDirX * dist + jx;
    const py = p.ay + p.sDirY * dist + jy;

    let drawR, drawG, drawB, drawA, s;

    if (p.isSparkle) {
      /* Sparkles flicker independently — sharp on/off twinkle */
      const flicker = Math.pow(Math.abs(Math.sin(t * p.sparkleFreq + p.jitPhase)), 4);
      drawA = p.alpha * flicker;
      if (drawA < 0.05) continue;          /* skip invisible sparkles */
      drawR = p.r; drawG = p.g; drawB = p.b;
      s = p.size * (0.7 + 0.6 * flicker); /* size also pulses */
    } else {
      /* Normal particles: brightness pulses with breath */
      const brightMult = 0.75 + 0.25 * (1 - pBreathe);
      drawR = Math.round(p.r * brightMult);
      drawG = Math.round(p.g * brightMult);
      drawB = Math.round(p.b * brightMult);
      drawA = p.alpha * brightMult;
      s = p.size;
    }

    ctx.fillStyle = `rgba(${drawR},${drawG},${drawB},${drawA.toFixed(2)})`;
    ctx.fillRect(px - s * 0.5, py - s * 0.5, s, s + 1);
  }

  requestAnimationFrame(draw);
}

init();
requestAnimationFrame(draw);
</script>
</body>
</html>
