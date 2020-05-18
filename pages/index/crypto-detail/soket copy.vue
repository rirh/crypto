<template>
	<view class="">
		<canvas style="width: 300px; height: 200px;" canvas-id="firstCanvas"></canvas>
		
	</view>
</template>

<script>
import config from 'config';
import moment from 'moment';
export default {
	data() {
		return {
			kline: [],
			end: '',
			activetab: '近1天',
			tabs: [
				{
					lable: '近1小时',
					value: 1,
					keyworld: 'hours'
				},
				{
					lable: '近1天',
					value: 1,
					keyworld: 'days'
				},
				{
					lable: '近1周',
					value: 1,
					keyworld: 'weeks'
				},
				{
					lable: '近1月',
					value: 1,
					keyworld: 'months'
				},
				{
					lable: '近1年',
					value: 1,
					keyworld: 'years'
				}
			]
		};
	},
	props: {
		instrument_id: {
			type: String,
			default: 'BTC-USDT'
		}
	},
	mounted() {
		this.get_kline();
		this.init_canvas();
	},

	methods: {
		init_canvas() {
			const context = uni.createCanvasContext('firstCanvas');
			context.setStrokeStyle('#00ff00');
			context.setLineWidth(5);
			context.rect(0, 0, 200, 200);
			context.stroke();
			context.setStrokeStyle('#ff0000');
			context.setLineWidth(2);
			context.moveTo(160, 100);
			context.arc(100, 100, 60, 0, 2 * Math.PI, true);
			context.moveTo(140, 100);
			context.arc(100, 100, 40, 0, Math.PI, false);
			context.moveTo(85, 80);
			context.arc(80, 80, 5, 0, 2 * Math.PI, true);
			context.moveTo(125, 80);
			context.arc(120, 80, 5, 0, 2 * Math.PI, true);
			context.stroke();
			context.draw();
		},
		handle_change_active(data) {
			this.activetab = data.lable;
			this.change_end_time();
			this.get_kline();
		},
		change_end_time() {
			const { value, keyworld } = this.tabs.find(e => e.lable === this.activetab);
			this.end = moment()
				.subtract(value, keyworld)
				.format();
		},
		switch_end() {
			this.end = moment()
				.subtract(1, 'days')
				.format();
		},
		get_kline() {
			uni.showLoading({
				mask: true,
				title: 'loading'
			});
			const instrument_id = this.instrument_id;
			const end = this.end;
			uni.request({
				url: `${config.host}/app/get_kline/`,
				// url: 'https://www.btcalory.com/api/512334D90902F08B/timer/',
				method: 'GET',
				data: { instrument_id, start: moment().format(), end, granularity: 60 },
				success: ({ statusCode, data }) => {
					uni.hideLoading();
					// console.log(statusCode, data);
					if (statusCode === 200) {
						this.kline = data;
					}
				}
			});
		}
	}
};
</script>

<style scoped lang="scss">
.tab {
	padding: 10rpx 30rpx;
	border-radius: 10rpx;
}
.active {
	background-color: #4d4d4d;
}
</style>
