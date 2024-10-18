<script setup>
// Importar los módulos necesarios de Firebase y Firestore
import { ref, onMounted } from 'vue';
import { collection, getDocs, addDoc, updateDoc, doc } from 'firebase/firestore'; // Importar updateDoc y doc
import { db } from '../firebase'; // Importa Firestore desde firebase.js

// Lista para almacenar las tareas
const todos = ref([]);

// Función para convertir Firestore Timestamp a Date
const convertTimestampToDate = (timestamp) => {
    const date = new Date(timestamp.seconds * 1000); // Convertir a milisegundos
    return date.toLocaleDateString(); // Formato de fecha
};

// Función para obtener las tareas de Firestore
const fetchTodos = async () => {
    try {
        const querySnapshot = await getDocs(collection(db, 'to-do'));
        todos.value = querySnapshot.docs.map(doc => ({
            id: doc.id,
            cliente: doc.data().cliente,
            completado: doc.data().completado,
            fechaInicio: convertTimestampToDate(doc.data().fecha_inicio),
            fechaLimite: convertTimestampToDate(doc.data().fecha_limite),
            precio: doc.data().precio,
            tarea: doc.data().tarea,
            estado: doc.data().completado ? 'Completado' : 'Pendiente'
        }));

        console.log('Datos obtenidos:', todos.value);
    } catch (error) {
        console.error('Error obteniendo los datos:', error);
    }
};

// Función para agregar una nueva tarea
const newTask = ref({
    tarea: '',
    cliente: '',
    precio: '',
    fechaInicio: '',
    fechaLimite: ''
});

const addTask = async (event) => {
    event.preventDefault(); // Evitar que la página se recargue al enviar el formulario
    try {
        await addDoc(collection(db, 'to-do'), {
            tarea: newTask.value.tarea,
            cliente: newTask.value.cliente,
            precio: Number(newTask.value.precio),
            fecha_inicio: new Date(newTask.value.fechaInicio),
            fecha_limite: new Date(newTask.value.fechaLimite),
            completado: false // Inicialmente, la tarea no está completada
        });
        // Limpiar el formulario
        newTask.value = { tarea: '', cliente: '', precio: '', fechaInicio: '', fechaLimite: '' };
        fetchTodos(); // Volver a obtener la lista de tareas
    } catch (error) {
        console.error('Error al agregar la tarea:', error);
    }
};

// Función para actualizar el estado de completado de una tarea
const updateTaskStatus = async (todo) => {
    try {
        const taskDoc = doc(db, 'to-do', todo.id); // Referencia al documento
        await updateDoc(taskDoc, { completado: todo.completado }); // Actualizar el campo completado
    } catch (error) {
        console.error('Error al actualizar el estado de la tarea:', error);
    }
};

// Llamar a fetchTodos cuando el componente se monte
onMounted(fetchTodos);
</script>

<template>
    <h1>To-Do</h1>
    <form @submit="addTask" class="mb-4">
        <div class="mb-3">
            <input type="text" v-model="newTask.tarea" class="form-control" placeholder="Ingrese su tarea" required>
        </div>
        <div class="mb-3">
            <input type="text" v-model="newTask.cliente" class="form-control" placeholder="Ingrese su cliente" required>
        </div>
        <div class="mb-3">
            <input type="number" v-model="newTask.precio" class="form-control" placeholder="Ingrese su precio" required>
        </div>
        <div class="mb-3">
            <input type="date" v-model="newTask.fechaInicio" class="form-control" required>
        </div>
        <div class="mb-3">
            <input type="date" v-model="newTask.fechaLimite" class="form-control" required>
        </div>
        <input type="submit" class="btn btn-primary" value="Enviar">
    </form>
    
    <table class="table table-striped">
        <thead>
            <tr>
                <th>Completar</th>
                <th>ID</th>
                <th>Fecha Inicio</th>
                <th>Fecha Limite</th>
                <th>Cliente</th>
                <th>Tarea</th>
                <th>Precio</th>
                <th>Estado</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="todo in todos" :key="todo.id">
                <td>
                    <input type="checkbox" v-model="todo.completado" @change="updateTaskStatus(todo)">
                </td>
                <td>{{ todo.id }}</td>
                <td>{{ todo.fechaInicio }}</td>
                <td>{{ todo.fechaLimite }}</td>
                <td>{{ todo.cliente }}</td>
                <td>{{ todo.tarea }}</td>
                <td>{{ todo.precio }}</td>
                <td>{{ todo.estado }}</td>
            </tr>
        </tbody>
    </table>
</template>
