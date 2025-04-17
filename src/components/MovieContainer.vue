
<template>
	<div class="vertContainer">
		<div class="vert">
			<img 
				:src="getImage(movie)"
				class="moviePosterBlurredBg"
				alt=""
			>
			<div
				id="imgContainer"
				@mousedown="hideInfo()"
				@mouseup="showInfo()"
				@touchstart="hideInfo()"
				@touchend="showInfo()"
			>
				<img
					:src="getImage(props.movie)"
					id="moviePoster"
					:class="correct ? '' : 'guess' + guessCount"
					alt=""
				>
			</div>
			<div
				id="guessBox"
				:class="guessBoxShowing ? 'open' : 'closed'"
			>
				<button 
					id="guessButton"
					@click="doGuess">{{guessInput == "" ? 'Skip for hint' : 'Guess'}} ({{guessCount}}/6)
				</button>
				<input 
					v-model="guessInput" 
					name="guessInput" 
					placeholder="Guess the movie" 
					type="text" 
					id="guessInput" 
					v-on:keyup.enter="doGuess()"
				>
				<div
					id="hintsContainer"
					class="hintsContainer"
				>
					<div
						id="hints"
						class="hints"
					>
						<p
							v-for="(hint, n) in hints"
							:key="n"
						>
							{{hint}}
						</p>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script setup>

import { ref, defineProps } from "vue";
//import SampleData from "../classes/sampleData.js";

const props = defineProps(['index', 'movie']);

//const data = new SampleData();
let guessCount = ref(0);
let correct = false;

let guessBoxShowing = ref(true);

const guessInput = ref("");

const hints = ref([]);

function hideInfo() {
	guessBoxShowing.value = false;
}

function showInfo() {
	guessBoxShowing.value = true;
}

function stripPunctuation(str) {
	const diacriticsMap = {
		'ā': 'a', 'ē': 'e', 'ī': 'i', 'ō': 'o', 'ū': 'u', // Macrons
		'á': 'a', 'à': 'a', 'ä': 'a', 'â': 'a', 'ã': 'a', 'å': 'a', // Accents
		'é': 'e', 'è': 'e', 'ë': 'e', 'ê': 'e', // Accents
		'í': 'i', 'ì': 'i', 'ï': 'i', 'î': 'i', // Accents
		'ó': 'o', 'ò': 'o', 'ö': 'o', 'ô': 'o', 'õ': 'o', // Accents
		'ú': 'u', 'ù': 'u', 'ü': 'u', 'û': 'u', // Accents
		'ç': 'c', 'ñ': 'n', 'ş': 's', 'ý': 'y', // Other accents and letters
		'Á': 'A', 'À': 'A', 'Ä': 'A', 'Â': 'A', 'Ã': 'A', 'Å': 'A', // Uppercase accents
		'É': 'E', 'È': 'E', 'Ë': 'E', 'Ê': 'E', // Uppercase accents
		'Í': 'I', 'Ì': 'I', 'Ï': 'I', 'Î': 'I', // Uppercase accents
		'Ó': 'O', 'Ò': 'O', 'Ö': 'O', 'Ô': 'O', 'Õ': 'O', // Uppercase accents
		'Ú': 'U', 'Ù': 'U', 'Ü': 'U', 'Û': 'U', // Uppercase accents
		'Ç': 'C', 'Ñ': 'N', 'Ş': 'S', 'Ý': 'Y', // Uppercase other accents
	};

	let result = str.replace(/[āēīōūáàäâãåéèëêíìïîóòöôõúùüûçñşýÁÀÄÂÃÅÉÈËÊÍÌÏÎÓÒÖÔÕÚÙÜÛÇÑŞÝ]/g, match => diacriticsMap[match] || match);
	result = String(result).replace(/[^\w\s]/g, '').toLowerCase();
	const nilWords = [
		"and",
		"the",
		"a",
		"an",
		"of"
	];

	let words = result.split(" ");
	let resWords = [];
	words.forEach((word) => {
		if (!nilWords.includes(word)) {
			resWords.push(word);
		}
	});

	result = resWords.join('');

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
		return "It has actor " + props.movie.cast[2] + " in it.";
	}
	if (count == 2) {
		return "The movie was released on " + props.movie.release_date;
	}
	if (count == 3) {
		return "The director is " + props.movie.director;
	}
	if (count == 4) {
		return "It also has actor " + props.movie.cast[1] + " in it.";
	}
	if (count == 5) {
		return "It also has the actor " + props.movie.cast[0] + " in it.";
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
	hints.value.push(text);
}

</script>
