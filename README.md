
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>3D Cricket Game</title>
<style>
  * { box-sizing: border-box; }
  body {
    margin: 0; overflow: hidden;
    background: #102d20; color: white;
    font-family: Arial, sans-serif;
    touch-action: manipulation;
  }
  #hud {
    position: absolute; top: 12px; left: 12px; z-index: 2;
    background: #071d19dd; padding: 12px 16px;
    border-radius: 12px; line-height: 1.7;
    font-weight: bold; font-size: 16px;
  }
  #message {
    position: absolute; top: 18%; width: 100%;
    text-align: center; font-size: 28px;
    font-weight: bold; color: #ffe45c;
    text-shadow: 2px 2px 5px black;
    pointer-events: none;
  }
  #controls {
    position: absolute; bottom: 24px; width: 100%;
    display: flex; justify-content: center; gap: 12px;
    z-index: 2; padding: 0 12px;
  }
  button {
    border: 2px solid white; border-radius: 16px;
    padding: 16px 28px; font-size: 19px;
    font-weight: bold; color: white;
    background: #e23d28; box-shadow: 0 4px 0 #7b2016;
  }
  button:active { transform: translateY(3px); }
  #restart { background: #1769aa; box-shadow: 0 4px 0 #0b3658; }
  #hint {
    position: absolute; bottom: 100px; width: 100%;
    text-align: center; font-size: 13px;
    color: white; text-shadow: 1px 1px 3px black;
  }
</style>
</head>
<body>
<div id="hud">
  🏏 3D CRICKET<br>
  Score: <span id="score">0</span>/<span id="wickets">0</span><br>
  Overs: <span id="overs">0.0</span>
</div>
<div id="message">Ready to Play!</div>
<div id="hint">Tap SHOT when the ball comes near!</div>
<div id="controls">
  <button id="shot">🏏 SHOT</button>
  <button id="restart">↻ RESTART</button>
</div>

<script type="module">
import * as THREE from
  'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);
scene.fog = new THREE.Fog(0x87ceeb, 45, 110);

const camera = new THREE.PerspectiveCamera(
  60, innerWidth / innerHeight, 0.1, 200
);
camera.position.set(0, 7, 17);
camera.lookAt(0, 1, -5);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
document.body.appendChild(renderer.domElement);
renderer.domElement.style.position = 'absolute';
renderer.domElement.style.inset = '0';

scene.add(new THREE.HemisphereLight(0xffffff, 0x42623a, 2));

const sun = new THREE.DirectionalLight(0xffffff, 3);
sun.position.set(-8, 18, 8);
sun.castShadow = true;
scene.add(sun);

function material(color) {
  return new THREE.MeshStandardMaterial({ color, roughness: 0.85 });
}

function cube(name, x, y, z, sx, sy, sz, color) {
  const obj = new THREE.Mesh(
    new THREE.BoxGeometry(sx, sy, sz), material(color)
  );
  obj.name = name;
  obj.position.set(x, y, z);
  obj.castShadow = true;
  obj.receiveShadow = true;
  scene.add(obj);
  return obj;
}

function sphere(name, x, y, z, radius, color) {
  const obj = new THREE.Mesh(
    new THREE.SphereGeometry(radius, 20, 16), material(color)
  );
  obj.name = name;
  obj.position.set(x, y, z);
  obj.castShadow = true;
  scene.add(obj);
  return obj;
}

// Green cricket ground
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(100, 120),
  material(0x26823a)
);
ground.rotation.x = -Math.PI / 2;
ground.position.y = -0.08;
ground.receiveShadow = true;
scene.add(ground);

// Stadium boundary circle
const boundary = new THREE.Mesh(
  new THREE.RingGeometry(29.5, 30, 100),
  new THREE.MeshBasicMaterial({
    color: 0xffffff, side: THREE.DoubleSide
  })
);
boundary.rotation.x = -Math.PI / 2;
boundary.position.y = 0.01;
scene.add(boundary);

// Pitch
cube("Pitch", 0, 0, 0, 5, 0.12, 24, 0xc9aa76);

