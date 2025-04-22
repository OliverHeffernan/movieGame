<template>
	<div id="phoneContainer" @scroll="loadMore">
		<NoConnection :connection="connection"/>
		<TutInfo />
		<MovieContainer v-for="(movie, n) in movies" :key="n" :index="n - 1" :movie="movie.movie" />
		<LoadingScreen />
	</div>
</template>

<script setup>
import "./styles/main.css";
import SampleData from "./classes/sampleData.js";
import MovieContainer from "./components/MovieContainer.vue";
import TutInfo from "./components/TutInfo.vue";
import LoadingScreen from "./components/LoadingScren.vue";
import NoConnection from "./components/NoConnection.vue";
const data = new SampleData();

let connection = ref(true);

import { ref, onMounted } from "vue";

let page = [];
const movies = ref([]);


let loadings = 0;
let fetching = false;

onMounted(async () => {
	const vh = window.innerHeight * 0.01;
	document.documentElement.style.setProperty('--vh', `${vh}px`);
	for (let i = 0; i < 10; i++) {
		await addMovie();
	}
});

async function addMovie() {
	await checkForMorePages();

	let index = getRandomInt(0, page.length - 1);

	let movie = page[index];
	page.splice(index, 1);

	await checkForMorePages();

	if (movie != undefined) {	
		movies.value.push({
			movie: movie
		});
	}
}

async function checkForMorePages() {
	if (page.length == 0) {
		const temp = await getPage();
		page = temp.results;
	}
}

async function loadMore() {
	if (loadings == 0 && !fetching) {
		await addMovie();
		loadings++;
		setTimeout(() => {
			loadings--;
		}, 2000);

	}
}

async function fetchResults() {
	fetching = true;
	try {
		const controller = new AbortController();
		const timeout = 10000;
		const timeoutId = setTimeout(() => controller.abort(), timeout);

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
		fetching = false;
		return JSON.parse(text);
	} catch (error) {
		fetching = false;
		return "ERROR";
	}
}

function getRandomInt(a, b) {
	const min = Math.ceil(Math.min(a, b));
	const max = Math.floor(Math.max(a, b));
	return Math.floor(Math.random() * (max - min + 1)) + min;
}

async function getPage() {
	let _page;
	if (connection.value) {
		_page = await fetchResults();
	}
	else {
		return getSampleData();
	}
	let attempts = 0;
	while (
		(
			_page === "ERROR" ||
			!_page ||
			typeof _page !== "object" ||
			!Array.isArray(_page.results)
		) &&
		attempts < 5 &&
		connection.value
	) {
		console.warn("getPage: retrying due to invalid data", _page);
		_page = await fetchResults();
		attempts++;
	}
	if (connection.value == false) {
		return getSampleData();
	} else if (_page === "ERROR") {
		connection.value = false;
		return getSampleData();
	}

	return _page;
}

function getSampleData() {
	let returning = {
		"results": []
	};

	for (let i = 0; i < data.data.results.length - 1; i++) {
		returning.results.push(data.data.results[i]);
	}

	return returning;
}

</script>
