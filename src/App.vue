<script>
import axios from 'axios';
axios.defaults.baseURL = 'http://localhost:3005';

import dayjs from 'dayjs';
import 'dayjs/locale/ru';
import relativeTime from 'dayjs/plugin/relativeTime';
dayjs.locale('ru');
dayjs.extend(relativeTime);

export default {
	data() {
		return {
            shop: [],
            user: null,
            plants: null,
            plantIndex: null,
            count: 1,
            
		};
	},

	methods: {
		async loadShop() {
            let response = await axios.get('/shop');
            this.shop = response.data;
        },

        async loadUser() {
            let response = await axios.get('/user');
            this.user = response.data;
        },

        async loadGarden() {
            let response = await axios.get('/garden');
            this.plants = response.data;
        },
        
        getPrice() {
            let plant = this.shop[this.plantIndex]
            let price = 0
            if (plant) {
                price = plant.buyPrice * this.count
            }
            return price;
        },

        getTimeToHarvest(date) {
            let day = dayjs(date);
            let now = dayjs();
            if (now > day) {
                return 'Готово!'
            }
            return day.fromNow()
        },

        async buy(evt) {
            evt.preventDefault();
            await axios.post('shop/buy',{
                count: this.count,
                title: this.shop[this.plantIndex].title
            });
            this.loadUser();
            this.loadGarden();
        },

        async harvest(item) {
            axios.post('/garden/harvest', {
                id: item._id
            });
            this.loadUser()
            this.loadGarden()
        }

	},
    mounted() {
        this.loadShop()
        this.loadUser()
        this.loadGarden()
        setInterval(() => {
            this.loadShop()
            this.loadGarden()
        }, 1000);
    }
};
</script>

<template>
	<div class="app">
		<div class="container my-3">

			<!-- Форма покупки растения -->
			<form @submit="buy" class="shop">
				<div class="row top-row">
					<div class="col">
						<!-- Магазин -->
						<div class="shop-plants">
							<!-- Растение -->
							<label
                                v-for="(item,index) in shop"
                                class="shop-item"
                            >
								<img :src="'src/assets/' + item.image + '.png'">
								<p>{{ item.title }}</p>
								<p>$ {{ item.buyPrice }} | {{ item.harvestPrice }}</p>
								<input
                                    v-model="plantIndex"
                                    :value="index"
                                    type="radio"
                                >
							</label>
						</div>
					</div>
					<div class="col">
						<div class="shop-form">
							<!-- Количество -->
							<div class="shop-info-row">
								<div class="shop-info-left">
									<label for="count" class="form-label">Количество:</label>
								</div>
								<div class="shop-info-right">
									<input min="0" v-model="count" type="number" id="count" class="form-control count-input">
								</div>
							</div>
							<div class="shop-info-row">
								<div class="shop-info-left">
									Итого:
								</div>
								<div class="shop-info-right">
									$ {{ getPrice() }}
								</div>
							</div>

							<div v-if="user" class="shop-info-row">
								<div class="shop-info-left">
									Баланс {{ user.username }}a:
								</div>
								<div class="shop-info-right">
									$ {{ user.gold }}
								</div>
							</div>

							<div class="shop-info-row">
								<div class="shop-info-left">
								</div>
								<div class="shop-info-right">
									<button type="submit" class="btn btn-primary">
										Посадить
									</button>
								</div>
							</div>
						</div>
					</div>
				</div>
			</form>


			<!-- Огород с растениями -->
			<div class="garden">
				<!-- Растение в огороде -->
				<div
                    v-for="(item, index) in plants" 
                    class="garden-plant"
                    @click="harvest(item)"
                >
					<img :src="'src/assets/'+ item.image +'.png'">
					<p> {{ getTimeToHarvest(item.timeToHarvest) }}</p>
				</div>
			</div>
		</div>
	</div>
</template>

<style>
.shop {
	border: 4px solid black;
	padding: 10px;
}

.top-row {
	padding: 20px;
}

.shop-info-row {
	display: flex;
	align-items: center;
	font-size: 20px;
	gap: 20px;
	height: 40px;
	margin: 20px;
}
.shop-info-left {
	width: 120px;
	text-align: right;
}
.shop-info-right {
	width: 100px;
}
.total {
	font-size: 20px;
	height: 40px;
	display: flex;
	align-items: center;
}

.count-input  {
	font-size: 20px;
}

.btn-primary {
	font-size: 20px;
}

.user-info {
	font-size: 30px;
	padding-left: 40px;
	padding-right: 10px;
	text-align: right;
}

.shop-plants {
	display: flex;
	flex-wrap: wrap;
	justify-content: flex-start;
}

.shop-item {
    width: 150px;
    padding: 20px;
	text-align: center;
}

.shop-item img {
	width: 100%;
}


.garden {
	border: 4px solid black;
	padding: 20px;
	display: flex;
	flex-wrap: wrap;
	background-image: url('src/assets/background.jpg');
	background-size: cover;
	min-height: 300px;
}

.garden-plant {
	width: 150px;
	padding: 20px;
	text-align: center;
	color: white;
}

.garden-plant img {
	width: 100%;
}
</style>
