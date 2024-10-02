<template>
  <form>
    <!-- Nome do Evento -->
    <v-text-field
      v-model="state.nomeEvento"
      :counter="50"
      :error-messages="v$.nomeEvento.$errors.map(e => e.$message)"
      label="Nome do Evento"
      required
      @blur="v$.nomeEvento.$touch"
      @input="v$.nomeEvento.$touch"
    ></v-text-field>

    <!-- Data do Evento -->
    <v-text-field
      v-model="state.dataEvento"
      label="Data do Evento (DD/MM/AAAA)"
      required
      type="date"
      :error-messages="v$.dataEvento.$errors.map(e => e.$message)"
      @blur="v$.dataEvento.$touch"
      @input="v$.dataEvento.$touch"
    ></v-text-field>

    <!-- Local do Evento -->
    <v-text-field
      v-model="state.localEvento"
      :counter="100"
      :error-messages="v$.localEvento.$errors.map(e => e.$message)"
      label="Local do Evento"
      required
      @blur="v$.localEvento.$touch"
      @input="v$.localEvento.$touch"
    ></v-text-field>

    <!-- Adicionar Participantes -->
    <v-row>
      <v-col cols="8" md="6">
        <v-text-field
          v-model="novoParticipante"
          label="Adicionar Participante"
        ></v-text-field>
      </v-col>
      <v-col cols="4" md="2" class="d-flex align-center">
        <v-btn color="primary" @click="adicionarParticipante">Adicionar</v-btn>
      </v-col>
    </v-row>

    <!-- Lista de Participantes -->
    <v-row>
      <v-col cols="12" md="6">
        <v-list>
          <v-list-item
            v-for="(participante, index) in state.participantes"
            :key="index"
          >
            <v-list-item-content>{{ participante.nome }}</v-list-item-content>
            <v-list-item-action>
              <v-btn icon @click="removerParticipante(index)">
                <v-icon>mdi-delete</v-icon>
              </v-btn>
            </v-list-item-action>
          </v-list-item>
        </v-list>
      </v-col>
    </v-row>

    <!-- Botões -->
    <v-btn
      class="me-4"
      @click="cadastrarEvento"
      :disabled="!v$.$anyDirty || v$.$pending || v$.$invalid"
    >
      Cadastrar Evento
    </v-btn>
    <v-btn @click="clear">
      Limpar
    </v-btn>
  </form>
</template>

<script setup>
import { reactive, toRefs } from 'vue'
import useVuelidate from '@vuelidate/core'
import { required } from '@vuelidate/validators'

// Estado inicial
const initialState = {
  nomeEvento: '',
  dataEvento: '',
  localEvento: '',
  participantes: [],
}

const state = reactive({
  ...initialState,
})

let novoParticipante = ''

// Função de validação de data
const isValidDate = (dateString) => {
  const regex = /^(0[1-9]|[12][0-9]|3[01])\/(0[1-9]|1[0-2])\/\d{4}$/;
  if (!regex.test(dateString)) return false;

  const [day, month, year] = dateString.split('/').map(Number);
  const date = new Date(year, month - 1, day);
  return date.getFullYear() === year && date.getMonth() === month - 1 && date.getDate() === day;
}

// Validações
const customDateValidator = (value) => {
  return isValidDate(value) || 'Data deve estar no formato DD/MM/AAAA e ser uma data válida';
}

// Regras de validação
const rules = {
  nomeEvento: { required },
  dataEvento: { required: true, customDateValidator },
  localEvento: { required },
}

const v$ = useVuelidate(rules, state)

// Função para adicionar participante
function adicionarParticipante() {
  if (novoParticipante) {
    // Adiciona o participante como objeto com eventID
    const eventID = Date.now(); // Você deve usar o mesmo eventID do evento quando for cadastrar
    state.participantes.push({ nome: novoParticipante, eventID });
    novoParticipante = '';
  }
}

// Função para remover participante
function removerParticipante(index) {
  state.participantes.splice(index, 1);
}

// Função para formatar a data
function formatDate(dateString) {
  const [year, month, day] = dateString.split('-');
  return `${day}/${month}/${year}`; // Converte para DD/MM/AAAA
}

function cadastrarEvento() {
  v$.value.$touch();
  if (!v$.value.$invalid) {
    // Gera um eventID único
    const eventID = Date.now();

    // Carrega eventos e participantes existentes do localStorage
    const events = JSON.parse(localStorage.getItem('Events')) || [];
    const participants = JSON.parse(localStorage.getItem('Participants')) || {};

    // Formata a data para DD/MM/AAAA
    const formattedDataEvento = formatDate(state.dataEvento);

    // Cria o evento e adiciona ao array de eventos
    const novoEvento = {
      eventID,
      nome: state.nomeEvento,
      data: formattedDataEvento, // Usa a data formatada
      local: state.localEvento,
    };
    events.push(novoEvento);

    // Adiciona participantes ao objeto de participantes com eventID
    participants[eventID] = state.participantes.map(participante => ({
      nome: participante.nome,
      eventID: eventID // Associa o eventID a cada participante
    }));

    // Salva eventos e participantes no localStorage
    localStorage.setItem('Events', JSON.stringify(events));
    localStorage.setItem('Participants', JSON.stringify(participants));

    // Limpa o formulário
    clear();
    alert('Evento cadastrado com sucesso!');
  }
}

// Função para limpar o formulário
function clear() {
  v$.value.$reset();

  for (const [key, value] of Object.entries(initialState)) {
    state[key] = value;
  }
}
</script>

<style scoped>
.v-btn {
  margin-top: 20px;
}
</style>
