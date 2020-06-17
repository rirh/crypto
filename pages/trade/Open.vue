<template>
	<div><button @click="handle_take_order" class="cu-btn">尝试开单</button></div>
</template>

<script>
import config from 'config';
export default {
	data() {
		return {
			key: ''
		};
	},
	methods: {
		handle_take_order() {
			uni.request({
				url: `${config.host}/app/take_order/`,
				method: 'GET',
				data: {
					instrument_id:'EOS-USDT-SWAP',
					// 参数填数字
					// 0：普通委托（order_type不填或填0都是普通委托）
					// 1：只做Maker（Post only）
					// 2：全部成交或立即取消（FOK）
					// 3：立即成交并取消剩余（IOC）
					// 4：市价委托
					order_type:'0',
					// 可填参数：
					// 1:开多
					// 2:开空
					// 3:平多
					// 4:平空
					type:'1',
					size:'1',
					// 是否以对手价下单。
					// 0:不是; 1:是。当以对手价下单，order_type只能选择0（普通委托）
					match_price:'0',
					price:'1'			
				},
				success: res => {
					if (res.statusCode === 200) {
						this.trades = res.data;
					}
				},
				fail: () => {},
				complete: () => {}
			});
		}
	}
};
</script>

<style scoped></style>
