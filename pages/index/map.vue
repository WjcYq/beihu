<template>
	<view>
		<web-view :src="url" @message="handleMessage"></web-view>
	</view>
</template>

<script>
	export default {
		props: ['center', 'warningData', 'RTAShow', ],
		computed: {
			url() {
				let center = JSON.stringify(this.center);
				let warningData = JSON.stringify(this.warningData);
				let RTAShow = this.RTAShow ? 1 : 0;
				return `/hybrid/html/map.html?center=${center}&warningData=${warningData}&RTAShow=${RTAShow}`;
			}
		},
		methods: {
			handleMessage(evt) {
				console.log('接收到的消息：' + JSON.stringify(evt.detail.data));
			}
		},
		onReady() {
			// #ifdef APP-PLUS
			var currentWebview = this.$scope.$getAppWebview() //此对象相当于html5plus里的plus.webview.currentWebview()。在uni-app里vue页面直接使用plus.webview.currentWebview()无效，非v3编译模式使用this.$mp.page.$getAppWebview()
			// #endif
		}
	}
</script>

<style>
</style>
