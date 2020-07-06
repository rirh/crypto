<template>
	<view class="">
		<scroll-view scroll-y class="DrawerPage" :class="modalName == 'viewModal' ? 'show' : ''">
			<view class="flex padding">
				<view style="width: 50%;" class="text-center text-white" @click="showModal($event, 'instrument')" data-target="viewModal">
					instrument:&nbsp;{{ query.instrument_id || 'all' }}
				</view>
				<view style="width: 50%;" class="text-center text-white" @click="showModal($event, 'type')" data-target="viewModal">type:&nbsp;{{ query.type || 'all' }}</view>
			</view>
			<Empty v-if="order_arr.length <= 0"></Empty>
			<view v-else class="">
				<view v-for="order in order_arr" :key="order.client_oid" class=""><OrderItem @on-revoke="handle_cancel(order)" :params="order"></OrderItem></view>
			</view>

			<!-- <view class="cu-list menu card-menu margin-top-xl margin-bottom-xl shadow-lg">
				<view class="cu-item arrow" v-for="(item, index) in 20" :key="index">
					<view class="content">
						<text class="cuIcon-github text-grey"></text>
						<text class="text-grey">{{ index + 1 }}</text>
					</view>
				</view>
			</view> -->
		</scroll-view>
		<view class="DrawerClose" :class="modalName == 'viewModal' ? 'show' : ''" @click="hideModal"><text class="cuIcon-pullright"></text></view>
		<scroll-view scroll-y class="DrawerWindow" :class="modalName == 'viewModal' ? 'show' : ''">
			<!-- instrument -->
			<view class="" v-if="type === 'instrument'">
				<scroll-view scroll-x class="margin-left nav" scroll-with-animation :scroll-left="0">
					<view class="cu-item" @click="handle_set_margined(0)" :class="tabcur === 0 ? 'text-white cur' : ''">Coin Margined</view>
					<view class="cu-item" @click="handle_set_margined(1)" :class="tabcur === 1 ? 'text-white cur' : ''">USDT Margined</view>
				</scroll-view>
				<view class="cu-list menu card-menu margin-top-xl margin-bottom-xl shadow-lg">
					<view class="cu-item" style="background-color:#000 ;" @click="handle_async(instrument)" v-for="(instrument, index) in instrument_arr" :key="index">
						<view class="content">
							<text class="text-white">{{ instrument.type }}</text>
						</view>
					</view>
				</view>
			</view>
			<!-- type -->
			<view class="" v-if="type === 'type'">
				<view class="cu-list menu card-menu margin-top-xl margin-bottom-xl shadow-lg">
					<view class="cu-item" style="background-color:#000 ;" @click="handle_async_type(type)" v-for="(type, index) in types_arr" :key="index">
						<view class="content">
							<text class="text-white">{{ type.lable }}</text>
						</view>
					</view>
				</view>
			</view>
		</scroll-view>
	</view>
