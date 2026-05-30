# 🤖 Guía de Desarrollo para Agentes IA (AGENTS.md)

Este documento es una guía de referencia rápida para agentes de Inteligencia Artificial que trabajen en este repositorio. Su objetivo es evitar que tengas que analizar todo el código desde cero, prevenir que reinventes la rueda y asegurar la coherencia del diseño y arquitectura del proyecto.

---

## 📌 Filosofía y Reglas del Proyecto

1. **Mantén la Simplicidad (Single-Page Application)**:
   * Todo el núcleo de la interfaz, estilos CSS y lógica JS están autocontenidos en [index.html](file:///home/nicorz/IdeaProjects/fulvito/public/index.html).
   * **NO** dividas el archivo en múltiples componentes React/Vue/Svelte ni agregues herramientas de compilación complejas (Webpack, Vite, Babel) a menos que el usuario lo solicite explícitamente.
2. **Estilo CSS Puro (Vanilla CSS)**:
   * El diseño utiliza CSS puro con variables globales `:root` definidas al inicio de [index.html](file:///home/nicorz/IdeaProjects/fulvito/public/index.html).
   * **NO** uses Tailwind CSS, Bootstrap u otros frameworks a menos que el usuario te lo pida e indique la versión exacta.
   * Respeta la estética premium y futurista: Glassmorphism (`backdrop-filter`), degradados vivos en el botón de grabado y colores armoniosos oscuros.
3. **No Comprometas Secretos**:
   * Las credenciales de Firebase se importan desde un archivo separado: [firebase-config.js](file:///home/nicorz/IdeaProjects/fulvito/public/firebase-config.js). Este archivo está configurado en `.gitignore` para evitar fugas de información.
   * **NUNCA** incrustes claves de API ni secrets directamente en Git ni en `index.html`.
4. **Manejo del Sistema de Logs**:
   * Cualquier proceso de inicialización, cambio de estado, error o descarga debe dejar registro en la consola lateral de logs visuales de la interfaz utilizando la función Javascript `addLog(message)`. Esto es vital para la depuración en vivo por parte del usuario.

---

## 🛠️ Arquitectura y Flujos Clave

### 🔍 1. Zoom e Interactividad de la Cámara (Viewport)
* **Objetivo**: Permitir al usuario ampliar el video para analizar jugadas al detalle (por ejemplo, en jugadas de fútbol).
* **Fórmula de Renderizado**: El zoom es puramente visual mediante transformaciones CSS aplicadas sobre `#main-video`.
  ```javascript
  video.style.transform = `translate(${zoomX}px, ${zoomY}px) scale(${zoomScale})`;
  ```
* **Controladores**:
  * Evento `wheel` en el viewport para controlar `zoomScale` (límite `1.0x` a `5.0x`).
  * Eventos de arrastre del ratón (`mousedown`/`mousemove`/`mouseup`) y táctiles (`touchstart`/`touchmove`/`touchend`) para modificar las coordenadas de paneo `zoomX` y `zoomY`.
  * La función `resetZoom()` limpia los offsets y restaura la escala a `1.0x`.
* **Guía para cambios**: Cualquier cambio en el reproductor debe asegurar que las coordenadas de zoom y pan se reseteen al cargar un video nuevo o al cambiar el tamaño del viewport.

### ✂️ 2. Lógica de Recorte con un Solo Botón (Grabador de Clip)
* **Estado del Intervalo**: Controlado por variables globales `clipStart` (milisegundo de inicio) y `clipEnd` (milisegundo de fin).
* **Lógica del Botón Único (`#record-btn`)**:
  * **Bandera `isRecordingState`**: Si es falsa, el primer clic inicia la grabación virtual (`clipStart = video.currentTime`), arranca el temporizador visual (`recordingTimer`) y cambia la apariencia del botón a modo de grabación parpadeante. Si es verdadera, el segundo clic marca `clipEnd = video.currentTime`, limpia los temporizadores y dispara automáticamente la función de exportación (`triggerAutomaticExport()`).
* **Línea de Tiempo**: `#timeline-start-marker`, `#timeline-end-marker` y `#timeline-range-highlight` se posicionan dinámicamente en base a porcentajes sobre la duración total del video (`(time / video.duration) * 100`).

### 📦 3. Motor Dual de Exportación
Al finalizar el marcado de límites, se ejecuta `triggerAutomaticExport()` que evalúa el método elegido en el selector de la interfaz (`#export-method`):

#### Método A: FFmpeg.wasm (WebAssembly Lossless)
* **Cómo funciona**: Escribe el archivo original en el sistema de archivos virtual en memoria de WebAssembly (`FS('writeFile')`), ejecuta el recorte a nivel binario copiando los tracks (`-c copy`) lo cual no altera los codecs ni degrada la calidad, lee el archivo procesado en memoria y limpia el sistema virtual (`FS('unlink')`).
* **Código Clave**:
  ```javascript
  ffmpegInstance.FS('writeFile', inputName, await fetchFile(currentFile));
  await ffmpegInstance.run('-ss', clipStart.toFixed(3), '-i', inputName, '-t', duration.toFixed(3), '-c', 'copy', outputName);
  const data = ffmpegInstance.FS('readFile', outputName);
  ```
* **Consideraciones**: Requiere aislamiento de origen cruzado (COOP/COEP) en el servidor para habilitar `SharedArrayBuffer`. Si falla su carga, el código realiza una captura de excepción y conmuta automáticamente a MediaRecorder.

#### Método B: MediaRecorder (Navegador Nivel 2 Fallback)
* **Cómo funciona**: Captura el stream del elemento video mediante `video.captureStream()`, sintoniza el video a `clipStart`, inicia un codificador nativo (`MediaRecorder`), silencia temporalmente el audio y reproduce el video a velocidad 1x hasta llegar a `clipEnd`, momento en el que detiene el grabador y genera el Blob.
* **Consideraciones**: Este método re-codifica el video en tiempo real. Es más lento y depende de las capacidades del navegador para codificar en caliente (`video/mp4` o `video/webm`).

---

## 📋 Lista de Tareas Comunes para Agentes

Si vas a agregar una nueva funcionalidad, comprueba esta lista primero:

* [ ] **¿Añadirás un nuevo botón de control?** Ubícalo dentro de `#player-wrapper` en la sección del layout que corresponda (`controls-left`, `controls-center` o `controls-right`) para no romper el grid responsivo de móviles.
* [ ] **¿Vas a modificar lógica de carga de FFmpeg?** Recuerda mantener la lógica de fallback hacia el CDN de `unpkg.com` en caso de que los archivos locales no se carguen o den un timeout.
* [ ] **¿Actualizaste la interfaz?** Valida la visualización en pantallas móviles simulando una resolución menor a 768px. Los controles de video deben apilarse y reordenarse fluidamente.
* [ ] **¿Se añadieron dependencias en package.json?** Registra los cambios e inicia el servidor local mediante `npm run dev` para corroborar que no haya conflictos de CORS o de carga de recursos locales.
