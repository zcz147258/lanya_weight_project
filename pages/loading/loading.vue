<template>
	<view class="container">
			<view class="else" v-if="data">
				<view class="item" >
					<!-- <view class="date">测量时间：{{ '2020-2-9 18:25:00' }}</view> -->
					<view class="center">
						<view class="left">
							<view class="des" style="margin-top: 30rpx;">当日体重：{{ data.weight }}kg</view>
						</view>
					</view>
					<view class="last">
						<view class="details">
							<view >体脂率：{{ data.ratioOfFat?data.ratioOfFat:'--' }}% <text v-if="data.ratioOfFat">(参考建议：{{ calc('ratioOfFat',data.ratioOfFat,data.ratioOfFatRange) }})</text></view>
							<view >BMI：{{ data.bmi?data.bmi:'--' }}  <text v-if="data.bmi">(参考建议：{{ calc('bmi',data.bmi,data.bmiRange) }})</text></view>
							<view >BMR：{{ data.bmr?data.bmr:'--' }}  <text v-if="data.bmr">(参考建议：{{ calc('bmr',data.bmr,data.bmrRange) }})</text></view>
							<view >体型：{{ data.bodyShape?data.bodyShape:'--' }} <text v-if="data.bodyShape">(参考建议：{{ calc('bodyShape',data.bodyShape,'other') }})</text></view>
							<view >肌肉率：{{ data.ratioOfMuscle?data.ratioOfMuscle:'--' }}% <text v-if="data.ratioOfMuscle">(参考建议：{{ calc('ratioOfMuscle',data.ratioOfMuscle,data.ratioOfMuscleRange) }})</text></view>
							<view >蛋白质率：{{ data.ratioOfProtein?data.ratioOfProtein:'--' }} <text v-if="data.ratioOfProtein">(参考建议：{{ calc('ratioOfProtein',data.ratioOfProtein,data.ratioOfProteinRange) }})</text></view>
							<view >骨骼肌率：{{ data.ratioOfSkeletalMuscle?data.ratioOfSkeletalMuscle:'--' }} <text v-if="data.ratioOfSkeletalMuscle">(参考建议：{{ calc('ratioOfSkeletalMuscle',data.ratioOfSkeletalMuscle,data.ratioOfSkeletalMuscleRange) }})</text></view>
							<view >皮下脂肪：{{ data.ratioOfSubcutaneousFat?data.ratioOfSubcutaneousFat:'--' }} <text v-if="data.ratioOfSubcutaneousFat">(参考建议：{{ calc('ratioOfSubcutaneousFat',data.ratioOfSubcutaneousFat,data.ratioOfSubcutaneousFatRange) }})</text></view>
							<view >内脏脂肪：{{ data.levelOfVisceralFat?data.levelOfVisceralFat:'--' }} <text v-if="data.levelOfVisceralFat">(参考建议：{{ calc('levelOfVisceralFat',data.levelOfVisceralFat,data.levelOfVisceralFatRange) }})</text></view>
							<view >水份率：{{ data.ratioOfWater?data.ratioOfWater:'--' }} <text v-if="data.ratioOfWater">(参考建议：{{ calc('ratioOfWater',data.ratioOfWater,data.ratioOfWaterRange) }})</text></view>
							<view >骨量：{{ data.weightOfBone?data.weightOfBone:'--' }} <text v-if="data.weightOfBone">(参考建议：{{ calc('weightOfBone',data.weightOfBone,data.weightOfBoneRange) }})</text></view>
							<view >脂肪重量：{{ data.weightOfFat?data.weightOfFat:'--' }} <text v-if="data.weightOfFat">(参考建议：{{ calc('weightOfFat',data.weightOfFat,data.weightOfFatRange) }})</text></view>
							<view >肌肉重量：{{ data.weightOfMuscle?data.weightOfMuscle:'--' }} <text v-if="data.weightOfMuscle">(参考建议：{{ calc('weightOfMuscle',data.weightOfMuscle,data.weightOfMuscleRange) }})</text></view>
							<view >蛋白质重量：{{ data.weightOfProtein?data.weightOfProtein:'--' }} <text v-if="data.weightOfProtein">(参考建议：{{ calc('weightOfProtein',data.weightOfProtein,data.weightOfProteinRange) }})</text></view>
							<view >水分重量：{{ data.weightOfWater?data.weightOfWater:'--' }} <text v-if="data.weightOfWater">(参考建议：{{ calc('weightOfWater',data.weightOfWater,data.weightOfWaterRange) }})</text></view>
						</view>
					</view>
				</view>
			</view>
			<view class="fixed" v-if="data">
				<view @click="Submit">保存信息</view>
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
			console.log('蓝牙不可用')
			uni.showToast({
				icon:'none',
				mask:true,
				title:'请打开蓝牙'
			})
			this._updateLog('蓝牙不可用');
	    }
	});
	const deviceTypeList = [{
	    protocol: 0x03,
	    deviceType: 0x03,
	    subType: 0x06,
	    vendor: 0x0f
	}];
	var vendorKey='16TZB2IYM5SL39H3'
	const waitTime = 120; //扫描等待时长，单位秒
	export default {
		data() {
			return {
				extraData:[],
				theMonthNumber:'',
				currentArrInfo:[],
				currentDay:0,//当前天数
				month:0,
				year:0,
				data:null,
				des:{
					bmi:[ '偏瘦','健康','偏胖','肥胖'],
					bmr: ['不足','标准'],
					bodyShape:['隐性胖','偏胖型','运动型偏胖','缺乏锻炼型','标准型', '标准运动型','偏瘦型','偏瘦运动型','运动健美型'],
					levelOfVisceralFat:['正常','偏高','严重偏高'],//内脏脂肪等级
					ratioOfFat:['偏瘦','健康','警戒','轻度肥胖','重度肥胖'],//体脂率区间
					ratioOfMuscle:['不足','标准','优'],
					ratioOfProtein:['不足','标准','优'],//蛋白质率区间
					ratioOfSkeletalMuscle:['不足','标准','优'],//肌肉率区间
					ratioOfSubcutaneousFat:['偏低','正常','偏高'],//皮下脂肪率
					weight:['不足','过轻','标准','过重','肥胖'],//体重率区间
					stateOfNutrition:['严重营养不良','中度营养不良','营养不良','标准','营养过剩','营养严重过剩'],//营养状态
					weightOfBone:['偏低','正常','优'],//骨重区间
					weightOfFat:['偏瘦','健康','警戒','轻度肥胖','重度肥胖'],
					weightOfMuscle:['不足','标准','优'],
					weightOfProtein:['不足','标准','优'],//蛋白质质量区间
					weightOfWater:['缺失','标准','优'],//水分重量区间	
					ratioOfWater:['缺失','标准','优']
				},
				userDataInfo:{}//用户信息
			}
		},
		onLoad() {
			uni.showLoading({
				mask:true,
				title:'正在测量中',
			})
			wx.authorize({
				  scope: 'scope.userLocation',
				  success() {
					  console.log('位置已经打开');
				  }
			 })
			 let that = this;
			this.scanWeight()
			//获取用户档案信息
			uni.request({
				url:this.BaseUrl + 'fast-scale/archives/info',
				header:{
					'Authorization':'Bearer dsadasdasdasdasdjasldjalksdjalskjdla'
				},
				success: (res) => {
					this.userDataInfo = res.data.data
				}
			})
		},
		computed:{
			calc(){
				return (key,num,arr)=>{
					if(arr == 'other' && num){
						return this.des[key][num-1]
					}
					if(arr && num){
						let index = this.CalcTheIndex(num,arr);
							return this.des[key][index]
					}else{
						return '--'
					}
				}
			}
		},
		methods: {
			CalcTheIndex(num,arr){
				let arr1 = arr;
				if(arr1){
					if(typeof arr1 == 'string'){
						arr1 = JSON.parse(arr1)
						console.log(arr1)
					}
					let index = null
					for (var i = 0; i < arr1.length; i++) {
						if((num <= arr1[i+1]) && (num >= arr1[i])){
							index = i;
							break;
						}
					}
					return index
				}
			},
			Submit(){//
				uni.request({
					url:this.BaseUrl + 'fat-scale/weighting-record',
					method:'POST',
					header:{
						'Authorization':'Bearer dsadasdasdasdasdjasldjalksdjalskjdla'
					},
					data:this.data,
					success: (res) => {
						console.log(res)
						if(res.data.data){
							uni.showToast({
								title:'上传数据成功',
							})
							setTimeout(function(){
								uni.navigateBack({
									delta:1
								})
							})
						}else{
							uni.showToast({
								title:res.data.message,
								icon:'none',
								mask:true
							})
						}
					}
				})
			},
			scanWeight(){
				let that = this
				myPlugin.startBleScan({
					 onDataAvailable: (res) => { // 收到体重数据，code为200时此函数要返回用户信息
					    console.log(res)
					    if (res.code === 200) {
					        console.log('收到动态体重数据', res);
							this.txt = JSON.stringify(res)
							let age,height,gender
							if(this.userDataInfo.genderDesc == "男"){
								gender = 0
							}else{
								gender = 1
							}
							age = new Date().getFullYear() - Number(this.userDataInfo.birthday.substring(0,4))
							console.log(Number(this.userDataInfo.birthday.substring(0,4)))
							height = this.userDataInfo.height
							 return {
								age,
								height,
								gender
							}; // gender 0 男性 1 女性 2男运动员 3女运动员
					        // this._updateLog('收到动态体重数据' + res.details.weight);
					    } else {
							that.data = res.details
							uni.hideLoading()
					        console.log('扫描完成 收到了数据', res);
							console.log('体脂率为:'+res.details.ratioOfFat)
							console.log('肌肉率为:'+res.details.ratioOfMuscle)
							console.log('蛋白率为:'+res.details.ratioOfProtein)
							console.log('骨骼肌率为:'+res.details.ratioOfProtein)
							console.log('骨骼肌率为:'+res.details.weight)
					        // this._updateLog('扫描完成');
					    }
					},
					onScaleFail: (reason) => { // 称重失败回调
						console.log('扫描失败', reason);
						// this._updateLog('扫描失败');
					},
					onConnSucess: () => {
						 // this._updateLog('\n 连接成功');
						 console.log('连接成功')
						 /**
						 * 如果您的称支持同步单位，这里返回要同步的单位；
						 * 如果不支持，请忽略。
						 * 部分称必须写
						 * kg：0；lb：1；jin：2；st-lb：3；st：4
						 */
						 return {'vtUnit':1};
					 }
				}, deviceTypeList, vendorKey, waitTime)
			}
		}
	}
