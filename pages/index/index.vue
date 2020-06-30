<template>
	<view class="wapper" :style="{ 'padding-top': `${statusBarHeight - 5}px` }" v-if="show">
		<view class="cu-bar search ">
			<view class="search-form round">
				<text class="cuIcon-search"></text>
				<input v-model="filter" @focus="InputFocus" @blur="InputBlur" :adjust-position="false" type="text" placeholder="search instrument " confirm-type="search" />
			</view>
			<view class="" style="width: 23vw"></view>
		</view>
		<view class="header">
			<view class="item" :class="{ active: cur === quote }" v-for="(k, quote) in cryptos" @click="handle_change_quote(quote)" :key="quote">{{ quote }}</view>
		</view>
		<scroll-view class="list" scroll-with-animation scroll-y>
			<view v-if="list.length" class="item" @click="handle_detail(crypto)" v-for="crypto in list" :key="crypto.instrument_id">
				<view class="flex-sub">
					<view>
						<text class="base">{{ crypto.instrument_id.split('-')[0] }}</text>
						<text class="quote">/{{ crypto.instrument_id.split('-')[1] }}</text>
					</view>
					<view class="base_volume_24h">
						<text>24H Vol {{ Math.floor(crypto.base_volume_24h) }}</text>
					</view>
				</view>
				<view class="flex-sub ">
					<view class="last">
						<text>{{ crypto.last }}</text>
					</view>
					<view class="usd">
						<text>${{ to2Fixed(crypto.last * btc.last) }}</text>
						<text class="calory">10 C</text>
					</view>
				</view>
				<view class="flex-sub ptc">
					<text :style="{ 'background-color': ((crypto.last - crypto.open_24h) / crypto.open_24h) * 100 > 0 ? '#2d8cf0' : '#ed4014' }" class="pr">{{ ptc(crypto) }}</text>
				</view>
			</view>
			<Empty v-else></Empty>
		</scroll-view>
	</view>
</template>

<script>
import config from 'config';
import pako from 'pako';
import Empty from 'components/empty.vue';
export default {
	components: {
		Empty
	},
	onLoad() {
		this.get_list();
		this.merge_list();
	},
	// onPullDownRefresh() {
	// 	this.get_list();
	// },
	computed: {
		statusBarHeight() {
			const { statusBarHeight } = uni.getSystemInfoSync();
			return statusBarHeight;
		},
		list() {
			let result = [];
			if (this.filter) {
				const list = this.cryptos[this.cur];
				const tag = this.filter.toLocaleUpperCase();
				result = list.filter(e => ~e.instrument_id.indexOf(tag));
			} else {
				result = this.cryptos[this.cur];
			}
			return result || [];
		}
	},
	data() {
		return {
			show: false,
			cur: '',
			btc: {},
			cryptos: {},
			InputBottom: 0,
			filter: ''
		};
	},
	onTabItemTap() {
		uni.vibrateShort();
	},
	methods: {
		InputFocus(e) {
			this.InputBottom = e.detail.height;
		},
		InputBlur(e) {
			this.InputBottom = 0;
		},
		to2Fixed(e) {
			return e.toFixed(2) || '';
		},
		// ptc_color({ last, open_24h }) {
		// 	const pr = ((last - open_24h) / open_24h) * 100;

		// 	const color = pr > 0 ? '$color-green' : '$color-red';
		// 	return {
		// 		'background-color': color
		// 	};
		// },

		ptc({ last, open_24h }) {
			const pr = ((last - open_24h) / open_24h) * 100;
			return `${(pr > 0 && '+') || ''}${pr.toFixed(2)}%`;
		},
		handle_change_quote(quote) {
			this.cur = quote;
			uni.vibrateShort();
		},
		handle_detail({ instrument_id }) {
			this.filter = '';
			uni.navigateTo({
				url: `/pages/index/crypto-detail/crypto-detail?instrument_id=${instrument_id}`
			});
		},
		merge_list() {
			var socketTask = uni.connectSocket({
				url: 'wss://real.okex.com:8443/ws/v3', //仅为示例，并非真实接口地址。
				// url: 'wss://api-aws.huobi.pro/ws', //仅为示例，并非真实接口地址。
				complete: () => {}
			});
			uni.onSocketOpen(function(res) {
				console.log('WebSocket连接已打开！');
				uni.sendSocketMessage({
					data: { op: 'subscribe', args: ['spot/ticker:ETH-USDT', 'spot/ticker:EOS-USDT'] }
					// data: { ping: Date.now() }
				});
			});
			uni.onSocketError(function(res) {
				console.log('WebSocket连接打开失败，请检查！');
			});
			uni.onSocketMessage(res => {
				console.log(res);
				console.log(pako);
				// let text = pako.inflate(res.data);
				// let msg = JSON.parse(text);
				// console.log(text);
				// var enc = new TextDecoder('utf-8');
				// var res = JSON.parse(new Uint8Array(res.data)); //转化成json对象
				// console.log(res);
			});
			// uni.sendSocketMessage({
			// 	data: { op: 'subscribe', args: ['spot/ticker:ETH-USDT', 'spot/ticker:EOS-USDT'] }
			// });
		},
		async get_list() {
			uni.showLoading({
				mask: true,
				title: 'loading'
			});
			uni.request({
				url: `${config.host}/app/get_all_ticker/`,
				// url: 'https://www.btcalory.com/api/512334D90902F08B/timer/',
				method: 'GET',
				// data: { user: '4225417A1091368B', language: 'CN', logId: 1588295682897, refresh: '', exchange: '', account: '', symbol: '', brief: '', t_type: '' },
				success: ({ statusCode, data }) => {
					uni.hideLoading();
					if (statusCode === 200) {
						this.show = true;
						const list = {};
						for (var i = 0; i < data.length; i++) {
							if (data[i].instrument_id === 'BTC-USDT') this.btc = data[i];
							const [base, quote] = data[i].instrument_id.split('-');
							if (!list[quote]) list[quote] = [];
							list[quote].push(data[i]);
						}
						this.cryptos = list;
						const [,quote] = Object.keys(this.cryptos);
						this.cur = quote;
						// this.show=true
					}
				}
			});
		}
	}
};
</script>

