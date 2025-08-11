<template>
  <div :class="['min-h-screen w-full flex flex-col transition-colors duration-500', getBackgroundColor()]">
    <!-- Barra de controles (aparece ao passar o mouse no topo) -->
    <div
      class="fixed top-0 left-0 right-0 z-50 transition-transform duration-300"
      :style="{ transform: showControls ? 'translateY(0)' : 'translateY(-100%)' }"
      @mouseenter="showControls = true"
    >
      <div class="bg-black/80 backdrop-blur-sm p-4 flex justify-between items-center">
        <div class="flex items-center gap-4">
          <span class="text-white">Setor: {{ currentStatus.sectorName }}</span>
          <span class="text-white">Meta: {{ formatTime(targetTime) }}</span>
          <button
            @click="soundEnabled = !soundEnabled"
            class="flex items-center gap-2 bg-white/10 border-white/20 text-white hover:bg-white/20 px-3 py-1 rounded border transition-colors"
          >
            <BellIcon v-if="soundEnabled" class="h-4 w-4" />
            <BellOffIcon v-else class="h-4 w-4" />
            Som {{ soundEnabled ? "Ativo" : "Inativo" }}
          </button>
        </div>

        <div class="flex items-center gap-2">
          <button
            @click="toggleTimer"
            class="flex items-center gap-2 bg-white/10 border-white/20 text-white hover:bg-white/20 px-3 py-1 rounded border transition-colors"
          >
            <PauseIcon v-if="isRunning" class="h-4 w-4" />
            <PlayIcon v-else class="h-4 w-4" />
            {{ isRunning ? "Pausar" : "Iniciar" }}
          </button>

          <button
            @click="resetTimer"
            class="flex items-center gap-2 bg-white/10 border-white/20 text-white hover:bg-white/20 px-3 py-1 rounded border transition-colors"
          >
            <RotateCcwIcon class="h-4 w-4" />
            Reset
          </button>

          <button
            @click="simulateWarning"
            class="flex items-center gap-2 bg-yellow-600/80 border-yellow-500 text-white hover:bg-yellow-600 px-3 py-1 rounded border transition-colors"
          >
            Teste Atenção
          </button>

          <button
            @click="simulateAlarm"
            class="flex items-center gap-2 bg-red-600/80 border-red-500 text-white hover:bg-red-600 px-3 py-1 rounded border transition-colors"
          >
            Teste Alarme
          </button>
        </div>
      </div>
    </div>

    <!-- Área para mostrar controles -->
    <div class="h-4 w-full cursor-pointer" @mouseenter="showControls = true" @mouseleave="showControls = false" />

    <!-- Conteúdo principal -->
    <div class="flex-1 flex flex-col items-center justify-center p-2">
      <!-- Logo/Nome DASS -->
      <div class="mb-3">
        <h1 :class="['text-6xl md:text-7xl lg:text-8xl font-black tracking-wider drop-shadow-lg', getTextColor()]">
          DASS
        </h1>

        <div class="flex flex-row">
          <ClockIcon :class="['h-8 w-8', getTextColor()]" />
          <h2 :class="['text-3xl md:text-4xl font-bold ml-2', getTextColor()]">TAKT TIME</h2>
        </div>
      </div>

      <!-- Indicadores de status -->
      <div class="relative">
        <!-- Server Connection Alert Message -->
        <div
          v-if="!wsConnected"
          class="w-[min(92vw,40rem)] rounded-2xl border border-blue-400/60 bg-white/80 dark:bg-zinc-900/80 backdrop-blur-md shadow-2xl px-6 py-6 sm:px-8 sm:py-8 text-center animate-fade-in"
        >
          <h1
            class="text-3xl sm:text-4xl lg:text-5xl font-extrabold leading-tight tracking-wide text-blue-700 dark:text-blue-300 break-words [text-wrap:balance] mx-auto"
          >
            Estabelecendo Conexão
            <span class="inline-flex align-middle gap-1 ml-2">
              <span class="loading-dot"></span>
              <span class="loading-dot [animation-delay:150ms]"></span>
              <span class="loading-dot [animation-delay:300ms]"></span>
            </span>
          </h1>

          <p class="mt-3 text-sm sm:text-base text-gray-700/90 dark:text-gray-300">Aguardando resposta do servidor…</p>
        </div>

        <div :class="wsConnected ? '' : 'hidden'">
          <!-- Barra Progresso Takt Time -->
          <div class="w-full max-w-4xl mb-8">
            <!-- Barra de progresso -->
            <div class="mt-4">
              <div
                class="bg-gray-200 rounded-lg h-20 md:h-24 lg:h-28 overflow-hidden shadow-lg border-4 border-gray-300"
              >
                <div
                  :class="[
                    'h-full transition-all duration-1000 ease-linear flex items-center justify-center relative',
                    getProgressBarColor(),
                    { 'animate-pulse': currentStatus.level === 'alarm' },
                    { blur: isRunning === false },
                  ]"
                  :style="{ width: `${Math.max(2, progressPercentage)}%` }"
                >
                  <!-- Tempo restante sobre a barra -->
                  <div class="text-white font-mono font-black text-2xl md:text-3xl lg:text-4xl drop-shadow-lg">
                    {{ formatTime(taktTime) }}
                  </div>

                  <!-- Efeito de piscar quando em alarme -->
                  <div
                    v-if="currentStatus.level === 'alarm'"
                    class="absolute inset-0 bg-white opacity-30 animate-ping"
                  />
                </div>
              </div>

              <!-- Informações adicionais da barra -->
              <div v-if="isRunning" class="flex justify-center items-center mt-3">
                <span :class="['text-lg opacity-70', getTextColor()]"> 00:00 </span>
                <span :class="['text-lg font-bold', getTextColor()]">
                  {{ Math.round(progressPercentage) }}% restante
                </span>
                <span :class="['text-lg opacity-70', getTextColor()]">
                  {{ formatTime(targetTime) }}
                </span>
              </div>
            </div>
          </div>

          <!-- Sinalizador Principal -->
          <div
            :class="[
              'w-64 h-64 md:w-80 md:h-80 lg:w-96 lg:h-96 rounded-full border-8 flex flex-col items-center justify-center shadow-2xl transition-all duration-500',
              getBorderColor(),
              {
                'animate-pulse shadow-red-500/50': currentStatus.level === 'alarm',
                'shadow-yellow-500/50': currentStatus.level === 'warning',
              },
            ]"
          >
            <!-- Status -->
            <div
              :class="[
                'text-3xl md:text-4xl lg:text-5xl font-black mb-4',
                getTextColor(),
                { 'animate-bounce': currentStatus.level === 'alarm' },
              ]"
            >
              {{ getStatusText() }}
            </div>

            <!-- Setor -->
            <div :class="['text-xl md:text-2xl lg:text-3xl font-bold mb-6', getTextColor()]">
              {{ currentStatus.sectorName }}
            </div>

            <div
              :class="[
                'w-12 h-12 md:w-16 md:h-16 lg:w-20 lg:h-20 rounded-full shadow-lg',
                {
                  'bg-green-500': currentStatus.level === 'normal',
                  'bg-yellow-600': currentStatus.level === 'warning',
                  'bg-red-700': currentStatus.level === 'alarm',
                  'animate-ping': currentStatus.level === 'alarm',
                },
              ]"
            />
          </div>
        </div>
      </div>
    </div>

    <!-- Rodapé com informações -->
    <div class="p-6 text-center">
      <div :class="['text-sm opacity-60', getTextColor()]">
        DASS - Sistema de Alarmes de Takt Time | Setor: {{ currentStatus.sectorName }} | Timer:
        {{ isRunning ? "Ativo" : "Pausado" }}
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, shallowRef, reactive, computed, onMounted, onUnmounted, watch } from "vue";
import {
  Bell as BellIcon,
  BellOff as BellOffIcon,
  Play as PlayIcon,
  Pause as PauseIcon,
  RotateCcw as RotateCcwIcon,
  Clock as ClockIcon,
} from "lucide-vue-next";
import { API_URL, WS_URL } from "../config/ip";
import alarmSound from "../assets/alarme.wav";

