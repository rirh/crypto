<template>
	<view>
		<view>
			<canvas
				id="kline"
				canvas-id="kline"
				class="kline"
				style="width: 400px; height: 600px"
				@touchstart="KLineTouchStart"
				@touchmove="KLineTouchMove"
				@touchend="KLineTouchEnd"
			></canvas>
		</view>

		<view class="button-sp-area">
			<button class="mini-btn" type="default" size="mini" @click="ChangeKLinePeriod(PERIOD_ID.KLINE_DAY_ID)">日线</button>
			<button class="mini-btn" type="default" size="mini" @click="ChangeKLinePeriod(PERIOD_ID.KLINE_WEEK_ID)">周线</button>
			<button class="mini-btn" type="default" size="mini" @click="ChangeKLinePeriod(PERIOD_ID.KLINE_MINUTE_ID)">1分钟</button>
			<button class="mini-btn" type="default" size="mini" @click="ChangeKLinePeriod(PERIOD_ID.KLINE_15MINUTE_ID)">15分钟</button>
		</view>

		<view class="button-sp-area">
			<button class="mini-btn" type="default" size="mini" @click="ChangeKLineIndex(0, 'BOLL')">BOLL</button>
			<button class="mini-btn" type="default" size="mini" @click="ChangeKLineIndex(1, 'RSI')">RSI</button>
			<button class="mini-btn" type="default" size="mini" @click="ChangeKLineIndex(2, 'WR')">WR</button>
			<button class="mini-btn" type="default" size="mini" @click="ChangeKLineIndex(0, 'MA')">MA</button>
			<button class="mini-btn" type="default" size="mini" @click="ChangeKLineIndex(1, 'VOL')">VOL</button>
			<button class="mini-btn" type="default" size="mini" @click="ChangeKLineIndex(1, 'MACD')">MACD</button>
		</view>

		<view class="button-sp-area">
			<button class="mini-btn" type="default" size="mini" @click="ChangeSymbol('000001.sz')">000001.sz</button>
			<button class="mini-btn" type="default" size="mini" @click="ChangeSymbol('600000.sh')">600000.sh</button>
		</view>
		
	</view>
</template>

<script>
import { JSCommon } from 'js_sdk/jones-hqchart/umychart.uniapp/umychart.wechat.3.0.js';

function DefaultData() {}

DefaultData.GetKLineOption = function() {
	let data = {
		Type: '历史K线图',

		//窗口指标
		Windows: [{ Index: 'MA', Modify: false, Change: false }, { Index: 'VOL', Modify: false, Change: false }, { Index: 'MACD', Modify: false, Change: false }],

		CorssCursorTouchEnd: true,
		IsShowRightMenu: false, //是否显示右键菜单
		CorssCursorInfo: { Left: 2, Right: 2 },

		//边框
		Border: {
			Left: 1,
			Right: 1, //右边间距
			Top: 1,
			Bottom: 25
		},

		KLine: {
			Right: 1, //复权 0 不复权 1 前复权 2 后复权
			Period: 0, //周期: 0 日线 1 周线 2 月线 3 年线
			PageSize: 12,
			IsShowTooltip: false
		},

		//子框架设置 (Height 窗口高度比例值)
		Frame: [
			{
				SplitCount: 3,
				//Height:4,
				IsShowLeftText: true,
				IsShowRightText: false
			},
			{
				SplitCount: 2,
				//Height:2,
				IsShowLeftText: true,
				IsShowRightText: false
			},
			{
				SplitCount: 2,
				//Height:2,
				IsShowLeftText: true,
				IsShowRightText: false
			}
		],

		ExtendChart: [{ Name: 'KLineTooltip' }]
	};

	return data;
};

//周期枚举
var PERIOD_ID = {
	KLINE_DAY_ID: 0,
	KLINE_WEEK_ID: 1,
	KLINE_MONTH_ID: 2,
	KLINE_YEAR_ID: 3,

	KLINE_MINUTE_ID: 4,
	KLINE_5MINUTE_ID: 5,
	KLINE_15MINUTE_ID: 6,
	KLINE_30MINUTE_ID: 7,
	KLINE_60MINUTE_ID: 8
};

var g_KLine = {
	JSChart: null
};

export default {
	name: 'HQChart',

	data() {
		let data = {
			Symbol: '600000.sh',

			KLine: {
				Option: DefaultData.GetKLineOption(),
				IsShow: false,
				Display: 'none',
				Width: 400,
				Height: 600
			},

			PERIOD_ID: PERIOD_ID
		};

		return data;
	},

	onLoad() {},

	onReady() {
		this.ChangeKLinePeriod(PERIOD_ID.KLINE_DAY_ID);
	},

	methods: {
		CreateKLineChart: function() {
			if (this.KLine.JSChart) return;

			let element = new JSCommon.JSCanvasElement();
			// #ifdef APP-PLUS
			element.IsUniApp = true; //canvas需要指定下 是uniapp的app
			// #endif
			element.ID = 'kline';
			element.Height = this.KLine.Height; //高度宽度需要手动绑定!!
			element.Width = this.KLine.Width;

			g_KLine.JSChart = JSCommon.JSChart.Init(element);
			this.KLine.Option.NetworkFilter = this.NetworkFilter;
			this.KLine.Option.Symbol = this.Symbol;
			g_KLine.JSChart.SetOption(this.KLine.Option);
		},

		//K线周期切换
		ChangeKLinePeriod: function(period) {
			if (!g_KLine.JSChart) {
				//不存在创建
				this.KLine.Option.Period = period;
				this.CreateKLineChart();
			} else {
				g_KLine.JSChart.ChangePeriod(period);
			}
		},

		//切换指标 windowIndex=窗口索引 0开始, name=指标名字/ID
		ChangeKLineIndex: function(windowIndex, name) {
			if (!g_KLine.JSChart) return;

			g_KLine.JSChart.ChangeIndex(windowIndex, name);
		},

		//切换股票
		ChangeSymbol: function(symbol) {
			if (!g_KLine.JSChart) return;

			g_KLine.JSChart.ChangeSymbol(symbol);
		},

		NetworkFilter: function(data, callback) {
			console.log('[HQChart:NetworkFilter] data', data.Name);
		},

		//KLine事件
		KLineTouchStart: function(event) {
			if (g_KLine.JSChart) g_KLine.JSChart.OnTouchStart(event);
		},

		KLineTouchMove: function(event) {
			if (g_KLine.JSChart) g_KLine.JSChart.OnTouchMove(event);
		},

		KLineTouchEnd: function(event) {
			if (g_KLine.JSChart) g_KLine.JSChart.OnTouchEnd(event);
		}
	}
};
</script>

<style>
.content {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
}

.logo {
	height: 200upx;
	width: 200upx;
	margin-top: 200upx;
	margin-left: auto;
	margin-right: auto;
	margin-bottom: 50upx;
}

.text-area {
	display: flex;
	justify-content: center;
}

.title {
	font-size: 36upx;
	color: #8f8f94;
}
</style>
