# 🌐 Cyber Globe 3D — Guía de personalización (Three.js)

Todo el modelo vive en el `<script>` al final de `index.html`, justo después de que se carga `three.min.js`. A continuación se explica cada sección y qué valores puedes cambiar para obtener distintos resultados.

---

## 1. CÁMARA

```js
const camera = new THREE.PerspectiveCamera(45, W / H, 0.1, 1000);
camera.position.z = 3.0;
```

| Propiedad | Valor actual | Qué hace |
|---|---|---|
| `45` (FOV) | `45` | Campo de visión en grados. Más alto = zoom out (más escena visible). Más bajo = zoom in (efecto telescopio). Rango útil: `30`–`75` |
| `camera.position.z` | `3.0` | Distancia de la cámara a la esfera. Más alto = esfera más pequeña en pantalla. Más bajo = esfera más grande y cercana |

---

## 2. TEXTURA CYBER (la superficie de la esfera)

```js
const earthTex = makeCyberTexture(2048);
```

La función `makeCyberTexture(size)` dibuja toda la apariencia de la superficie en un canvas 2D que luego se aplica como textura.

### 2a. Resolución de la textura
```js
makeCyberTexture(2048)
```
- `2048` = píxeles del canvas interno. Más alto = más detalle pero más RAM de GPU.
- Valores útiles: `512`, `1024`, `2048`, `4096`

### 2b. Color de fondo de la esfera
```js
ctx.fillStyle = '#020a10';
```
- Color base oscuro de la textura (el "mar" del globo).
- Cámbialo por cualquier color CSS: `'#000000'`, `'#0d0d0d'`, `'#001a00'`, etc.

### 2c. Grid de líneas (la cuadrícula sobre la esfera)
```js
ctx.strokeStyle = 'rgba(34,211,238,0.18)';
ctx.lineWidth = 0.8;
const step = size / 28;
```
| Propiedad | Qué controla |
|---|---|
| `rgba(34,211,238,0.18)` | Color y opacidad de las líneas. El cuarto número (`0.18`) es la transparencia (0 = invisible, 1 = sólido) |
| `lineWidth = 0.8` | Grosor de las líneas del grid |
| `size / 28` | Densidad del grid. Número más pequeño = más líneas, más denso |

### 2d. Líneas diagonales de circuito
```js
ctx.strokeStyle = 'rgba(34,211,238,0.08)';
ctx.lineWidth = 0.5;
```
- Igual que el grid pero más tenues y en diagonal.
- El `0.08` controla qué tan visibles son.

### 2e. Nodos de ciudad (puntos brillantes)
```js
const nodes = [
    [0.22,0.22],[0.15,0.30], ...
];
```
- Cada par `[x, y]` es una coordenada normalizada (`0.0` = izquierda/arriba, `1.0` = derecha/abajo) en la textura.
- Agregar más pares = más nodos brillantes en la esfera.

```js
const grd = ctx.createRadialGradient(x, y, 0, x, y, 10);
grd.addColorStop(0, 'rgba(34,211,238,0.9)');  // Centro del glow
grd.addColorStop(0.3, 'rgba(34,211,238,0.4)'); // Medio
grd.addColorStop(1, 'rgba(34,211,238,0)');     // Borde (transparente)
```
- El `10` es el radio del glow en píxeles de textura.
- Los colores de `addColorStop` controlan la transición del brillo.

```js
ctx.arc(x, y, 2, 0, Math.PI * 2); // Punto central blanco
ctx.fillStyle = '#ffffff';
```
- El `2` es el radio del punto blanco central. Súbelo para puntos más grandes.

### 2f. Líneas de conexión entre nodos
```js
ctx.strokeStyle = 'rgba(34,211,238,0.3)';
ctx.lineWidth = 0.8;
const connections = [[0,1],[1,2],[0,3], ...];
```
- `connections` es un array de pares de índices de `nodes` que se conectan con una línea.
- `[0,1]` conecta el nodo 0 con el nodo 1.
- La opacidad `0.3` controla qué tan visibles son los "circuitos".

