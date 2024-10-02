<template>
  <div>
    <h1>Eventos Cadastrados</h1>

    <v-card>
      <v-card-title>
        <span>Lista de Eventos</span>
      </v-card-title>
      <v-card-text>
        <v-list>
          <v-list-item
            v-for="event in eventos"
            :key="event.eventID"
          >
            <v-list-item-content>
              <v-list-item-title>{{ event.nome }}</v-list-item-title>
              <v-list-item-subtitle>
                <span>Data: {{ formatDate(event.data) }}</span>
                <span> | Local: {{ event.local }}</span>
              </v-list-item-subtitle>
              <v-list-item-subtitle>
                <v-btn @click="openParticipantsDialog(event.eventID)" class="mr-2">Ver Participantes</v-btn>
                <v-btn @click="openEditEventDialog(event)" color="primary" class="mr-2">Editar Evento</v-btn>
                <v-btn @click="openEventDetailsDialog(event)" color="info">Ver Detalhes</v-btn>
              </v-list-item-subtitle>
            </v-list-item-content>
            <v-list-item-action>
              <v-btn icon @click="removerEvento(event.eventID)">
                <v-icon>mdi-delete</v-icon>
              </v-btn>
            </v-list-item-action>
          </v-list-item>
          <v-divider class="my-3"></v-divider> <!-- Espaçador entre os eventos -->
        </v-list>
      </v-card-text>
    </v-card>

    <!-- Dialogo para mostrar participantes -->
    <v-dialog v-model="participantsDialog" max-width="500px">
      <v-card>
        <v-card-title>
          Participantes do Evento
          <v-spacer></v-spacer>
          <v-btn icon @click="participantsDialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-list>
            <v-list-item
              v-for="(participant, index) in currentParticipants"
              :key="index"
            >
              <v-list-item-content>{{ participant.nome }}</v-list-item-content>
              <v-list-item-action>
                <v-btn icon @click="editParticipant(participant, index)">
                  <v-icon>mdi-pencil</v-icon>
                </v-btn>
                <v-btn icon @click="removeParticipant(currentEventID, index)">
                  <v-icon>mdi-delete</v-icon>
                </v-btn>
              </v-list-item-action>
            </v-list-item>
            <v-list-item v-if="!currentParticipants.length">
              <v-list-item-content>Nenhum participante cadastrado.</v-list-item-content>
            </v-list-item>
          </v-list>
        </v-card-text>
        <v-card-actions>
          <v-btn @click="addParticipant(currentEventID)" color="success">Adicionar Participante</v-btn>
          <v-btn @click="participantsDialog = false" color="primary">Fechar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Novo diálogo para mostrar detalhes do evento -->
    <v-dialog v-model="eventDetailsDialog" max-width="500px">
      <v-card>
        <v-card-title>
          Detalhes do Evento
          <v-spacer></v-spacer>
          <v-btn icon @click="eventDetailsDialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-list>
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title><strong>Nome:</strong> {{ currentEventDetails.nome }}</v-list-item-title>
                <v-list-item-subtitle><strong>Data:</strong> {{ formatDate(currentEventDetails.data) }}</v-list-item-subtitle>
                <v-list-item-subtitle><strong>Local:</strong> {{ currentEventDetails.local }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-list-item>
              <v-list-item-content>
                <strong>Participantes:</strong>
                <v-list>
                  <v-list-item
                    v-for="(participant, index) in currentEventDetails.participants"
                    :key="index"
                  >
                    <v-list-item-content>{{ participant.nome }}</v-list-item-content>
                  </v-list-item>
                </v-list>
                <v-list-item v-if="!currentEventDetails.participants.length">
                  <v-list-item-content>Nenhum participante cadastrado.</v-list-item-content>
                </v-list-item>
              </v-list-item-content>
            </v-list-item>
          </v-list>
        </v-card-text>
        <v-card-actions>
          <v-btn @click="eventDetailsDialog = false" color="primary">Fechar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Dialogo para editar evento -->
    <v-dialog v-model="editEventDialog" max-width="500px">
  <v-card>
    <v-card-title>
      Editar Evento
      <v-spacer></v-spacer>
      <v-btn icon @click="editEventDialog = false">
        <v-icon>mdi-close</v-icon>
      </v-btn>
    </v-card-title>
    <v-card-text>
      <v-container>
        <v-row>
          <v-col cols="12">
            <v-card class="mb-2">
              <v-card-title>Nome do Evento</v-card-title>
              <v-card-text>
                <v-text-field v-model="editedEvent.nome" label="Nome do Evento"></v-text-field>
              </v-card-text>
            </v-card>
          </v-col>
          <v-col cols="12">
            <v-card class="mb-2">
              <v-card-title>Data</v-card-title>
              <v-card-text>
                <!-- Removi o @input para evitar conflito -->
                <v-text-field
                  v-model="editedEvent.data"
                  type="date"
                  label="Data"
                ></v-text-field>
              </v-card-text>
            </v-card>
          </v-col>
          <v-col cols="12">
            <v-card class="mb-2">
              <v-card-title>Local</v-card-title>
              <v-card-text>
                <v-text-field v-model="editedEvent.local" label="Local"></v-text-field>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-card-text>
    <v-card-actions>
      <v-btn @click="saveEvent" color="primary">Salvar</v-btn>
      <v-btn @click="editEventDialog = false">Cancelar</v-btn>
    </v-card-actions>
  </v-card>
</v-dialog>

    <!-- Dialogo para editar participante -->
    <v-dialog v-model="editParticipantDialog" max-width="400px">
      <v-card>
        <v-card-title>
          Editar Participante
          <v-spacer></v-spacer>
          <v-btn icon @click="editParticipantDialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-text-field v-model="editedParticipantName" label="Nome do Participante"></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-btn @click="saveParticipant" color="primary">Salvar</v-btn>
          <v-btn @click="editParticipantDialog = false">Cancelar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const eventos = ref([]);
const participantsDialog = ref(false);
const editEventDialog = ref(false);
const editParticipantDialog = ref(false);
const eventDetailsDialog = ref(false); // Novo diálogo para mostrar detalhes do evento
const currentParticipants = ref([]);
const editedEvent = ref({});
const editedParticipantName = ref('');
const currentEventID = ref(null);
const currentEventDetails = ref({}); // Armazena os detalhes do evento atual
let currentParticipantIndex = ref(null);

const loadEvents = () => {
  const storedEvents = JSON.parse(localStorage.getItem('Events')) || [];
  eventos.value = storedEvents;
}

const formatDate = (dateString) => {
  // Se a data estiver no formato "AAAA-MM-DD", converta para "DD/MM/AAAA"
  if (dateString.includes('-')) {
    const [year, month, day] = dateString.split('-');
    return `${day}/${month}/${year}`;
  }

  // Se a data já estiver no formato "DD/MM/AAAA", retorne sem mudanças
  const [day, month, year] = dateString.split('/');
  return day && month && year ? `${day}/${month}/${year}` : 'Data inválida';
}

// Função para validar e formatar a entrada da data
const formatDateInput = () => {
  if (editedEvent.value.data.includes('-')) {
    const [year, month, day] = editedEvent.value.data.split('-');
    editedEvent.value.data = `${day}/${month}/${year}`;
  }
}

const getParticipants = (eventID) => {
  const participants = JSON.parse(localStorage.getItem('Participants')) || {};
  return participants[eventID] || [];
}

// Função para abrir o diálogo de participantes
const openParticipantsDialog = (eventID) => {
  currentEventID.value = eventID;
  currentParticipants.value = getParticipants(eventID);
  participantsDialog.value = true;
}

// Função para abrir o diálogo de detalhes do evento
const openEventDetailsDialog = (event) => {
  currentEventDetails.value = { ...event, participants: getParticipants(event.eventID) };
  eventDetailsDialog.value = true;
}

// Função para remover um evento
const removerEvento = (eventID) => {
  eventos.value = eventos.value.filter(event => event.eventID !== eventID);
  saveEventsToLocalStorage();
}

// Função para abrir o diálogo de edição do evento
const openEditEventDialog = (event) => {
  editedEvent.value = { ...event };
  editEventDialog.value = true;
}

// Função para salvar as edições do evento
const saveEvent = () => {
  const index = eventos.value.findIndex(event => event.eventID === editedEvent.value.eventID);
  if (index !== -1) {
    eventos.value[index] = { ...editedEvent.value };
    saveEventsToLocalStorage();
    editEventDialog.value = false;
  }
}

// Função para adicionar um participante
const addParticipant = (eventID) => {
  const participantName = prompt('Nome do Participante:');
  if (participantName) {
    const participants = JSON.parse(localStorage.getItem('Participants')) || {};
    if (!participants[eventID]) participants[eventID] = [];
    participants[eventID].push({ nome: participantName });
    localStorage.setItem('Participants', JSON.stringify(participants));
    currentParticipants.value = participants[eventID]; // Atualiza a lista de participantes
  }
}

// Função para remover um participante
const removeParticipant = (eventID, index) => {
  const participants = JSON.parse(localStorage.getItem('Participants')) || {};
  participants[eventID].splice(index, 1);
  localStorage.setItem('Participants', JSON.stringify(participants));
  currentParticipants.value = participants[eventID]; // Atualiza a lista de participantes
}

// Função para abrir o diálogo de edição do participante
const editParticipant = (participant, index) => {
  currentParticipantIndex.value = index;
  editedParticipantName.value = participant.nome;
  editParticipantDialog.value = true;
}

// Função para salvar as edições do participante
const saveParticipant = () => {
  const participants = JSON.parse(localStorage.getItem('Participants')) || {};
  participants[currentEventID.value][currentParticipantIndex.value].nome = editedParticipantName.value;
  localStorage.setItem('Participants', JSON.stringify(participants));
  currentParticipants.value = participants[currentEventID.value]; // Atualiza a lista de participantes
  editParticipantDialog.value = false;
}

// Função para salvar eventos no localStorage
const saveEventsToLocalStorage = () => {
  localStorage.setItem('Events', JSON.stringify(eventos.value));
}

// Carrega os eventos ao montar o componente
onMounted(() => {
  loadEvents();
});
</script>

<style scoped>
/* Adicionando espaçamento entre os eventos */
.v-list-item {
  margin-bottom: 20px; /* Espaço entre cada evento */
}

/* Estilo para os botões */
.v-btn {
  margin-right: 10px; /* Espaço entre os botões */
}
</style>
