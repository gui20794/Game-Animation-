<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Game Animator - Crie Jogos</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: Arial, sans-serif; background: #f4f4f4; overflow: hidden; }
        #toolbar { position: fixed; top: 0; left: 0; right: 0; background: #333; color: white; padding: 10px; display: flex; gap: 10px; z-index: 10; }
        button { background: #4CAF50; color: white; border: none; padding: 8px 12px; border-radius: 5px; cursor: pointer; font-size: 14px; touch-action: manipulation; }
        button:active { background: #45a049; }
        #canvas { width: 100vw; height: calc(100vh - 120px); background: #87CEEB; display: block; touch-action: none; }
        #timeline { position: fixed; bottom: 0; left: 0; right: 0; height: 100px; background: #ddd; overflow-x: auto; display: flex; gap: 5px; padding: 10px; }
        .keyframe { width: 40px; height: 40px; background: #ff9800; border-radius: 50%; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 12px; }
        #controls { position: fixed; bottom: 110px; right: 10px; background: rgba(0,0,0,0.7); color: white; padding: 10px; border-radius: 10px; }
        #export { position: fixed; top: 60px; left: 50%; transform: translateX(-50%); background: #2196F3; }
    </style>
</head>
<body>
    <div id="toolbar">
        <button onclick="addSprite('ball')">Adicionar Bola</button>
        <button onclick="addSprite('box')">Adicionar Caixa</button>
        <button onclick="playPause()">▶ Play/Pausa</button>
        <button onclick="clearAll()">Limpar</button>
        <button id="export" onclick="exportGame()" style="display:none;">📱 Exportar Jogo</button>
    </div>
    <canvas id="canvas"></canvas>
    <div id="timeline"></div>
    <div id="controls">
        Velocidade: <input type="range" id="speed" min="0.1" max="3" step="0.1" value="1" onchange="speed = this.value">
    </div>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight - 120;

        let sprites = []; // Array de sprites {type, x, y, vx, vy, rot, keyframes:[]}
        let isPlaying = false;
        let time = 0;
        let speed = 1;
        let editingSprite = null;

        // Adicionar sprite
        function addSprite(type) {
            const sprite = {
                id: Date.now(),
                type,
                x: canvas.width / 2,
                y: canvas.height / 2,
                vx: 0, vy: 0,
                rot: 0,
                keyframes: [{t:0, x:canvas.width/2, y:canvas.height/2, vx:1, vy:1, rot:0}]
            };
            sprites.push(sprite);
            updateTimeline();
            document.getElementById('export').style.display = 'block';
        }

        // Atualizar timeline
        function updateTimeline() {
            const tl = document.getElementById('timeline');
            tl.innerHTML = '';
            for (let i = 0; i < 20; i++) {
                const kf = document.createElement('div');
                kf.className = 'keyframe';
                kf.textContent = i;
                kf.onclick = () => setKeyframe(i * 1000);
                tl.appendChild(kf);
            }
        }

        // Definir keyframe
        function setKeyframe(t) {
            if (editingSprite) {
                editingSprite.keyframes.push({t, x:editingSprite.x, y:editingSprite.y, vx:editingSprite.vx, vy:editingSprite.vy, rot:editingSprite.rot});
                editingSprite.keyframes.sort((a,b) => a.t - b.t);
            }
        }

        // Desenhar
        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            sprites.forEach(sprite => {
                ctx.save();
                ctx.translate(sprite.x, sprite.y);
                ctx.rotate(sprite.rot * Math.PI / 180);
                if (sprite.type === 'ball') {
                    ctx.beginPath();
                    ctx.arc(0, 0, 20, 0, Math.PI*2);
                    ctx.fillStyle = '#ff5722';
                    ctx.fill();
                } else {
                    ctx.fillStyle = '#2196F3';
                    ctx.fillRect(-20, -20, 40, 40);
                }
                ctx.restore();
            });
        }

        // Atualizar animação
        function update() {
            if (!isPlaying) return;
            time += 16 * speed; // ~60fps
            sprites.forEach(sprite => {
                // Interpolar keyframes simples
                const kf = sprite.keyframes.find(k => k.t <= time) || sprite.keyframes[0];
                sprite.x += kf.vx;
                sprite.y += kf.vy;
                sprite.rot += 1;
                // Bounce
                if (sprite.x > canvas.width + 20 || sprite.x < -20) sprite.vx *= -1;
                if (sprite.y > canvas.height + 20 || sprite.y < -20) sprite.vy *= -1;
            });
        }

        // Loop
        function loop() {
            update();
            draw();
            requestAnimationFrame(loop);
        }

        // Eventos touch
        canvas.addEventListener('touchstart', (e) => {
            e.preventDefault();
            const rect = canvas.getBoundingClientRect();
            const x = e.touches[0].clientX - rect.left;
            const y = e.touches[0].clientY - rect.top;
            // Selecionar sprite para editar
            editingSprite = sprites.find(s => Math.hypot(s.x - x, s.y - y) < 25);
        });

        canvas.addEventListener('touchmove', (e) => {
            e.preventDefault();
            if (editingSprite) {
                const rect = canvas.getBoundingClientRect();
                editingSprite.x = e.touches[0].clientX - rect.left;
                editingSprite.y = e.touches[0].clientY - rect.top;
            }
        });

        // Play/Pausa
        function playPause() {
            isPlaying = !isPlaying;
        }

        // Limpar
        function clearAll() {
            sprites = [];
            time = 0;
            updateTimeline();
        }

        // Exportar jogo
        function exportGame() {
            const gameCode = `<!DOCTYPE html><html><head><title>Meu Jogo</title><meta name="viewport" content="width=device-width,initial-scale=1"><style>body{margin:0;height:100vh;background:#87CEEB;}canvas{width:100%;height:100vh;display:block;}</style></head><body><canvas id="c"></canvas><script>
let sprites=[${JSON.stringify(sprites.map(s=>({...s,keyframes:s.keyframes.slice(0,5)})))]; // Limita keyframes
const c=document.getElementById('c');c.width=innerWidth;c.height=innerHeight;const ctx=c.getContext('2d');let t=0;
function loop(){ctx.clearRect(0,0,c.width,c.height);sprites.forEach(s=>{ctx.save();ctx.translate(s.x,s.y);ctx.rotate(s.rot*Math.PI/180);
if(s.type==='ball'){ctx.beginPath();ctx.arc(0,0,20,0,Math.PI*2);ctx.fillStyle='#ff5722';ctx.fill();}else{ctx.fillStyle='#2196F3';ctx.fillRect(-20,-20,40,40);}
ctx.restore();s.x+=s.vx;s.y+=s.vy;s.rot+=1;if(s.x>c.width+20||s.x<-20)s.vx*=-1;if(s.y>c.height+20||s.y<-20)s.vy*=-1;});t+=16;requestAnimationFrame(loop);}
loop();
</script></body></html>`;
            const blob = new Blob([gameCode], {type: 'text/html'});
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'meu-jogo.html';
            a.click();
        }

        updateTimeline();
        loop();
    </script>
</body>
</html>
