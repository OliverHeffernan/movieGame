<template>
	<div id="sample" :class="connection ? 'close' : 'open'"><i class="fa-solid fa-exclamation"></i><p>Failed to connect to the server. Using sample data.</p></div>
</template>
<script setup>
import { defineProps } from "vue";

defineProps(["connection"]);
</script>
<style scoped>
#sample {
	position: fixed;

	background-color: #0F0F0F;
	padding: 8px;
	border-radius: 0 0 8px 0;
	color: #D57575;
	z-index: 10;
	display: flex;
	transition: all 0.2s;
}

#sample i {
	color: #D57575;
}

#sample p {
	display: block;
	padding-left: 10px;
}

.open {
	opacity: 1;
	top: 0;
	left: 0;
}

.close {
	opacity: 0;
	top: -500px;
	left: -500px;
}
</style>
