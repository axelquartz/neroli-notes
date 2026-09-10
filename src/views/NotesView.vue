<template>
  <div class="notes-page">
    <v-container class="notes-inner py-8 py-md-10 px-6 px-md-8" max-width="1200">
      <v-row class="mb-6 mb-md-8 align-center">
        <v-col cols="12" sm>
          <div class="notes-heading">
            <h1 class="notes-title">Neroli Notes</h1>
            <p class="notes-subtitle">The ultimate notetaker app</p>
          </div>
        </v-col>
        <v-col cols="12" sm="auto">
          <v-btn
            class="new-note-btn"
            rounded="lg"
            elevation="0"
            prepend-icon="mdi-plus"
            @click="dialog = true"
          >
            New note
          </v-btn>
        </v-col>
      </v-row>

      <v-divider class="notes-divider mb-8" />

      <v-row v-if="loading">
        <v-col class="text-center py-12">
          <v-progress-circular
            indeterminate
            color="#F4D35E"
          ></v-progress-circular>
        </v-col>
      </v-row>

      <v-row v-else-if="notas.length === 0">
        <v-col class="text-center py-12">
          <v-icon
            icon="mdi-note-off-outline"
            size="64"
            class="notes-subtitle mb-2"
          ></v-icon>
          <p class="notes-subtitle">
            No hay notas creadas todavía. ¡Escribe la primera!
          </p>
        </v-col>
      </v-row>

      <v-row v-else>
        <v-col
          cols="12"
          sm="6"
          lg="4"
          v-for="nota in notas"
          :key="nota.id"
        >
          <v-card class="note-card pa-5 rounded-xl d-flex flex-column" elevation="0">
            <v-card-title class="note-card-title px-0 pt-0 pb-3">
              {{ nota.title || "Sin título" }}
            </v-card-title>
            <v-card-text class="px-0 pb-6 note-body flex-grow-1">
              {{ nota.content }}
            </v-card-text>
            <v-card-actions class="px-0 pb-0 mt-auto justify-space-between">
              <span class="text-caption notes-date">
                {{ new Date(nota.created_at).toLocaleDateString() }}
              </span>
              <v-btn
                icon="mdi-trash-can-outline"
                variant="text"
                color="error"
                @click="eliminarNota(nota.id)"
              ></v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>

      <v-dialog v-model="dialog" max-width="500">
        <v-card class="dialog-card pa-4 rounded-xl">
          <v-card-title class="text-h5 font-weight-bold">Crear Nota</v-card-title>
          <v-card-text>
            <v-text-field
              v-model="nuevaNota.title"
              label="Título"
              variant="outlined"
              density="comfortable"
              class="mb-3"
            ></v-text-field>
            <v-textarea
              v-model="nuevaNota.content"
              label="Contenido"
              variant="outlined"
              density="comfortable"
              rows="4"
            ></v-textarea>
          </v-card-text>
          <v-card-actions class="justify-end">
            <v-btn variant="text" @click="dialog = false">Cancelar</v-btn>
            <v-btn
              class="new-note-btn"
              rounded="lg"
              elevation="0"
              @click="guardarNota"
              :loading="saving"
            >
              Guardar
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-container>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { supabase } from "@/supabase";

interface Nota {
  id: number;
  title: string;
  content: string;
  created_at: string;
}

const notas = ref<Nota[]>([]);
const loading = ref(true);
const dialog = ref(false);
const saving = ref(false);

const nuevaNota = ref({
  title: "",
  content: "",
});

const fetchNotas = async () => {
  loading.value = true;
  const { data, error } = await supabase
    .from("notes")
    .select("*")
    .order("created_at", { ascending: false });
  if (error) {
    console.error("Error al cargar notas:", error.message);
  } else {
    notas.value = data || [];
  }
  loading.value = false;
};

const guardarNota = async () => {
  if (!nuevaNota.value.title && !nuevaNota.value.content) return;
  saving.value = true;
  const { error } = await supabase
    .from("notes")
    .insert([
      { title: nuevaNota.value.title, content: nuevaNota.value.content },
    ]);

  if (error) {
    console.error("Error al guardar:", error.message);
  } else {
    nuevaNota.value.title = "";
    nuevaNota.value.content = "";
    dialog.value = false;
    await fetchNotas();
  }
  saving.value = false;
};

const eliminarNota = async (id: number) => {
  const { error } = await supabase.from("notes").delete().eq("id", id);
  if (error) {
    console.error("Error al eliminar:", error.message);
  } else {
    await fetchNotas();
  }
};

onMounted(() => {
  fetchNotas();
});
</script>

<style scoped>
.notes-page {
  min-height: 100vh;
  background: #121212;
}

.notes-heading {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.notes-title {
  margin: 0;
  color: #f5f5f5;
  font-size: 32px;
  font-weight: 700;
  line-height: 1.15;
  letter-spacing: -0.02em;
}

.notes-subtitle {
  margin: 0;
  color: #9e9e9e;
  font-size: 14px;
  font-weight: 400;
  line-height: 1.3;
}

.notes-date {
  color: #9e9e9e;
  font-size: 12px;
  line-height: 1.2;
}

.new-note-btn {
  background: #f4d35e !important;
  color: #1a1a1a !important;
  font-size: 14px;
  font-weight: 600;
  text-transform: none;
  letter-spacing: 0;
  height: 36px;
  padding: 0 16px;
  border-radius: 8px !important;
}

.notes-divider {
  opacity: 0.12;
  border-color: #ffffff !important;
}

.note-card {
  background: #2a2a2a !important;
  color: #f5f5f5;
  height: 100%;
}

.note-card-title {
  font-size: 18px;
  font-weight: 700;
  line-height: 1.3;
  letter-spacing: 0;
}

.note-body {
  color: #cfcfcf;
  font-size: 14px;
  line-height: 1.5;
  display: -webkit-box;
  -webkit-line-clamp: 6;
  line-clamp: 6;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.dialog-card {
  background: #2a2a2a !important;
  color: #f5f5f5;
}
</style>