interface SignalStatus {
  level: "normal" | "warning" | "alarm";
  sectorName: string;
  timestamp: Date;
  message: string;
}

// Estados reativos
const currentStatus = reactive<SignalStatus>({
  level: "normal",
  sectorName: "Costura",
  timestamp: new Date(),
  message: "Sistema funcionando normalmente",
});

const taktTime = ref(60); // Tempo atual em segundos
const targetTime = ref(60); // Tempo meta em segundos
const isRunning = ref(false); // Se o timer está rodando
const soundEnabled = ref(true);
const showControls = ref(false);

let timerInterval: number | null = null;

// Computed properties
const progressPercentage = computed(() => (taktTime.value / targetTime.value) * 100);

// Funções helper
const getBackgroundColor = () => {
  switch (currentStatus.level) {
    case "alarm":
      return "bg-red-500";
    case "warning":
      return "bg-yellow-500";
    case "normal":
      return "bg-white";
  }
};

const getTextColor = () => {
  switch (currentStatus.level) {
    case "alarm":
      return "text-white";
    case "warning":
      return "text-black";
    case "normal":
      return "text-black";
  }
};

const getBorderColor = () => {
  switch (currentStatus.level) {
    case "alarm":
      return "border-red-700";
    case "warning":
      return "border-yellow-700";
    case "normal":
      return "border-gray-300";
  }
};

