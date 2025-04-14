<template>
	<div id="phoneContainer" @scroll="loadMore">
		<MovieContainer v-for="n in numLoaded" :key="n" :index="n - 1" />
	</div>
</template>

<script setup>
import "./styles/main.css";
import MovieContainer from "./components/MovieContainer.vue";
import { ref } from "vue";

const numLoaded = ref(10);

let loading = false;

async function loadMore() {
	if (!loading) {
		numLoaded.value++;
		loading = true;
		setTimeout(() => {
			loading = false;
		}, 1000);
	}
}
</script>
