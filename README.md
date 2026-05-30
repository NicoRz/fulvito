# ⚽ Fulvito Video Clipper - Editor de Video Local

Una aplicación web de una sola página (SPA) premium, fluida y moderna diseñada para recortar videos en formato MP4 de manera **100% local y segura**. La aplicación aprovecha la potencia de WebAssembly a través de **FFmpeg.wasm** para realizar recortes binarios sin pérdida de calidad de forma ultra rápida, incorporando a su vez un sistema de grabación interactivo de respaldo mediante **MediaRecorder** de HTML5.

---

## 🚀 Características Principales

### 1. Carga de Archivos Instantánea
* **Drag & Drop**: Arrastra cualquier archivo de video `.mp4` directamente al área de trabajo.
* **Selector Local**: Botón estilizado para buscar y seleccionar videos desde la interfaz de archivos del sistema operativo.
* El archivo se procesa enteramente en el navegador del usuario utilizando `URL.createObjectURL`, garantizando total privacidad (el video nunca se sube a servidores externos).

### 2. Zoom & Pan Cinemático (Interactivo)
* **Zoom Dinámico**: Ampliación de hasta **5.0x** utilizando la rueda del ratón (scroll wheel) o mediante gestos multitáctiles (pinch-to-zoom) en dispositivos móviles.
* **Arrastre y Desplazamiento (Pan)**: Al hacer zoom, puedes hacer clic y arrastrar (o deslizar con un dedo en móviles) para desplazar el encuadre y analizar detalles del video con precisión.
* **Restablecimiento Rápido**: Doble clic sobre el viewport de video o un botón flotante con diseño premium restablece instantáneamente la escala a `1.0x`.
* **Nota Crítica**: El zoom es puramente visual para facilitar el análisis del contenido (por ejemplo, jugadas de fútbol). El clip exportado mantendrá la resolución y relación de aspecto original (1x) sin pérdida alguna de resolución por re-encuadre.

### 3. Sistema de Recorte de Un Solo Botón (Grabar Clip)
* **Flujo Simplificado**: Un único botón central de grabación inspirado en cámaras de filmación profesionales.
  * **Primer Clic**: Marca el punto de inicio del clip (`clipStart`) basándose en el segundo actual del video.
  * **Segundo Clic**: Marca el punto de finalización (`clipEnd`) y detiene la grabación virtual del fragmento.
* **Visualización en Línea de Tiempo**: Se resaltan los marcadores de inicio (color verde), fin (color rojo) y el fragmento seleccionado en la barra de progreso utilizando gradientes.
* **Exportación Automática**: Nada más marcar el punto final, la aplicación inicia automáticamente el procesamiento y la descarga directa del clip resultante.

### 4. Motor Dual de Exportación (Procesamiento Inteligente)
La aplicación cuenta con una arquitectura de procesamiento redundante para asegurar compatibilidad en todo tipo de navegadores y condiciones de conectividad:
* **FFmpeg.wasm (Predeterminado - Alto Rendimiento)**:
  * Utiliza WebAssembly para ejecutar comandos nativos de FFmpeg en el hilo del navegador.
  * Comando utilizado: `-ss [inicio] -i input.mp4 -t [duración] -c copy output.mp4`.
  * El argumento `-c copy` realiza una copia directa de los streams de audio y video. **No hay re-codificación**, lo que se traduce en una velocidad asombrosa y preservación absoluta de la calidad original.
* **MediaRecorder (Fallback / Respaldo)**:
  * Si el motor de FFmpeg falla al cargar o el navegador no lo soporta (por políticas de cabeceras COOP/COEP o falta de soporte WebAssembly), el sistema conmuta automáticamente al método MediaRecorder.
  * Graba el flujo de video en tiempo real (`captureStream()`) reproduciendo el fragmento silenciosamente a velocidad `1x`.
  * Genera un archivo codificado en `video/mp4` o `video/webm` según soporte del navegador.

### 5. Panel de Control Lateral (Configuración y Telemetría)
* **Selector de Método**: Permite alternar manualmente entre FFmpeg y MediaRecorder para pruebas.
* **Intervalo del Clip**: Desglose con precisión de milisegundos del inicio, final y duración exacta del fragmento a exportar.
* **Instrucciones Integradas**: Una guía visual rápida sobre cómo interactuar con los gestos de zoom y pan.
* **Registro de Consola Integrado (Console Logs)**: Una caja de logs interactiva que muestra el historial técnico de lo que ocurre tras bambalinas (carga de FFmpeg, comandos ejecutados, eventos de error, etc.).

### 6. Firebase y Métricas
* Integra **Firebase Analytics** para trackear el comportamiento del usuario y uso del sistema (eventos como carga de videos, velocidad de reproducción, inicio/fin de recortes, éxitos y fallos en las exportaciones).
* El despliegue está configurado para la plataforma global de **Firebase Hosting**.

---

## 📁 Estructura del Proyecto

El proyecto está estructurado de manera minimalista y ultra-eficiente como una Single Page Application (SPA), eliminando la necesidad de empaquetadores complejos como Webpack o Vite:

```bash
├── .agents/                    # Recursos y configuraciones específicas de agentes IA
├── .firebase/                  # Caché local y archivos temporales de Firebase
├── .firebaserc                 # Vinculación del proyecto con la ID de Firebase (fulvito-93)
├── firebase.json               # Configuración de Firebase Hosting y reglas de redirección
├── package.json                # Definición de dependencias de desarrollo y scripts npm
├── skills-lock.json            # Estado de los plugins/skills del entorno de desarrollo
└── public/                     # Directorio principal del servidor web público
    ├── index.html              # Frontend único (Estructura HTML5, estilos CSS3 y código JS)
    ├── firebase-config.js      # Configuración de credenciales de Firebase (Ignorado en Git)
    ├── ffmpeg.min.js           # Envoltura JS de FFmpeg.wasm para uso offline
    ├── ffmpeg-core.js          # Cargador WebAssembly de FFmpeg
    └── ffmpeg-core.wasm        # Binario de WebAssembly con el motor FFmpeg (24MB)
```

---

## 🛠️ Detalle de los Archivos del Servidor (`public/`)

### 📦 `public/index.html`
Es el núcleo central de la aplicación. Contiene:
* **Estructura HTML5**: Layout cinemático con cabecera (Slim Header), visor de video centrado, controles inferiores unificados, notificaciones dinámicas (Toast) y el modal de progreso.
* **Diseño y Estilos (CSS)**:
  * Sistema visual moderno y oscuro con variables custom (`:root`).
  * Efectos premium de desenfoque de fondo (**Glassmorphism**) mediante `backdrop-filter`.
  * Degradados fluidos (`linear-gradient`) y animaciones interactivas (vibración de botón de grabación, pulso de estado).
  * Adaptabilidad completa (**Mobile-Responsive**) mediante Media Queries, reorganizando los botones y paneles en móviles.
* **Lógica JS (JavaScript Vanilla)**:
  * Control del reproductor de video HTML5, volumen, velocidad de reproducción y pantalla completa personalizada en ventana.
  * Listeners interactivos y fórmulas matemáticas de coordenadas para realizar Zoom y Pan mediante eventos `wheel`, `mousedown`/`mousemove` y gestos de toques `touchstart`/`touchmove`.
  * Lógica de grabación de clips con temporizadores y actualización dinámica de elementos del DOM.
  * Inicialización de Firebase Analytics cargado asíncronamente.
  * Lógica de descarga automatizada de archivos generados.

### ⚙️ `public/firebase-config.js`
Archivo de configuración que expone el objeto global `firebaseConfig` con las API keys del proyecto `fulvito-93`. Este archivo se encuentra excluido de git por motivos de seguridad en producción y uso del *GitHub Secret Scanner*.

### ⚡ Binarios Locales (`ffmpeg*`)
La inclusión directa de `ffmpeg.min.js`, `ffmpeg-core.js` y `ffmpeg-core.wasm` en la carpeta pública garantiza que la aplicación pueda inicializar su motor de video de forma local, disminuyendo los tiempos de carga de red y ofreciendo una experiencia robusta incluso en entornos offline o con conexiones limitadas.

---

## 💻 Ejecución y Despliegue

### Requisitos Previos
Tener instalado Node.js y npm en tu máquina local.

### Ejecución en Local
1. Clona el repositorio y sitúate en el directorio raíz.
2. Levanta el servidor web local ejecutando:
   ```bash
   npm run dev
   ```
   *Esto iniciará un servidor en [http://localhost:3000](http://localhost:3000) apuntando a la carpeta `/public` usando `http-server`.*

### Despliegue en Firebase Hosting
1. Asegúrate de tener instalado e inicializado el CLI de Firebase (`npm install -g firebase-tools`).
2. Haz login en tu cuenta:
   ```bash
   firebase login
   ```
3. Despliega la aplicación directamente ejecutando:
   ```bash
   firebase deploy
   ```
   *El sitio estará disponible públicamente en la URL configurada en Firebase (por ejemplo, [https://fulvito-93.web.app](https://fulvito-93.web.app)).*

---

## 📝 Notas Técnicas y Seguridad

* **Cross-Origin Isolation**: Para que FFmpeg.wasm funcione correctamente de manera local en navegadores modernos, el servidor web (Firebase o local) debe retornar las cabeceras HTTP de aislamiento de origen cruzado:
  * `Cross-Origin-Opener-Policy: same-origin`
  * `Cross-Origin-Embedder-Policy: require-corp`
  Estas reglas aseguran que `SharedArrayBuffer` (requerido por WebAssembly) esté disponible. Firebase Hosting las maneja automáticamente según la configuración especificada.
* **Fallback a CDN**: En caso de que el cargador local falle por problemas de carga de archivos de gran tamaño (el archivo .wasm pesa 24MB), el cargador de JS incluye una lógica de redundancia que descarga los recursos alternativos desde el CDN de `unpkg.com` garantizando el funcionamiento.
* **Formato MP4**: FFmpeg.wasm está configurado para manejar codecs estándar h264/AAC contenidos en MP4.