const getStatusText = () => {
  switch (currentStatus.level) {
    case "alarm":
      return "ALARME";
    case "warning":
      return "ATENÇÃO";
    case "normal":
      return "NORMAL";
  }
};

const getProgressBarColor = () => {
  switch (currentStatus.level) {
    case "alarm":
      return "bg-red-600";
    case "warning":
      return "bg-yellow-600";
    case "normal":
      return "bg-green-500";
  }
};

const formatTime = (seconds: number) => {
  const mins = Math.floor(seconds / 60);
  const secs = seconds % 60;
  return `${mins.toString().padStart(2, "0")}:${secs.toString().padStart(2, "0")}`;
};

// Função para tocar som de alarme
const playAlarmSound = () => {
  if (!soundEnabled.value) return;
  const audio = new Audio(alarmSound);
  audio.volume = 0.5;
  audio.play().catch((err) => {
    console.error("Erro ao tocar som de alarme:", err);
  });
};

// Funções de controle
const resetTimer = () => {
  taktTime.value = targetTime.value;
  Object.assign(currentStatus, {
    level: "normal",
    sectorName: currentStatus.sectorName,
    timestamp: new Date(),
    message: "Sistema funcionando normalmente",
  });
};

const toggleTimer = () => {
  isRunning.value = !isRunning.value;
};

const simulateAlarm = () => {
  taktTime.value = 0;
  Object.assign(currentStatus, {
    level: "alarm",
    sectorName: currentStatus.sectorName,
    timestamp: new Date(),
    message: "TESTE: Simulação de alarme ativada",
  });
  playAlarmSound();
};

const simulateWarning = () => {
  taktTime.value = 30;
  Object.assign(currentStatus, {
    level: "warning",
    sectorName: currentStatus.sectorName,
    timestamp: new Date(),
    message: "TESTE: Simulação de atenção ativada",
  });
};