// Pitch markings
for (const z of [-9, 9]) {
  cube("Crease", 0, 0.075, z, 4.2, 0.025, 0.15, 0xffffff);
}

// Three stumps at each end
function makeStumps(z) {
  for (let x = -0.35; x <= 0.36; x += 0.35) {
    cube("Stump", x, 0.55, z, 0.09, 1.1, 0.09, 0xf6e8c8);
  }
  cube("Bail", 0, 1.12, z, 0.9, 0.07, 0.12, 0xffdf65);
}
makeStumps(-9);
makeStumps(9);

// Simple 3D batsman
const batter = new THREE.Group();
scene.add(batter);
batter.position.set(0, 0, -7.2);

function bodyPart(parent, geometry, color, x, y, z) {
  const mesh = new THREE.Mesh(geometry, material(color));
  mesh.position.set(x, y, z);
  mesh.castShadow = true;
  parent.add(mesh);
  return mesh;
}

bodyPart(batter, new THREE.CapsuleGeometry(0.38, 0.65, 5, 10),
  0x1749d1, 0, 1.5, 0);
bodyPart(batter, new THREE.SphereGeometry(0.29, 16, 12),
  0xdca477, 0, 2.25, 0);
bodyPart(batter, new THREE.BoxGeometry(0.22, 0.8, 0.25),
  0xffffff, -0.2, 0.45, 0);
bodyPart(batter, new THREE.BoxGeometry(0.22, 0.8, 0.25),
  0xffffff, 0.2, 0.45, 0);

// Bat
const bat = bodyPart(
  batter, new THREE.BoxGeometry(0.22, 1.45, 0.13),
  0xc88a42, 0.55, 1.05, 0.12
);
bat.rotation.z = -0.45;

// Bowler (simple model)
const bowler = new THREE.Group();
scene.add(bowler);
bowler.position.set(0, 0, 8);

bodyPart(bowler, new THREE.CapsuleGeometry(0.34, 0.6, 4, 10),
  0xd93030, 0, 1.35, 0);
bodyPart(bowler, new THREE.SphereGeometry(0.25, 16, 12),
  0xdca477, 0, 2.05, 0);
bodyPart(bowler, new THREE.BoxGeometry(0.2, 0.8, 0.2),
  0xffffff, -0.16, 0.4, 0);
bodyPart(bowler, new THREE.BoxGeometry(0.2, 0.8, 0.2),
  0xffffff, 0.16, 0.4, 0);

// Cricket ball
const ball = sphere("Ball", 0, 1.2, 7, 0.19, 0xd71920);

let score = 0, wickets = 0, legalBalls = 0;
let ballActive = false, shotUsed = false;
let ballVelocity = new THREE.Vector3();
let ballState = "ready", messageTimer = 0;
let shotTimer = 0;

const scoreEl = document.querySelector("#score");
const wicketsEl = document.querySelector("#wickets");
const oversEl = document.querySelector("#overs");
const messageEl = document.querySelector("#message");

function say(text, duration = 1.5) {
  messageEl.textContent = text;
  messageTimer = duration;
}

function updateScore() {
  scoreEl.textContent = score;
  wicketsEl.textContent = wickets;
  oversEl.textContent =
    Math.floor(legalBalls / 6) + "." + (legalBalls % 6);
}

function resetBall() {
  ball.position.set(0, 1.2, 7);
  ballVelocity.set(0, 0, 0);
  ballActive = false;
  shotUsed = false;
  ballState = "ready";
}

function bowl() {
  if (ballActive || wickets >= 10 || legalBalls >= 60) return;
  ball.position.set((Math.random() - 0.5) * 0.7, 1.35, 7);
  ballVelocity.set((Math.random() - 0.5) * 0.6, 0.05, -0.32);
  ballActive = true;
  shotUsed = false;
  ballState = "bowled";
  say("Ball coming!", 0.7);
}

