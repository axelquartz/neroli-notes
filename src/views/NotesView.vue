<template>
  <v-container class="py-8" max-width="800">
    <!-- Cabecera -->
    <v-row class="mb-6 align-center">
      <v-col>
        <h1 class="text-h4 font-weight-bold text-grey-darken-4">Mis Notas</h1>
        <p class="text-subtitle-1 text-grey">Sincronizadas con Supabase</p>
      </v-col>
      <v-col cols="auto">
        <v-btn
          color="primary"
          prepend-icon="mdi-plus"
          elevation="2"
          @click="dialog = true"
        >
          Nueva Nota
        </v-btn>
      </v-col>
    </v-row>

    <!-- Lista de Notas -->
    <v-row v-if="loading">
      <v-col class="text-center py-12">
        <v-progress-circular
          indeterminate
          color="primary"
        ></v-progress-circular>
      </v-col>
    </v-row>

    <v-row v-else-if="notas.length === 0">
      <v-col class="text-center py-12">
        <v-icon
          icon="mdi-note-off-outline"
          size="64"
          class="text-grey-lighten-1 mb-2"
        ></v-icon>
        <p class="text-grey">
          No hay notas creadas todavía. ¡Escribe la primera!
        </p>
      </v-col>
    </v-row>

    <v-row v-else>
      <v-col cols="12" md="6" v-for="nota in notas" :key="nota.id">
        <v-card class="pa-4 elevation-1 rounded-lg" border>
          <v-card-title class="text-h6 font-weight-bold px-0 pt-0">
            {{ nota.title || "Sin título" }}
          </v-card-title>
          <v-card-text class="px-0 text-body-1 text-grey-darken-2">
            {{ nota.content }}
          </v-card-text>
          <v-card-actions class="px-0 pb-0 justify-space-between">
            <span class="text-caption text-grey">
              {{ new Date(nota.created_at).toLocaleDateString() }}
            </span>
            <v-btn
              icon="mdi-delete-outline"
              variant="text"
              color="error"
              size="small"
              @click="eliminarNota(nota.id)"
            ></v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>

    <!-- Modal / Diálogo para Nueva Nota -->
    <v-dialog v-model="dialog" max-width="500">
      <v-card class="pa-4 rounded-lg">
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
            color="primary"
            variant="flat"
            @click="guardarNota"
            :loading="saving"
            >Guardar</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
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

// Obtener notas de Supabase (tabla 'notes')
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

// Guardar nueva nota (tabla 'notes')
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

// Eliminar nota (tabla 'notes')
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
