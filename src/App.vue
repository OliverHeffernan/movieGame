<template>
	<div class="vert">
		<img id="moviePoster" class="guess0" alt="">
		<div id="loading" v-if="!movieLoaded">Finding a film...</div>
		<input placeholder="Guess the movie" type="text" id="guessInput">
		<button id="guessButton" @click="doGuess">Guess</button>
		<div id="hints">
		</div>
	</div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,100..900;1,100..900&display=swap');
* {
	background-color: rgb(25,25,25);
	color: white;
	font-family: "Roboto", sans-serif;
}

#loading {
	width: min(calc(100vw - 20px), 400px);
	height: min(calc((100vw - 20px)*1.5), 600px);
	margin-left: 50vw;
	transform: translateX(-50%);
	display: flex;
	align-items: center;
	justify-content: center;
	text-align: center;
	background-color: black;
}

#hints {
	margin-left: 50vw;
	transform: translateX(-50%);
	width: min(calc(100vw - 20px), 400px);
	list-style: none;
}

#hints p {
	padding: 10px;
	border-top: 2px solid rgb(50,50,50);
}

#guessInput {
	margin-left: 50vw;
	transform: translateX(-50%);
	width: min(calc(100vw - 20px), 400px);
	border: 2px solid rgb(50,50,50);
	padding: 10px;
	border-radius: 5px;
	font-size: 20px;
	margin-top: 10px;
}

#moviePoster {
	margin-left: 50vw;
	transform: translateX(-50%);
	width: min(calc(100vw - 20px), 400px);
	overflow: hidden;
	height: auto;
	clip-path: inset(0);
}

#guessButton {
	margin-left: 50vw;
	transform: translateX(-50%);
	width: min(calc(100vw - 20px), 400px);
	padding: 10px;
	border-radius: 10px;
	border: none;
	background-color: #1B82B4;
	box-shadow: inset 0px -4px 0px rgba(0,0,0,0.5);
	font-size: 20px;
	margin-top: 10px;
}

#guessButton:hover {
	box-shadow: inset 0px -5px 0px rgba(0,0,0,0.6);
}

#guessButton:active {
	box-shadow: inset 0px -4px 0px rgba(0,0,0,0.5);
}

.vert {
	display: flex;
	flex-direction: column;
}

.vert > * {
	display: block;
}
</style>

<script setup>
import { ref } from "vue";
import SampleData from "./classes/sampleData.js";
const data = new SampleData();
let guessMovie;
let credits;
let director;
let guessCount = 0;

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
	guessCount++;
	if (stripPunctuation(guess) == stripPunctuation(guessMovie.title)) {
		document.getElementById("moviePoster").className = "";
		addHint(guess + " is correct!!");
	} else {
		document.getElementById("moviePoster").className = "guess" + guessCount;
		doHint(guessCount, guess);
	}
}

function getHint(count) {
	if (count == 1) {
		return "It has actor " + credits.cast[2].name + " in it.";
	}
	if (count == 2) {
		return "The movie was released on " + guessMovie.release_date;
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
		addHint(guess + " is incorrect. The answer is " + guessMovie.title);
	}
	else if (count < 6) {
		addHint(guess + " is incorrect. Hint: " + getHint(count));
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

		const response = await fetch('https://192.168.68.59:8443/movie', {
			method: 'GET',
			signal: controller.signal,
		});

		clearTimeout(timeoutId);

		if (!response.ok) {
			throw new Error(`HTTP error! Status: ${response.status}`);
		}

		const text = await response.text();
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
	//while (movie == null || (movie.success === false || movie.adult || movie.poster_path === null || movie.original_language != "en" || parseInt(movie.release_date.split("-")[0]) < 1970 || movie.vote_average < 7 || movie.vote_count < 7000)) {
		//movie = await fetchResults();
	//}

	while (movie == null || (movie.success == false || movie.adult || movie.poster_path == null || movie.original_language != "en" || movie.vote_average < 7 || movie.vote_count < 6500)) {
		movie = await fetchResults();
	}
	guessMovie = movie;
	credits = await fetchCredits(movie.id);

	//console.log(movie.poster_path);
	document.getElementById("moviePoster").src = "https://image.tmdb.org/t/p/w600_and_h900_bestv2/" + movie.poster_path;

	for (let i = 0; i < credits.crew.length; i++) {
		if (credits.crew[i].job == "Director" || credits.crew[i].known_for_department == "Directing") {
			director = credits.crew[i].name;
			break;
		}
	}

	movieLoaded.value = true;
}

getMovie();
</script>

<style>
.guess0 {
	filter: blur(60px);
}
.guess1 {
	filter: blur(55px);
}
.guess2 {
	filter: blur(45px);
}
.guess3 {
	filter: blur(30px);
}
.guess4 {
	filter: blur(25px);
}
.guess5 {
	filter: blur(20px);
}
</style>
