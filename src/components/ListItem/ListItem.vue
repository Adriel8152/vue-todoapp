<script setup lang="ts">
	import { Ref, ref } from 'vue';
	import { deleteDoc, doc, updateDoc } from 'firebase/firestore';
	import db from '../../../firebase/config.ts';
	import { Todo } from '../List/interface';
	import { ClickAwayListener } from '../';
	import { useNotification } from '@kyvg/vue3-notification';

	const props = defineProps<{
		todo: Todo,
		getTodos: Function,
		checkCheckboxTodos: Function,
		anyCheckboxSelected: Boolean,
	}>();

	const { notify } = useNotification();

	const submenuOpen: Ref<boolean> = ref(false);

	const toggleSubmenuOpen = () => {
		submenuOpen.value = !submenuOpen.value
	}

	const toggleComplete = async () => {
		try {
			await updateDoc(doc(db, "todos", props.todo.id), {
				completed: !props.todo.completed,
			})
	
			props.getTodos();
		} catch(error) {
			notify({
				type: "error",
				title: "Error",
				text: "Ocurrió un error al actualizar la tarea"
			})

			console.log(error);
		}
	}

	const handleDelete = async () => {
		try {
			// Devuelve undefined
			await deleteDoc(doc(db, "todos", props.todo.id));

			props.getTodos();
		} catch(error) {
			notify({
				type: "error",
				title: "Error",
				text: "Ocurrió un error al eliminar la tarea"
			})

			console.error(error)
		}
	}

</script>


<template>
	<ClickAwayListener zIndex="3" :showClickAway="submenuOpen" :toggleSubmenuOpen="toggleSubmenuOpen" />

	<li :class="{selected: props.todo.selected, completed: props.todo.completed}" class="todo_item bg-slate-200 text-gray-950 rounded-xl py-3 px-6 list-none flex gap-2 items-center">
		<!-- Checkbox para seleccionar varios todos -->
		<span>
			<input type="checkbox" @change="props.checkCheckboxTodos" v-model="props.todo.selected"
        class="before:content[''] translate-y-0.5 peer relative cursor-pointer appearance-none bg-white text-current w-4 h-4 border border-slate-500 border-solid rounded-sm transition-all before:flex before:h-full before:w-full before:border-2 before:border-white before:rounded-sm checked:border-indigo-500 checked:bg-indigo-500 checked:before:bg-indigo-500" />
		</span>
		
		<!-- Título -->
		<span :class="{selected: props.anyCheckboxSelected}" class="todo_title_container bg-inherit transition-transform flex-1">
			<span class="todo_title">{{ props.todo.title }}</span>
			<span v-if="todo.completed" class="ml-2 text-blue-600 font-semibold text-xs"> - <i class="bi bi-check2-square"></i> Completada</span>
		</span>

		<!-- Submenú -->
		<div :class="{button_submenu_active: submenuOpen}" class="rounded-full aspect-square text-gray-900 flex place-content-center transition-colors bg-black bg-opacity-0 hover:bg-opacity-10 cursor-pointer relative">
			<i @click="toggleSubmenuOpen" class="bi bi-three-dots-vertical flex p-2 rounded-full"></i>
			<div :class="{submenu_active: submenuOpen}" class="submenu absolute z-10 bg-white rounded-md p-1 top-full right-0 flex flex-col border-solid border border-slate-300 shadow-md">

				<span @click="toggleComplete" class="transition-colors bg-black bg-opacity-0 hover:bg-opacity-10 py-2 px-4 select-none">
					<span v-if="!props.todo.completed" class="text-blue-600 flex flex-nowrap gap-3 items-center whitespace-nowrap"><i class="bi bi-check2-square pointer-events-none"></i> Completar</span>
					<span v-if="props.todo.completed" class="text-orange-600 flex flex-nowrap gap-3 items-center whitespace-nowrap"><i class="bi bi-x-square pointer-events-none"></i> Descompletar</span>
				</span>
				<span @click="handleDelete" class="flex flex-nowrap gap-3 items-center whitespace-nowrap transition-colors bg-black bg-opacity-0 hover:bg-opacity-10 py-2 px-4 text-red-600 select-none">
						<i class="bi bi-trash-fill"></i> Eliminar
				</span>
			</div>
		</div>
	
	</li>
</template>


<style scoped>
	/* Estilos cuando un submenu está activo */
	.button_submenu_active {
		background-color: rgba(0,0,0,0.1);
	}
	
	.submenu {
		overflow: hidden;
		clip-path: polygon(100% 0, 100% 0, 100% 0, 100% 0);
		transition: clip-path 0.1s ease-in-out;
	}
	
	.submenu_active {
		clip-path: polygon(0 0, 100% 0, 100% 100%, 0 100%);
	}

	.todo_item.selected {
		box-shadow: 0 0 0 3.5px #6366f1 inset;
	}

	.todo_item:hover .todo_title_container,
	.todo_title_container.selected {
		transform: translateX(0px);
	}

	.todo_item.completed .todo_title {
		font-style: italic;
		text-decoration: line-through;
	}

	.todo_title_container {
		transform: translateX(-24px);
	}
</style>