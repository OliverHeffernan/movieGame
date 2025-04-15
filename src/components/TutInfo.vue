<template>
	<div id="tut" :class="open ? 'open' : 'closed'">
		<ul>
			<li>Enter your guess in the input field at the bottom of the screen.</li>
			<li>You will get a hint after every guess. And the poster will become less blurred.</li>
			<li>If you have no clue you can press the "skip for hint" button, to sip this guess and get a hint.</li>
			<li>You have a maximum of 6 guesses.</li>
		</ul>
		<button @click="closeTut">Let's Go!</button>
	</div>
</template>
<script setup>
import { ref } from "vue";

const open = ref(true);

function closeTut() {
	open.value = false;
}
</script>
<style scoped>
#tut {
	position: absolute;
	z-index: 5;

	width: min(calc(100% - 80px), 800px);
	
	left: 50%;
	top: 50%;
	transform: translate(-50%, -50%);

	background-color: black;
	display: block;
	padding: 20px;

	border-radius: 20px;

	border: 1px solid rgb(60,60,60);
	transition: all 0.2s;
}

.open {
	opacity: 1;
}

.closed {
	opacity: 0;
	display: none;
}

#tut ul {
	list-style-position: inside;
}

#tut ul li {
	margin-bottom: 10px;
}

#tut ul li:last-child {
	margin-bottom: 0;
}

#tut button {
	width: 100%;
	background-color: #1A82B4;
	border: none;
	font-size: 20px;
	padding: 10px 0;
	margin-top: 10px;
	border-radius: 10px;
}
</style>
