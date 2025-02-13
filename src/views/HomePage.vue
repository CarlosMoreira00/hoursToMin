<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Relógio Circular</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <ion-item>
        <ion-input v-model="inicio" type="time"></ion-input>
      </ion-item>

      <ion-item>
        <ion-input v-model="fim" type="time"></ion-input>
      </ion-item>

      <ion-button expand="full" @click="calcularDiferenca">Calcular e Iniciar</ion-button>

      <div v-if="contadorMinutos !== null" class="clock-container">
        <svg class="clock" width="200" height="200" viewBox="0 0 200 200">
          <!-- Círculo de fundo -->
          <circle cx="100" cy="100" r="90" class="clock-background" />
          <!-- Círculo de tempo -->
          <circle 
            cx="100" 
            cy="100" 
            r="90" 
            class="clock-progress" 
            :stroke-dasharray="circunferencia"
            :stroke-dashoffset="offset" />
        </svg>
        <div class="time-display">{{ formatarTempo() }}</div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref } from "vue";
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonItem, IonLabel, IonInput, IonButton } from "@ionic/vue";

// Definição do estado
const inicio = ref(null);
const fim = ref(null);
const contadorMinutos = ref(0);
let intervalId = null;

// Circunferência do círculo (calculada com a fórmula 2πr)
const raio = 90;
const circunferencia = 2 * Math.PI * raio;

// O offset será usado para animar a diminuição do círculo
const offset = ref(circunferencia);

// Variáveis de tempo
const tempoRestanteMs = ref(0);  // Variável para controlar o tempo restante em milissegundos

const calcularDiferenca = () => {
  if (inicio.value && fim.value) {
    // Converte os horários de início e fim para minutos
    const inicioDate = new Date(`1970-01-01T${inicio.value}:00`);
    const fimDate = new Date(`1970-01-01T${fim.value}:00`);

    let diffMs = fimDate - inicioDate; // Diferença em milissegundos

    // Caso a diferença seja negativa, assumimos que o fim foi no dia seguinte
    if (diffMs < 0) {
      diffMs += 24 * 60 * 60 * 1000; // Adiciona 24 horas (em milissegundos)
    }

    // Converte a diferença para minutos
    const diffMinutos = diffMs / 1000 / 60;
    contadorMinutos.value = diffMinutos; // Atualiza o contador de minutos

    // Converte para milissegundos
    tempoRestanteMs.value = diffMinutos * 60 * 1000;

    // Inicia o contador
    iniciarContador(tempoRestanteMs.value);
  }
};

const iniciarContador = (tempoTotalMs) => {
  if (intervalId) clearInterval(intervalId);  // Limpa o intervalo anterior

  intervalId = setInterval(() => {
    if (tempoRestanteMs.value > 0) {
      tempoRestanteMs.value -= 10; // Decrementa 10ms
      const porcentagemTempoRestante = tempoRestanteMs.value / tempoTotalMs; // Calcula a porcentagem de tempo restante
      offset.value = circunferencia * (1 - porcentagemTempoRestante); // Atualiza o "offset" do círculo
    } else {
      clearInterval(intervalId); // Para o intervalo quando o tempo acabar
    }
  }, 10); // Atualiza a cada 10ms
};

// Função para formatar o tempo restante
const formatarTempo = () => {
  const totalMs = tempoRestanteMs.value;

  const minutosRestantes = Math.floor(totalMs / 60000);
  const segundosRestantes = Math.floor((totalMs % 60000) / 1000);
  const milissegundosRestantes = totalMs % 1000;

  return `${minutosRestantes.toString().padStart(2, "0")}:${segundosRestantes.toString().padStart(2, "0")}:${milissegundosRestantes.toString().padStart(3, "0")}ms`;
};
</script>

<style scoped>
.clock-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
}

.clock {
  transform: rotate(-90deg); /* Gira o círculo para que ele comece de cima */
}

.clock-background {
  fill: none;
  stroke: #e6e6e6;
  stroke-width: 10;
}

.clock-progress {
  fill: none;
  stroke: #4caf50;
  stroke-width: 10;
  stroke-linecap: round;
  transition: stroke-dashoffset 0.1s linear;
}

.time-display {
  font-size: 24px;
  margin-top: 20px;
}
</style>

