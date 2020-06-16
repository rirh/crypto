<template>
	<view>
		<view class="text-white">
			<text class="text-df text-bold">{{ hold.instrument_id }}</text>
			<text style="height: 35rpx;" class="cu-tag margin-left-sm radius " :class="[hold.side === 'long' ? 'bg-blue' : 'bg-red']">
				{{ hold.side }}&nbsp;{{ hold.leverage }}x
			</text>
			<text>{{ holding.margin_mode }}</text>
		</view>
		<view class="flex margin-top-sm text-grey ">
			<view class="" style="width: 50%;">
				<view class="">Open Price</view>
				<view class="text-white margin-top-sm text-bold">{{ hold.avg_cost }}</view>
			</view>
			<view class="" style="width: 50%;">
				<view class=" ">PL</view>
				<view class="text-white margin-top-sm text-bold">{{ hold.settled_pnl }} USDT</view>
			</view>
		</view>
		<view class="flex margin-top text-grey ">
			<view class="" style="width: 50%;">
				<view class="">Est. Liquidation Price</view>
				<view class="text-white margin-top-sm text-bold">{{ hold.liquidation_price }}</view>
			</view>
			<view class="" style="width: 50%;">
				<view class="">PL Ratio</view>
				<view class="text-white margin-top-sm text-bold">{{ hold.settled_pnl }} USDT</view>
			</view>
		</view>
		<view class="flex padding-top-lg justify-around text-grey ">
			<view class="flex justify-between" style="width: 50%;">
				<text style="width: 50%;">UPL</text>
				<text style="width: 50%;">{{ hold.unrealized_pnl }}</text>
			</view>
			<view class="flex justify-around" style="width: 50%;">
				<text class="" style="width: 50%;">Margin</text>
				<text class="" style="width: 50%;">{{ hold.margin }} USDT</text>
			</view>
		</view>
		<view class="flex margin-top-xs justify-around text-grey ">
			<view class="flex justify-between" style="width: 50%;">
				<text style="width: 50%;">Total Position</text>
				<text style="width: 50%;">{{ hold.position }}</text>
			</view>
			<view class="flex justify-between" style="width: 50%; ">
				<text class="" style="width: 50%;">Margin Ratio</text>
				<text class="" style="width: 50%;">{{ hold.maint_margin_ratio }} Count</text>
			</view>
		</view>
		<view class="flex margin-top-xs justify-around text-grey ">
			<view class="flex" style="width: 50%;">
				<text style="width: 50%;">Avail</text>
				<text style="width: 50%;">{{ hold.avail_position }}</text>
			</view>
			<view class="flex justify-between" style="width: 50%;">
				<text style="width: 50%;" class="">MMR</text>
				<text style="width: 50%;" class="">{{ hold.maint_margin_ratio }} %</text>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	props: {
		hold: {
			type: Object | Array,
			default: () => {}
		}
	}
};
</script>

<style scoped></style>
