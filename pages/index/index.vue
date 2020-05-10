<template>
	<view class="wapper" v-if="show">
		<view class="header">
			<view class="item" :class="{ active: cur === quote }" v-for="(k, quote) in cryptos" @click="handle_change_quote(quote)" :key="quote">{{ quote }}</view>
		</view>
		<view class="list">
			<view class="item" v-for="crypto in list" :key="crypto.instrument_id">
				<view class="flex-sub">
					<view>
						<text class="base">{{ crypto.instrument_id.split('-')[0] }}</text>
						<text class="quote">/{{ crypto.instrument_id.split('-')[1] }}</text>
					</view>
					<view class="base_volume_24h">
						<text>24H成交量 {{ Math.floor(crypto.base_volume_24h) }}</text>
					</view>
				</view>
				<view class="flex-sub ">
					<view class="last">
						<text>{{ crypto.last }}</text>
					</view>
					<view class="usd">
						<text>${{ to2Fixed(crypto.last * btc.last) }}</text>
					</view>
				</view>
				<view class="flex-sub ptc">
					<text :style="{ 'background-color': ((crypto.last - crypto.open_24h) / crypto.open_24h) * 100 > 0 ? '#2d8cf0' : '#ed4014' }" class="pr">{{ ptc(crypto) }}</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
// const host = 'https://crypto.huzhihui.org.cn';
const host = 'http://127.0.0.1:8000';

export default {
	onLoad() {
		this.get_list();
		this.merge_list();
	},
	// onPullDownRefresh() {
	// 	this.get_list();
	// },
	computed: {
		list() {
			return this.cryptos[this.cur] || [];
		}
	},
	data() {
		return {
			show: false,
			cur: '',
			btc: {},
			cryptos: {}
		};
	},
	methods: {
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
		merge_list() {
			var socketTask = uni.connectSocket({
				url: 'wss://real.okex.com:8443/ws/v3', //仅为示例，并非真实接口地址。
				complete: () => {}
			});
			uni.onSocketOpen(function(res) {
				console.log('WebSocket连接已打开！');
				uni.sendSocketMessage({
					data: { op: 'subscribe', args: ['spot/ticker:ETH-USDT', 'spot/ticker:EOS-USDT'] }
				});
			});
			uni.onSocketError(function(res) {
				console.log('WebSocket连接打开失败，请检查！');
			});
			uni.onSocketMessage(function(res) {
				console.log((res.data));
				// uni.sendSocketMessage({
				// 	data: { op: 'subscribe', args: ['spot/ticker:ETH-USDT', 'spot/ticker:EOS-USDT'] }
				// });
			});
		},
		get_list() {
			uni.showLoading({
				title: '加载中...'
			});
			uni.request({
				url: `${host}/app/get_ticker/`,
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
						const [quote] = Object.keys(this.cryptos);
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
@media (prefers-color-scheme: dark) {
	.page {
		background-color: #1b1c1e;
		color: white;
	}
}
page {
	position: absolute;
	background-color: #f4f4f4;
	font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
}
$text-xl: 28rpx;
$text-ml: 24rpx;
$text-sm: 20rpx;
$color-bg: #f4f4f4;
$color-green: #2d8cf0;
$color-red: #ed4014;
$color-text-title: #000;
$color-text-sub: #666;
$color-text-less: #999;
$color-color-white: white;

.wapper {
	position: fixed;
	display: flex;
	flex-direction: column;
	height: 100%;
	width: 100%;
	top: 0;
	overflow: hidden;
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
	overflow-y: scroll;
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
		.quote {
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
	// padding: 10rpx 15rpx;
	// border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}
</style>
