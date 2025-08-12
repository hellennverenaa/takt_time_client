<template>
  <div class="min-h-screen w-full relative flex items-center justify-center p-4 bg-gradient-to-b from-blue-900 to-gray-900">
    <!-- Background overlay -->
    <div class="absolute inset-0 bg-black/40"></div>

    <!-- Conteúdo principal -->
    <div class="relative z-10 w-full max-w-2xl">
      <!-- Header com logo DASS -->
      <div class="text-center mb-8">
        <h1 class="text-6xl md:text-7xl lg:text-8xl font-black text-white drop-shadow-2xl tracking-wider">
          DASS
        </h1>
        <div class="text-white/90 text-xl md:text-2xl mt-4 drop-shadow-lg">
          Sistema de Monitoramento Industrial
        </div>
      </div>

      <!-- Formulário -->
      <div class="bg-white/95 backdrop-blur-md shadow-2xl border-0 rounded-lg p-6">
        <div class="text-center pb-4">
          <h2 class="text-2xl font-bold text-gray-800 flex items-center justify-center gap-2">
            <svg class="h-6 w-6 text-blue-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 6H5a2 2 0 00-2 2v9a2 2 0 002 2h14a2 2 0 002-2V8a2 2 0 00-2-2h-5m-8 0V4a2 2 0 012-2h4a2 2 0 012 2v2m-8 0h8" />
            </svg>
            Identificação do Funcionário
          </h2>
        </div>

        <form @submit.prevent="handleSubmit" class="space-y-6">
          <!-- Etapa 1: Identificação -->
          <div v-if="currentStep === 1">
            <div class="text-center mb-6">
              <svg class="h-12 w-12 mx-auto text-blue-600 mb-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 6H5a2 2 0 00-2 2v9a2 2 0 002 2h14a2 2 0 002-2V8a2 2 0 00-2-2h-5m-8 0V4a2 2 0 012-2h4a2 2 0 012 2v2m-8 0h8" />
              </svg>
              <h3 class="text-xl font-semibold text-gray-800">
                Digite sua matrícula ou bipe seu crachá
              </h3>
            </div>

            <div class="space-y-4">
              <div>
                <label class="text-base font-medium text-gray-700">
                  Matrícula do Funcionário
                </label>
                <div class="flex gap-2 mt-2">
                  <div class="relative flex-1">
                    <svg class="absolute left-3 top-1/2 transform -translate-y-1/2 h-5 w-5 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
                    </svg>
                    <input
                      v-model="formData.operatorId"
                      @input="handleEmployeeSearch"
                      type="text"
                      placeholder="Digite a matrícula (ex: 001, 002...)"
                      class="pl-12 h-12 text-base w-full border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                      :disabled="isLoadingEmployee"
                    />
                    <div v-if="isLoadingEmployee" class="absolute right-3 top-1/2 transform -translate-y-1/2">
                      <div class="animate-spin h-5 w-5 border-2 border-blue-600 border-t-transparent rounded-full"></div>
                    </div>
                  </div>
                  
                  <button
                    type="button"
                    @click="simulateBadgeRead"
                    :disabled="isLoadingEmployee"
                    class="h-12 px-4 border-2 border-dashed border-blue-300 hover:border-blue-500 hover:bg-blue-50 rounded-md flex items-center gap-2 text-blue-600 disabled:opacity-50"
                  >
                    <svg class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z" />
                    </svg>
                    Bipar Crachá
                  </button>
                </div>
                <p class="text-xs text-gray-500 mt-1">
                  Experimente: 001, 002, 003... ou clique em "Bipar Crachá"
                </p>
              </div>

              <!-- Informações do funcionário encontrado -->
              <div v-if="employeeFound" class="bg-green-50 border border-green-200 rounded-lg p-4 space-y-3 animate-fade-in">
                <div class="flex items-center gap-2 text-green-800">
                  <svg class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
                  </svg>
                  <span class="font-semibold">✓ Funcionário Encontrado</span>
                </div>
                
                <div class="grid grid-cols-2 gap-3">
                  <div>
                    <span class="text-sm font-medium text-gray-600">Nome:</span>
                    <div class="text-base font-semibold text-gray-800">{{ employeeFound.name }}</div>
                  </div>
                  <div>
                    <span class="text-sm font-medium text-gray-600">Matrícula:</span>
                    <div class="text-base font-semibold text-gray-800">{{ employeeFound.id }}</div>
                  </div>
                  <div>
                    <span class="text-sm font-medium text-gray-600">Setor:</span>
                    <div class="text-base font-semibold text-gray-800">{{ employeeFound.sector }}</div>
                  </div>
                  <div>
                    <span class="text-sm font-medium text-gray-600">Célula:</span>
                    <div class="text-base font-semibold text-gray-800">{{ employeeFound.cell }}</div>
                  </div>
                  <div class="col-span-2">
                    <span class="text-sm font-medium text-gray-600">Cargo:</span>
                    <div class="text-base font-semibold text-gray-800">{{ employeeFound.position }}</div>
                  </div>
                  <div v-if="employeeFound.leaderName" class="col-span-2 bg-blue-50 rounded p-2">
                    <span class="text-sm font-medium text-blue-700">Líder Direto:</span>
                    <div class="text-base font-semibold text-blue-800">{{ employeeFound.leaderName }}</div>
                  </div>
                </div>
              </div>

              <!-- Mensagem se não encontrar -->
              <div v-if="formData.operatorId && !employeeFound && !isLoadingEmployee" class="bg-red-50 border border-red-200 rounded-lg p-4">
                <div class="flex items-center gap-2 text-red-800">
                  <div class="w-5 h-5 bg-red-500 rounded-full flex items-center justify-center">
                    <span class="text-white text-xs font-bold">!</span>
                  </div>
                  <span class="font-semibold">Funcionário não encontrado</span>
                </div>
                <p class="text-red-700 text-sm mt-1">
                  Verifique a matrícula digitada ou tente bipar o crachá novamente.
                </p>
              </div>
            </div>
          </div>

          <!-- Etapa 2: Confirmação -->
          <div v-if="currentStep === 2">
            <div class="text-center mb-6">
              <svg class="h-12 w-12 mx-auto text-yellow-600 mb-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
              </svg>
              <h3 class="text-xl font-semibold text-gray-800">
                Confirmação dos Dados
              </h3>
              <p class="text-gray-600 text-sm mt-2">
                Revise as informações antes de iniciar
              </p>
            </div>

            <div class="bg-gray-50 rounded-lg p-4 space-y-3">
              <div class="grid grid-cols-2 gap-4">
                <div>
                  <span class="text-sm font-medium text-gray-500">Funcionário:</span>
                  <div class="text-base font-semibold text-gray-800">{{ formData.operatorName }}</div>
                </div>
                <div>
                  <span class="text-sm font-medium text-gray-500">Matrícula:</span>
                  <div class="text-base font-semibold text-gray-800">{{ formData.operatorId }}</div>
                </div>
                <div>
                  <span class="text-sm font-medium text-gray-500">Setor:</span>
                  <div class="text-base font-semibold text-gray-800">{{ formData.sectorName }}</div>
                </div>
                <div>
                  <span class="text-sm font-medium text-gray-500">Célula:</span>
                  <div class="text-base font-semibold text-gray-800">{{ formData.operatorCell }}</div>
                </div>
                <div v-if="formData.leaderName" class="col-span-2">
                  <span class="text-sm font-medium text-gray-500">Líder Responsável:</span>
                  <div class="text-base font-semibold text-gray-800">{{ formData.leaderName }}</div>
                </div>
                <div class="col-span-2">
                  <span class="text-sm font-medium text-gray-500">Tempo Meta (Takt Time):</span>
                  <div class="text-base font-semibold text-gray-800">
                    02:00 minutos (fixo para todos os setores)
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Botões de navegação -->
          <div class="flex justify-between pt-6">
            <button
              v-if="currentStep > 1"
              type="button"
              @click="prevStep"
              class="px-6 py-2 border border-gray-300 rounded-md hover:bg-gray-50"
            >
              Voltar
            </button>
            
            <div :class="currentStep === 1 ? 'ml-auto' : ''">
              <button
                v-if="currentStep < 2"
                type="button"
                @click="nextStep"
                :disabled="!isStepValid"
                class="px-6 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-md disabled:opacity-50 flex items-center gap-2"
              >
                Próximo
                <svg class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                </svg>
              </button>
              <button
                v-else
                type="submit"
                :disabled="isSubmitting"
                class="px-8 py-2 bg-green-600 hover:bg-green-700 text-white rounded-md disabled:opacity-50 flex items-center gap-2"
              >
                <div v-if="isSubmitting" class="animate-spin h-4 w-4 border-2 border-white border-t-transparent rounded-full"></div>
                <span>{{ isSubmitting ? 'Iniciando...' : 'Iniciar Sistema' }}</span>
                <svg v-if="!isSubmitting" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
              </button>
            </div>
          </div>
        </form>
      </div>

      <!-- Footer -->
      <div class="text-center mt-8">
        <div class="text-white/70 text-sm">
          DASS © 2025 - Sistema de Monitoramento Industrial
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import { EmployeeService } from '../services/EmployeeService.js'

