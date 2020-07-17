<script>
export default {
	onLaunch: function() {
		uni.navigateTo({
			url: '/pages/auth/auth'
		});
		console.log('App Launch');
	},
	onShow: function() {
		console.log('App Show');
	},
	onHide: function() {
		console.log('App Hide');
	}
};
</script>

<style lang="scss">
page {
	background-color: $color-bg;
	font-family: 'SF Pro Text', 'SF Pro Icons', 'Helvetica Neue', 'Helvetica', 'Arial', sans-serif;
}
/*每个页面公共css */
@import 'css/main.css';
@import 'css/icon.css';
</style>