</script>

<style>
	page{
		background-color: #f4f2f2;
	}
</style>
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
	.container{
		padding: 30rpx;
		.else{
			.item{
				color: #515151;
				margin-top: 25rpx;
				padding: 35rpx;
				background-color: white;
				border-radius: 20rpx;
				.date{
					padding-bottom: 35rpx;
					font-weight: bolder;
					border-bottom: 1rpx solid #efefef;
				}
				.center{
					font-size: 28rpx;
					display: flex;
					justify-content: space-between;
					align-items: center;
					border-bottom: 1rpx solid #efefef;
					.left{
						padding: 30rpx 0rpx 30rpx 50rpx;
						.des{
							
						}
						.des::before{
							content: "";
							display: inline-block;
							width: 12rpx;
							height: 12rpx;
							margin-right: 10rpx;
							border-radius: 50%;
							background-color: #FEB835;
							margin-bottom: 5rpx;
						}
					}
					.right{
						display: flex;
						align-items: center;
						.tobuka{
							padding: 10rpx 25rpx;
							background-color: #FEB835;
							border-radius: 30rpx;
							color: white;
							font-weight: bolder;
						}
						.already{
							padding: 10rpx 25rpx;
							background-color: white;
							border-radius: 30rpx;
							border: 3rpx solid #FEB835;
							color: #FEB835;
							font-weight: bolder;
						}
					}
				}
				.last{
					font-size: 28rpx;
					margin-top: 25rpx;
					.intro{
						display: flex;
						justify-content: space-between;
						>view:not(:nth-child(1)){
							color: #FEB835;
							font-weight: bolder;
						}
					}
					.details{
						margin-top: 25rpx;
						padding-left: 50rpx;
						>view:not(:nth-child(1)){
							margin-top: 30rpx;
						}
						view::before{
							content: "";
							display: inline-block;
							width: 12rpx;
							height: 12rpx;
							margin-right: 10rpx;
							border-radius: 50%;
							background-color: #FEB835;
							margin-bottom: 5rpx;
						}
					}
				}
			}
		}
	}

</style>
