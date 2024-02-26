<script setup lang="ts">
	import { Ref, ref, onMounted } from 'vue';
	import { doc, writeBatch } from 'firebase/firestore';
	import db from '../../../firebase/config';
	import { ListItem, BatchMenu } from '../';
	import { Todo } from './interface';
	import { useNotification } from '@kyvg/vue3-notification';

	const props = defineProps<{
		todos: Array<Todo>,
		getTodos: Function,
		loadingTodos: Boolean,
	}>()

	// Este estado guarda si hay algún checkbox seleccionado de cualquier Todo
	const selectedTodos: Ref<Array<Todo>> = ref([]);
	const anyCheckboxSelected = ref(false);
	const checkboxSelectedAmount = ref(0);
	const selectAllCheckbox = ref(false);

	const { notify } = useNotification();
	
	// Esta función comprueba si existe algún checkbox seleccionado entre todos los Todos
	const checkCheckboxTodos = () => {
		const selectedValues = props.todos.map(todo => todo.selected);

		// Almacena en un estado si selectedValues tiene algún elemento en true
		anyCheckboxSelected.value = selectedValues.includes(true);

		checkboxSelectedAmount.value = selectedValues.filter(value => value === true).length;

		// Si se seleccionan todos los todos, se cambia el valor a true
		selectAllCheckbox.value = !selectedValues.includes(false);

		selectedTodos.value = props.todos.filter(todo => todo.selected);
	}
	
	const toggleSelectAll = () => {
		selectAllCheckbox.value = !selectAllCheckbox.value;

		props.todos.forEach(todo => {
			todo.selected = selectAllCheckbox.value;
		});

		checkCheckboxTodos();
	}

	const completeMany = async () => {
		// Si no hay todos seleccionados, sale de la función
		if(selectedTodos.value.length === 0) return;
		
		// Se declara batch dentro de la función para que no cause error al hacer el commit, si se declarara fuera tendría alcance global y se guardaría el commit() causando error
		const batch = writeBatch(db);

		selectedTodos.value.forEach(selectedTodo => {
			const ref = doc(db, "todos", selectedTodo.id);
			
			batch.update(ref, {"completed": true});
		});

		try {
			// Manda la información a la DB
			await batch.commit();
		} catch(err){
			notify({
				type: "error",
				title: "Error",
				text: "Ocurrió un error al completar las tareas"
			})
			
			console.log(err);
		}

		// Resetea los valores
		selectedTodos.value = [];
		anyCheckboxSelected.value = false;
		checkboxSelectedAmount.value = 0;

		// Actualiza los todos
		props.getTodos();

		return;
	}

	const decompleteMany = async () => {
		// Si no hay todos seleccionados, sale de la función
		if(selectedTodos.value.length === 0) return;
		
		// Se declara batch dentro de la función para que no cause error al hacer el commit, si se declarara fuera tendría alcance global y se guardaría el commit() causando error
		const batch = writeBatch(db);

		selectedTodos.value.forEach(selectedTodo => {
			const ref = doc(db, "todos", selectedTodo.id);
			
			batch.update(ref, {"completed": false});
		});

		try {
			// Manda la información a la DB
			await batch.commit();
		} catch(err){
			notify({
				type: "error",
				title: "Error",
				text: "Ocurrió un error al descompletar las tareas"
			})

			console.log(err);
		}

		// Resetea los valores
		selectedTodos.value = [];
		anyCheckboxSelected.value = false;
		checkboxSelectedAmount.value = 0;

		// Actualiza los todos
		props.getTodos(); 
	}

	const deleteMany = async () => {
		// Si no hay todos seleccionados, sale de la función
		if(selectedTodos.value.length === 0) return;
		
		// Se declara batch dentro de la función para que no cause error al hacer el commit, si se declarara fuera tendría alcance global y se guardaría el commit() causando error
		const batch = writeBatch(db);

		selectedTodos.value.forEach(selectedTodo => {
			const ref = doc(db, "todos", selectedTodo.id);
			
			batch.delete(ref);
		});

		try {
			// Manda la información a la DB
			await batch.commit();
		} catch(err){
			notify({
				type: "error",
				title: "Error",
				text: "Ocurrió un error al eliminar las tareas"
			})

			console.log(err);
		}

		// Resetea los valores
		selectedTodos.value = [];
		anyCheckboxSelected.value = false;
		checkboxSelectedAmount.value = 0;

		// Actualiza los todos
		props.getTodos(); 
	}

	// En cuando se monta el componente, se revisan los Todos
	onMounted(() => {
		checkCheckboxTodos();
	});

</script>


<template>
	
	<div>
		<BatchMenu :anyCheckboxSelected="anyCheckboxSelected" :checkboxSelectedAmount="checkboxSelectedAmount" :selectAllCheckbox="selectAllCheckbox" :toggleSelectAll="toggleSelectAll" :completeMany="completeMany" :decompleteMany="decompleteMany" :deleteMany="deleteMany" />

		<ul class="flex flex-col gap-2">
			<!-- TODO: Colocar skeleton de bootstrap -->
			<template v-if="loadingTodos">
				<span class="text-center py-8">Cargando...</span>
			</template>

			<template v-if="!loadingTodos" v-for="todo in props.todos" :key="todo.id">
				<ListItem :getTodos="getTodos" :todo="todo" :checkCheckboxTodos="checkCheckboxTodos" :anyCheckboxSelected="anyCheckboxSelected" />
			</template>

			<template v-if="!loadingTodos && todos.length === 0">
				<span class="text-center">No se encontraron tareas pendientes. Registre alguna.</span>
			</template>
		</ul>
	</div>
</template>


<style scoped>

</style>