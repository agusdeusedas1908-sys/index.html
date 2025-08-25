<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>AR Ley Justina – Órganos 3D</title>
  <meta name="description" content="Experiencia WebAR educativa sobre la Ley Justina: coloca modelos 3D de órganos y accede a información al tocarlos." />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap" rel="stylesheet">
  <style>
    :root{--bg:#0b0f14;--panel:#10161d;--accent:#e11d48;--text:#e6f0ff;--muted:#9fb3c8}
    html,body{height:100%;margin:0}
    body{font-family:Inter,system-ui,Segoe UI,Roboto,Helvetica,Arial,sans-serif;background:var(--bg);color:var(--text);overflow:hidden}
    .ui{position:fixed;inset:0;pointer-events:none}
    .topbar{position:absolute;top:12px;left:12px;right:12px;display:flex;gap:8px;align-items:center;justify-content:space-between;pointer-events:auto}
    .brand{display:flex;align-items:center;gap:10px}
    .logo{width:32px;height:32px;border-radius:10px;background:linear-gradient(135deg,var(--accent),#ef4444)}
    .title{font-weight:800;letter-spacing:.2px}
    .pill{background:rgba(255,255,255,.06);border:1px solid rgba(255,255,255,.12);backdrop-filter:blur(6px);padding:8px 12px;border-radius:999px;color:var(--text)}
    .controls{position:absolute;left:12px;right:12px;bottom:14px;display:flex;flex-wrap:wrap;gap:10px;pointer-events:auto}
    .card{background:rgba(16,22,29,.9);border:1px solid rgba(255,255,255,.08);box-shadow:0 10px 30px rgba(0,0,0,.4);border-radius:16px;padding:12px}
    .row{display:flex;gap:10px;flex-wrap:wrap}
    button,select{background:var(--panel);color:var(--text);border:1px solid rgba(255,255,255,.12);border-radius:12px;padding:10px 14px;font-weight:600}
    button:hover{border-color:rgba(255,255,255,.28)}
    .accent{background:linear-gradient(135deg,var(--accent),#ef4444);border:none}
    .accent:hover{filter:brightness(1.05)}
    .reticle{position:absolute;inset:auto;left:50%;top:50%;transform:translate(-50%,-50%);width:44px;height:44px;border:2px solid rgba(255,255,255,.8);border-radius:50%;opacity:.0;pointer-events:none;transition:opacity .2s}
    .reticle.show{opacity:1}
    .toast{position:absolute;left:50%;transform:translateX(-50%);top:60px;background:rgba(16,22,29,.95);border:1px solid rgba(255,255,255,.1);padding:10px 14px;border-radius:12px}
    .modal{position:fixed;inset:0;background:rgba(0,0,0,.5);display:none;align-items:center;justify-content:center;padding:20px;pointer-events:auto}
    .modal.open{display:flex}
    .sheet{max-width:680px;width:100%;background:#0f1720;border:1px solid rgba(255,255,255,.08);border-radius:20px;box-shadow:0 20px 60px rgba(0,0,0,.5)}
    .sheet header{display:flex;gap:12px;align-items:center;padding:16px 18px;border-bottom:1px solid rgba(255,255,255,.06)}
    .sheet header .dot{width:12px;height:12px;border-radius:50%;background:var(--accent)}
    .sheet .content{padding:18px}
    .grid{display:grid;grid-template-columns:1fr;gap:10px}
    @media(min-width:600px){.grid{grid-template-columns:1fr 1fr}}
    .badge{font-size:12px;color:#e2e8f0;background:#12202d;border:1px solid rgba(255,255,255,.08);padding:4px 8px;border-radius:999px}
    a{color:#93c5fd}
    canvas{display:block}
  </style>
</head>
<body>
  <div id="app"></div>

  <!-- UI Overlay -->
  <div class="ui" aria-live="polite">
    <div class="topbar">
      <div class="brand">
        <div class="logo" aria-hidden="true"></div>
        <div>
          <div class="title">AR Ley Justina</div>
          <div class="pill" style="margin-top:4px;font-size:12px">Colocá un órgano en tu entorno y tocá para ver información</div>
        </div>
      </div>
      <div class="row">
        <span class="badge">Tu cámara se usa solo localmente</span>
      </div>
    </div>

    <div class="controls">
      <div class="card" style="flex:1;min-width:280px">
        <div class="row" style="align-items:center;justify-content:space-between">
          <label for="organ" style="font-weight:600">Órgano</label>
          <select id="organ" title="Seleccioná un órgano">
            <option value="corazon">Corazón</option>
            <option value="rinon">Riñón</option>
            <option value="higado">Hígado</option>
            <option value="pulmon">Pulmón</option>
            <option value="cornea">Córnea</option>
          </select>
          <button id="place" class="accent" title="Colocar en el espacio">Colocar</button>
          <button id="info" title="Ver información">Información</button>
          <button id="reset" title="Quitar órganos">Reiniciar</button>
        </div>
      </div>
    </div>

    <div id="reticle" class="reticle" aria-hidden="true"></div>

    <div id="toast" class="toast" style="display:none"></div>
  </div>

  <!-- Modal Información -->
  <div id="modal" class="modal" role="dialog" aria-modal="true" aria-labelledby="h-title">
    <div class="sheet">
      <header>
        <div class="dot"></div>
        <div>
          <div id="h-title" style="font-weight:800">Ley Justina — Información</div>
          <div style="font-size:12px;color:var(--muted)">Argentina • Ley 27.447 (donación de órganos)</div>
        </div>
        <div style="margin-left:auto"><button id="closeModal">Cerrar</button></div>
      </header>
      <div class="content">
        <div class="grid">
          <div>
            <h3 style="margin:.2rem 0 0.5rem 0">Sobre el órgano seleccionado</h3>
            <ul id="organFacts" style="line-height:1.6"></ul>
          </div>
          <div>
            <h3 style="margin:.2rem 0 0.5rem 0">Puntos clave de la Ley Justina</h3>
            <ul style="line-height:1.6">
              <li>Presume que <strong>toda persona mayor de edad es donante</strong> salvo manifestación expresa en contrario.</li>
              <li>Busca <strong>agilizar y transparentar</strong> los procesos de donación y trasplante.</li>
              <li>Refuerza el rol de <strong>INCUCAI</strong> y los organismos jurisdiccionales.</li>
              <li>Promueve la <strong>concientización y educación</strong> sobre donación de órganos y tejidos.</li>
            </ul>
            <p style="font-size:12px;color:var(--muted)">Esta app es educativa y no recopila datos personales. Consultá fuentes oficiales para trámites.</p>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Three.js & helpers -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/webxr/ARButton.js"></script>

  <script>
  // --- Utilidades UI ---
  const toastEl = document.getElementById('toast');
  function toast(msg, ms=2200){
    toastEl.textContent = msg; toastEl.style.display='block';
    setTimeout(()=>toastEl.style.display='none', ms);
  }

  // --- Datos por órgano ---
  const ORGANS = {
    corazon: {
      name: 'Corazón',
      color: 0xd81b60,
      facts: [
        'Late ~100.000 veces al día; su trasplante puede revertir insuficiencia cardíaca terminal.',
        'Criterios estrictos de compatibilidad (grupo sanguíneo, tamaño, urgencia).',
        'Tiempo de isquemia fría suele ser < 4–6 horas.'
      ],
      geometry: () => new THREE.SphereGeometry(0.08, 32, 32), // forma simple
      stretch: {x:1, y:1.2, z:1}
    },
    rinon: {
      name: 'Riñón',
      color: 0x8b5cf6,
      facts: [
        'Filtra desechos y equilibra líquidos; diálisis es terapia sustitutiva, el trasplante mejora calidad de vida.',
        'Puede donarse de donante fallecido o vivo relacionado.',
        'Alta prevalencia de lista de espera.'
      ],
      geometry: () => new THREE.SphereGeometry(0.07, 28, 28),
      stretch: {x:1.2, y:0.9, z:0.7}
    },
    higado: {
      name: 'Hígado',
      color: 0x7c3aed,
      facts: [
        'Órgano metabólico clave; puede regenerarse parcialmente.',
        'Trasplante indicado en cirrosis avanzada y falla hepática.',
        'Puede realizarse trasplante parcial (segmento).'
      ],
      geometry: () => new THREE.SphereGeometry(0.09, 28, 28),
      stretch: {x:1.4, y:0.7, z:1}
    },
    pulmon: {
      name: 'Pulmón',
      color: 0x10b981,
      facts: [
        'Intercambio de oxígeno/CO₂; trasplante unilateral o bilateral.',
        'Indicaciones: fibrosis quística, EPOC avanzada, fibrosis pulmonar.',
        'Tiempo de isquemia fría suele ser < 6–8 horas.'
      ],
      geometry: () => new THREE.SphereGeometry(0.07, 24, 24),
      stretch: {x:1.7, y:1.1, z:0.7}
    },
    cornea: {
      name: 'Córnea (tejido)',
      color: 0x22d3ee,
      facts: [
        'La córnea es el “vidrio” del ojo; su trasplante recupera la visión.',
        'Almacenamiento en banco de tejidos con criterios de calidad.',
        'Donación no altera estética del donante.'
      ],
      geometry: () => new THREE.SphereGeometry(0.05, 24, 24),
      stretch: {x:1, y:1, z:1}
    }
  };

  // --- Escena AR con Three.js ---
  let renderer, scene, camera, xrRefSpace, xrHitTestSource = null, reticleEl, reticleVisible=false;
  let reticleObj, placed = [];
  let raycaster = new THREE.Raycaster();
  let tap = new THREE.Vector2();

  init();

  function init(){
    const container = document.getElementById('app');

    renderer = new THREE.WebGLRenderer({ antialias:true, alpha:true });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.xr.enabled = true;
    container.appendChild(renderer.domElement);

    scene = new THREE.Scene();
    camera = new THREE.PerspectiveCamera();

    const light = new THREE.HemisphereLight(0xffffff, 0x444444, 1.1);
    scene.add(light);

    // Retícula para ubicar
    const ringGeom = new THREE.RingGeometry(0.06, 0.07, 32).rotateX(-Math.PI/2);
    const ringMat = new THREE.MeshBasicMaterial({ color: 0xffffff, opacity: 0.9, transparent:true });
    const dotGeom = new THREE.CircleGeometry(0.005, 16).rotateX(-Math.PI/2);
    const dot = new THREE.Mesh(dotGeom, ringMat);
    const ring = new THREE.Mesh(ringGeom, ringMat);
    reticleObj = new THREE.Group();
    reticleObj.add(ring); reticleObj.add(dot);
    reticleObj.matrixAutoUpdate = false; reticleObj.visible = false;
    scene.add(reticleObj);

    // Botón AR
    document.body.appendChild(ARButton.createButton(renderer, { requiredFeatures:['hit-test'] }));

    renderer.setAnimationLoop(render);

    // Eventos
    window.addEventListener('resize', onResize);
    document.getElementById('place').addEventListener('click', placeSelected);
    document.getElementById('reset').addEventListener('click', resetAll);
    document.getElementById('info').addEventListener('click', openInfo);
    document.getElementById('closeModal').addEventListener('click', closeInfo);
    document.addEventListener('pointerdown', onPointerDown);

    reticleEl = document.getElementById('reticle');

    renderer.xr.addEventListener('sessionstart', onSessionStart);
    renderer.xr.addEventListener('sessionend', onSessionEnd);

    toast('Tocá “ENTRAR EN AR” para activar la cámara');
  }

  async function onSessionStart(){
    const session = renderer.xr.getSession();
    const viewerSpace = await session.requestReferenceSpace('viewer');
    xrRefSpace = await session.requestReferenceSpace('local');
    const hitTestSource = await session.requestHitTestSource({ space: viewerSpace });
    xrHitTestSource = hitTestSource;
    reticleEl.classList.add('show');
    toast('Apuntá a una superficie plana y presioná “Colocar”.');
  }

  function onSessionEnd(){
    xrHitTestSource = null; xrRefSpace = null; reticleEl.classList.remove('show');
  }

  function onResize(){
    renderer.setSize(window.innerWidth, window.innerHeight);
  }

  function render(timestamp, frame){
    if (frame && xrHitTestSource && xrRefSpace){
      const hitTestResults = frame.getHitTestResults(xrHitTestSource);
      if (hitTestResults.length){
        const pose = hitTestResults[0].getPose(xrRefSpace);
        reticleObj.visible = true; reticleVisible = true;
        reticleObj.matrix.fromArray(pose.transform.matrix);
      } else {
        reticleObj.visible = false; reticleVisible = false;
      }
    }
    renderer.render(scene, camera);
  }

  function makeOrganMesh(key){
    const spec = ORGANS[key];
    const geom = spec.geometry();
    geom.computeVertexNormals();
    const mat = new THREE.MeshStandardMaterial({ color: spec.color, roughness:0.6, metalness:0.05 });
    const mesh = new THREE.Mesh(geom, mat);
    mesh.scale.set(spec.stretch.x, spec.stretch.y, spec.stretch.z);
    mesh.userData = { key };
    // latido/respiración suave
    mesh.tick = (t)=>{
      const k = key==='pulmon' ? (Math.sin(t*1.8)*0.03+1) : (Math.sin(t*2.6)*0.02+1);
      mesh.scale.set(spec.stretch.x*k, spec.stretch.y*k, spec.stretch.z*k);
    };
    return mesh;
  }

  function placeSelected(){
    const key = document.getElementById('organ').value;
    if (!reticleVisible){ toast('No se detecta una superficie. Mové el dispositivo.'); return; }
    const organ = makeOrganMesh(key);
    organ.applyMatrix4(reticleObj.matrix);
    organ.position.y += 0.03; // levanta un poco
    scene.add(organ);
    placed.push(organ);
    toast(`${ORGANS[key].name} colocado. Tocá el órgano para ver info.`);
  }

  function resetAll(){
    placed.forEach(m => scene.remove(m));
    placed = [];
    toast('Escena reiniciada.');
  }

  function openInfo(forKey){
    const key = forKey || document.getElementById('organ').value;
    const list = document.getElementById('organFacts');
    list.innerHTML = '';
    ORGANS[key].facts.forEach(f=>{
      const li=document.createElement('li'); li.textContent=f; list.appendChild(li);
    });
    document.getElementById('h-title').textContent = `Información — ${ORGANS[key].name}`;
    document.getElementById('modal').classList.add('open');
  }
  function closeInfo(){ document.getElementById('modal').classList.remove('open'); }

  function onPointerDown(ev){
    if (!placed.length) return;
    const rect = renderer.domElement.getBoundingClientRect();
    tap.x = ((ev.clientX - rect.left) / rect.width) * 2 - 1;
    tap.y = -((ev.clientY - rect.top) / rect.height) * 2 + 1;

    raycaster.setFromCamera(tap, camera);
    const intersects = raycaster.intersectObjects(placed, false);
    if (intersects.length){
      const key = intersects[0].object.userData.key;
      openInfo(key);
    }
  }

  // Animación orgánica
  (function loop(){
    requestAnimationFrame(loop);
    const t = performance.now()/1000;
    placed.forEach(m=>m.tick && m.tick(t));
  })();
  </script>

  <!-- Sección QR (nota): generá un QR que apunte a la URL pública donde hospedes este archivo -->
</body>
</html>
