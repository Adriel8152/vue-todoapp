<script setup lang="ts">
	const props = defineProps<{
		zIndex: string,
		toggleSubmenuOpen: Function,
		showClickAway: Boolean
	}>();
</script>

<template>
	<div v-if="props.showClickAway" @click="props.toggleSubmenuOpen" :style="{zIndex: props.zIndex}" class="absolute left-0 top-0 flex w-screen h-screen bg-black bg-opacity-20 overflow-hidden" />
</template>

<style>

</style>