<template>
	<div id="hello">hello</div>
	<img id="moviePoster" class="guess0" src="" alt="">
	<input type="text" id="guessInput">
	<ul id="hints">
	</ul>
	<button @click="doGuess">Guess</button>
</template>

<script setup>

let guessMovie;
let credits;
let director;
let guessCount = 0;

function stripPunctuation(str) {
	let result = String(str).replace(/[^\w\s]/g, '').toLowerCase();
	console.log(result);
	return result;
}

function doGuess() {
	console.log("do guess");
	let guess = document.getElementById("guessInput").value;
	if (guess == "") {
		return;
	}
	guessCount++;
	if (stripPunctuation(guess) == stripPunctuation(guessMovie.title)) {
		document.getElementById("moviePoster").className = "";
		console.log("correct");
	} else {
		document.getElementById("moviePoster").className = "guess" + guessCount;
		doHint(guessCount);
	}
}

function getHint(count) {
	if (count == 1) {
		return "It has the actor " + credits.cast[0].name + " in it.";
	}
	if (count == 2) {
		return "It also has actor " + credits.cast[1].name + " in it.";
	}
	if (count == 3) {
		return "The director is " + director;
	}
	if (count == 4) {
		return "The movie was released on " + guessMovie.release_date;
	}
}

function doHint(count) {
	const ul = document.getElementById("hints");
	const li = document.createElement('li');
	li.innerText = getHint(count);
	ul.appendChild(li);
}

async function fetchResults() {
	try {
		const response = await fetch('https://192.168.68.59:8443/movie', {
			method: 'GET',
		});

		if (!response.ok) {
			throw new Error(`HTTP error! Status: ${response.status}`);
		}

		const text = await response.text();
		return JSON.parse(text.substring(8));
	} catch (error) {
		console.error('Error fetching result:', error);
		return "ERROR";
	}
}

async function fetchCredits(id) {
	try {
		const response = await fetch('https://192.168.68.59:8443/credits/' + id, {
			method: 'GET',
		});

		if (!response.ok) {
			throw new Error(`HTTP error! Status: ${response.status}`);
		}
		const text = await response.text();
		return JSON.parse(text.substring(8));
	} catch (error) {
		console.error('Error fetching credits:', error);
		return "ERROR";
	}
}

async function getMovie() {
	let movie = await fetchResults();
	while (movie == null || (movie.success === false || movie.adult || movie.poster_path === null || movie.original_language != "en" || parseInt(movie.release_date.split("-")[0]) < 1970 || movie.vote_average < 8 || movie.vote_count < 10000)) {
		movie = await fetchResults();
		console.log(movie);
	}
	guessMovie = movie;
	credits = await fetchCredits(movie.id);
	console.log(credits);

	document.getElementById("moviePoster").src = "https://image.tmdb.org/t/p/w600_and_h900_bestv2/" + movie.poster_path;

	for (let i = 0; i < credits.crew.length; i++) {
		if (credits.crew[i].job == "Director") {
			director = credits.crew[i].name;
			break;
		}
	}

	console.log(director);
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