// Timer para decrementar takt_time
const startTimer = () => {
  if (timerInterval) clearInterval(timerInterval);

  timerInterval = setInterval(() => {
    if (!isRunning.value) return;

    const prevTime = taktTime.value;
    const newTime = Math.max(0, prevTime - 1);
    taktTime.value = newTime;

    // Determinar o status baseado no tempo restante
    let newStatus: Partial<SignalStatus>;

    if (newTime <= 5) {
      // Alarme - tempo zerado
      newStatus = {
        level: "alarm",
        sectorName: currentStatus.sectorName,
        timestamp: new Date(),
        message: "ALARME: Takt Time zerado - Parada de produção",
      };
      // Tocar alarme apenas quando acabou de chegar a zero
      playAlarmSound();
      // if (prevTime > 0) {
      // }
    } else if (newTime <= 30) {
      // Warning - falta 1 minuto ou menos
      newStatus = {
        level: "warning",
        sectorName: currentStatus.sectorName,
        timestamp: new Date(),
        message: `ATENÇÃO: Faltam ${newTime} segundos para o takt time`,
      };
    } else {
      // Normal - mais de 1 minuto restante
      newStatus = {
        level: "normal",
        sectorName: currentStatus.sectorName,
        timestamp: new Date(),
        message: "Sistema funcionando normalmente",
      };
    }

    Object.assign(currentStatus, newStatus);
  }, 1000);
};

// Lifecycle hooks
const wsConnected = ref(false);
const WsClient = shallowRef<WebSocket | null>(null);
let heartBeatInterval: number | null = null;
const lastPingTime = ref<number | null>(null); // TODO: Continuar logica de verificação de ping/pong heartbeat
let retries = 0;

const heartBeat = (ws: WebSocket) => {
  if (ws && wsConnected.value) {
    console.log('oinging');
    
    lastPingTime.value = Date.now();
    ws.send(JSON.stringify({ type: "ping" }));
  }
};

function connect() {
  const ws = new WebSocket(WS_URL);
  WsClient.value = ws;

  ws.addEventListener("open", () => {
    console.log("WebSocket conectado");
    wsConnected.value = true;
    retries = 0;

    const registerMessage = {
      type: "register",
      payload: { id: "cost-2-2408" },
    };
    ws.send(JSON.stringify(registerMessage));
  });

  ws.onmessage = (event) => {
    try {
      const data = JSON.parse(event.data);

      if (data.type === "taktAlert") {
        const { message, takt_time, clientId } = data;

        isRunning.value = true;
        taktTime.value = takt_time;
        console.log(`${message} (ID: ${clientId})`);

        startTimer();
      }
    } catch {
      console.log("Mensagem não-JSON:", event.data);
    }
  };

  ws.addEventListener("close", () => {
    console.log("WebSocket desconectado");
    wsConnected.value = false;
    // backoff exponencial simples
    const delay = Math.min(30000, 1000 * 2 ** retries++);
    setTimeout(connect, delay);
  });

  ws.addEventListener("error", (err) => {
    console.error("Erro no WebSocket:", err);
    // força fechamento para disparar o fluxo de reconexão
    ws.close();
  });

  if (heartBeatInterval) clearInterval(heartBeatInterval);
  heartBeatInterval = setInterval(() => heartBeat(ws), 30000);
}

onMounted(() => {
  connect();
});

onUnmounted(() => {
  if (timerInterval) {
    clearInterval(timerInterval);
  }
});

// Watch para reiniciar timer quando isRunning muda
watch(isRunning, () => {
  if (isRunning.value) {
    startTimer();
  }
  if (heartBeatInterval) {
    clearInterval(heartBeatInterval);
    heartBeatInterval = null;
  }
});
</script>

<style scoped>
@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(6px) scale(0.98);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}
.animate-fade-in {
  animation: fade-in 0.35s ease-out both;
}

@keyframes dot {
  0%,
  80%,
  100% {
    opacity: 0.2;
    transform: translateY(0);
  }
  40% {
    opacity: 1;
    transform: translateY(-2px);
  }
}
.loading-dot {
  width: 0.5rem;
  height: 0.5rem;
  border-radius: 9999px;
  background: rgb(59 130 246); /* blue-500 */
  display: inline-block;
  animation: dot 1.2s ease-in-out infinite;
}
</style>
