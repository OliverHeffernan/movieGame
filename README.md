# movie-game
This is a simple guess the movie game. It uses the [TMDb](https://www.themoviedb.org/) API for the movie data. The API is accessed through a [Rust backend](https://github.com/OliverHeffernan/movieGameBackend) running on a raspberry pi.

## Frontend
The front end is written in vuejs.

## Instructions
- Enter your guess in the input field at the bottom of the screen.
- You will get a hint after every guess. And the poster will become less blurred.
- If you have no clue you can press the "skip for hint" button, to sip this guess and get a hint.
- You have a maximum of 6 guesses.

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
