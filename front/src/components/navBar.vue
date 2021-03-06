<template>
	<header class="nav">
		<h1>
			<router-link to="/"> Table basse </router-link>
		</h1>

		<nav
			class="controls"
			v-if="$router.currentRoute.name === 'Home'"
		>
			<vue-slider
				@change="zoomChanged"
				:value="settings.zoom"
				class="zoom"
				:min="0.5"
				:max="4"
				:interval="0.1"
				:tooltip="'none'"
				:contained="'true'"
				:dot-attrs="{ 'aria-label': 'Varier la taille des images des oeuvres' }"
			/>
			<v-select
				aria-controls="items-lists-container"
				@input="selectCategory"
				:options="categories"
				:value="settings.currentCategory"
				:searchable="false"
				:clearable="false"
				:selectable="
          (option) => {
            return countsByCat[option.code] > 0 || option.code === 'all';
          }
        "
			>
				<template v-slot:option="option">
					<span class="inline-label">{{ option.label }}</span>
					<span
						class="inline-count"
						v-if="option.code === 'all'"
					>{{
            countsByCatTotal
          }}</span>
					<span
						class="inline-count"
						v-else
					>{{
            countsByCat[option.code]
          }}</span>
				</template>
			</v-select>

			<v-select
				aria-controls="items-lists-container"
				@input="selectYear"
				:options="years.yearsWithItems"
				:value="years.yearsWithItems[years.period.start || 0]"
				:searchable="false"
				:clearable="false"
				:selectable="
          (option) =>
            this.countsByYear[option][this.settings.currentCategory.code] > 0 ||
            this.settings.currentCategory.code === 'all'
        "
			>
				<template v-slot:option="option">
					<span class="inline-label">{{ option.label }}</span>
					<span
						class="inline-count"
						v-if="settings.currentCategory.code === 'all'"
					></span>
					<span
						class="inline-count"
						v-else
					>{{
            countsByYear[option.label][settings.currentCategory.code]
          }}</span>
				</template>
			</v-select>
		</nav>
	</header>
</template>

<script>
import Vue from "vue";
import { mapState } from "vuex";

import VueSlider from "vue-slider-component";
import "vue-slider-component/theme/material.css";
import vSelect from "vue-select";
Vue.component("v-select", vSelect);
import "vue-select/dist/vue-select.css";
import { myMixin } from "../mixin.js";

export default {
	components: { VueSlider },
	mixins: [myMixin],

	computed: {
		...mapState([
			"items",
			"categories",
			"settings",
			"years",
			"countsByYear",
			"countsByCat",
		]),
		countsByCatTotal() {
			return Object.values(this.countsByCat).reduce((previous, current) => {
				return previous + current;
			});
		},
	},

	created() { },
	methods: {
		zoomChanged(val) {
			//debounce a été pensé pour être utilisé en callback, d'où le () supplémentaire.
			this.debounce(this.changeTransformOrigin, 500)();
			this.$store.dispatch("setZoom", val);

		},
		async selectCategory(val) {
			this.$router.push({ path: `/items/${val.code}/` });
			this.$store.dispatch("setCategory", val);

			const { start, end } = this.years.period;
			this.years.yearsWithItems.slice(start, end).forEach((year) => {
				this.$store.dispatch("getItems", year);
			});
		},
		async selectYear(year, event) {
			this.$router.push({
				path: `/items/${this.settings.currentCategory.code}/${year}`,
			});
			//bricolage car le select renvoie la valeur sélectionnée,
			// alors qu'on voudrait l'index de cette valeur dans le tableau des années chargées dans le select
			const index = this.years.yearsWithItems.findIndex((el) => el === year);

			await this.$store.dispatch("setPeriod", {
				start: index,
				end: index + 1,
			});
			await this.$store.dispatch("getItems");
		},
		//countsByYearTotal: function (option) {
		//	return Object.values(this.countsByYear[option.label]).reduce((previous, current) => {
		//		return previous + current
		//	})
		//},
	},
};
</script>