export default {
  name: 'DassInitialForm',
  emits: ['form-submit'],
  data() {
    return {
      formData: {
        sectorName: '',
        operatorName: '',
        operatorId: '',
        operatorCell: '',
        leaderName: '',
        leaderId: ''
      },
      currentStep: 1,
      isSubmitting: false,
      employeeFound: null,
      isLoadingEmployee: false
    }
  },
  computed: {
    isStepValid() {
      if (this.currentStep === 1) {
        return this.formData.operatorId && this.formData.operatorName && this.formData.sectorName
      }
      return true
    }
  },
  methods: {
    playSuccessSound() {
      try {
        const audioContext = new (window.AudioContext || window.webkitAudioContext)()
        const oscillator = audioContext.createOscillator()
        const gainNode = audioContext.createGain()
        
        oscillator.connect(gainNode)
        gainNode.connect(audioContext.destination)
        
        oscillator.frequency.setValueAtTime(600, audioContext.currentTime)
        oscillator.frequency.setValueAtTime(800, audioContext.currentTime + 0.1)
        
        gainNode.gain.setValueAtTime(0.2, audioContext.currentTime)
        gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.2)
        
        oscillator.start(audioContext.currentTime)
        oscillator.stop(audioContext.currentTime + 0.2)
      } catch (error) {
        // Ignore
      }
    },

    async handleEmployeeSearch() {
      if (!this.formData.operatorId.trim()) {
        this.employeeFound = null
        return
      }

      this.isLoadingEmployee = true
      
      await new Promise(resolve => setTimeout(resolve, 800))
      
      const employee = EmployeeService.findEmployeeById(this.formData.operatorId)
      
      if (employee) {
        this.employeeFound = employee
        this.playSuccessSound()
        
        this.formData.operatorName = employee.name
        this.formData.sectorName = employee.sector
        this.formData.operatorCell = employee.cell
        this.formData.leaderName = employee.leaderName || ''
        this.formData.leaderId = employee.leaderId || ''
      } else {
        this.employeeFound = null
        this.formData.operatorName = ''
        this.formData.sectorName = ''
        this.formData.operatorCell = ''
        this.formData.leaderName = ''
        this.formData.leaderId = ''
      }
      
      this.isLoadingEmployee = false
    },

    async simulateBadgeRead() {
      this.isLoadingEmployee = true
      
      // Som de beep do scanner
      try {
        const audioContext = new (window.AudioContext || window.webkitAudioContext)()
        const oscillator = audioContext.createOscillator()
        const gainNode = audioContext.createGain()
        
        oscillator.connect(gainNode)
        gainNode.connect(audioContext.destination)
        
        oscillator.frequency.setValueAtTime(1000, audioContext.currentTime)
        gainNode.gain.setValueAtTime(0.3, audioContext.currentTime)
        gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.1)
        
        oscillator.start(audioContext.currentTime)
        oscillator.stop(audioContext.currentTime + 0.1)
      } catch (error) {
        // Ignore
      }
      
      await new Promise(resolve => setTimeout(resolve, 1200))
      
      const employee = EmployeeService.simulateBadgeRead()
      
      this.employeeFound = employee
      this.playSuccessSound()
      
      this.formData.operatorId = employee.id
      this.formData.operatorName = employee.name
      this.formData.sectorName = employee.sector
      this.formData.operatorCell = employee.cell
      this.formData.leaderName = employee.leaderName || ''
      this.formData.leaderId = employee.leaderId || ''
      
      this.isLoadingEmployee = false
    },

    nextStep() {
      if (this.currentStep < 2) {
        this.currentStep++
      }
    },

    prevStep() {
      if (this.currentStep > 1) {
        this.currentStep--
      }
    },

    async handleSubmit() {
      this.isSubmitting = true
      
      await new Promise(resolve => setTimeout(resolve, 1500))
      
      this.$emit('form-submit', this.formData)
    }
  }
}
</script>

<style scoped>
.animate-fade-in {
  animation: fadeIn 0.5s ease-in-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>