---

## 3. ESFERA PRINCIPAL

```js
new THREE.SphereGeometry(0.65, 80, 80)
```
| Parámetro | Valor | Qué hace |
|---|---|---|
| `0.65` | radio | Tamaño de la esfera. `1.0` = tamaño estándar. Bájalo para esfera más pequeña |
| `80, 80` | segmentos W × H | Suavidad de la esfera. `32` = notarás caras planas. `80` = esfera perfectamente redonda |

### Material de la esfera (`MeshPhongMaterial`)
```js
new THREE.MeshPhongMaterial({
    map: earthTex,           // textura de color
    emissiveMap: earthTex,   // textura de emisión (brilla sola)
    emissive: new THREE.Color(0x0a1520),  // color base de la emisión
    emissiveIntensity: 0.6,  // qué tanto brilla sola
    specular: new THREE.Color(0x0d6677),  // color del reflejo especular
    shininess: 6,            // dureza del reflejo (bajo = difuso, alto = círculo duro)
})
```

| Propiedad | Efecto al subir | Efecto al bajar |
|---|---|---|
| `emissiveIntensity` | La esfera brilla más por sí sola, ignora más la iluminación | Se ve más oscura sin luz |
| `shininess` | Reflejo más pequeño y duro (círculo visible) | Reflejo más difuso y suave (recomendado: 1–15) |
| `specular` color | Tono del reflejo especular | — |

---

## 4. WIREFRAME (red sobre la esfera)

```js
new THREE.SphereGeometry(0.651, 28, 28)  // radio levemente mayor que la esfera
new THREE.MeshBasicMaterial({
    color: 0x22d3ee,
    wireframe: true,
    transparent: true,
    opacity: 0.06,   // ← muy sutil
})
```
- `opacity: 0.06` = barely visible. Súbelo a `0.15`–`0.3` para que la red sea más notoria.
- `color: 0x22d3ee` = cyan. Cámbialo a `0x00ff00` para verde, `0xff6600` para naranja, etc.
- El radio debe ser ligeramente mayor que la esfera principal (`0.65`) para que quede por encima y no haya z-fighting.

---

## 5. ATMÓSFERA INTERIOR (glow suave alrededor)

```js
new THREE.SphereGeometry(0.69, 64, 64)   // levemente mayor que la esfera
new THREE.MeshPhongMaterial({
    color: 0x22d3ee,
    transparent: true,
    opacity: 0.07,           // ← muy sutil, solo una capa de luz
    side: THREE.FrontSide,   // visible desde afuera
    depthWrite: false,       // no bloquea lo que hay detrás
})
```
- Aumenta `opacity` a `0.15`–`0.25` para un glow más dramático.
- Cambia `color` para un ambiente diferente (verde, morado, naranja...).

---

## 6. HALO EXTERIOR

```js
new THREE.SphereGeometry(0.77, 64, 64)
new THREE.MeshPhongMaterial({
    color: 0x22d3ee,
    transparent: true,
    opacity: 0.035,
    side: THREE.BackSide,    // ← se renderiza por dentro, visible desde afuera
    depthWrite: false,
})
```
- Es la capa más externa. `BackSide` hace que la cara interior de la esfera sea visible, creando el efecto de "aura".
- `opacity: 0.035` es muy sutil. Súbelo a `0.08`–`0.12` para un halo muy notorio.

---

## 7. ANILLOS ORBITALES

```js
function makeRing(radius, tube, color, opacity, tiltX, tiltZ)

const ring1 = makeRing(0.86, 0.003, 0x22d3ee, 0.55, Math.PI / 2, 0.3);
const ring2 = makeRing(0.96, 0.002, 0x60a5fa, 0.35, Math.PI / 2.5, -0.5);
const ring3 = makeRing(1.05, 0.002, 0x22d3ee, 0.20, Math.PI / 1.8, 0.8);
```

