<template>
  <div class="takt">
    <div class="container">
      <h1 class="title text-3xl font-bold text-blue-600 mb-4">🔧 Monitor de Takt Time</h1>

      <div v-if="!alarmActive" class="waiting text-lg text-gray-600">Aguardando sinal de alarme...</div>

      <div v-else class="alarm">
        🚨 <strong>{{ celula }}</strong> atingiu o Takt Time!
      </div>
    </div>
  </div>
  <div class="watermark">DASS</div>
  <div class="marca-dagua">DASS</div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const celula = ref("");
const alarmActive = ref(false);
// const tempoExibicao = 10000 // milissegundos (ex: 10s)

function tocarAlarme() {
  const audio = new Audio("/alarme.mp3");
  audio.play();
}

const ws = ref(null);

const connect = () => {
  ws.value = new WebSocket("ws://10.110.30.222:3043");

  ws.value.onopen = () => {
    console.log("WebSocket conectado");
    const registerData = JSON.stringify({
      type: "register",
      payload: {
        id: "fab2-cel-2308",
      },
    });
    ws.value.send(registerData);
  };

  ws.value.onmessage = (event) => {
    console.log("Mensagem recebida:", event.data);
    alarmActive.value = true;
    tocarAlarme();
  };

  ws.value.onclose = () => {
    console.log("WebSocket desconectado");
  };

  ws.value.onerror = (error) => {
    console.error("WebSocket erro:", error);
  };
};

onMounted(() => {
  connect();
});

// Simulação do recebimento do sinal externo (substitua depois)
// function simularSinalRecebido(nomeCelula) {
//     celula.value = nomeCelula
//     alarmActive.value = true
//     // tocarAlarme()

//     // Após X segundos, limpa a tela
//     setTimeout(() => {
//         alarmActive.value = false
//         celula.value = ''
//     }, tempoExibicao)
// }

// Simula sinais recebidos (para teste)
// Substitua isso depois por WebSocket, API, MQTT, etc
// setTimeout(() => simularSinalRecebido("Célula Costura"), 3000)
// setTimeout(() => simularSinalRecebido("Célula Montagem"), 20000)

// function tocarAlarme() {
//   const audio = new Audio('/alarm.mp3')
//   audio.play()
// }
</script>

<style scoped>
@keyframes pulsar {
  0% {
    transform: scale(1);
    opacity: 1;
  }

  50% {
    transform: scale(1.1);
    opacity: 0.7;
  }

  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.container.ativo .conteudo {
  animation: pulsar 1s infinite;
}

.container {
  border: yellow 8px solid;
}

.marca-dagua {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 120px;
  font-weight: bold;
  color: rgba(0, 0, 0, 0.05);
  z-index: 0;
  pointer-events: none;
  user-select: none;
}

.takt {
  min-height: 100vh;
  background-color: #e92412d2;
  display: flex;
  justify-content: center;
  align-items: center;
  font-family: "Segoe UI", sans-serif;
}

.container {
  background: white;
  padding: 40px;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  text-align: center;
  width: 500px;
}

.title {
  font-size: 26px;
  color: #222;
  margin-bottom: 30px;
}

.waiting {
  font-size: 18px;
  color: #555;
}

.alarm {
  font-size: 45px;
  color: #d30202;
  font-weight: bold;
  animation: pulse 1s infinite;
}

.watermark {
  position: fixed;
  bottom: 20px;
  right: 20px;
  font-size: 18px;
  color: rgb(238, 235, 235);
  font-weight: bold;
  user-select: none;
  pointer-events: none;
}

@keyframes pulse {
  0% {
    opacity: 1;
    transform: scale(1);
  }

  50% {
    opacity: 0.5;
    transform: scale(1.05);
  }

  100% {
    opacity: 1;
    transform: scale(1);
  }
}
</style>
