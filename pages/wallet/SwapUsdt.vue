<template>
	<view class="">
		<view class="text-bold">{{ swap.underlying }}账户权益 {{ swap.total_avail_balance }}</view>
		<view class="flex">
			<view class="text-left padding-top" style="width: 50%;">
				<view class="text-grey text-sm">已经实现盈亏</view>
				<view class="padding-top-xs">{{ swap.realized_pnl }}</view>
			</view>
			<view class="text-left padding-top" style="width: 50%;">
				<view class="text-grey text-sm">未实现盈亏</view>
				<view class="padding-top-xs">{{ swap.unrealized_pnl }}</view>
			</view>
		</view>
		<view class="flex">
			<view class="text-left padding-top" style="width: 50%;">
				<view class="text-grey text-sm">已用保证金</view>
				<view class="padding-top-xs">{{ swap.margin }}</view>
			</view>
			<view class="text-left padding-top" style="width: 50%;">
				<view class="text-grey text-sm">保证金率</view>
				<view class="padding-top-xs">{{ swap.margin_ratio }}</view>
			</view>
		</view>
		<view class="flex">
			<view class="text-left padding-top" style="width: 50%;">
				<view class="text-grey text-sm">维持保证金率</view>
				<view class="padding-top-xs">{{ swap.maint_margin_ratio || '0.00' }}</view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	props: {
		swap: {
			type: Object,
			default: () => {}
		}
	}
};
</script>

<style scoped></style>