| Parámetro | Qué controla |
|---|---|
| `radius` (1er arg) | Distancia del anillo al centro de la esfera |
| `tube` (2do arg) | Grosor del anillo. `0.003` = muy fino. `0.01` = notablemente grueso |
| `color` (3er arg) | Color en hex. `0x22d3ee` = cyan, `0x60a5fa` = azul |
| `opacity` (4to arg) | Transparencia. `0.55` = bastante visible, `0.1` = casi invisible |
| `tiltX` (5to arg) | Inclinación en X. `Math.PI/2` = 90°, `Math.PI/4` = 45° |
| `tiltZ` (6to arg) | Inclinación en Z. Positivo/negativo cambia la dirección |

**Para agregar un 4to anillo:**
```js
const ring4 = makeRing(1.18, 0.002, 0xff6600, 0.30, Math.PI / 3, 1.2);
```

**Velocidades de rotación** (en la función `animate`):
```js
ring1.rotation.z += 0.003;   // velocidad anillo 1
ring2.rotation.z -= 0.002;   // negativo = sentido contrario
ring3.rotation.y += 0.0015;  // rota en eje Y en vez de Z
```

---

## 8. PARTÍCULAS ORBITALES

```js
const orbCount = 500;       // cantidad de partículas
orbRadii[i] = 0.9 + Math.random() * 3.2;   // distancia min-max al centro
orbSpeeds[i] = (Math.random() * 0.003 + 0.0005) * (Math.random() > 0.5 ? 1 : -1);
orbY[i] = (Math.random() - 0.5) * 6.0;     // dispersión vertical (-3 a +3)
```

| Variable | Qué controla |
|---|---|
| `orbCount` | Número total de partículas. Más = más denso (y más CPU). Máximo razonable: ~800 |
| `0.9 + Math.random() * 3.2` | Radio mínimo (`0.9`) y máximo (`0.9+3.2 = 4.1`). Controla desde dónde hasta dónde llegan las partículas |
| `* 6.0` | Dispersión vertical total. `6.0` = desde -3 hasta +3 unidades. Aumentar llena más arriba y abajo |
| `0.003 + 0.0005` | Velocidad máxima y mínima de cada partícula |

### Material de partículas
```js
const orbMat = new THREE.PointsMaterial({
    color: 0x22d3ee,   // color
    size: 0.038,       // tamaño de cada punto
    transparent: true,
    opacity: 0.85
});
```
- `size`: tamaño de cada partícula en unidades de mundo 3D. `0.01` = microscópico, `0.1` = muy grande.
- `color`: color uniforme de todas las partículas.

### Pulso de opacidad (animación)
```js
orbMat.opacity = 0.55 + Math.sin(t * 2.5) * 0.3;
```
- Esto hace que las partículas "parpadeen" suavemente.
- `0.55` = opacidad base
- `0.3` = amplitud del parpadeo (sube y baja 0.3)
- `2.5` = velocidad del pulso. Más alto = parpadeo más rápido
- Para desactivar el parpadeo: `orbMat.opacity = 0.85;`

---

## 9. LUCES

```js
const keyLight = new THREE.DirectionalLight(0x88ddff, 1.2);
keyLight.position.set(4, 2, 4);              // posición X, Y, Z

const rimLight = new THREE.DirectionalLight(0x22d3ee, 0.3);
rimLight.position.set(-4, -1, -3);           // luz de relleno desde atrás

scene.add(new THREE.AmbientLight(0x0a1520, 5.0)); // luz ambiente
```

| Luz | Tipo | Color | Intensidad | Efecto |
|---|---|---|---|---|
| `keyLight` | `DirectionalLight` | `#88ddff` (azul claro) | `1.2` | Luz principal, viene de arriba-derecha-frente |
| `rimLight` | `DirectionalLight` | `#22d3ee` (cyan) | `0.3` | Contraluz desde atrás, da profundidad |
| Ambiente | `AmbientLight` | `#0a1520` (azul muy oscuro) | `5.0` | Iluminación uniforme base, sin sombras |

