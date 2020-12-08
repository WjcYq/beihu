<template>
	<view class="index">
		<u-row gutter="0">
			<u-col span="8">
				<view class="map-header">
					<u-image width="70rpx" height="80rpx" src="../../static/common/img/u57.png"></u-image>
					<view class="map-header-title span">北湖分局移动作战指挥平台</view>
					<view class="map-header-user">
						赵某某
						<u-image width="70rpx" height="80rpx" style="margin:0 40rpx" src="../../static/common/img/u73.png"></u-image>
					</view>
				</view>
			</u-col>
			<u-col span="4">
				<u-tabs bg-color="rgba(2, 167, 240, 1)" height="100" :list="list" :is-scroll="false" :current="current" font-size="44"
				 inactive-color="#fff" active-color="#fff" :bar-style="barStyle" @change="changeTabIndex"></u-tabs>
			</u-col>
		</u-row>
		<view class="map" :style="{height:mapHeight+'px'}">
			<map-talks :center="center" :warningData="warningData" :RTAShow="layer.RTAShow"></map-talks>
		</view>
		<u-row gutter="0" align="top" class="list">
			<u-col span="1" offset="7">
				<view class="button" :class="{'active':layer.videoShow}" @click="mapAddLayer('videoShow')">视频监控</view>
				<view class="button" :class="{'active':layer.northShow}" @click="mapAddLayer('northShow')">北斗警车</view>
				<view class="button" :class="{'active':layer.fourGShow}" @click="mapAddLayer('fourGShow')">4G警车</view>
				<view class="button" :class="{'active':layer.RTAShow}" @click="mapAddLayer('RTAShow')">实时警情</view>
			</u-col>
			<u-col span="4">
				<view class="dataList" :style="{height:mapHeight+'px'}">
					<view class="dataInfo" v-for="(data,index) in warningData" :key="index" v-if="current===0">
						<view>{{index+1+'、'}}{{data.aAddress}}</view>
						<view>报警人：{{data.aPersonName}}</view>
						<view>报警电话：{{data.aPersonPhone}}</view>
						<view>报警时间：{{data.aDate}}</view>
						<view>出警单位：{{data.deptName}}</view>
						<view>{{data.remark}}</view>
						<u-image class="icon" width="72rpx" height="72rpx" src="../../static/common/img/u75.png"></u-image>
						<u-image class="line" width="100%" height="4rpx" src="../../static/common/img/u25.svg"></u-image>
					</view>
					<view class="dataInfo" v-for="(data,index) in instructData" :key="index" v-if="current===1">
						<view>{{index+1+'、'}}{{data.title}}</view>
						<view>发送人：{{data.createUserName}}</view>
						<view>发送时间：{{data.instructionsdate}}</view>
						<view>关联警情：{{data.warningAddress}}(报警地址)</view>
						<view>指令内容：</view>
						<view>{{data.content}}</view>
						<u-row gutter="40">
							<u-col span="4" v-for="img in data.imageList" :key="img.id">
								<u-image width="100%" height="400rpx" :src="img.path" mode="heightFix"></u-image>
							</u-col>
							<u-col span="4" v-for="video in data.videoList" :key="video.id">
								<video style="height: 400rpx;width:100%" :controls="false" :enable-progress-gesture="false" :src="video.path">
								</video>
							</u-col>
						</u-row>
						<u-image class="icon" width="72rpx" height="72rpx" src="../../static/common/img/u142.png"></u-image>
						<u-image class="line" width="100%" height="4rpx" src="../../static/common/img/u25.svg"></u-image>
					</view>
				</view>
			</u-col>
		</u-row>

	</view>
</template>

<script>
	import mapTalks from './map.vue';
	export default {
		components: {
			mapTalks
		},
		data() {
			return {
				webviewStyles: {
					progress: {
						color: '#FF3333'
					}
				},
				mapHeight: 0,
				list: [{
					name: '警情',
					count: 0,
				}, {
					name: '指令',
					count: 0,
				}],
				current: 0,
				center: [],
				layer: {
					videoShow: false,
					northShow: false,
					fourGShow: false,
					RTAShow: false,
				},
				RTAlayer: null,
				infoWindowList: [],
				warningData: [],
				instructData: [],
			}
		},
		computed: {
			barStyle() {
				let barStyle = {
					width: '48%',
					transform: 'translate(2%, -100%)',
					backgroundImage: 'url(../../static/common/img/u67.svg)',
					backgroundRepeat: 'no-repeat',
					backgroundSize: 'cover',
					backgroundPosition: 'center',
					height: '8rpx',
					backgroundColor: 'transparent',
				}
				if (this.current === 1) {
					barStyle.transform = 'translate(106%, -100%)';
				}
				return barStyle;
			}
		},
		methods: {
			mapAddLayer(layer) {
				this.layer[layer] = !this.layer[layer];
			},
			changeTabIndex(index) {
				this.current = index;
			},
			getWarningData() {
				let params = {
					deptCode: "370899440501"
				}
				this.$axios.get('/rest/alarm/getTodayAlarmList', {
					params
				}).then(res => {
					if (res.code == 1) {
						this.warningData = res.data;
					}
				})
			},
			getInstructData() {
				let params = {
					userId: "806"
				}
				this.$axios.get('/rest/alarm/getInstructions', {
					params
				}).then(res => {
					if (res.code == 1) {
						this.instructData = res.data;
					}
				})
			}
		},
		mounted() {
			uni.getLocation({
				type: 'wgs84',
				success: (loaction) => {
					this.center = [loaction.longitude, loaction.latitude];
					uni.getSystemInfo({
						success: (res) => {
							this.mapHeight = res.windowHeight - 50;
							this.getWarningData();
							this.getInstructData();
						}
					});
				}
			});
		},
	}
</script>

<style lang="scss" scoped>
	.index {
		.map-header {
			font-family: 'Arial Normal', 'Arial';
			font-weight: 400;
			font-style: normal;
			background-color: rgba(2, 167, 240, 1);
			display: flex;
			align-items: center;
			justify-content: center;
			position: relative;
			height: 100rpx;

			&-user {
				font-size: 32rpx;
				color: #F2F2F2;
				position: absolute;
				right: 0;
				display: flex;
				align-items: flex-end;
			}

			.span {
				font-size: 52rpx;
				color: #FFFFFF;
			}
		}

		/deep/.u-tab-item {
			border-left: 6rpx solid #fff;
		}

		.map {
			position: absolute;
			z-index: -1;
			width: 100%;
		}

		.list {
			position: absolute;
			z-index: 9999999;
			width: 100%;
		}

		.button {
			width: 200rpx;
			height: 70rpx;
			margin: 30rpx 20rpx;
			background-color: rgba(1, 84, 120, 1);
			border-radius: 10rpx;
			display: flex;
			justify-content: center;
			align-items: center;
			font-size: 32rpx;
			color: #FFFFFF;
			cursor: pointer;
		}

		.button:hover {
			background-color: rgba(2, 167, 240, 1);
		}

		.active {
			background-color: rgba(2, 167, 240, 1);
		}

		.dataList {
			width: 100%;
			background-image: url(../../static/common/img/u18.svg);
			background-repeat: no-repeat;
			background-size: cover;
			display: flex;
			flex-flow: column;

			.dataInfo {
				margin: 30rpx 20rpx 0;
				font-size: 30rpx;
				color: #FFFFFF;
				font-weight: 400;
				display: flex;
				flex-flow: column;
				position: relative;
				border-bottom: 1px;

				.icon {
					position: absolute;
					top: 10rpx;
					right: 20rpx;
				}
			}
		}
	}
</style>
