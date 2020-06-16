<template>
	<view class="">
		<view class="padding text-sm" v-for="(hold, j) in trade" :key="j"><Holding :hold="hold"></Holding></view>
	</view>
</template>

<script>
import Holding from './Holding.vue';
export default {
	components: {
		Holding
	},
	props: {
		trade: {
			type: Array | Object,
			default: () => {}
		}
	}
};
</script>

<style scoped></style>
