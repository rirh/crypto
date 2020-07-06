<template>
	<view>
		<view class="flex justify-between align-center padding ">
			<text class="text-white">Instrument</text>
			<text class="text-white text-bold">{{ form.instrument_id }}</text>
		</view>
		<view class="flex justify-between align-center padding">
			<view
				class="text-gray"
				v-for="type in type_arr"
				:key="type.value"
				:class="[`${type.value === form.type && 'bg-blue cu-tag text-white'}`]"
				@click="handle_change_type(type)"
			>
				<text>{{ type.label }}</text>
			</view>
		</view>

		<view class="padding-left padding-right padding-bottom flex  justify-between align-center">
			<view
				class="text-gray"
				v-for="order in order_type_arr"
				:key="order.value"
				:class="[`${order.value === form.order_type && 'bg-blue cu-tag text-white'}`]"
				@click="handle_change_order_type(order)"
			>
				<text>{{ order.label }}</text>
			</view>
		</view>
		<view class="padding-left padding-right padding-bottom">
			<input
				class="price padding-left text-lg text-bold shadow"
				auto-focus
				:placeholder="`${type_arr.find(e => e.value === form.type).label} price`"
				type="number"
				:value="form.price"
			/>
		</view>
		<view class="flex justify-between align-center padding-left padding-right padding-bottom ">
			<text class="text-white">Match Price</text>
			<switch class="blue" @change="handle_change_match_price" :class="form.match_price === '1' ? 'checked' : ''" :checked="form.match_price === '1' ? true : false"></switch>
		</view>
		<view class="flex justify-between align-center padding-left padding-right padding-bottom ">
			<text class="text-white">Size</text>
			<text class="text-white text-bold">{{ form.size }} Count</text>
		</view>
		<view class="flex justify-between align-center padding-left padding-right padding-bottom "></view>
		<view class="padding"><button :loading="loading" @click="handle_take_order" class="cu-btn block bg-blue margin-tb-sm lg">尝试开单</button></view>
	</view>
</template>

<script>
import config from 'config';
export default {
	data() {
		return {
			loading: false,
			form: {
				instrument_id: 'EOS-USDT-SWAP',
				// 可填参数：
				// 1:开多
				// 2:开空
				// 3:平多
				// 4:平空
				type: '1',
				// 参数填数字
				// 0：普通委托（order_type不填或填0都是普通委托）
				// 1：只做Maker（Post only）
				// 2：全部成交或立即取消（FOK）
				// 3：立即成交并取消剩余（IOC）
				// 4：市价委托
				order_type: '0',
				//对手价 0 no 1 yes
				match_price: '0',
				price: '',
				size: '1'
			},
			type_arr: [
				{
					label: 'Open long',
					value: '1'
				},
				{
					label: 'Open short',
					value: '2'
				},
				{
					label: 'Close long',
					value: '3'
				},
				{
					label: 'Close short',
					value: '4'
				}
			],
			order_type_arr: [
				{
					label: 'Normal',
					value: '0'
				},
				{
					label: 'Post only',
					value: '1'
				},
				{
					label: 'FOK',
					value: '2'
				},
				{
					label: 'IOC',
					value: '3'
				},
				{
					label: 'Makter',
					value: '4'
				}
			],
			key: ''
		};
	},
	methods: {
		handle_change_match_price(e) {
			this.form.match_price = e.detail.value ? '1' : '0';
		},
		handle_change_order_type(order) {
			this.form.order_type = order.value;
		},
		handle_change_type(type) {
			this.form.type = type.value;
		},
		handle_take_order() {
			if (this.loading) return;
			this.loading = true;
			uni.request({
				url: `${config.host}/app/take_order/`,
				method: 'GET',
				data: this.form,
				success: res => {
					this.loading = false;
					if (res.statusCode === 200) {
						uni.showToast({
							icon: 'none',
							duration: 5000,
							title: `${this.form.instrument_id} ${this.type_arr.find(e => e.value === this.form.type).label} success`
						});
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
.price {
	background-color: rgba($color: #000000, $alpha: 0.4);
	color: #fff;
	border-radius: 20rpx;
	height: 90rpx;
	line-height: 90px;
	display: flex;
	justify-content: flex-start;
	align-items: center;
}
</style>
