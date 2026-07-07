<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>VJ LUIGI LONG // WEBGL TERMINAL PORTFOLIO</title>
    
    <!-- Fuentes Web Premium -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@300;400;600&family=Space+Grotesk:wght@400;700&display=swap" rel="stylesheet">

    <style>
        :root {
            --font-display: 'Space Grotesk', sans-serif;
            --font-mono: 'IBM Plex Mono', monospace;
            --accent-green: #00ff66;
            --bg-dark: #020203;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
            -webkit-user-drag: none;
        }

        body, html {
            background-color: var(--bg-dark);
            color: #f0f0f5;
            font-family: var(--font-mono);
            overflow-x: hidden;
            width: 100%;
        }

        /* --- EFECTO MONITOR CRT SONY TRINITRON & SCANLINES --- */
        .crt-monitor {
            position: fixed;
            top: 0; left: 0; width: 100vw; height: 100vh;
            pointer-events: none;
            z-index: 99999;
            box-shadow: inset 0 0 120px rgba(0,0,0,0.9);
            background: linear-gradient(rgba(18, 16, 16, 0) 50%, rgba(0, 0, 0, 0.35) 50%);
            background-size: 100% 4px;
            animation: scanlineScroll 12s linear infinite, crtFlicker 0.15s infinite;
        }

        .crt-vignette {
            position: fixed;
            inset: 0;
            background: radial-gradient(circle, transparent 50%, rgba(0,0,0,0.75) 100%);
            pointer-events: none;
            z-index: 99998;
        }

        @keyframes scanlineScroll {
            0% { background-position: 0 0; }
            100% { background-position: 0 100%; }
        }

        @keyframes crtFlicker {
            0% { opacity: 0.995; }
            50% { opacity: 1; }
            100% { opacity: 0.992; }
        }

        /* --- RETÍCULA ESTILO AUTOCAD --- */
        .autocad-grid {
            position: fixed;
            inset: 0;
            background-size: 50px 50px;
            background-image: 
                linear-gradient(to right, rgba(0, 255, 102, 0.02) 1px, transparent 1px),
                linear-gradient(to bottom, rgba(0, 255, 102, 0.02) 1px, transparent 1px);
            z-index: -1;
            pointer-events: none;
        }

        /* --- CANVAS PARA FONDO THREE.JS --- */
        #webgl-canvas-container {
            position: fixed;
            top: 0; left: 0; width: 100vw; height: 100vh;
            z-index: -2;
            background: #000;
        }

        /* --- CURSOR HUD PERSONALIZADO --- */
        #hud-cursor {
            position: fixed;
            top: 0; left: 0;
            width: 30px; height: 30px;
            border: 1px solid var(--accent-green);
            border-radius: 50%;
            pointer-events: none;
            transform: translate(-50%, -50%);
            z-index: 100000;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: width 0.3s, height 0.3s, background-color 0.3s;
        }

        #hud-cursor .dot {
            width: 4px; height: 4px;
            background-color: var(--accent-green);
            border-radius: 50%;
        }

        #hud-cursor .hud-text {
            font-size: 9px;
            color: #000;
            font-weight: bold;
            display: none;
            text-transform: uppercase;
            font-family: var(--font-mono);
        }

        body.hover-node #hud-cursor {
            width: 75px; height: 75px;
            background-color: var(--accent-green);
        }
        body.hover-node #hud-cursor .dot { display: none; }
        body.hover-node #hud-cursor .hud-text { display: block; }

        /* --- LOADER INTERACTIVO KERNEL --- */
        #kernel-loader {
            position: fixed;
            inset: 0;
            background-color: #010102;
            z-index: 99990;
            padding: 3rem;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
        }

        .loader-console {
            font-family: var(--font-mono);
            color: var(--accent-green);
            font-size: 14px;
            line-height: 1.7;
            max-width: 800px;
        }

        .loader-footer {
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-top: 1px solid rgba(0, 255, 102, 0.2);
            padding-top: 2rem;
        }

        .action-trigger {
            background: transparent;
            border: 1px solid var(--accent-green);
            color: var(--accent-green);
            padding: 12px 35px;
            font-family: var(--font-mono);
            cursor: pointer;
            visibility: hidden;
            letter-spacing: 2px;
            transition: all 0.3s;
            font-weight: 600;
        }

        .action-trigger:hover {
            background: var(--accent-green);
            color: #000;
            box-shadow: 0 0 20px var(--accent-green);
        }

        /* --- DISEÑO DE LA INTERFAZ PRINCIPAL --- */
        header {
            position: fixed;
            top: 0; left: 0; width: 100%;
            padding: 2.5rem;
            display: flex;
            justify-content: space-between;
            align-items: center;
            z-index: 900;
            transition: all 0.4s ease;
        }

        header.scrolled {
            padding: 1.2rem 2.5rem;
            background: rgba(2, 2, 3, 0.35);
            backdrop-filter: blur(15px);
            border-bottom: 1px solid rgba(0, 255, 102, 0.15);
            color: var(--accent-green);
        }

        .brand-glitch {
            font-weight: 700;
            font-family: var(--font-display);
            letter-spacing: 2px;
            font-size: 1.3rem;
        }

        main {
            opacity: 0;
        }

        section {
            min-height: 100vh;
            padding: 200px 4rem; /* Gran Espaciado */
            display: flex;
            flex-direction: column;
            justify-content: center;
            border-bottom: 1px solid rgba(255,255,255,0.03);
        }

        .section-tag {
            color: var(--accent-green);
            font-size: 11px;
            letter-spacing: 5px;
            margin-bottom: 2.5rem;
        }

        /* Titulares Gigantes */
        .giant-headline {
            font-family: var(--font-display);
            font-size: calc(4.5vw + 3rem);
            font-weight: 700;
            line-height: 1.02;
            text-transform: uppercase;
            mix-blend-mode: difference;
            margin-bottom: 2.5rem;
            letter-spacing: -2px;
        }

        /* --- CONTENEDOR DE PROYECTOS FULL WIDTH --- */
        .full-width-projects {
            width: 100%;
            margin-top: 4rem;
        }

        .project-row {
            display: flex;
            justify-content: space-between;
            align-items: flex-end;
            border-bottom: 1px solid rgba(255,255,255,0.1);
            padding: 3.5rem 0;
            text-decoration: none;
            color: inherit;
            transition: padding-left 0.4s, border-color 0.4s;
            position: relative;
            cursor: none;
        }

        .project-row:hover {
            padding-left: 2.5rem;
            border-color: var(--accent-green);
        }

        .proj-meta {
            font-size: 1.5rem;
            color: rgba(255,255,255,0.3);
            font-weight: 300;
        }

        .proj-title {
            font-family: var(--font-display);
            font-size: 4.5rem;
            font-weight: 700;
            text-transform: uppercase;
        }

        .proj-year {
            font-family: var(--font-mono);
            color: var(--accent-green);
            font-size: 1.2rem;
        }

        /* --- TERMINAL DE CONSOLA REAL --- */
        .live-hardware-console {
            background: rgba(10, 10, 15, 0.85);
            border: 1px solid rgba(0, 255, 102, 0.15);
            padding: 2.5rem;
            border-radius: 4px;
            font-family: var(--font-mono);
            min-height: 280px;
            color: #b5ffce;
            line-height: 1.8;
            font-size: 14px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
        }

        /* --- MODAL INMERSIVO FULLSCREEN PARA CASOS DE ESTUDIO --- */
        #case-study-overlay {
            position: fixed;
            inset: 0;
            background: #010103;
            z-index: 10000;
            display: none;
            padding: 5rem;
            overflow-y: auto;
        }

        .overlay-close {
            position: absolute;
            top: 3rem; right: 3rem;
            background: transparent;
            border: 1px solid var(--accent-green);
            color: var(--accent-green);
            padding: 12px 25px;
            cursor: pointer;
            font-family: var(--font-mono);
            font-weight: 600;
            transition: all 0.3s;
        }

        .overlay-close:hover {
            background: var(--accent-green);
            color: #000;
        }
    </style>
