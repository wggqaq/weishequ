<script>
	export default {
	

		onLaunch: function() {
			console.log('App Launch')
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		},
		
	}
</script>

/* App.vue */
<style lang="scss">
	@import "uview-ui/index.scss";

	.tabbar_icon {
		width: 25rpx !important;
		height: 25rpx !important;
	}
</style>