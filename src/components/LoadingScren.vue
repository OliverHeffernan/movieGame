<template>
	<div id="load" :class="open ? 'open' : 'closed'">
		<div id="blob">
			<i class="fa-solid fa-spinner"></i>
			<h4>{{ messages[getRandomInt(0, messages.length - 1)] }}</h4>
		</div>
	</div>
</template>
<script setup>
import { defineProps } from "vue";
defineProps(['open']);

function getRandomInt(a, b) {
	const min = Math.ceil(Math.min(a, b));
	const max = Math.floor(Math.max(a, b));
	return Math.floor(Math.random() * (max - min + 1)) + min;
}

const messages = [
	"Warming up the Oscars… for your performance.",
	"Cue the dramatic music… loading in progress.",
	"Casting you as the lead role… of this guessing game.",
	"Loading the trivia… may the film gods be with you.",
	"Loading like a movie sequel no one asked for.",
	"Trying to remember that actor’s name? Yeah, us too.",
	"You’re the main character now. Don’t mess it up.",
	"Movie knowledge loading... please stand by for greatness.",
	"If this takes too long, blame the CGI.",
	"This loading screen has better pacing than some films.",
	"This is the way.",
	"I am Groot.",
	"One does not simply… skip this loading screen.",
	"To infinity... and beyon-wait, still loading.",
	"The name's Screen, Loading Screen.",
];


</script>

<style scoped>
#load {
	position: relative;
	width: 100%;
	height: 100%;
	background-color: black;

	transition: all 0.2s;
}

@keyframes spin {
	from {
		rotate: 0deg;
	}
	to {
		rotate: 360deg;
	}
}

#load i {
	font-size: 30px;
	width: 100%;
	text-align: center;
	animation-name: spin;
	animation-duration: 2s;
	animation-iteration-count: infinite;
}

#load #blob {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
}

h4 {
	text-align: center;
	margin-top: 10px;
}

.open {
	opacity: 1;
}

.closed {
	opacity: 0;
	display: none;
}
</style>
