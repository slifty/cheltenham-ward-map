<template>
	<WardMap />
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import WardMap from './components/WardMap.vue'

export default defineComponent({
	name: 'App',
	components: {
		WardMap,
	},
})
</script>

<style>
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	height: 100%;
}
html, body { height: 100%; }
</style>