<style  lang="scss">
@import "@/variables.scss";

.nav {
	display: flex;
	flex-wrap: wrap;
}

.controls {
	vertical-align: middle;
	display: flex;
	flex-wrap: wrap;
	height: auto;
	align-content: center;
	align-items: center;
	font-size: 1.2em;
}

.nav {
	justify-content: space-between;
	h1 {
		margin-left: 1em;
	}
}

.controls > div {
	width: 18vw !important;
	min-width: 220px;
	margin: 1em;
	max-width: 250px;
}

@media screen and (max-width: 650px) {
	.controls > div {
		min-width: auto;
		max-width: none;
		width: 92vw !important;
	}
	.controls .vue-slider {
		display: none !important;
	}
	.controls .vue-slider-dot {
		width: 16px !important;
		height: 16px !important;
	}
}

$themeColor: $accent-color;
$dotBgColor: scale-color($accent-color, $lightness: 50%);
$bgColor: #bbb;

.controls .vue-slider-rail {
	background-color: $white-3;
}

.controls .vue-slider-process {
	background-color: scale-color($themeColor, $blackness: 30%);
}
.controls .vue-slider-dot-handle-focus {
	background-color: scale-color($accent-color, $red: 30%);
}
.controls .vue-slider-dot-handle-focus:after {
	background-color: scale-color($accent-color, $red: 30%);
	width: 150%;
	height: 150%;
}

@media screen and (max-width: 30em) {
}

@import "~vue-slider-component/lib/theme/material.scss";

.controls .v-select {
	background-color: hsl(234, 50%, 20%);
	border-bottom: 1px solid $white-3;
}

@media screen and (min-width: 601px) {
	.controls .v-select .vs__dropdown-toggle {
		padding: 0.3em;
	}
}
@media screen and (max-width: 600px) {
	.controls .v-select .vs__dropdown-toggle {
		padding: 0.6em 0.3em;
	}
}

.controls .v-select .vs__open-indicator {
	fill: hsl(234.3, 33%, 69%);
}

.controls .v-select .vs__selected {
	color: $white-1;
	opacity: 1;
	margin: 0;
	padding: 0;
}

.controls .vs__selected-options {
	display: flex;
	flex-basis: 100%;
	flex-grow: 1;
	flex-wrap: wrap;
	padding: 0 8px;
	position: relative;
	align-content: center;
}

.controls .v-select .vs__dropdown-menu {
	background-color: $black-3;
	box-shadow: 2px 2px 4px $black-4;
}

.controls .v-select .vs__dropdown-option {
	color: $white-1;
}

.controls .v-select .vs__dropdown-option:hover {
	background: $black-1;
}
.controls .vs__dropdown-option--highlight {
	background-color: $black-1;
}

.controls .v-select .vs__dropdown-option--disabled {
	color: $white-3;
}

.controls .v-select .vs__dropdown-option {
	&.vs__dropdown-option--disabled:hover {
		background-color: $black-3;
	}
}

.controls .v-select .vs__dropdown-option--disabled .inline-label {
	text-decoration: line-through;
}

.controls .v-select .inline-count {
	color: $white-3;
}

@media screen and (min-width: 601px) {
	.controls .v-select .vs__dropdown-option {
		display: flex;
		justify-content: space-between;
		padding: 0.6rem;
	}
}

@media screen and (max-width: 600px) {
	.controls .v-select .vs__dropdown-option {
		display: flex;
		justify-content: space-between;
		padding: 1.2rem;
	}
}

.controls .v-select .vs__dropdown-menu {
	max-height: 440px;
}

.vs__actions {
	position: absolute;
	width: 100%;
	height: 100%;
	z-index: 1;
}

.v-select .vs__actions {
	justify-items: flex-start;
	align-self: center;
	justify-content: flex-end;
	padding: 4px 20px 0 3px;
}
</style>