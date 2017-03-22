<template>
<div id="app">
	<date @daterange="daterange" :show="true" start="2015-03-03" end="2017-04-04"></date>
</div>
</template>

<script>
import date from './components/date'

export default {
	name: 'app',
	components: {
		date
	},
	methods: {
		daterange (o) {
			console.dir(o);
		}
	}
}
</script>

<style>
/* reset */

html {
	margin: 0;
	padding: 0;
	font-size: 10px;
}

body {
	width: 100%;
	margin: 0;
	padding: 0;

	font-size: 2rem;
	font-family: 'Avenir', Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
}

li,
p {
	margin: 0;
	padding: 0;
}

ul,
ol {
	margin: 0;
	padding: 0;
	list-style-type: none;
}

a {
	text-decoration: none;
}

h1,
h2,
h3,
h4,
h5,
h6 {
	margin: 0;
	padding: 0;
	font-weight: normal;
	color: inherit;
	font-size: inherit;
}
</style>
