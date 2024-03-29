<script>
import axios from "axios";
import ApartmentCard from "../partials/ApartmentCard.vue";
import { fetchImageApartment } from "../../store";

export default {
	data() {
		return {
			slidePromoList: [],
			currentSlide: 0,
			cardsForSlide: 3,
			movingCard: false,
			filter: {
				sponsorships: 1,
			}
		};
	},
	computed: {
		// Calcola le card visibili in base all'indice corrente e al numero di card da dover mostrare
		visibleSlide() {
			// Creo l'indice di inizio e fine del range di carte da mostrare
			const start = this.currentSlide;
			const end = start + this.cardsForSlide;
			// Verifico se l'array contiene almeno 3 elementi
			if (this.slidePromoList.length < this.cardsForSlide) {
				return this.slidePromoList;
			}
			else {
				// Se si è raggiunta la fine dell'array, riporto l'indice "currentSlide" a 0 e ritorno le prime carte dell'array
				if (start >= this.slidePromoList.length) {
					this.currentSlide = 0;
					return this.slidePromoList.slice(0, this.cardsForSlide);
				}
				// Se sono nei limiti dell'array, ritorno il range di carte tra l'indice di inizio e quello di fine
				if (end <= this.slidePromoList.length) {
					return this.slidePromoList.slice(start, end);
				}
				// Individuo le due porzioni di array da mostrare
				const firstPart = this.slidePromoList.slice(start); // dall'indice "start" fino alla fine dell'array			
				const secondPart = this.slidePromoList.slice(0, end - this.slidePromoList.length); // dall'inizio dell'array fino all'indice "end" sottraendo la sua lunghezza
				// Restituisco le due parti concatenate
				return firstPart.concat(secondPart);
			}
		},
	},
	methods: {
		fetchData(url) {
			axios.get(url, { params: this.filter }).then((response) => {
				this.slidePromoList = response.data.apartments.data;
			});
		},
		onNextClick() {
			this.movingCard = true;
			this.currentSlide++;
			// Rimuovo la classe di animazione
			setTimeout(() => {
				this.movingCard = false;
			}, 300);
		},
		onPrevClick() {
			this.movingCard = true;
			this.currentSlide--;
			if (this.currentSlide < 0) {
				this.currentSlide = this.slidePromoList.length - 1; // Per tornare all'ultima slide
			}
			setTimeout(() => {
				this.movingCard = false;
			}, 300);
		},
		fetchImageApartment,
	},
	mounted() {
		this.fetchData('http://127.0.0.1:8000/api/apartments/');
	},
	components: { ApartmentCard }
}
</script>

<template>
	<div class="container pb-4 pt-4" v-if="slidePromoList.length > 0">
		<h2 class="pb-2 text-white fw-bold text-center"><i class="fa-solid fa-star"></i> Le nostre migliori strutture <i
				class="fa-solid fa-star"></i></h2>
		<!-- Sezione carosello per dispositivi più grandi di 769px -->
		<div class="slider-container">
			<div class="promo-slider-box">
				<div class="row row-cols-3 justify-content-center">
					<div v-for="(card, i) in visibleSlide" :key="i" class="col">
						<ApartmentCard :card="card" :thumbfunction="fetchImageApartment(card)" :movingCard="movingCard">
						</ApartmentCard>
					</div>
				</div>
				<!-- Pulsanti di navigazione -->
				<div v-if="slidePromoList.length > 3">
					<div class="prev text-center" @click="onPrevClick">
						<button type="button" class="btn slider-btn">
							<i class="fa-solid fa-chevron-left"></i>
						</button>
					</div>
					<div class="next text-center" @click="onNextClick">
						<button type="button" class="btn slider-btn">
							<i class="fa-solid fa-chevron-right"></i>
						</button>
					</div>
				</div>
			</div>
		</div>

		<!-- Sezione card per dispositivi più piccoli di 769px -->
		<div class="card-container">
			<div class="row row-cols-1 row-cols-md-3 gy-4 justify-content-center">
				<div v-for="(card, i) in slidePromoList" :key="i" class="col">
					<ApartmentCard :card="card" :thumbfunction="fetchImageApartment(card)" :movingCard="movingCard">
					</ApartmentCard>
				</div>
			</div>
		</div>
	</div>
</template>

<style lang="scss" scoped>
@use "../../style/partials/variables" as *;

.container {
	background-color: rgba(255, 255, 255, 0.12);
	padding: 0px 30px;
	border-radius: 20px;
	border: 2px dashed white;
	margin-top: 50px;
	display: flex;
	flex-direction: column;
	align-items: center;
}

.promo-slider-box {
	position: relative;

	.prev {
		position: absolute;
		top: 50%;
		left: -60px;
		transform: translate(-50%, -50%);
	}

	.next {
		position: absolute;
		top: 50%;
		right: -98px;
		transform: translate(-50%, -50%);
	}
}

.slider-container {
	display: none;
	width: 91%;
}

.card-container {
	display: block;
}

.slider-btn {
	color: $partial-primary-color;
	background-color: $partial-secondary-color;
	border: 2px solid $partial-primary-color;

	&:hover {
		color: white;
		background-color: $secondary-color;
		border-color: $partial-primary-color;
	}
}

@media (min-width: 820px) {
	.slider-container {
		display: block;
	}

	.card-container {
		display: none;
	}
}
</style>