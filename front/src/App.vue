<template>
	<div
		id="app "
		:class="`${activeView}`"
	>
		<navBar />
		<router-view />
	</div>
</template>

<script>
import Vue from 'vue'
import { mapState } from 'vuex'
import router from './router'
export default {
	name: 'App',

	components: {
		navBar: () => import('./components/navBar.vue'),
	},
	computed: {
		...mapState(['items', 'categories', 'settings', 'years']),
		activeView() {
			return this.$route.name
		}

	},
	methods: {},

	created() {

	},
}


</script>

<style lang="scss">
@import "@/variables.scss";

@import url("https://fonts.googleapis.com/css2?family=Aleo:wght@400;700&display=swap");

#app {
	font-family: aleo, sans-serif;

	letter-spacing: 0.05em;
}

.notFound header.nav {
	display: none;
}

.controls,
.load-more,
body {
	font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica,
		Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
}

a:visited,
a:link {
	color: $white-1;
	text-decoration: none;
}
body {
	margin: 0;
	background: $black-1;
	color: $white-1;
	overflow-x: hidden;
}
#nav {
	padding: 30px;
	a {
		font-weight: bold;
		color: $white-1;
		&.router-link-exact-active {
			color: #42b983;
		}
	}
}

@keyframes fadein {
	from {
		opacity: 0;
	}
	to {
		opacity: 1;
	}
}
.sr-only {
	position: absolute;
	width: 1px;
	height: 1px;
	padding: 0;
	margin: -1px;
	overflow: hidden;
	clip: rect(0, 0, 0, 0);
	border: 0;
}
</style>
