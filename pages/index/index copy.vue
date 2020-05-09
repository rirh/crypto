<template>
	<view class="constain">
		<view class="card">
			<view class=""></view>
			<view class="" style="width:100%;display: flex;justify-content: space-between;">
				<view class="">{{ now }}</view>
				<view class="">
					<text style="color: #666;font-size: 32rpx;margin-right: 10rpx;font-weight: bold;Ï">zh:</text>
					<switch :checked="iszh" @change="iszh = !iszh" />
				</view>
			</view>

			<view class="view">
				<view class="" v-for="(k, v) in params" :key="v">{{ iszh ? `${zh[v] || '参数'}&nbsp;:&nbsp;` : `${v || ''}&nbsp;:&nbsp;` }}{{ k }}</view>
			</view>
		</view>
		<scroll-view scroll-x="true" class="tag-wapper" :style="{ 'margin-bottom': `${statusBarHeight}px` }" scroll-with-animation>
			<view class="tag-contain">
				<view :class="['tag', item.name === current.name && 'current']" v-for="(item, index) in list" :key="index" @click="handle_change_current(item)">
					{{ item.name }}
				</view>
			</view>
		</scroll-view>
	</view>
</template>

<script>
const host = 'https://api.huzhihui.org.cn';
// import Fab from 'components/uni-fab/uni-fab.vue';
// import uniCountdown from 'components/uni-countdown/uni-countdown.vue'

const statusBarHeight = uni.getSystemInfoSync().statusBarHeight;
export default {
	components: {
		// Fab,  uniCountdown
	},
	data() {
		return {
			iszh: false,
			running: false,
			now:'',
			statusBarHeight: statusBarHeight,
			zh: {
				instrument_id: '币对名称',
				product_id: '币对名称',
				last: '最新成交价',
				last_qty: '最新成交的数量',
				best_ask: '卖一价',
				best_ask_size: '卖一价对应的量',
				ask: '卖方深度',
				bid: '买方深度',
				best_bid: '买一价',
				best_bid_size: '买一价对应的数量',
				open_24h: '24小时开盘价',
				high_24h: '24小时最高价',
				low_24h: '24小时最低价',
				base_volume_24h: '24小时成交量，按交易货币统计',
				quote_volume_24h: '24小时成交量，按计价货币统计',
				timestamp: '系统时间戳'
			},
			current: {},
			params: {},
			timer: null,
			list: [
				{
					name: 'BSVUSDT',
					value: 'BSV-USDT'
				},
				{
					name: 'BTCUSDT',
					value: 'BTC-USDT'
				},
				{
					name: 'EOSUSDT',
					value: 'EOS-USDT'
				},
				{
					name: 'ETHUSDT',
					value: 'ETH-USDT'
				},
				{
					name: 'BCHUSDT',
					value: 'BCH-USDT'
				}
			]
		};
	},
	onShareAppMessage(e) {},
	onLoad() {
		this.handle_change_current(this.list[0]);
		this.running = true;
		this.async_symbol();
		this.async_timer()
	},
	onShow() {
		this.running = true;
		this.async_symbol();
		this.async_timer()
		
	},
	onHide() {
		this.running = false;
	},
	methods: {
		fetch(url) {
			return new Promise((reslove, reject) => {
				uni.request({
					url,
					method: 'GET',
					data: {},
					success: res => {
						reslove(res);
					},
					fail: () => {
						reject();
					},
					complete: () => {}
				});
			});
		},
		async async_timer() {
			const url = `https://crypto.huzhihui.org.cn`;
			const res = await this.fetch(url);
			// this.now=res.data.iso
		},
		async async_symbol() {
			const timer = async () => {
				// if (this.timer) clearTimeout(this.tim);
				const symbol = this.current.value;
				const url = `${host}/crypto/${symbol}`;
				const { data } = await this.fetch(url);
				if (data.code === 200) {
					uni.hideLoading();
					this.params = data.body;
					if (this.running) {
						// timer();
					}

					// this.timer = setTimeout(() => , 500);
				}
			};
			timer();
		},
		handle_change_current(item) {
			uni.vibrateShort();
			uni.showLoading({
				title: '加载中...'
			});
			this.current = item;
			//
		}
	}
};
</script>

<style lang="scss" scoped>
page {
	background-color: #f4f4f4;
}
.constain {
	position: fixed;
	height: 100%;
	width: 100%;
	left: 0;
	top: 0;
	display: flex;
	flex-direction: column;
	background-image: linear-gradient(180deg, #1ac8d1, #e2cc9e);
	background-repeat: no-repeat;
	background-size: 100% 100%;
}
.tag-wapper {
	margin: 20rpx;
	border-radius: 40rpx;
	width: 90%;
	overflow: scroll;
	padding: 20rpx;
	background-color: rgba($color: #000, $alpha: 0.1);
	.tag-contain {
		display: flex;
		justify-content: flex-start;
		align-items: center;
		.tag {
			padding: 20rpx;
			margin: 10rpx 20rpx;
			color: #666;
			font-size: 28rpx;
			background-color: #f4f4f4;
			border-radius: 30rpx;
			font-weight: bold;
		}
		.current {
			background-color: #268974;
			color: #fff;
		}
	}
}
.card {
	flex: 1;
	display: flex;
	justify-content: flex-end;
	align-items: center;
	flex-direction: column;
	.view {
		margin: 20rpx;
		border-radius: 40rpx;
		box-shadow: 0 0 10rpx rgba(255, 255, 255, 0.3);
		width: 90%;
		padding: 20rpx;
		background-color: rgba($color: #f4f4f4, $alpha: 0.3);
		color: #333;
		font-weight: bold;
		font-size: 28rpx;
	}
}
</style>
