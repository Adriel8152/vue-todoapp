<script setup lang="ts">
	import { addDoc, collection, Timestamp } from 'firebase/firestore';
	import { Ref, ref } from 'vue';
	import { useNotification } from '@kyvg/vue3-notification';
	import db from '../../../firebase/config.ts';


	const props = defineProps<{
		getTodos: Function,
	}>()


	const inputInitialValue = "";
	const todoInput: Ref<string> = ref(inputInitialValue);

	const { notify } = useNotification();

	// Se usa mediante el evento @input
	// const handleChangeInput = () => {
	// 	console.log(addTodoInput.value)
	// }


	// TODO: Poner fecha de creación del TODO para organizarlos al mostrarlos
	const addTodo = async (value: string) => {
		try {
			await addDoc(collection(db, "todos"), {
				title: value,
				completed: false,
				createdAt: Timestamp.now(),
			});

			notify({
				type: "success",
				title: "Tarea agregada!",
				text: "La tarea se agregó con éxito"
			})
		} catch(error) {
			notify({
				type: "error",
				title: "Error",
				text: "Ocurrió un error al guardar la tarea"
			})

			console.error("Error en la petición: ", error);
		}
	}


	const clearInput = () => {
		todoInput.value = inputInitialValue;
	}


	const handleAddTodo = (e: Event) => {
		e.preventDefault();
		if(!Boolean(todoInput.value)) return;

		addTodo(todoInput.value);

		// Se llama a la función para actualizar nuevamente la lista de Todos
		props.getTodos();

		clearInput();
	}
</script>


<template>
	<form @submit="handleAddTodo" class="flex items-center gap-4 justify-center">
		<input v-model="todoInput" class="flex-1 min-h-12 rounded-3xl py-2 px-4 text-gray-900" placeholder="Agregar una tarea" />
		<button class="bg-indigo-600 py-2 px-6 rounded-lg transition-colors leading-0 hover:bg-indigo-700 active:bg-indigo-800">Agregar</button>
	</form>
</template>


<style scoped>

</style>