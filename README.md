<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>🍹Determination Drinks - Encuesta Undertale🍹</title>

<!-- Fuente estilo pixel art -->
<link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet" />

<style>
  body {
    margin: 0;
    padding: 20px;
    font-family: "Press Start 2P", monospace;
    color: white;
    background: url("images/fondo.jpg") no-repeat center center fixed;
    background-size: cover;
    text-align: center;
  }
  form {
    background: rgba(0,0,0,0.75);
    border: 2px solid white;
    border-radius: 12px;
    max-width: 400px;
    margin: 30px auto;
    padding: 25px;
  }
  label {
    display: block;
    margin: 15px 0 8px;
    font-size: 11px;
  }
  select, textarea, input[type="text"] {
    width: 100%;
    padding: 10px;
    font-family: inherit;
    font-size: 12px;
    background: #000;
    color: #fff;
    border: 2px solid white;
    border-radius: 5px;
    resize: none;
    box-sizing: border-box;
  }
  textarea {
    font-family: monospace;
  }
  button {
    width: 100%;
    padding: 12px;
    margin-top: 20px;
    font-family: inherit;
    font-size: 14px;
    background: white;
    color: black;
    border: 2px solid black;
    border-radius: 5px;
    cursor: pointer;
  }
  button:hover {
    background: black;
    color: white;
    border-color: white;
  }
  .mensaje {
    display: none;
    margin-top: 20px;
    font-size: 14px;
    color: #0f0;
  }
</style>

</head>

<body>
  <h1>Determination Drinks 💗</h1>

  <form id="miForm">
    <label>1. ¿Qué tipo de endulzante preferís?</label>
    <select name="endulzante" required>
      <option value="" disabled selected>-- Elegí una opción --</option>
      <option value="azucar">Azúcar</option>
      <option value="miel">Miel</option>
      <option value="edulcorante">Edulcorante</option>
      <option value="otro">Otro</option>
    </select>

    <textarea name="otroEndulzante" rows="1" placeholder="Si elegiste otro, ¿cuál?..."></textarea>

    <label>2. ¿Qué frutas te gustan más en un licuado?</label>
    <textarea name="frutas" rows="3" placeholder="Ej: banana, frutilla, mango..." required></textarea>

    <label>3. ¿Cuánto pagarías por un licuado?</label>
    <select name="precio" required>
      <option value="" disabled selected>-- Elegí un rango --</option>
      <option value="$700 - $1000">$700 - $1000</option>
      <option value="$1000 - $1500">$1000 - $1500</option>
      <option value="menos de $1800">menos de $1800</option>
    </select>

    <label>4. ¿Te llamaría la atención si la persona que atiende hace cosplay de la temática?</label>
    <select name="cosplay" required>
      <option value="" disabled selected>Seleccionar</option>
      <option value="sí">Sí</option>
      <option value="no">No</option>
      <option value="depende">Depende</option>
    </select>

    <button type="submit">Enviar Respuestas</button>
  </form>

  <p class="mensaje" id="mensajeFinal">* Gracias por tu respuesta, humano.</p>

  <!-- Audio de envío -->
  <audio id="submit-sound" src="sound/savepoint.mp3" preload="auto"></audio>
  <!-- Audio de tipeo -->
  <audio id="typing-sound" src="sound/typing.mp3" preload="auto"></audio>

<script type="module">
  import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-app.js";
  import { getFirestore, doc, getDoc, setDoc, updateDoc, increment } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-firestore.js";

  const firebaseConfig = {
    apiKey: "AIzaSyD1djuXf4a5y9uucZEHBrL_HWgWr0PqH3M",
    authDomain: "encuestaundertale.firebaseapp.com",
    projectId: "encuestaundertale",
    storageBucket: "encuestaundertale.firebasestorage.app",
    messagingSenderId: "34519487798",
    appId: "1:34519487798:web:7d53e84d8551693cf84f97",
    measurementId: "G-QMTMF22EHY"
  };

  const app = initializeApp(firebaseConfig);
  const db = getFirestore(app);

  const form = document.getElementById("miForm");
  const audio = document.getElementById("submit-sound");
  const typingSound = document.getElementById("typing-sound");
  const mensajeFinal = document.getElementById("mensajeFinal");
  const usuarioVotoKey = "usuarioYaVoto";

  form.addEventListener("submit", async (e) => {
    e.preventDefault();

    if(localStorage.getItem(usuarioVotoKey)) {
      alert("Ya has votado. Gracias por tu participación.");
      return;
    }

    const formData = new FormData(form);
    const endulzante = formData.get("endulzante");
    const otroEndulzante = formData.get("otroEndulzante");
    const frutas = formData.get("frutas");
    const precio = formData.get("precio");
    const cosplay = formData.get("cosplay");

    if(!endulzante || !frutas || !precio || !cosplay) {
      alert("Por favor completa todas las preguntas obligatorias.");
      return;
    }

    audio.currentTime = 0;
    audio.play();

    try {
      const votosRef = doc(db, 'votos', 'endulzante');
      const docSnap = await getDoc(votosRef);

      if (!docSnap.exists()) {
        // Crear documento con conteos iniciales
        await setDoc(votosRef, {
          azucar: 0,
          miel: 0,
          edulcorante: 0,
          otro: 0
        });
      }

      // Actualizar el conteo del endulzante seleccionado
      await updateDoc(votosRef, {
        [endulzante]: increment(1)
      });

      // Aquí podrías guardar también frutas, precio, cosplay en otros documentos/colecciones

      localStorage.setItem(usuarioVotoKey, "true");
      mensajeFinal.style.display = "block";
      form.reset();

    } catch (error) {
      console.error("Error guardando voto:", error);
      alert("Hubo un error al enviar tu voto, intenta nuevamente.");
    }
  });

  let typingTimeout;

  function playTypingSound() {
    typingSound.currentTime = 0;
    typingSound.play();
  }

  function stopTypingSound() {
    typingSound.pause();
  }

  form.querySelectorAll("input[type=text], textarea, select").forEach(input => {
    input.addEventListener("keydown", () => {
      if (typingSound.paused) {
        playTypingSound();
      }
      clearTimeout(typingTimeout);
      typingTimeout = setTimeout(() => {
        stopTypingSound();
      }, 300);
    });
  });

</script>

</body>
</html>
