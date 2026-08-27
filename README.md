<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulação Estável de Buraco de Minhoca - Tríade MHD+SAMS+SROS</title>
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
            background-color: #000005;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        canvas {
            display: block;
            width: 100vw;
            height: 100vh;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
        }
        /* HUD Técnico Lateral */
        #hud {
            position: absolute;
            top: 20px;
            left: 20px;
            z-index: 10;
            color: #00ffcc;
            background: rgba(0, 5, 15, 0.85);
            padding: 20px;
            border-radius: 8px;
            border: 1px solid #00ffcc;
            box-shadow: 0 0 15px rgba(0, 255, 204, 0.2);
            pointer-events: none;
            max-width: 320px;
        }
        h1 {
            font-size: 14px;
            text-transform: uppercase;
            letter-spacing: 2px;
            margin-bottom: 12px;
            color: #fff;
            border-bottom: 1px solid #00ffcc;
            padding-bottom: 5px;
        }
        .status-item {
            margin-bottom: 8px;
            font-size: 12px;
        }
        .status-value {
            color: #fff;
            font-weight: bold;
        }
        .flash {
            animation: pulse 1s infinite alternate;
        }
        @keyframes pulse {
            from { opacity: 0.5; }
            to { opacity: 1; }
        }
    </style>
</head>
<body>

    <!-- Canvas para renderização gráfica -->
    <canvas id="wormholeCanvas"></canvas>

    <!-- Interface HUD -->
    <div id="hud">
        <h1>Métrica Einstein-Rosen</h1>
        <div class="status-item">PORTAL: <span class="status-value flash" style="color: #00ff55;">MHD + SAMS + SROS ATIVOS</span></div>
        <div class="status-item">REATOR MHD: <span class="status-value" id="mhd-val">143.00x c/coletor</span></div>
        <div class="status-item">SAMS FLUIDO: <span class="status-value" style="color: #ffaa00;">Ajuste em Milissegundos</span></div>
        <div class="status-item">SROS LASERS: <span class="status-value" id="sros-val">100% Sincronizado</span></div>
    </div>

    <script>
        const canvas = document.getElementById('wormholeCanvas');
        const ctx = canvas.getContext('2d');

        function resize() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resize);
        resize();

        // Parâmetros da simulação
        const particleCount = 250; // Quantidade balanceada para desempenho máximo
        const particles = [];
        let rotationAngle = 0;

        // Geração inicial do túnel matemático
        for (let i = 0; i < particleCount; i++) {
            let angle = Math.random() * Math.PI * 2;
            let radius = 80 + Math.random() * 200; // Raio das paredes do túnel
            
            particles.push({
                x: Math.cos(angle) * radius,
                y: Math.sin(angle) * radius,
                z: Math.random() * 800, // Profundidade
                type: Math.random() > 0.5 ? 'sros' : 'energy', // Azul ou Rosa
                size: 1.5 + Math.random() * 2
            });
        }

        function draw() {
            // Fundo preto com leve transparência para criar o rastro de velocidade das estrelas
            ctx.fillStyle = 'rgba(0, 0, 5, 0.15)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            const centerX = canvas.width / 2;
            const centerY = canvas.height / 2;
            rotationAngle += 0.005; // Rotação do reator MHD

            // Ordena as partículas para desenhar as do fundo primeiro
            particles.sort((a, b) => b.z - a.z);

            for (let i = 0; i < particleCount; i++) {
                let p = particles[i];

                // Move as partículas para frente imitando a viagem da nave
                p.z -= 6;

                // Reinjeta as partículas no fundo ao saírem da tela
                if (p.z <= 10) {
                    p.z = 800;
                    let angle = Math.random() * Math.PI * 2;
                    let radius = 80 + Math.random() * 200;
                    p.x = Math.cos(angle) * radius;
                    p.y = Math.sin(angle) * radius;
                }

                // Cálculo simples de perspectiva 3D para 2D
                let factor = 250 / p.z;
                
                // Aplica rotação simulando o efeito espiral do túnel
                let cosR = Math.cos(rotationAngle);
                let sinR = Math.sin(rotationAngle);
                let rotX = p.x * cosR - p.y * sinR;
                let rotY = p.x * sinR + p.y * cosR;

                let screenX = centerX + rotX * factor;
                let screenY = centerY + rotY * factor;
                let sizeOnScreen = p.size * factor;

                // Restringe o desenho à área visível da tela
                if (screenX > 0 && screenX < canvas.width && screenY > 0 && screenY < canvas.height) {
                    
                    // Define as cores com base no sistema da tríade
                    if (p.type === 'sros') {
                        ctx.fillStyle = `rgba(0, 255, 238, ${1 - p.z/800})`; // Ciano brilhante para os lasers SROS
                    } else {
                        ctx.fillStyle = `rgba(255, 0, 180, ${1 - p.z/800})`; // Magenta para a energia negativa
                    }

                    // Desenha a partícula na tela
                    ctx.beginPath();
                    ctx.arc(screenX, screenY, sizeOnScreen, 0, Math.PI * 2);
                    ctx.fill();
                }
            }

            // Desenha teias de linhas conectando partículas próximas (Rede Full Duplex do SROS)
            for (let i = 0; i < particles.length - 1; i += 15) {
                let p1 = particles[i];
                let p2 = particles[i + 1];

                let f1 = 250 / p1.z;
                let f2 = 250 / p2.z;

                let x1 = centerX + (p1.x * Math.cos(rotationAngle) - p1.y * Math.sin(rotationAngle)) * f1;
                let y1 = centerY + (p1.x * Math.sin(rotationAngle) + p1.y * Math.cos(rotationAngle)) * f1;
                
                let x2 = centerX + (p2.x * Math.cos(rotationAngle) - p2.y * Math.sin(rotationAngle)) * f2;
                let y2 = centerY + (p2.x * Math.sin(rotationAngle) + p2.y * Math.cos(rotationAngle)) * f2;

                if (x1 > 0 && x1 < canvas.width && x2 > 0 && x2 < canvas.width && p1.z > 100) {
                    ctx.beginPath();
                    ctx.moveTo(x1, y1);
                    ctx.lineTo(x2, y2);
                    ctx.strokeStyle = `rgba(0, 255, 200, ${0.05 * (1 - p1.z/800)})`;
                    ctx.lineWidth = 0.5;
                    ctx.stroke();
                }
            }

            // Atualização dinâmica dos dados numéricos do HUD técnico
            const liveTime = Date.now() * 0.002;
            document.getElementById('mhd-val').innerText = (143.0 + Math.sin(liveTime) * 0.8).toFixed(2) + " Eq/Plck";
            document.getElementById('sros-val').innerText = "Conectado (" + (100 - Math.random() * 0.02).toFixed(3) + "%)";

            requestAnimationFrame(draw);
        }

        // Executa a animação
        draw();
    </script>
</body>
</html>