**Consejos:**
- `DirectionalLight` no tiene posición real en el mundo, solo **dirección**. `position.set(4,2,4)` = la luz viene *desde* ese punto apuntando al origen.
- Si el `shininess` del material es alto, la `keyLight` producirá un círculo duro visible. Mantenlo bajo (`< 15`).
- Sube la `AmbientLight` para que la esfera se vea más pareja sin sombras fuertes.

---

## 10. ANIMACIÓN Y CONTROL DEL RATÓN

```js
earth.rotation.y += 0.0012 + targetX;       // velocidad de rotación Y + influencia del ratón
earth.rotation.x += (targetY - earth.rotation.x) * 0.03;  // suavizado en X
```

| Variable | Efecto |
|---|---|
| `0.0012` | Velocidad de auto-rotación. `0.005` = gira mucho más rápido |
| `mouseX * 0.012` | Cuánto afecta el movimiento horizontal del ratón a la rotación. Sube para más reactividad |
| `mouseY * 0.25` | Cuánto afecta el movimiento vertical del ratón a la inclinación |
| `* 0.04` y `* 0.03` | Factor de suavizado (lerp). Más bajo = respuesta más lenta y suave |

---

## 11. CANVAS Y TAMAÑO DEL PANEL

```js
const W = Math.round(window.innerWidth * 0.45);  // 45% del ancho de pantalla
const H = window.innerHeight;                     // altura completa
```
- `0.45` corresponde al `width: 45vw` del CSS del `#globeCanvas`.
- Si cambias el ancho en CSS, actualiza este valor también para que el renderer tenga el aspect ratio correcto.

---

## 12. COLORES DE REFERENCIA usados en el proyecto

| Hex | RGB | Nombre |
|---|---|---|
| `0x22d3ee` / `#22d3ee` | `34, 211, 238` | Cyan principal (Tailwind `cyan-400`) |
| `0x60a5fa` / `#60a5fa` | `96, 165, 250` | Azul claro (Tailwind `blue-400`) |
| `0x88ddff` / `#88ddff` | `136, 221, 255` | Azul muy claro (keyLight) |
| `0x0a1520` / `#0a1520` | `10, 21, 32` | Azul muy oscuro (fondo / emissive) |
| `0x0d6677` / `#0d6677` | `13, 102, 119` | Cyan oscuro (specular suave) |

Para cambiar toda la paleta a **verde** por ejemplo, reemplaza `0x22d3ee` → `0x00ff88` en toda la escena.

---

## Recetas rápidas

### Esfera más grande
```js
// Cambia todos los radios proporcionalmente
SphereGeometry(1.0, ...)   // esfera
SphereGeometry(1.001, ...) // wireframe
SphereGeometry(1.06, ...)  // atmósfera
SphereGeometry(1.18, ...)  // halo
makeRing(1.32, ...)        // anillo 1
makeRing(1.48, ...)        // anillo 2
makeRing(1.62, ...)        // anillo 3
```

### Más partículas y más dispersas
```js
const orbCount = 800;
orbRadii[i] = 1.2 + Math.random() * 5.0;
orbY[i] = (Math.random() - 0.5) * 10.0;
```

### Rotación más rápida
```js
earth.rotation.y += 0.005 + targetX;
```

### Sin reactividad al ratón
```js
// Eliminar o poner a 0 la influencia del mouse:
mouseX = 0; mouseY = 0;
```

### Partículas más brillantes y sin parpadeo
```js
const orbMat = new THREE.PointsMaterial({
    color: 0x22d3ee, size: 0.05,
    transparent: true, opacity: 1.0
});
// En animate(), eliminar la línea del pulso:
// orbMat.opacity = 0.55 + Math.sin(t * 2.5) * 0.3;
```
