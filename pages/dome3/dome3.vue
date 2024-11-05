<template>
	<view class="layout">
		<view class="picbox">
			<view class="box add" @click="chooseUrl" v-if="tempFilePaths.length == 0">
				+
			</view>
			<view class="box" v-for="item,index in tempFilePaths" :key="index">
				<image :src="item.temp" mode="aspectFill" ></image>
				<view class="closepic" @click="closepic">
					x
				</view>
				<view class="loadpic" v-show="item.progress > 1">
					{{item.progress}}%
				</view>
			</view>
		</view>
	</view>
	<view class="urlshow" v-for="item,index in tempFilePaths" :key="index">
		<image :src="item.url" mode="aspectFill"></image>
			
		
		
		<view class="urlrmove" @click="urlrmove(item.fileID)">
			x
		</view>
	</view>
	
</template>

<script setup>
	import {
		ref
	} from 'vue';
	import dayjs from 'dayjs';
	// 调用云对象
	const dcloudDome3 = uniCloud.importObject("dcloudDome3");
	//数据定义
	let tempFilePaths = ref([]);
	// 关闭图片
	let closepic = function() {
		tempFilePaths.value = [];
	}
	// 选择图片&&上传图片
	let chooseUrl = async function() {
		let res = await uni.chooseImage({
			count: 1,
		});
		tempFilePaths.value = res.tempFilePaths.map(item => ({
			temp: item,
			staus: 0,
			progress: 0
		}));
		// console.log(tempFilePaths.value)
		// 创建存储变量名
		let cloudName = dayjs().format('YYYYMMDD') + "/"+ Date.now() + Math.floor(Math.random() * 1000) + '.png';
		// console.log(cloudName)
		// 上传到云存储
		let res2 = await uniCloud.uploadFile({
			filePath:tempFilePaths.value[0].temp,
			cloudPath:cloudName,
			cloudPathAsRealPath: true,
			
			onUploadProgress: (e) => {
				tempFilePaths.value[0].progress = Math.round(
				            (e.loaded * 100) / e.total
				          )
			},
			
		});
		// 关闭上传动画
		// tempFilePaths.value[0].staus = 0;
		console.log(res2)
		tempFilePaths.value[0].fileID = res2.fileID;
		// 获取临时文件链接
		// 转换url
	let res3 = await uniCloud.getTempFileURL({
			fileList: [res2.fileID],
		})
		tempFilePaths.value[0].url = res3.fileList[0].tempFileURL;
		console.log(tempFilePaths)
		
	}
	// 图片放大
	// const enlargedIMG = (url) => {
	// 	console.log(url);
	// 	uni.previewImage({
	// 		urls: [url.temp]
	// 	})
	// }
	// 删除云图片
	let urlrmove = function(url){
		dcloudDome3.remove([url]).then(res => {
			console.log(res)
			closepic()
		}).catch(err => {
			console.log(err)
		})
	}
</script>

<style lang="scss" scoped>
	.layout {
		padding: 30rpx;

		.picbox {
			display: grid;
			grid-template-columns: repeat(3, 1fr);
			grid-gap: 15rpx;

			.box {
				position: relative;
				border-radius: 10rpx;
				border: 1px solid #ccc;
				width: 200rpx;
				height: 200rpx;

				.closepic {
					position: absolute;
					top: 0;
					right: 0;
					width: 30rpx;
					height: 30rpx;
					background-color: #ccc;
					display: flex;
					align-items: center;
					justify-content: center;
					z-index: 100;
				}

				.loadpic {
					position: absolute;
					bottom: 0;
					left: 0;
					width: 100%;
					height: 100%;
					background-color: rgba(0, 0, 0, 0.3);
					color: #ffffff;
					display: flex;
					align-items: center;
					justify-content: center;
				}

				&.add {
					background-color: #ccc;
					display: flex;
					align-items: center;
					justify-content: center;
				}

				image {
					width: 100%;
					height: 100%;
				}
			}
		}
	}
	.urlshow{
		position: relative;
		
		image{
			position: absolute;
		}
		
		.urlrmove{
			position: absolute;
			top: 0;
			right: 0;
			width: 60rpx;
			height: 60rpx;
			background-color: #ccc;
			display: flex;
			align-items: center;
			justify-content: center;
			z-index: 100;
		}
	}
</style>