<style scoped lang="scss">
page {
	position: absolute;
	background-color: $color-bg;
	font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
}

.wapper {
	position: fixed;
	display: flex;
	flex-direction: column;
	height: 100%;
	width: 100%;
	top: 0;
	overflow: hidden;
	background-color: $color-bg;
}
.header {
	display: flex;
	padding: 10rpx 0rpx;
	justify-content: space-around;
	align-items: flex-start;
	flex-wrap: nowrap;
	overflow-y: scroll;
	width: 750rpx;
	border-bottom: 1px solid rgba(0, 0, 0, 0.1);
	background-color: $color-bg;

	.item {
		margin: 0 5rpx;
		color: $color-text-sub;
		height: 80rpx;
		font-size: $text-xl;
		display: flex;
		justify-content: center;
		align-items: center;
		font-weight: 400;
	}

	.active {
		color: $color-green;
	}
}

.list {
	flex: 1;
	overflow-x: hidden;
	height: 100%;
	.item {
		display: flex;
		justify-content: center;
		align-items: center;
		padding: 15rpx 20rpx;
		border-bottom: 1px solid rgba(0, 0, 0, 0.03);
		.flex-sub {
			flex: 1;
		}

		.base {
			font-weight: bold;
			font-size: $text-xl;
			color: $color-text-title;
		}
		.quote,
		.calory {
			font-size: $text-sm;
			color: $color-text-less;
			margin-left: 10rpx;
		}
		.base_volume_24h {
			font-size: $text-sm;
			color: $color-text-less;
		}
		.last {
			font-weight: bold;
			font-size: $text-xl;
			color: $color-text-title;
		}
		.usd {
			font-size: $text-ml;
			color: $color-text-less;
		}
		.ptc {
			display: flex;
			justify-content: flex-end;
			align-items: center;
			.pr {
				color: $color-color-white;
				border-radius: 15rpx;
				font-weight: bold;
				font-size: $text-xl;
				padding: 13rpx 0;
				width: 150rpx;
				text-align: center;
			}
		}
	}
	.cu-bar .search-form {
		background-color: #000 !important;
	}
	// padding: 10rpx 15rpx;
	// border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}
</style>
