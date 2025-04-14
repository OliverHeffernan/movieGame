
<template>
	<div class="vertContainer">
		<div class="vert">
			<img :src="getImage(movie)" name="moviePosterBlurredBg" class="moviePosterBlurredBg" alt="">
			<div id="imgContainer" @mousedown="hideInfo()" @mouseup="showInfo()">
				<img :src="getImage(props.movie)" name="moviePoster" id="moviePoster" :class="correct ? '' : 'guess' + guessCount" alt="">
				<!--<div id="loading" v-if="!movieLoaded">Finding a film...</div>-->
			</div>
			<div id="guessBox" :class="guessBoxShowing ? 'open' : 'closed'">
				<button id="guessButton" @click="doGuess">{{guessInput == "" ? 'Skip for hint' : 'Guess'}} ({{guessCount}}/6)</button>
				<input v-model="guessInput" name="guessInput" placeholder="Guess the movie" type="text" id="guessInput" v-on:keyup.enter="doGuess()">
				<div id="hintsContainer" class="hintsContainer">
					<!--
					<div id="hints" class="hints">
						<div id="topBar"></div>
					</div>
					-->
					<div id="hints" class="hints">
						<p v-for="(hint, n) in hints" :key="n">{{hint}}</p>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script setup>

import { ref, defineProps } from "vue";
//import SampleData from "../classes/sampleData.js";

const props = defineProps(['index', 'movie', 'credits']);

//const data = new SampleData();
let director = getDirector(props.credits);
let guessCount = ref(0);
let correct = false;

//let connection = true;
//let sampleNumber = 0;

let guessBoxShowing = ref(true);

const guessInput = ref("");

const hints = ref([]);
function getDirector(credits) {
	let result = "";
	for (let i = 0; i < credits.crew.length; i++) {
		if (credits.crew[i].job == "Director" || credits.crew[i].known_for_department == "Directing") {
			result = credits.crew[i].name;
			break;
		}
	}

	return result;
}

function hideInfo() {
	guessBoxShowing.value = false;
}

function showInfo() {
	guessBoxShowing.value = true;
}

function stripPunctuation(str) {
	let result = String(str).replace(/[^\w\s]/g, '').toLowerCase();
	return result;
}

function doGuess() {
	const guess = guessInput.value;
	guessInput.value = "";
	if (guessCount.value >= 6 || correct) {
		return;
	}

	guessCount.value++;
	if (stripPunctuation(guess) == stripPunctuation(props.movie.title) || guess.toLowerCase() == props.movie.title.toLowerCase()) {
		addHint(guess + " is correct!!");
		correct = true;
	} else {
		doHint(guessCount.value, guess);
	}
}

function getImage(movie) {
	return "https://image.tmdb.org/t/p/w600_and_h900_bestv2/" + movie.poster_path;
}

function getHint(count) {
	if (count == 1) {
		return "It has actor " + props.credits.cast[2].name + " in it.";
	}
	if (count == 2) {
		return "The movie was released on " + props.movie.release_date;
	}
	if (count == 3) {
		return "The director is " + director;
	}
	if (count == 4) {
		return "It also has actor " + props.credits.cast[1].name + " in it.";
	}
	if (count == 5) {
		return "It also has the actor " + props.credits.cast[0].name + " in it.";
	}
}

function doHint(count, guess) {
	if (count == 6) {
		addHint("❌ " + guess + ". The answer is " + props.movie.title);
	}
	else if (count < 6) {
		addHint("❌ " + guess + ".💡 " + getHint(count));
	}
}

function addHint(text) {
	/*
	const ul = document.getElementsByClassName("hints")[props.index];
	const li = document.createElement('p');
	li.innerText = text;
	ul.appendChild(li);
	*/

	hints.value.push(text);
}

</script>
