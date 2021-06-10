<template>
	<view class="container">
			<view class="else">
				<view class="item" v-for="(item,index) in list" :key="index">
					<view class="date">测量时间：{{ item.createTime }}</view>
					<view class="center">
						<view class="left">
							<view class="des" style="margin-top: 30rpx;">当日体重：{{ item.weight }}kg</view>
						</view>
					</view>
					<view class="last">
						<!-- <view class="intro">
							<view>测量详细数据</view>
							<view v-if="item.weight">展开</view>
							<view v-else>收起</view>
						</view> -->
						<view class="details">
							<view >体脂率：{{ item.ratioOfFat?item.ratioOfFat:'--' }}% <text >(参考建议：{{ calc('ratioOfFat',item.ratioOfFat,item.ratioOfFatRange) }})</text></view>
							<view >BMI：{{ item.bmi?item.bmi:'--' }}  <text >(参考建议：{{ calc('bmi',item.bmi,item.bmiRange) }})</text></view>
							<view >BMR：{{ item.bmr?item.bmr:'--' }}  <text >(参考建议：{{ calc('bmr',item.bmr,item.bmrRange) }})</text></view>
							<view >体型：{{ item.bodyShape?item.bodyShape:'--' }} <text >(参考建议：{{ calc('bodyShape',item.bodyShape,'other') }})</text></view>
							<view >肌肉率：{{ item.ratioOfMuscle?item.ratioOfMuscle:'--' }}% <text >(参考建议：{{ calc('ratioOfMuscle',item.ratioOfMuscle,item.ratioOfMuscleRange) }})</text></view>
							<view >蛋白质率：{{ item.ratioOfProtein?item.ratioOfProtein:'--' }}%  <text >(参考建议：{{ calc('ratioOfProtein',item.ratioOfProtein,item.ratioOfProteinRange) }})</text></view>
							<view >骨骼肌率：{{ item.ratioOfSkeletalMuscle?item.ratioOfSkeletalMuscle:'--' }} % <text >(参考建议：{{ calc('ratioOfSkeletalMuscle',item.ratioOfSkeletalMuscle,item.ratioOfSkeletalMuscleRange) }})</text></view>
							<view >皮下脂肪：{{ item.ratioOfSubcutaneousFat?item.ratioOfSubcutaneousFat:'--' }} <text >(参考建议：{{ calc('ratioOfSubcutaneousFat',item.ratioOfSubcutaneousFat,item.ratioOfSubcutaneousFatRange) }})</text></view>
							<view >内脏脂肪：{{ item.levelOfVisceralFat?item.levelOfVisceralFat:'--' }} <text >(参考建议：{{ calc('levelOfVisceralFat',item.levelOfVisceralFat,item.levelOfVisceralFatRange) }})</text></view>
							<view >水份率：{{ item.ratioOfWater?item.ratioOfWater:'--' }} <text >(参考建议：{{ calc('ratioOfWater',item.ratioOfWater,item.ratioOfWaterRange) }})</text></view>
							<view >骨量：{{ item.weightOfBone?item.weightOfBone:'--' }} <text >(参考建议：{{ calc('weightOfBone',item.weightOfBone,item.weightOfBoneRange) }})</text></view>
							<view >脂肪重量：{{ item.weightOfFat?item.weightOfFat:'--' }} <text >(参考建议：{{ calc('weightOfFat',item.weightOfFat,item.weightOfFatRange) }})</text></view>
							<view >肌肉重量：{{ item.weightOfMuscle?item.weightOfMuscle:'--' }} <text>(参考建议：{{ calc('weightOfMuscle',item.weightOfMuscle,item.weightOfMuscleRange) }})</text></view>
							<view >蛋白质重量：{{ item.weightOfProtein?item.weightOfProtein:'--' }} <text >(参考建议：{{ calc('weightOfProtein',item.weightOfProtein,item.weightOfProteinRange) }})</text></view>
							<view >水分重量：{{ item.weightOfWater?item.weightOfWater:'--' }} <text >(参考建议：{{ calc('weightOfWater',item.weightOfWater,item.weightOfWaterRange) }})</text></view>
						</view>
					</view>
				</view>
			</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				extraData:[],
				theMonthNumber:'',
				currentArrInfo:[],
				currentDay:0,//当前天数
				month:0,
				year:0,
				pageNo:1,
				totalPage:'',
				list:[],
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
				}
			}
		},
		onLoad() {
			this.getinfo()
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
		onReachBottom() {
			console.log('触底')
			this.pageNo += 1
			if(this.pageNo>this.totalPage){
				uni.showToast({
					icon:'none',
					mask:true,
					title:'没有更多数据'
				})
			}else{
				this.getinfo()
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
			getinfo(){
				uni.request({
					url:this.BaseUrl + 'fat-scale/weighting-record/list?pageSize=5&pageNo='+ this.pageNo,
					
					header:{
						'Authorization':'Bearer dsadasdasdasdasdjasldjalksdjalskjdla'
					},
					success: (res) => {
						this.totalPage = res.data.data.totalPage
						this.list = res.data.data.list
					}
				})
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
