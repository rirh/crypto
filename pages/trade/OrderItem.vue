<template>
	<!-- 
	参数名	参数类型	描述
	instrument_id	String	合约名称，如BTC-USD-SWAP
	client_oid	String	用户设置的订单ID
	size	String	委托数量
	timestamp	String	创建时间
	filled_qty	String	成交数量
	fee	String	手续费
	order_id	String	订单id
	price	String	委托价格
	price_avg	String	成交均价
	type	String	1:开多
	2:开空
	3:平多
	4:平空
	contract_val	String	合约面值
	order_type	String	0：普通委托
	1：只做Maker（Post only）
	2：全部成交或立即取消（FOK）
	3：立即成交并取消剩余（IOC）
	4：市价委托
	state	String	-2:失败
	-1:撤单成功
	0:等待成交
	1:部分成交
	2:完全成交
	3:下单中
	4:撤单中
	trigger_price	String	强平的真实触发价格，仅强平单会返回此字段
	leverage	String	杠杆倍数 -->
	<view>
		<view class="padding flex justify-start align-center">
			<view class="flex-sub">
				<text :class="[params.type === '1' || params.type === '3' ? 'bg-blue' : 'bg-red']" class="text-white padding-xs radius">{{ type_to_text(params.type) }}</text>
				<text class="margin-left text-white ">{{ params.instrument_id.split('-').join('') }}</text>
				<text class="margin-left text-white text-xs">{{ to_time(params.timestamp) }}</text>
			</view>
			<button @click="handle_cancel" class="cu-btn bg-red">Cancel</button>
		</view>
		<view class="flex text-white padding-left padding-right">
			<view style="width: 50%;" class="">
				<text>Order Anmt:{{ params.size }} Count</text>
			</view>
			<view style="width: 50%;" class="">
				<text>Order Price: $ {{ params.price }}</text>
			</view>
		</view>
		<view class="flex text-white margin-top-xs padding-left padding-right">
			<view style="width: 50%;" class="">
				<text>Filled Anmt:{{ params.filled_qty }} Count</text>
			</view>
			<view style="width: 50%;" class="">
				<text>Margin: $ {{ `${Number(params.contract_val) / Number(params.leverage).toFixed(2)}` }} USDT</text>
			</view>
		</view>
	</view>
</template>

<script>
import moment from 'moment';
export default {
	props: {
		params: {
			type: Object,
			default: () => {}
		}
	},
	onReady() {},
	methods: {
		handle_cancel(){
			this.$emit('on-revoke')
		},
		type_to_text(type) {
			let result = '';
			type = Number(type);
			switch (type) {
				case 1:
					result = 'Open long';
					break;
				case 2:
					result = 'Open short';
					break;
				case 3:
					result = 'Close long';
					break;
				case 4:
					result = 'Close short';
					break;
				default:
					break;
			}
			return result;
		},
		to_time(time) {
			return moment(time).format('HH:mm');
		}
	}
};
</script>

<style scoped></style>
