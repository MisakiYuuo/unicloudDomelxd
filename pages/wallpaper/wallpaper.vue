<template>
	<view class="layout">
		<view class="add" @click="handleAdd">
			<button size="mini" type="primary">添加</button>
		</view>
		<view class="selectelist">
			<uni-data-select collection="dome3" 
			v-model="formData.classid" 
			field="name as text,_id as value"
				where="status == true" label="name" 
				:clear="true" 
				@change="selecL"></uni-data-select>
		</view>
		<view class="list">
			<view class="item" v-for="item in listData" :key="item._id">
				<view class="left">
					<image :src="item.picurl.url" mode="aspectFill" @click="enlargedIMG(item.picurl.url)"></image>
				</view>
				<view class="right">
					<view class="desc">{{item.description}}</view>
					<view class="classname">
						<text>{{item.classid[0].name}}</text>
						<text>{{dayjs(item.createTime).format("YYYY-MM-DD HH:mm")}}</text>
					</view>
				</view>
			</view>
		</view>
	</view>



	<uni-popup ref="popup" border-radius="10px 10px 0 0" :is-mask-click="false">

		<view class="popupOut">
			<uni-file-picker v-model="formData.picurl" return-type="object">
			</uni-file-picker>
			<textarea placeholder="请输入壁纸描述" v-model="formData.description"></textarea>
			<uni-data-select 
			collection="dome3" 
			v-model="formData.classid" 
			field="name as text,_id as value"
			where="status == true" label="name" 
			:clear="true" 
			></uni-data-select>
			<view class="group">
				<button type="primary" size="mini" @click="onSubmit" :disabled="disabled">提交</button>
				<button type="default" size="mini" @click="onCancel">取消</button>
			</view>
		</view>

	</uni-popup>
</template>

<script setup>
	import {
		computed,
		ref
	} from 'vue';
	import dayjs from 'dayjs';
	const db = uniCloud.database();
	const listData = ref([]);
	const popup = ref(null);
	const formData = ref({
		description: "",
		picurl: {},
		classid: ""
	})
	// 分类筛选
	const selecL = async (e) => {
		let res = await db.collection("demo-wallpaper").where({
			classid:e
		}).get()
		console.log(res.result.data[0]);
		if(res.result.data[0]){
			listData.value = res.result.data
		}else{
			uni.showToast({
				title:"暂无数据"
			})
			getData()
		}
		
		
	}
	// 图片放大
	const enlargedIMG = (url) => {
		uni.previewImage({
			urls: [url]
		})
	}
	//判断提交按钮是否禁用
	const disabled = computed(() => {
		if (formData.value.description && formData.value.classid && Object.keys(formData.value.picurl).length >
			0) {
			return false;
		} else {
			return true
		}
	})
//数据初始化
	const getData = async () => {
		let wallTemp = db.collection("demo-wallpaper").field("description,classid,picurl,createTime").orderBy(
			"createTime desc").getTemp()
		let classTemp = db.collection("dome3").field("_id,name").getTemp()
		let res = await db.collection(wallTemp, classTemp).get();
		// console.log(res.result.data , "123");
		listData.value = res.result.data;
		// console.log(res);
		// console.log("获取数据");

	}
	// 测试数据用
	const gg = () => {
		// console.log("获取数据",listData);
		getData()
	}
	// 添加数据
	const handleAdd = () => {
		popup.value.open();
	}
// 提交数据
	const onSubmit = async () => {
		uni.showLoading({
			title: "提交中"
		})
		let res = await db.collection("demo-wallpaper").add(formData.value)
		// console.log("提交成功", res);
		uni.showToast({
			title: "提交成功"
		})
		init();
		popup.value.close();
		getData();
	}
// 取消数据
	const onCancel = () => {
		popup.value.close();
	}
// 数据初始化
	const init = () => {
		formData.value = {
			description: "",
			picurl: {},
			classid: ""
		}
	}

// 调用
	getData();
</script>

<style lang="scss" scoped>
	.layout {
		padding: 30rpx;

		.add {
			margin-bottom: 30rpx;
		}

		.list {
			.item {
				padding: 15rpx 0;
				border-top: 1px solid #eee;
				display: flex;

				.left {
					width: 200rpx;
					height: 200rpx;

					image {
						width: 100%;
						height: 100%;
						border-radius: 8rpx;
					}
				}

				.right {
					flex: 1;
					padding-left: 30rpx;
					display: flex;
					flex-direction: column;
					justify-content: space-between;

					.desc {
						font-size: 30rpx;
						line-height: 1.6em;
					}

					.classname {
						font-size: 28rpx;
						color: #999;
						padding-top: 20rpx;
						display: flex;
						align-items: center;
						justify-content: space-between;
					}
				}
			}

			.item:last-child {
				border-bottom: 1px solid #eee;
			}
		}
	}

	.popupOut {
		width: 660rpx;
		background: #fff;
		border-radius: 10rpx;
		min-height: 400rpx;
		padding: 30rpx;
		box-sizing: border-box;

		textarea {
			border: 1px solid #efefef;
			padding: 10rpx;
			width: 100%;
			height: 150rpx;
			box-sizing: border-box;
			margin: 30rpx 0;
		}

		.group {
			padding-top: 60rpx;
			padding-bottom: 30rpx;
			gap: 30rpx;
			display: flex;

			button {
				width: 100%;
			}
		}
	}
</style>