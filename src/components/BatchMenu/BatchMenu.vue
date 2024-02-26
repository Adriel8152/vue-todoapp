<script setup lang="ts">
	const props = defineProps<{
		anyCheckboxSelected: Boolean,
		checkboxSelectedAmount: number,
		toggleSelectAll: Function,
		selectAllCheckbox: Boolean,
		completeMany: Function,
		decompleteMany: Function,
		deleteMany: Function,
	}>();
</script>

<template>
	<div :class="{anyCheckboxSelected: props.anyCheckboxSelected}" class="flex flex-row overflow-x-scroll pb-3 justify-center opacity-0 pointer-events-none">
	<!-- Opción de todos seleccionados -->
		<span @click="props.toggleSelectAll" class="text-sm py-2 px-4 rounded-sm cursor-pointer hover:bg-white hover:bg-opacity-5 whitespace-nowrap">
			<input type="checkbox" v-model="props.selectAllCheckbox" class="before:content[''] translate-y-0.5 peer relative cursor-pointer appearance-none bg-white text-current w-4 h-4 border border-slate-500 border-solid rounded-sm transition-all before:flex before:h-full before:w-full before:border-2 before:border-white before:rounded-sm checked:border-indigo-500 checked:bg-indigo-500 checked:before:bg-indigo-500 mr-2"/>
			{{ props.checkboxSelectedAmount + " Seleccionado"}}{{ props.checkboxSelectedAmount > 1 ? "s" : '' }}
		</span>

		<!-- Opción de completar todos -->
		<span @click="props.completeMany" class="text-sm py-2 px-4 rounded-sm cursor-pointer hover:bg-white hover:bg-opacity-5 whitespace-nowrap text-blue-400"><i class="bi bi-check2-square pointer-events-none mr-2"></i>Completar</span>
		
		<!-- Opción de descompletar todos -->
		<span @click="props.decompleteMany" class="text-sm py-2 px-4 rounded-sm cursor-pointer hover:bg-white hover:bg-opacity-5 whitespace-nowrap text-orange-300"><i class="bi bi-x-square pointer-events-none mr-2"></i>Descompletar</span>
		
		<!-- Opción de eliminar todos -->
		<span @click="props.deleteMany" class="text-sm py-2 px-4 rounded-sm cursor-pointer hover:bg-white hover:bg-opacity-5 whitespace-nowrap text-red-400"><i class="bi bi-trash3 pointer-events-none mr-2"></i>Eliminar</span>
	</div>
</template>

<style scoped>
	.anyCheckboxSelected {
		opacity: 1;
		pointer-events: all;
	}
</style>