function playShot() {
  if (!ballActive || shotUsed) {
    if (!ballActive) say("Wait for the ball!", 0.8);
    return;
  }

  // A timing window makes shots more successful near the batter.
  const distance = Math.abs(ball.position.z - (-5.7));
  shotUsed = true;

  if (distance < 2.8 && ball.position.y < 3.3) {
    const timing = Math.max(0, 1 - distance / 3.3);
    const power = 0.6 + timing * 0.65;
    const angle = (Math.random() - 0.5) * 0.75;

    ballVelocity.set(
      Math.sin(angle) * 0.28,
      0.24 + power * 0.2,
      -power
    );
    ballState = "hit";
    shotTimer = 0;
    bat.rotation.z = 0.9;
    say("Great Shot!", 0.8);
  } else {
    say("Missed the timing!", 0.8);
  }
}

document.querySelector("#shot").addEventListener("pointerdown", playShot);
document.querySelector("#restart").addEventListener("click", () => {
  score = 0;
  wickets = 0;
  legalBalls = 0;
  resetBall();
  updateScore();
  say("New innings!", 1.5);
});

window.addEventListener("keydown", e => {
  if (e.code === "Space") {
    e.preventDefault();
    playShot();
  }
});

// Stadium seating and floodlight towers
for (let i = 0; i < 48; i++) {
  const a = i / 48 * Math.PI * 2;
  const x = Math.sin(a) * 33;
  const z = Math.cos(a) * 33;
  const seat = cube("Stadium Seat", x, 1.1, z,
    2.1, 2.2, 1.3, i % 2 ? 0x194d96 : 0xd22e35);
  seat.lookAt(0, 0, 0);
}

for (const x of [-19, 19]) {
  for (const z of [-22, 22]) {
    cube("Floodlight Pole", x, 7, z, 0.3, 14, 0.3, 0x777777);
    const lamp = sphere("Floodlight", x, 14.1, z, 0.7, 0xffffff);
  }
}

// Main game loop
let previousTime = 0;
function animate(time) {
  requestAnimationFrame(animate);
  const dt = Math.min((time - previousTime) / 1000 || 0, 0.04);
  previousTime = time;

  if (messageTimer > 0) {
    messageTimer -= dt;
    if (messageTimer <= 0) messageEl.textContent = "";
  }

  if (ballActive) {
    ball.position.addScaledVector(ballVelocity, dt * 30);

    if (ballState === "bowled") {
      ballVelocity.y -= 0.012 * dt * 30;
      if (ball.position.y < 0.2) {
        ball.position.y = 0.2;
        ballVelocity.y = Math.abs(ballVelocity.y) * 0.45;
      }

      // Ball passes the batter
      if (ball.position.z < -10) {
        ballActive = false;
        legalBalls++;
        if (!shotUsed) {
          wickets++;
          say("OUT! Wicket lost!", 1.4);
        }
        updateScore();
        shotTimer = 1.5;
      }
    } else if (ballState === "hit") {
      ballVelocity.y -= 0.012 * dt * 30;
      if (ball.position.y < 0.2) {
        ball.position.y = 0.2;
        ballVelocity.y *= -0.4;
      }

      if (ball.position.z < -30 || Math.abs(ball.position.x) > 30) {
        let runs = 1;
        const dist = Math.hypot(ball.position.x, ball.position.z);
        if (dist > 27) runs = 6;
        else if (dist > 18) runs = 4;
        else if (dist > 11) runs = 2;
        score += runs;
        legalBalls++;
        say(runs === 6 ? "SIX! 🚀" : runs === 4 ? "FOUR! 🏏" :
             runs + " RUN" + (runs > 1 ? "S" : ""), 1.5);
        updateScore();
        ballActive = false;
        shotTimer = 1.7;
      }
    }
  } else {
    shotTimer -= dt;
    if (shotTimer <= 0 && wickets < 10 && legalBalls < 60) {
      bowl();
    }
  }

  // Animate the bat during the shot
  if (bat.rotation.z > -0.45) {
    bat.rotation.z -= dt * 3;
    if (bat.rotation.z < -0.45) bat.rotation.z = -0.45;
  }

  renderer.render(scene, camera);
}
updateScore();
requestAnimationFrame(animate);

window.addEventListener("resize", () => {
  camera.aspect = innerWidth / innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
});
</script>
</body>
</html>
