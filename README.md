# Eventos-Online.github.io
Pedir canciones
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Pedir Canción</title>
  <script src="https://www.gstatic.com/firebasejs/10.8.1/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.8.1/firebase-firestore-compat.js"></script>
  <style>
    body { font-family: sans-serif; padding: 20px; max-width: 600px; margin: auto; }
    input, button, textarea { padding: 8px; margin: 5px 0; width: 100%; }
    .admin-controls { margin-top: 20px; border-top: 1px solid #ccc; padding-top: 20px; }
    .peticion { border: 1px solid #ddd; padding: 10px; margin: 5px 0; }
    .validada { background-color: #d1ffd1; }
    .no-validada { background-color: #ffd1d1; }
    #estadoPeticion { margin-top: 10px; font-weight: bold; }
    .respuesta { font-style: italic; margin-top: 5px; color: #444; }
  </style>
</head>
<body>
  <h1>🎶 Pide una canción</h1>

  <div id="formulario">
    <input type="text" id="cancionInput" placeholder="Nombre de la canción" />
    <button onclick="enviarPeticion()">Enviar</button>
    <div id="estadoPeticion"></div>
  </div>

  <div class="admin-controls">
    <h2>🔐 Modo Administrador</h2>
    <input type="password" id="contrasenaInput" placeholder="Contraseña admin" />
    <button onclick="activarAdmin()">Entrar</button>
  </div>

  <div id="adminPanel" style="display:none;">
    <h3>Peticiones recibidas</h3>
    <div id="listaPeticiones"></div>
  </div>

  <script>
    // Reemplaza con tu configuración de Firebase
    const firebaseConfig = {
      apiKey: "TU_API_KEY",
      authDomain: "TU_AUTH_DOMAIN",
      projectId: "TU_PROJECT_ID",
      storageBucket: "TU_STORAGE_BUCKET",
      messagingSenderId: "TU_MESSAGING_SENDER_ID",
      appId: "TU_APP_ID"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();
    const peticionesRef = db.collection("peticiones");

    const estadoDiv = document.getElementById("estadoPeticion");

    function enviarPeticion() {
      const cancion = document.getElementById("cancionInput").value.trim();
      if (cancion === "") return alert("Por favor, escribe una canción.");
      peticionesRef.add({ cancion, validada: false, respuesta: "" }).then(docRef => {
        localStorage.setItem("peticionID", docRef.id);
        escucharEstadoUsuario(docRef.id);
      });
      document.getElementById("cancionInput").value = "";
    }

    function escucharEstadoUsuario(id) {
      peticionesRef.doc(id).onSnapshot(doc => {
        const data = doc.data();
        if (!data) return;
        let mensaje = "";
        if (data.validada === true) {
          mensaje = "✅ ¡Tu canción ha sido aceptada!";
          estadoDiv.style.color = "green";
        } else if (data.validada === false) {
          mensaje = "❌ Tu canción no ha sido aceptada aún.";
          estadoDiv.style.color = "red";
        }
        estadoDiv.innerText = mensaje;
        if (data.respuesta) {
          estadoDiv.innerHTML += `<div class='respuesta'>💬 ${data.respuesta}</div>`;
        }
      });
    }

    const peticionGuardada = localStorage.getItem("peticionID");
    if (peticionGuardada) {
      escucharEstadoUsuario(peticionGuardada);
    }

    function activarAdmin() {
      const pass = document.getElementById("contrasenaInput").value;
      if (pass === "admin123") {
        document.getElementById("adminPanel").style.display = "block";
        escucharPeticiones();
      } else {
        alert("Contraseña incorrecta.");
      }
    }

    function escucharPeticiones() {
      peticionesRef.onSnapshot(snapshot => {
        const contenedor = document.getElementById("listaPeticiones");
        contenedor.innerHTML = "";
        snapshot.forEach(doc => {
          const data = doc.data();
          const div = document.createElement("div");
          div.className = "peticion " + (data.validada ? "validada" : "no-validada");

          div.innerHTML = `
            <strong>${data.cancion}</strong>
            <div style="margin: 5px 0;">
              <button onclick="validar('${doc.id}', true)">✅</button>
              <button onclick="validar('${doc.id}', false)">❌</button>
              <button onclick="eliminar('${doc.id}')">🗑️ Eliminar</button>
            </div>
            <textarea placeholder="Respuesta personalizada..." onchange="responder('${doc.id}', this.value)">${data.respuesta || ""}</textarea>
          `;
          contenedor.appendChild(div);
        });
      });
    }

    function validar(id, estado) {
      peticionesRef.doc(id).update({ validada: estado });
    }

    function eliminar(id) {
      if (confirm("¿Estás seguro de eliminar esta petición?")) {
        peticionesRef.doc(id).delete();
      }
    }

    function responder(id, texto) {
      peticionesRef.doc(id).update({ respuesta: texto });
    }
  </script>
</body>
</html>