</template>
<script>
import config from 'config';
import Empty from 'components/empty.vue';
import OrderItem from './OrderItem.vue';
export default {
	components: {
		Empty,
		OrderItem
	},
	data() {
		return {
			modalName: null,
			type: '',

			query: {
				instrument_id: '',
				state: -2,
				lable: ''
			},
			instrument_arr: [],
			types_arr: [
				{
					value: 0,
					lable: 'wait success order'
				},
				{
					value: -1,
					lable: 'recalled success order'
				},
				{
					value: 2,
					lable: 'success order'
				},
				{
					value: 3,
					lable: 'ordering'
				},
				{
					value: -2,
					lable: 'fail order'
				},

				{
					value: 1,
					lable: 'par success order'
				},

				{
					value: 4,
					lable: 'recalling'
				},
				{
					value: 6,
					lable: 'lce order'
				},
				{
					value: 7,
					lable: 'complate order'
				}
			],
			tabcur: 1,
			order_arr: []
		};
	},
	onReady() {
		this.init();
	},
	methods: {
		init() {
			this.fetch_instruments();
		},
		handle_cancel(params) {
			uni.request({
				url: `${config.host}/app/revoke_order/`,
				method: 'GET',
				data: {
					client_oid: params.client_oid
				},
				success: res => {
					if (res.statusCode === 200) {
						console.log(res);
						this.fetch_order_list();
					}
				},
				fail: () => {},
				complete: () => {
					uni.hideLoading();
				}
			});
		},
		handle_set_margined(i) {
			this.tabcur = i;
			this.instrument_arr = this.instrument_arr.filter(e => e.type !== 'all').map(e => ({ ...e, type: `${e.underlying_index}-${this.tabcur === 0 ? 'USD' : 'USDT'}` }));
			this.instrument_arr.unshift({ type: 'all' });
		},

		handle_async(params) {
			this.query.instrument_id = params.type;
			this.hideModal();
			this.fetch_order_list();
		},
		showModal(e, type) {
			this.modalName = e.currentTarget.dataset.target;
			this.type = type;
		},
		hideModal(e) {
			this.modalName = null;
		},
		handle_async_type(params) {
			this.query.type = params.lable;
			this.query.state = params.value;
			this.hideModal();
			this.fetch_order_list();
		},

		fetch_order_list() {
			uni.showLoading({
				mask: true,
				title: 'loading...'
			});
			uni.request({
				url: `${config.host}/app/get_order_list/`,
				method: 'GET',
				data: {
					instrument_id: `${this.query.instrument_id}-SWAP`,
					state: this.query.state
				},
				success: res => {
					if (res.statusCode === 200) {
						console.log(res);
						this.order_arr = res.data[0].order_info;
					}
				},
				fail: () => {},
				complete: () => {
					uni.hideLoading();
				}
			});
		},
		fetch_instruments() {
			uni.request({
				url: `${config.host}/app/get_instruments/`,
				method: 'GET',
				data: {},
				success: res => {
					if (res.statusCode === 200) {
						this.instrument_arr = res.data.map(e => ({ ...e, type: `${e.underlying_index}-${this.tabcur === 0 ? 'USD' : 'USDT'}` })).reverse();
						this.query.instrument_id = this.instrument_arr[0].type;
						this.query.type = this.types_arr[0].lable;
						this.query.state = this.types_arr[0].value;
						this.fetch_order_list();
					}
				},
				fail: () => {},
				complete: () => {}
			});
		}
	}
};
</script>

<style scoped lang="scss">
page {
	overflow: hidden;
}

.DrawerPage {
	position: fixed;
	width: 100vw;
	height: 100vh;
	left: 0vw;
	background-color: $color-bg;
	transition: all 0.4s;
}

.DrawerPage.show {
	transform: scale(0.9, 0.9);
	left: 85vw;
	box-shadow: 0 0 60rpx rgba(0, 0, 0, 0.2);
	transform-origin: 0;
}

.DrawerWindow {
	position: absolute;
	width: 85vw;
	height: 100vh;
	left: 0;
	top: 0;
	transform: scale(0.9, 0.9) translateX(-100%);
	opacity: 0;
	pointer-events: none;
	transition: all 0.4s;
	padding: 150rpx 0;
	margin-top: 120rpx;
}

.DrawerWindow.show {
	transform: scale(1, 1) translateX(0%);
	opacity: 1;
	pointer-events: all;
}

.DrawerClose {
	position: absolute;
	width: 40vw;
	height: 100vh;
	right: 0;
	top: 0;
	color: transparent;
	padding-bottom: 30rpx;
	display: flex;
	align-items: flex-end;
	justify-content: center;
	background-image: linear-gradient(90deg, rgba(0, 0, 0, 0.01), rgba(0, 0, 0, 0.6));
	letter-spacing: 5px;
	font-size: 50rpx;
	opacity: 0;
	pointer-events: none;
	transition: all 0.4s;
}

.DrawerClose.show {
	opacity: 1;
	pointer-events: all;
	width: 15vw;
	color: #fff;
}

.DrawerPage .cu-bar.tabbar .action button.cuIcon {
	width: 64rpx;
	height: 64rpx;
	line-height: 64rpx;
	margin: 0;
	display: inline-block;
}

.DrawerPage .cu-bar.tabbar .action .cu-avatar {
	margin: 0;
}

.DrawerPage .nav {
	flex: 1;
}

.DrawerPage .nav .cu-item.cur {
	border-bottom: 0;
	position: relative;
}

.DrawerPage .nav .cu-item.cur::after {
	content: '';
	width: 10rpx;
	height: 10rpx;
	background-color: currentColor;
	position: absolute;
	bottom: 10rpx;
	border-radius: 10rpx;
	left: 0;
	right: 0;
	margin: auto;
}

.DrawerPage .cu-bar.tabbar .action {
	flex: initial;
}
</style>
