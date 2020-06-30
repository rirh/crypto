<template>
	<view class="flex padding">
		<text style="width: 30%;">{{ account.currency }}</text>
		<text style="width: 40%;text-align: center;">{{ account.available }}</text>
		<text style="width: 30%;text-align: right;">{{ account.frozen }}</text>
	</view>
</template>

<script>
export default {
	props: {
		account: {
			type: Object|Array,
			default: () => {}
		}
	}
};
</script>

<style scoped></style>
