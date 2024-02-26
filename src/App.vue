<script setup lang="ts">
  import db from '../firebase/config';
  import { getDocs, collection } from 'firebase/firestore';
  import { Ref, ref, onMounted } from 'vue';
  import { InputAddTodo, List } from './components/index.ts';
	import { Todo } from './components/List/interface';
	import { Notifications, useNotification } from '@kyvg/vue3-notification';

  const todoData: Ref<Todo[]> = ref([]);
  const loadingTodos: Ref<boolean> = ref(false);

	const { notify } = useNotification();

	const getTodos: Function = async () => {
    // Limpia la lista de Todos
    todoData.value = [];
    loadingTodos.value = true;

		try {
			const querySnapshot = await getDocs(collection(db, "todos"));
	
			querySnapshot.forEach(doc => {
				// Se crea un nuevo objeto con la interfaz de Todo
				const TODO: Todo = {
					id: doc.id || '',
					title: doc.data().title || null,
					completed: doc.data().completed,
					selected: false,
				}
	
				// Se guardan en el estado
				todoData.value?.push(TODO);
			});
		} catch(error) {
			notify({
				type: "error",
				title: "Error",
				text: "Ocurrió un error al obtener las tareas"
			})

			console.log(error);
		}

    loadingTodos.value = false;
	}

	onMounted(() => {
		getTodos();
	})
</script>

<template>

	<main class="flex m-auto p-8 max-w-3xl flex-col gap-4">
		<h2 class="text-center text-4xl mb-3">ToDo App</h2>
    <InputAddTodo :getTodos="getTodos" />
    <hr />
    <List :getTodos="getTodos" :loadingTodos="loadingTodos" :todos="todoData" />
  </main>
	
	<hr class="border-gray-700" />
	<div class="flex items-center justify-center h-20">
		<span class="text-zinc-300"><a href="https://github.com/Adriel8152/vue-todoapp" target="_blank" noreferrer noopener class="p-4 hover:underline"><i class="bi bi-github mr-2"></i> Ir al repositorio</a></span>
	</div>
  
	<Notifications close-on-click position="top right" classes="my-notification" />
</template>

<style scoped>

</style>