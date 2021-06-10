<template>
	<view>
		<!-- <button type="primary" style="width: 50%;margin: auto;margin-top: 30rpx;" @click="reload">重新检测</button> -->
		<view class="container">
			<view class="center">
				<view>
					<view class="first">
						我的测量
					</view>
					<view class="third">
						<view class="item" style="text-align: center;">
							<image style="width: 30rpx;height: 30rpx;" src="https://pic-sys.gbn.red/health_03-3.png"></image>
							<view class="number" style="font-size: 40rpx;color: #FFF718;">{{data.targetWeight?data.targetWeight:'--'}}kg</view>
							<view class="text" style="font-size: 26rpx;color: #FFF718;">目标体重</view>
						</view>
					</view>
					<view class="fourth">
						<view class="item" style="margin-right: 50rpx;text-align: center;">
							<image style="width: 30rpx;height: 30rpx;" src="https://pic-sys.gbn.red/health_03-3.png"></image>
							<view class="number">{{data.bmi?data.bmi:'--'}}</view>
							<view class="text">BMI</view>
						</view>
						<view class="item" style="margin-right: 50rpx;text-align: center;">
							<image style="width: 30rpx;height: 30rpx;" src="https://pic-sys.gbn.red/health_03-2.png"></image>
							<view class="number">{{data.currentWeight?data.currentWeight:'--'}}kg</view>
							<view class="text">目前体重</view>
						</view>
						<view class="item" style="text-align: center;">
							<image style="width: 30rpx;height: 30rpx;" src="https://pic-sys.gbn.red/health_03-1.png"></image>
							<view class="number">{{data.ratioOfFat?data.ratioOfFat:'--'}}%</view>
							<view class="text">体脂率</view>
						</view>
					</view>
					<view class="fifth" @click="toClockDetails">
						<view>
							测量
						</view>
					</view>
				</view>
			</view>
			<view class="fixed">
				<view @click="lookDetails">查看历史测量</view>
			</view>
		</view>
	</view>
</template>

<script>
	var myPlugin = requirePlugin('myPlugin');
	myPlugin.checkBle({
	    success: () => { // 蓝牙可用
			console.log('蓝牙可用')
			this._updateLog('蓝牙可用');
			this.setData({
				isBleAvailable: true
			})
	    },
	    fail: () => { // 蓝牙不可用
			uni.showToast({
				icon:'none',
				mask:true,
				title:'请打开蓝牙'
			})
			this._updateLog('蓝牙不可用');
	    }
	});
	
	export default {
		data() {
			return {
				data:{
					
				},
				txt:'体重'
			}
		},
		onLoad() {
			//获取最近一次测量结果
			uni.request({
				url:this.BaseUrl + 'fast-scale/archives/last-count',
				header:{
					'Authorization':'Bearer dsadasdasdasdasdjasldjalksdjalskjdla'
				},
				success: (res) => {
					console.log(res)
					this.data = res.data.data
				}
			})
			//打开蓝牙监听 
			
		},
		methods: {
			lookDetails(){
				uni.navigateTo({
					url:'../detail/detail'
				})
			},
			toClockDetails(){//去测量详情
				// uni.navigateTo({
				// 	url:'../input/input'
				// })
				//获取用户档案信息
				uni.request({
					url:this.BaseUrl + 'fast-scale/archives/info',
					header:{
						'Authorization':'Bearer dsadasdasdasdasdjasldjalksdjalskjdla'
					},
					success: (res) => {
						if(!res.data.data){
							uni.navigateTo({
								url:'../input/input'
							})
						}else{
							uni.navigateTo({
								url:'../loading/loading'
							})
						}
					}
				})
				// uni.navigateTo({
				// 	url:'../loading/loading'
				// })
			},
			scanWeight(){
				let that = this
			}
		}
	}
</script>

<style lang="less">
	.fixed{
		padding: 30rpx;
		position: fixed;
		bottom: 10rpx;
		width: 90%;
		height: 100rpx;
		z-index: 998;
		view{
			color: white;
			font-size: 35rpx;
			font-weight: 480;
			text-align: center;
			background:linear-gradient(#FFC747,#FE9105);
			height: 100rpx;
			line-height: 100rpx;
			border-radius: 60rpx;
		}
	}
	.center{
		padding: 30rpx;
		>view{
			padding: 15rpx;
			background:linear-gradient(#FFC747,#FEA119);
			border-radius: 30rpx;
			box-shadow:2px 2px 5px 3px #cacaca;
			.first{
				padding-left: 30rpx;
				font-size: 35rpx;
				font-weight: bolder;
				color: white;
				margin-bottom: 30rpx;
			
				
			}
			.second{
				display: flex;
				justify-content: center;
				margin-bottom: 45rpx;
				.text{
					font-weight: bolder;
					font-size: 35rpx;
					margin-right: 10rpx;
					line-height: 62rpx;
					height: 62rpx;
				}
				.number{
					font-size: 40rpx;
					font-weight: bolder;
					line-height: 62rpx;
					height: 62rpx;
				}
			}
			.third{
				display: flex;
				justify-content: center;
				align-items: center;
				margin-bottom: 45rpx;
				.item{
					text-align: center;
					.text{
						text-align: center;
						font-size: 35rpx;
						font-weight: 600;
					}
					.number{
						font-size: 25rpx;
						text-align: center;
					}
				}
			}
			.fourth{
				display: flex;
				justify-content: center;
				align-items: center;
				margin-bottom: 45rpx;
				.item{
					.text{
						text-align: center;
						font-size: 25rpx;
						color: white;
						font-weight: 600;
					}
					.number{
						font-size: 30rpx;
						text-align: center;
						color: white;
						font-weight: 600;
						margin-bottom: 10rpx;
					}
				}
			}
			.fifth{
				display: flex;
				justify-content: center;
				align-items: center;
				margin-bottom: 30rpx;
				>view{
					border-radius: 50%;
					color: #FE9105;
					font-size: 40rpx;
					font-weight: bolder;
					background-color: white;
					padding: 35rpx;
					width: 80rpx;
					height: 80rpx;
					text-align: center;
					line-height: 80rpx;
					box-shadow:inset 0px 0px 3px 3px #e7e7e7;
				}
				
			}
		}
	}
</style>
