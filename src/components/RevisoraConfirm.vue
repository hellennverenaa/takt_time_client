<template>
  <v-dialog
    :model-value="openTaktConfirmDialog"
    @update:model-value="emit('update:openTaktConfirmDialog', $event)"
    max-width="400"
    persistent
  >
    <!-- <template v-slot:activator="{ props: activatorProps }">
      <v-btn v-bind="activatorProps"> Open Dialog </v-btn>
    </template> -->

    <v-card class="bg-green-lighten-3">
      <v-card-title>
        <h1 class="text-center">
          <strong>Takt Concluído <i class="mdi mdi-check-all" /></strong>
        </h1>
      </v-card-title>

      <v-card-text>
        <div>
          <p>Tempo de Takt Concluído! A produção foi Concluída?</p>
        </div>

        <div class="actions mt-5 d-flex justify-around">
          <v-btn prepend-icon="mdi mdi-check-outline" color="success" @click="confirm"> Confirmar </v-btn>
          <v-btn prepend-icon="mdi mdi-cancel" color="error" @click="reject"> Recusar </v-btn>
        </div>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>

<script lang="ts" setup>
import { ref, defineEmits, defineProps, toRefs, watch } from "vue";

const props = defineProps<{
  openTaktConfirmDialog: boolean;
}>();

const { openTaktConfirmDialog } = toRefs(props);
const dialogOpenedTimestamp = ref<Date | null>(null); // captura o timestamp quando o diálogo é aberto
const dialogClosedTimestamp = ref<Date | null>(null); // captura o timestamp quando o diálogo é fechado
const dialogDuration = ref<number | null>(null); // duração do diálogo em segundos
watch(openTaktConfirmDialog, (newValue) => {
  if (newValue) {
    dialogOpenedTimestamp.value = new Date();
  }
});

const emit = defineEmits<{
  (e: "update:openTaktConfirmDialog", value: boolean): void;
  (e: "resetTaktTime", value: any): void;
}>();

const close = () => {
  emit("update:openTaktConfirmDialog", false);
};

const confirm = () => {
  dialogClosedTimestamp.value = new Date();
  close();

  // TODO: Implementar lógica de confirmação
  dialogDuration.value = Math.round(
    (dialogClosedTimestamp.value!.getTime() - dialogOpenedTimestamp.value!.getTime()) / 1000
  );
  emit("resetTaktTime", dialogDuration.value);
};

const reject = () => {
  dialogClosedTimestamp.value = new Date();
  close();

  // TODO: Implementar lógica de rejeição
  dialogDuration.value = Math.round(
    (dialogClosedTimestamp.value!.getTime() - dialogOpenedTimestamp.value!.getTime()) / 1000
  );
  emit("resetTaktTime", dialogDuration.value);
};
</script>

<style scoped></style>
