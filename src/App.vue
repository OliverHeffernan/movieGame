<template>
	<div id="phoneContainer">
		<div class="vert">
			<div id="imgContainer">
				<img id="moviePoster" :class="correct ? '' : 'guess' + guessCount" alt="">
				<div id="loading" v-if="!movieLoaded">Finding a film...</div>
			</div>
			<div id="guessBox" class="closed">
				<button id="guessButton" @click="doGuess">Guess ({{guessCount}}/6)</button>
				<input placeholder="Guess the movie" type="text" id="guessInput">
				<div id="hintsContainer">
					<div id="hints">
						<div id="topBar"></div>
					</div>
				</div>
			</div>
			<button v-if="guessCount > 0" id="scrollHintsButton" @click="scrollHints()">View your hints</button>
		</div>
	</div>
</template>

<script setup>
import "./styles/main.css";

import { ref } from "vue";
import SampleData from "./classes/sampleData.js";
const data = new SampleData();
let guessMovie = ref("");
let credits;
let director;
let guessCount = ref(0);
let correct = false;

let connection = true;
let sampleNumber = 0;

let movieLoaded = ref(false);

function stripPunctuation(str) {
	let result = String(str).replace(/[^\w\s]/g, '').toLowerCase();
	return result;
}

function doGuess() {
	let guess = document.getElementById("guessInput").value;
	if (guess == "") {
		return;
	}
	guessCount.value++;
	if (stripPunctuation(guess) == stripPunctuation(guessMovie.value.title)) {
		document.getElementById("moviePoster").className = "";
		addHint(guess + " is correct!!");
		correct = true;
	} else {
		doHint(guessCount.value, guess);
	}

	scrollHints();
}

function getImage(movie) {
	return "https://image.tmdb.org/t/p/w600_and_h900_bestv2/" + movie.poster_path;
}

function getHint(count) {
	if (count == 1) {
		return "It has actor " + credits.cast[2].name + " in it.";
	}
	if (count == 2) {
		return "The movie was released on " + guessMovie.value.release_date;
	}
	if (count == 3) {
		return "The director is " + director;
	}
	if (count == 4) {
		return "It also has actor " + credits.cast[1].name + " in it.";
	}
	if (count == 5) {
		return "It also has the actor " + credits.cast[0].name + " in it.";
	}
}

function doHint(count, guess) {
	if (count == 6) {
		addHint("❌" + guess + "The answer is " + guessMovie.value.title);
	}
	else if (count < 6) {
		addHint("❌" + guess + "Hint: " + getHint(count));
	}
}

function addHint(text) {
	const ul = document.getElementById("hints");
	const li = document.createElement('p');
	li.innerText = text;
	ul.appendChild(li);
}

function getRandomInt(a, b) {
	const min = Math.ceil(Math.min(a, b));
	const max = Math.floor(Math.max(a, b));
	return Math.floor(Math.random() * (max - min + 1)) + min;
}

async function fetchResults() {
	try {
		const controller = new AbortController();
		const timeout = 5000;
		const timeoutId = setTimeout(() => controller.abort(), timeout);

		//const response = await fetch('https://192.168.68.59:8443/movie', {
		const response = await fetch('https://settling-donkey-brightly.ngrok-free.app/movie', {
			method: 'GET',
			signal: controller.signal,
			headers: {
				"ngrok-skip-browser-warning": "true"
			}
		});


		clearTimeout(timeoutId);

		if (!response.ok) {
			throw new Error(`HTTP error! Status: ${response.status}`);
		}

		const text = await response.text();
		console.log(text);
		return JSON.parse(text.substring(8));
	} catch (error) {
		console.error('Error fetching result:', error);
		//return "ERROR";
		connection = false;
		//console.log("connection failed");
		sampleNumber = getRandomInt(0, data.data.length - 1);
		//console.log(sampleNumber);
		return data.data[sampleNumber].movie;
	}
}

async function fetchCredits(id) {
	try {
		if (!connection) {
			throw new Error('No connection available');
		}

		const response = await fetch('https://192.168.68.59:8443/credits/' + id, {
			method: 'GET',
		});

		if (!response.ok) {
			throw new Error(`HTTP error! Status: ${response.status}`);
		}
		const text = await response.text();
		return JSON.parse(text.substring(8));
	} catch (error) {
		//console.error('Error fetching credits:', error);
		//return "ERROR";
		return data.data[sampleNumber];
	}
}

async function getMovie() {
	let movie = await fetchResults();

	while (movie == null || (movie.success == false || movie.adult || movie.poster_path == null || movie.original_language != "en" || movie.vote_average < 7.8 || movie.vote_count < 1000 || parseInt(movie.release_date.split("-")[0]) < 1965 )) {
		movie = await fetchResults();
		if (movie != null) {
			console.log(movie);
		}
	}
	guessMovie.value = movie;
	credits = await fetchCredits(movie.id);


	//console.log(movie.poster_path);
	document.getElementById("moviePoster").src = getImage(movie);

	for (let i = 0; i < credits.crew.length; i++) {
		if (credits.crew[i].job == "Director" || credits.crew[i].known_for_department == "Directing") {
			director = credits.crew[i].name;
			break;
		}
	}

	movieLoaded.value = true;
	document.getElementById("guessBox").className = "open";
}

function scrollHints() {
	console.log("scrolled");
	let elm = document.getElementById("hintsContainer");
	elm.scrollTo({ top: elm.scrollHeight, behaviour: 'smooth' });
}

/*
onMounted(() => {
	let elm = document.getElementById("hintsContainer");
	elm.addEventListener('scroll', () => {
		const halfway = elm.scrollHeight / 3;
		const current = elm.scrollTop + elm.clientHeight;

		if (current < halfway) {
			elm.scrollTo({ top: halfway, behavious: 'smooth' });
		}
	});
});
*/

getMovie();

</script>