</head>
<body>

    <!-- Capas del Monitor Analógico -->
    <div class="crt-monitor"></div>
    <div class="crt-vignette"></div>
    <div class="autocad-grid"></div>
    
    <!-- Cursor HUD -->
    <div id="hud-cursor"><div class="dot"></div><span class="hud-text">OPEN</span></div>

    <!-- PANTALLA DE CARGA PRO (KERNEL SYSTEM) -->
    <div id="kernel-loader">
        <div class="loader-console" id="typed-console"></div>
        <div class="loader-footer">
            <div style="color: rgba(255,255,255,0.3)">ENGINE_STATUS // UNSTABLE_BUILD_2026</div>
            <button class="action-trigger" id="enter-btn">ENTER_SYSTEM</button>
        </div>
    </div>

    <!-- CONTENEDOR RENDER WEBGL -->
    <div id="webgl-canvas-container"></div>

    <!-- INTERFAZ PRINCIPAL -->
    <div id="app-interface" style="opacity: 0;">
        <header id="global-header">
            <div class="brand-glitch" id="vj-title">VJ LUIGI LONG</div>
            <div id="utc-clock">00:00:00 UTC</div>
        </header>

        <main id="scroll-container">
            <!-- SECCIÓN 1: IDENTITY -->
            <section id="ch-signal" class="scroll-reveal">
                <div class="section-tag">[CH_01 // IDENTITY]</div>
                <h1 class="giant-headline">LIVE VISUALS<br>VIDEO MAPPING<br>DIGITAL EXPERIENCES</h1>
                <p style="max-width: 650px; color: rgba(255,255,255,0.6); font-size: 1.15rem; line-height: 1.9;">
                    Soy un Artista Digital y Operador de cámara en vivo con base en Neuquén, Argentina. Mi práctica se centra en la captura y manipulación de imagen en tiempo real para Shows Musicales, Eventos Corporativos y experiencias site-specific.

Desarrollo arquitecturas ópticas que dialogan con el espacio, entornos Interactivos que responden al movimiento y al sonido, y Proyecciones de Video Mapping sobre Monumentos y Fachadas.

Cada proyecto es una oportunidad para transformar la percepción del espectador y crear momentos únicos donde la tecnología se vuelve la emoción del evento.
                </p>
            </section>

            <!-- SECCIÓN 2: COMPILATION -->
            <section id="ch-works" class="scroll-reveal">
                <div class="section-tag">[CH_02 // COMPILATION]</div>
                <div class="full-width-projects">
                    
                    <a href="#" class="project-row" data-project="live-set">
                        <span class="proj-meta">01</span>
                        <h2 class="proj-title">LIVE SET</h2>
                        <span class="proj-year">2026</span>
                    </a>

                    <a href="#" class="project-row" data-project="ppoo">
                        <span class="proj-meta">02</span>
                        <h2 class="proj-title">ppoo0o000oooo</h2>
                        <span class="proj-year">2025</span>
                    </a>

                    <a href="#" class="project-row" data-project="creative">
                        <span class="proj-meta">03</span>
                        <h2 class="proj-title">LUIGI LONG CREATIVE</h2>
                        <span class="proj-year">2026</span>
                    </a>

                </div>
            </section>

            <!-- SECCIÓN 3: PIPELINE CONSOLE -->
            <section id="ch-output" class="scroll-reveal">
                <div class="section-tag">[CH_03 // PIPELINE]</div>
                <div class="live-hardware-console" id="hardware-terminal"></div>
            </section>

            <!-- SECCIÓN 4: TRANSFERS -->
            <section id="ch-transmit" class="scroll-reveal" style="border: none;">
                <div class="section-tag">[CH_04 // TRANSFERS]</div>
                <h2 class="giant-headline" style="font-size: 4rem;">ESTABLISH_UPLINK</h2>
                <div style="display: flex; flex-direction: column; gap: 1.8rem; margin-top: 2rem;">
                    <a href="mailto:longluigi89@gmail.com" style="color: var(--accent-green); text-decoration: none; font-size: 1.8rem; font-family: var(--font-display); font-weight: 700;">longluigi89@gmail.com ↗</a>
                    <a href="https://www.instagram.com/vj_luigi.long_/" target="_blank" style="color: #fff; text-decoration: none; font-size: 1.8rem; font-family: var(--font-display); font-weight: 700;">instagram.com/vj_luigi.long_ ↗</a>
                </div>
            </section>
        </main>
    </div>

    <!-- MODAL INMERSIVO FULLSCREEN -->
    <div id="case-study-overlay">
        <button class="overlay-close" id="close-overlay">CLOSE_MODULE</button>
        <div id="overlay-content" style="margin-top: 5rem;"></div>
    </div>

    <!-- CDNs de Dependencias Críticas -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/ScrollTrigger.min.js"></script>
    <script src="https://cdn.jsdelivr.net/gh/studio-freight/lenis@1.0.19/bundled/lenis.min.js"></script>

    <script>
        // Registrar ScrollTrigger de GSAP inmediatamente
        gsap.registerPlugin(ScrollTrigger);

        // ==========================================================================
        // 1. LOADER MAQUINA DE ESCRIBIR KERNEL
        // ==========================================================================
        const bootLogs = [
            "BOOTING KERNEL_VJ_v4.0.26...",
            "MOUNTING HARDWARE ACCELERATED GPU SUBROUTINES...",
            "CHECKING COMPILING GLSL SHADERS (POST-PROCESSING)...",
            "ALLOCATING GRAPHICS VRAM INODE BUFFER...",
            "INITIALIZING LIVE VISUAL ENGINE ROUTINES...",
            "CONNECTING TO NEUQUEN_NODE_AR NETWORK...",
            "OPENGL LAYER READY",
            "WEBGPU COMPATIBILITY MATRIX CHECK OK",
            "VIDEO MAPPING CORE CORE_ENGINE ENGAGED.",
            "STATUS: ALL SYSTEMS OPERATIONAL."
        ];

        const consoleElement = document.getElementById("typed-console");
        let logLine = 0;

        function typeConsole() {
            if (logLine < bootLogs.length) {
                let p = document.createElement("p");
                p.innerHTML = `> ${bootLogs[logLine]}`;
                consoleElement.appendChild(p);
                logLine++;
                setTimeout(typeConsole, 140);
            } else {
                document.getElementById("enter-btn").style.visibility = "visible";
            }
        }

        typeConsole();

        document.getElementById("enter-btn").addEventListener("click", () => {
            playSynthSound(440, "sawtooth", 0.12);
            
            gsap.to("#kernel-loader", {
                opacity: 0,
                duration: 0.8,
                onComplete: () => {
                    document.getElementById("kernel-loader").style.display = "none";
                    document.getElementById("app-interface").style.opacity = 1;
                    gsap.to("main", { opacity: 1, duration: 1 });
                    initApplication();
                }
            });
        });

        // ==========================================================================
        // AUDIO PROCEDURAL VIA WEB AUDIO API (Evita dependencias externas)
        // ==========================================================================
        function playSynthSound(freq, type, duration) {
            try {
                const ctx = new (window.AudioContext || window.webkitAudioContext)();
                const osc = ctx.createOscillator();
                const gain = ctx.createGain();
                osc.type = type;
                osc.frequency.setValueAtTime(freq, ctx.currentTime);
                gain.gain.setValueAtTime(0.08, ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + duration);
                osc.connect(gain);
                gain.connect(ctx.destination);
                osc.start();
                osc.stop(ctx.currentTime + duration);
            } catch(e) {}
        }

        function initApplication() {
            // ==========================================================================
            // 2. SCROLL SUAVE CON LENIS & INTERACCIÓN EN HEADER
            // ==========================================================================
            const lenis = new Lenis({
                duration: 1.2,
                easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t))
            });

            function raf(time) {
                lenis.raf(time);
                requestAnimationFrame(raf);
            }
            requestAnimationFrame(raf);

            window.addEventListener('scroll', () => {
                const header = document.getElementById('global-header');
                if (window.scrollY > 60) {
                    header.classList.add('scrolled');
                } else {
                    header.classList.remove('scrolled');
                }
            });

            // ==========================================================================
            // 3. HUD CURSOR CONTROL & EVENTOS HOVER
            // ==========================================================================
            const cursor = document.getElementById('hud-cursor');
            let mouseX = 0, mouseY = 0;
            let currentX = 0, currentY = 0;

            window.addEventListener('mousemove', (e) => {
                mouseX = e.clientX;
                mouseY = e.clientY;
            });

            function updateCursor() {
                currentX += (mouseX - currentX) * 0.15;
                currentY += (mouseY - currentY) * 0.15;
                cursor.style.left = `${currentX}px`;
                cursor.style.top = `${currentY}px`;
                requestAnimationFrame(updateCursor);
            }
            updateCursor();

            document.querySelectorAll('.project-row, .action-trigger, .overlay-close').forEach(item => {
                item.addEventListener('mouseenter', () => {
                    document.body.classList.add('hover-node');
                    playSynthSound(880, 'sine', 0.04);
                });
                item.addEventListener('mouseleave', () => {
                    document.body.classList.remove('hover-node');
                });
            });

            // ==========================================================================
            // 4. ANIMACIÓN GLITCH TITULAR INICIAL (300ms)
            // ==========================================================================
            const titleElement = document.getElementById('vj-title');
            gsap.fromTo(titleElement, 
                { x: () => (Math.random() - 0.5) * 30, y: () => (Math.random() - 0.5) * 30, opacity: 0.3 },
                { x: 0, y: 0, opacity: 1, duration: 0.3, ease: "power4.out" }
            );

            // ==========================================================================
            // 5. TRANSICIONES REVEAL DE SECCIONES CON GSAP
            // ==========================================================================
            document.querySelectorAll('.scroll-reveal').forEach(section => {
                gsap.fromTo(section, 
                    { opacity: 0, y: 80, filter: "blur(8px)" },
                    { 
                        opacity: 1, y: 0, filter: "blur(0px)",
                        duration: 1.4,
                        scrollTrigger: {
                            trigger: section,
                            start: "top 85%",
                            toggleActions: "play none none none"
                        }
                    }
                );
            });

            // ==========================================================================
            // 6. FONDO VIVO EN THREE.JS (Partículas + Ruido Procedural Reactivo)
            // ==========================================================================
            const container = document.getElementById('webgl-canvas-container');
            const scene = new THREE.Scene();
            const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            container.appendChild(renderer.domElement);

            const particleCount = 500;
            const geometry = new THREE.BufferGeometry();
            const positions = new Float32Array(particleCount * 3);

            for(let i=0; i<particleCount*3; i+=3) {
                positions[i] = (Math.random() - 0.5) * 12;
                positions[i+1] = (Math.random() - 0.5) * 12;
                positions[i+2] = (Math.random() - 0.5) * 6;
            }
            geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
            
            const material = new THREE.PointsMaterial({ 
                color: 0x00ff66, 
                size: 0.035, 
                transparent: true, 
                opacity: 0.75 
            });
            const particleSystem = new THREE.Points(geometry, material);
            scene.add(particleSystem);

            camera.position.z = 4;

            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            });

            function animateWebGL() {
                requestAnimationFrame(animateWebGL);
                
                // Movimiento cinético basado en el mouse del operador
                particleSystem.rotation.y += 0.0015;
                particleSystem.rotation.x += ((mouseY * 0.00015) - particleSystem.rotation.x) * 0.05;
                particleSystem.rotation.y += ((mouseX * 0.00015) - particleSystem.rotation.y) * 0.05;
                
                renderer.render(scene, camera);
            }
            animateWebGL();

            // ==========================================================================
            // 7. RELOJ UTC SCONCRONIZADO
            // ==========================================================================
            setInterval(() => {
                const d = new Date();
                document.getElementById('utc-clock').textContent = `${String(d.getUTCHours()).padStart(2,'0')}:${String(d.getUTCMinutes()).padStart(2,'0')}:${String(d.getUTCSeconds()).padStart(2,'0')} UTC`;
            }, 1000);

            // ==========================================================================
            // 8. CONSOLA EN TIEMPO REAL (CH_03 OUTPUT)
            // ==========================================================================
            const consoleLines = [
                "$ node --render-pipeline",
                "Loading deployment structural geometries...",
                "GPU pipeline fully engaged via context.",
                "Active Frame-buffer Matrix: 60 FPS stable.",
                "Matrix Latency: 2 ms // Vector stable.",
                "Real-time pipeline diagnostics: SECURE."
            ];
            const targetTerminal = document.getElementById('hardware-terminal');
            let terminalIdx = 0;

            function runRealConsole() {
                if(terminalIdx < consoleLines.length) {
                    let l = document.createElement('div');
                    l.textContent = `> ${consoleLines[terminalIdx]}`;
                    targetTerminal.appendChild(l);
                    terminalIdx++;
                    setTimeout(runRealConsole, 900);
                }
            }
            runRealConsole();

            // ==========================================================================
            // 9. CAPA DE CASOS DE ESTUDIO FULLSCREEN INMERSIVOS
            // ==========================================================================
            const projectDatabase = {
                "live-set": { title: "LIVE SET 2026", desc: "Performance audiovisual de alta densidad en tiempo real con modelado procedural generativo y sincronización de datos mediante arquitectura de red avanzada.", tech: "TouchDesigner / GLSL Shaders / Resolume / WebGL" },
                "ppoo": { title: "ppoo0o000oooo", desc: "Instalación interactiva algorítmica de carácter masivo basada en la captura de perturbaciones magnéticas externas y traducción de señales lumínicas.", tech: "WebGPU / Processing / Arduino Architecture / MaxMSP" },
                "creative": { title: "LUIGI LONG CREATIVE", desc: "Estructura monumental de Video Mapping proyectada sobre superficies arquitectónicas brutalistas complejas utilizando cálculo geométrico espacial avanzado.", tech: "MadMapper / Heavy-Duty Lasers / Unreal Engine / Blender 3D" }
            };

            document.querySelectorAll('.project-row').forEach(row => {
                row.addEventListener('click', (e) => {
                    e.preventDefault();
                    const key = row.getAttribute('data-project');
                    const data = projectDatabase[key];
                    
                    const overlay = document.getElementById('case-study-overlay');
                    const content = document.getElementById('overlay-content');
                    
                    content.innerHTML = `
                        <h1 style="font-family: var(--font-display); font-size: calc(3.5vw + 2rem); color: var(--accent-green); text-transform: uppercase; font-weight:700; line-height:1.1;">${data.title}</h1>
                        <p style="font-size: 1.4rem; margin: 3rem 0; max-width: 850px; line-height: 1.7; color: rgba(255,255,255,0.8);">${data.desc}</p>
                        <div style="border: 1px solid rgba(0, 255, 102, 0.3); padding: 2rem; display: inline-block; background: rgba(0,255,102,0.02);">
                            <strong style="color: var(--accent-green);">CORE_TECHNOLOGIES:</strong> <span style="font-family: var(--font-mono); font-size:1.1rem; margin-left:10px;">${data.tech}</span>
                        </div>
                    `;
                    
                    overlay.style.display = 'block';
                    playSynthSound(587.33, 'triangle', 0.25);
                });
            });

            document.getElementById('close-overlay').addEventListener('click', () => {
                document.getElementById('case-study-overlay').style.display = 'none';
                playSynthSound(293.66, 'sine', 0.15);
            });
        }
    </script>
</body>
</html>
