<template>
	<main class="Home">
		<div
			id="items-lists-container"
			aria-live="polite"
			aria-atomic="false"
		>
			<items
				class="items-container"
				:style="zoomFactor"
				v-for="year in displayedYears"
				:year="year"
				:key="year"
				:class="{ allCategories: allCategories }"
			/>
		</div>
		<button
			class="load-more"
			type="button"
			@click="getNextYear"
			aria-controls="items-lists-container"
		>
			Année précédente
		</button>
	</main>
</template>

<script>
import Vue from "vue";
import { mapState } from "vuex";
import items from "../components/items.vue";
import { myMixin } from "../mixin.js";

export default {
	components: { items },
	mixins: [myMixin],

	created() {
		this.init();

		window.addEventListener(
			"resize",
			this.debounce(this.changeTransformOrigin, 500)
		);

		//switch (this.settings.phase) {
		//	case 0: //init
		//		router.push('Init')
		//		break
		//	case 1: //loading
		//		console.log('loading')
		//		break
		//	case 2: //finished
		//		//show transition
		//		router.push('Home')
		//		break
		//	case 3: //error
		//}
	},
	computed: {
		...mapState([
			"items",
			"categories",
			"settings",
			"years",
			"countsByYear",
			"countsByCat",
		]),
		allCategories: function () {
			if (this.settings.currentCategory === "all") {
				return true;
			} else {
				return false;
			}
		},
		displayedYears() {
			const { start, end } = this.years.period;
			return this.years.yearsWithItems.slice(start, end);
		},
		zoomFactor: function () {
			return {
				"--zoom-factor": this.settings.zoom,
			};
		},
	},
	methods: {
		async init() {
			// On fixe la catégorie en la récupérant de l'URL



			//on récupère systématiquement des stats, au cas où il y a de nouveaux items
			const { countsByCat } = await this.$store.dispatch("getCounts");

			if (this.settings.initFront !== "true") {
				await this.$store.dispatch("loadNonEmptyYears");
				await this.$store.dispatch("setPeriod", { start: 0, end: 1 });
				this.$store.dispatch("setCategory", this.categories[6]);
				await this.$store.dispatch("getItems");
				this.$store.commit("INIT", "true");

			}
			const routeParams = this.$route.params;
			if (routeParams.category) {
				this.$store.dispatch(
					"setCategory",
					this.categories.find(
						(elem) => elem.code === this.$route.params.category
					)
				);
			}

			if (routeParams.year) {
				const index = this.years.yearsWithItems.findIndex((el) => el === routeParams.year);
				console.log(index)
				await this.$store.dispatch("setPeriod", {
					start: index,
					end: index + 1,
				});

			}
			await this.$store.dispatch("getItems");



			function isNotEqual(objA, objB) {
				const arrayA = Object.values(objA)
				return Object.values(objB).some(
					(el, index) =>
						el !== arrayA[index]
				)
			}

			//Si y a du nouveau par rapport au store
			if (isNotEqual(countsByCat, this.countsByCat)) {
				this.logOfRequests = []
				await this.$store.dispatch("getItems");
			}
		},

		async getNextYear() {
			const end = this.years.period.end + 1;
			await this.$store.dispatch("setPeriod", { end: end });
			await this.$store.dispatch("getItems");
		},
	},
};
</script>

<style scoped lang="scss">
@import "@/variables.scss";

@media screen and (min-width: 601px) {
	.Home {
		width: 99%;
	}
}
@media screen and (max-width: 600px) {
	.Home {
		width: 95%;
	}
}

.Home {
	margin: auto;
	position: relative;

	#items-lists-container {
		padding-bottom: 8rem;
	}

	.load-more {
		width: 100%;
		border: none;
		height: 3em;
		color: scale-color($accent-color, $lightness: 15%);
		background-color: rgba(38, 38, 41, 0.45);
		color: scale-color($color: $accent-color, $lightness: 25%);
		cursor: pointer;
		font-size: 1.3rem;
		margin: 2rem 0 0rem 0;
		padding: 0;
		position: absolute;
		bottom: 0;
		left: 0;
	}
}

//@import 'vue-accessible-modal/src/styles/core.scss';
</style>