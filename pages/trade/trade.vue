<template>
	<view class="contant" :style="{ 'padding-top': `${statusBarHeight - 5}px` }">
		<view class="flex justify-start align-center " style="height: 80rpx;overflow: hidden;">
			<text class="text-lg text-bold text-white margin-left-lg margin-top">{{ rete.rate }}</text>
		</view>
		<view class="padding flex justify-around types">
			<text
				class="text-lg"
				v-for="(type, index) in type_arr"
				:class="cursor === type.value ? 'text-blue text-bold' : 'text-white'"
				:key="index"
				@click="handle_async_type(type)"
			>
				{{ type.lable }}
			</text>
		</view>
		<view class="" style="height: 83vh;">
			<Open v-if="cursor === 0"></Open>
			<Close v-if="cursor === 1"></Close>
			<Order v-if="cursor === 2"></Order>
			<scroll-view v-if="cursor === 3" scroll-y style="height: 100%;">
				<view>
					<Position v-if="trades.length > 0" v-for="(trade, index) in trades" :key="index" :trade="trade.holding"></Position>
					<Empyt v-show="trades.length <= 0"></Empyt>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
import config from 'config';

import Open from './Open.vue';
import Close from './Close.vue';
import Order from './Order.vue';
import Position from './Postions.vue';
import Empyt from 'components/empty.vue';
import uniDrawer from 'components/uni-drawer/uni-drawer.vue';

import { get_position_swap } from 'constant';
export default {
	components: {
		Open,
		Close,
		Order,
		Position,
		Empyt,
		uniDrawer
	},
	data() {
		return {
			cursor: 2,
			type_arr: [
				{
					lable: 'Open',
					value: 0
				},
				{
					lable: 'Close',
					value: 1
				},
				{
					lable: 'Orders',
					value: 2
				},
				{
					lable: 'Positions',
					value: 3
				}
			],
			trades: [],
			rete: {}
		};
	},
	computed: {
		statusBarHeight() {
			const { statusBarHeight } = uni.getSystemInfoSync();
			return statusBarHeight;
		}
	},
	onTabItemTap() {
		uni.vibrateShort();
	},
	onLoad() {
		this.init();
	},
	methods: {
		handle_async_type({ value }) {
			this.cursor = value;
		},
		fetch_position_swap() {
			uni.request({
				url: `${config.host}/app/get_position_swap/`,
				method: 'GET',
				data: {},
				success: res => {
					if (res.statusCode === 200) {
						this.trades = res.data;
					}
				},
				fail: () => {},
				complete: () => {}
			});
		},
		fetch_rate() {
			uni.request({
				url: `${config.host}/app/get_rate/`,
				method: 'GET',
				data: {},
				success: res => {
					if (res.statusCode === 200) {
						this.rete = res.data;
					}
				},
				fail: () => {},
				complete: () => {}
			});
		},
		init() {
			this.fetch_position_swap();
			this.fetch_rate();
		}
	}
};
</script>

<style lang="scss" scoped>
.types {
	border-bottom: 1rpx solid #000;
}
.contant {
	position: fixed;
	height: 100%;
	width: 100%;
	overflow: hidden;
}
</style>
