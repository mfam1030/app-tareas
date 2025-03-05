<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Lista de Tareas</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
    
      <ion-item>
        <ion-input v-model="nuevaTarea" placeholder="Nueva tarea"></ion-input>
        <ion-button size="large" @click="agregarTarea">
          <ion-icon slot="icon-only" :icon="addCircleOutline"></ion-icon>
        </ion-button>
      </ion-item>

  
      <ion-list>
        <ion-card v-for="(tarea, index) in tareas" :key="index">
          <ion-card-header>
            <ion-card-title :class="{ completada: tarea.completada }">
              {{ tarea.texto }}
            </ion-card-title>
          </ion-card-header>

          <ion-card-content>
           
            <ion-checkbox
              v-model="tarea.completada"
              @ionChange="actualizarTarea(index)"
              :color="tarea.completada ? 'success' : 'primary'"
            >
              {{ tarea.completada ? 'Completada' : 'Marcar como completada' }}
            </ion-checkbox>
          </ion-card-content>

          <ion-button @click="editarTarea(index)" fill="clear" color="warning">Editar</ion-button>
          <ion-button @click="eliminarTarea(index)" fill="clear" color="danger">Eliminar</ion-button>
        </ion-card>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonItem, IonInput, IonButton,
  IonList, IonCard, IonCardHeader, IonCardTitle, IonCardContent, IonIcon, IonCheckbox,
  toastController, alertController
} from '@ionic/vue';
import { Preferences } from '@capacitor/preferences'; 
import { ref, onMounted } from 'vue';
import { addCircleOutline } from 'ionicons/icons';

export default {
  components: {
    IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonItem, IonInput, IonButton,
    IonList, IonCard, IonCardHeader, IonCardTitle, IonCardContent, IonIcon, IonCheckbox
  },
  setup() {
    const tareas = ref<{ texto: string; completada: boolean }[]>([]);
    const nuevaTarea = ref('');

   
    const cargarTareas = async () => {
      const { value } = await Preferences.get({ key: 'tareas' });
      if (value) {
        tareas.value = JSON.parse(value);
      }
    };

    
    const guardarTareas = async () => {
      await Preferences.set({
        key: 'tareas',
        value: JSON.stringify(tareas.value),
      });
    };

  
    const agregarTarea = async () => {
      if (nuevaTarea.value.trim() !== '') {
        tareas.value.push({ texto: nuevaTarea.value, completada: false });
        nuevaTarea.value = '';
        await guardarTareas(); 
        mostrarToast('Tarea agregada correctamente');
      }
    };

    // Editar una tarea
    const editarTarea = async (index: number) => {
      const alert = await alertController.create({
        header: 'Editar tarea',
        inputs: [
          {
            name: 'texto',
            type: 'text',
            value: tareas.value[index].texto,
            placeholder: 'Nuevo texto'
          }
        ],
        buttons: [
          {
            text: 'Cancelar',
            role: 'cancel'
          },
          {
            text: 'Guardar',
            handler: (data) => {
              if (data.texto.trim() !== '') {
                tareas.value[index].texto = data.texto;
                guardarTareas(); 
                mostrarToast('Tarea editada correctamente');
              }
            }
          }
        ]
      });
      await alert.present();
    };

  
    const eliminarTarea = async (index: number) => {
      tareas.value.splice(index, 1);
      await guardarTareas(); 
      mostrarToast('Tarea eliminada correctamente');
    };

   
    const actualizarTarea = async (index: number) => {
      await guardarTareas(); 
      mostrarToast(`Tarea marcada como ${tareas.value[index].completada ? 'completada' : 'pendiente'}`);
    };

    
    const mostrarToast = async (mensaje: string) => {
      const toast = await toastController.create({
        message: mensaje,
        duration: 2000,
        position: 'top', 
        color: 'success', 
      });
      await toast.present();
    };

   
    onMounted(() => {
      cargarTareas();
    });

    return {
      tareas,
      nuevaTarea,
      agregarTarea,
      editarTarea,
      eliminarTarea,
      actualizarTarea,
      addCircleOutline, 
    };
  },
};
</script>

<style scoped>
/* Estilo para tareas completadas */
.completada {
  text-decoration: line-through;
  color: #888;
}
</style>