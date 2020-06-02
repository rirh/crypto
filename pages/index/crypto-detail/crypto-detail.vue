<template>
	<view>
		<view v-show="baseinfo.last" class="flex justify-between align-center padding">
			<view class="">
				<view class="">
					<text class=" text-bold" :style="{ color: ((baseinfo.last - baseinfo.open_24h) / baseinfo.open_24h) * 100 > 0 ? '#2d8cf0' : '#ed4014', 'font-size': '60rpx;' }">
						{{ baseinfo.last }}
					</text>
				</view>
				<view class="text-28" style="color: #F4F4F4;">
					<text class="sub">≈</text>
					<text class="sub margin-right-xs">{{ baseinfo.last }}</text>
					<text class="sub">USD</text>
					<text class="margin-left-sm" :style="{ color: ((baseinfo.last - baseinfo.open_24h) / baseinfo.open_24h) * 100 > 0 ? '#2d8cf0' : '#ed4014' }">
						{{ ptc(baseinfo) }}
					</text>
				</view>
			</view>
			<view class="" style="color:#f4f4f4">
				<view class="flex justify-between">
					<text class="sub">高</text>
					<text class="text-white margin-left">{{ baseinfo.high_24h }}</text>
				</view>
				<view class="flex justify-between">
					<text class="sub">低</text>
					<text class="text-white margin-left">{{ baseinfo.low_24h }}</text>
				</view>
				<view class="flex justify-between">
					<text class="sub">24H</text>
					<text class="text-white margin-left">{{ baseinfo.quote_volume_24h }}</text>
				</view>
			</view>
		</view>
		<canvas style="width: 100%; height: 300px;" canvas-id="kline"></canvas>
		<view class="flex justify-center align-center">
			<text class="tab" @click="handle_change_active(item)" :class="{ active: item.lable === activetab }" v-for="(item, index) in tabs" :key="index">{{ item.lable }}</text>
		</view>
		<view class="padding-left padding-top">深度时间：{{ time_to_txt(depth.timestamp) }}</view>
		<view class="padding flex justify-between align-center">
			<view class="">
				<text>买盘</text>
				<text>数量({{ params.instrument_id && params.instrument_id.split('-')[0] }})</text>
			</view>
			<view class="">
				<text>价格({{ params.instrument_id && params.instrument_id.split('-')[1] }})</text>
			</view>
			<view class="">
				<text>数量({{ params.instrument_id && params.instrument_id.split('-')[0] }})</text>
				<text>买盘</text>
			</view>
		</view>
		<view class="flex padding-left padding-right padding-bottom t">
			<view class="" style="width: 50%;">
				<view class="flex depth" style="width: 100%;" v-for="(bid, i) in depth.bids" :key="i">
					<view class="">
						<text style="width: 40rpx;display: inline-block;">{{ i + 1 }}</text>
						<text class="margin-left">{{ to_fixd(bid[1]) }}</text>
					</view>
					<text class="flex-sub text-right">{{ bid[0] }}</text>
					<view class="tips ask-tips" :style="{ width: `${to_width(depth.bids, bid)}` }"></view>
				</view>
			</view>
			<view class="" style="width: 50%;">
				<view class="flex depth" style="width: 100%;" v-for="(ask, i) in depth.asks" :key="i">
					<text class="flex-sub text-left">{{ ask[0] }}</text>
					<view class="text-right">
						<text class="margin-right">{{ to_fixd(ask[1]) }}</text>
						<text style="width: 40rpx;display: inline-block;">{{ i + 1 }}</text>
					</view>

					<view class="tips bid-tips" :style="{ width: `${to_width(depth.asks, ask)}` }"></view>
				</view>
			</view>
			<!-- <view class="" style="width: 50%;">
				<view class="" v-for="(bid, j) in depth.bids" :key="j">{{ bid.toString() }}</view>
			</vie -->
		</view>
		<!-- <Soket :instrument_id="params.instrument_id" /> -->
		<!-- <Chart /> -->
	</view>
</template>

<script>
import moment from 'moment';
import config from 'config';
const { statusBarHeight } = uni.getSystemInfoSync();
export default {
	data() {
		return {
			statusBarHeight,
			granularity: 60,
			params: {},
			baseinfo: {},
			/**
			 *
				参数名	类型	    描述
				time	String	开始时间
				open	String	开盘价格
				high	String	最高价格
				low	String	最低价格
				close	String	收盘价格
				volume	String	交易量
			 */

			kline: [],
			/**
			 *
				参数名	类型	描述
				asks	List<String>	卖方深度
				bids	List<String>	买方深度
				timestamp	String	时间戳
			 */
			depth: {},

			end: '',
			activetab: '近1天',
			// 时间粒度granularity必须是[60 180 300 900 1800 3600 7200 14400 21600 43200 86400 604800]中的任一值，否则请求将被拒绝。
			// 这些值分别对应的是[1min 3min 5min 15min 30min 1hour 2hour 4hour 6hour 12hour 1day 1week]的时间段。
			tabs: [
				{
					lable: '近5分',
					value: 1,
					keyworld: 'minutes',
					granularity: 300
				},
				{
					lable: '近30分',
					value: 30,
					keyworld: 'minutes',
					granularity: 1800
				},
				{
					lable: '近4时',
					value: 4,
					keyworld: 'hours',
					granularity: 7200
				},
				{
					lable: '近1天',
					value: 1,
					keyworld: 'days',
					granularity: 86400
				},
				{
					lable: '近1周',
					value: 1,
					keyworld: 'weeks',
					granularity: 604800
				}
			]
		};
	},
	onLoad(params) {
		console.log(params.instrument_id);
		uni.setNavigationBarTitle({
			title: params.instrument_id
		});
		this.params = params;
		this.init();
	},
	methods: {
		to_fixd(e) {
			return e;
		},
		to_width(list, tag) {
			const max = Math.max(...list.map(e => e[1]));
			console.log(max);
			console.log(`${((tag[1] / max) * 100).toFixed(2)}%`);
			return `${((tag[1] / max) * 100).toFixed(2)}%`;
		},
		time_to_txt(time) {
			return moment(time).format('YYYY-MM-DD HH:mm:ss');
		},
		fetch_depth() {
			const { instrument_id } = this.params;
			uni.request({
				url: `${config.host}/app/get_depth/`,
				method: 'GET',
				data: {
					instrument_id,
					size: 15
				},
				success: res => {
					if (res.statusCode === 200) {
						this.depth = res.data;
						// setTimeout(() => {
						this.fetch_depth();
						// }, 1000);
					}
				},
				fail: () => {},
				complete: () => {}
			});
		},
		handle_change_active(data) {
			this.activetab = data.lable;
			this.change_end_time();
			this.get_kline();
		},
		change_end_time() {
			const { value, keyworld, granularity = 60 } = this.tabs.find(e => e.lable === this.activetab);
			this.granularity = granularity;
			this.end = moment()
				.subtract(value, keyworld)
				.toISOString();
		},
		switch_end() {
			this.end = moment()
				.subtract(1, 'days')
				.toISOString();
		},
		get_kline() {
			uni.showLoading({
				mask: true,
				title: 'loading'
			});
			const instrument_id = this.params.instrument_id;
			const granularity = this.granularity;
			const end = this.end;
			uni.request({
				url: `${config.host}/app/get_kline/`,
				// url: 'https://www.btcalory.com/api/512334D90902F08B/timer/',
				method: 'GET',
				data: {
					instrument_id,
					start: encodeURIComponent(moment().toISOString()),
					end: encodeURIComponent(moment().toISOString()),
					granularity
				},
				success: ({ statusCode, data }) => {
					uni.hideLoading();
					// console.log(statusCode, data);
					if (statusCode === 200) {
						this.kline = data;
						this.initcanvas();
					}
				}
			});
		},
		initcanvas() {
			this.context = uni.createCanvasContext('kline');
			this.drawkline();
		},
		drawkline() {
			const ctx = this.context;
			console.log(ctx.height, ctx.width);
			const height = 100;
			const { screenWidth } = uni.getSystemInfoSync();

			// '#2d8cf0' : '#ed4014'
			ctx.setStrokeStyle('#2d8cf0');
			ctx.setShadow(1, 100, 8, '#ed4014');

			// 计算最高价格
			const max_price = Math.max(...this.kline.map(([, , e]) => e));
			const min_price = Math.min(...this.kline.map(([, , , e]) => e));
			console.log(min_price, max_price);
			const len = this.kline.length;
			const clo = screenWidth / len;
			const income = max_price - min_price;
			const [[, open, high, low]] = this.kline;
			ctx.moveTo(clo, height - ((high - low) / income) * height);
			this.kline.forEach(([, open, high, low], i) => {
				const x = (i + 1) * clo;
				const y = height - ((high - low) / income) * height;
				ctx.lineTo(x, y);
				if (Number(low) === min_price) {
					ctx.setFillStyle('white');
					ctx.fillText(low, x, y + 20);
					ctx.stroke();
				} else if (Number(high) === max_price) {
					ctx.setFillStyle('white');
					ctx.fillText(high, x, y - 20);
					ctx.stroke();
				}
			});
			ctx.stroke();
			ctx.draw();
		},
		ptc({ last, open_24h }) {
			const pr = ((last - open_24h) / open_24h) * 100;
			return `${(pr > 0 && '+') || ''}${pr.toFixed(2)}%`;
		},
		get_specific_ticker() {
			uni.showLoading({
				mask: true,
				title: 'loading'
			});
			const { instrument_id } = this.params;
			uni.request({
				url: `${config.host}/app/get_specific_ticker/`,
				// url: 'https://www.btcalory.com/api/512334D90902F08B/timer/',
				method: 'GET',
				data: { instrument_id },
				success: ({ statusCode, data }) => {
					uni.hideLoading();
					// console.log(statusCode, data);
					if (statusCode === 200) {
						this.baseinfo = data;
					}
				}
			});
		},

		init() {
			this.get_specific_ticker();
			this.switch_end();
			this.get_kline();
			this.fetch_depth();
		}
	}
};
</script>

<style lang="scss">
page {
	background-color: #1b1c1e;
	color: white;
	.sub {
		color: $color-text-sub;
	}

	.tab {
		padding: 10rpx 30rpx;
		border-radius: 10rpx;
	}
	.active {
		background-color: #4d4d4d;
	}
	.depth {
		font-size: $uni-font-size-base;
		position: relative;
		padding: 10rpx 0;
		.tips {
			position: absolute;
			height: 100%;
			top: 0;
		}
		.ask-tips {
			right: 0;
			background-color: $color-green;
			opacity: 0.3;
		}
		.bid-tips {
			left: 0;
			background-color: $color-red;
			opacity: 0.3;
		}
	}
}
</style>
