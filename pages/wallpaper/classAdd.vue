<template>
	<view class="layout">
		<view class="add">
			<input class="ipt" type="text" v-model="classname" placeholder="请输入分类名称按enter添加" @confirm="onConfirm">
		</view>
		<view class="goTo">
			<button @click="goTo">点击跳转壁纸</button>
		</view>
		<view class="goTo">
			<button @click="goToo">点击跳转上传dome</button>
		</view>
		<view class="list">
			<view class="item" v-for="item in classList" :key="item._id">
				<view class="name">{{item.name}}</view>
				<view class="status">
					<switch :checked="item.status" style="transform:scale(0.6)" @change="(e)=>switchChange(e,item._id)" />
				</view>
				<view class="remove">
					<uni-icons type="trash" size="26" @click="handleRemove(item._id)"></uni-icons>
				</view>
			</view>
		</view>
	</view>
</template>

<script setup>
	import {
		ref
	} from 'vue';
	const classname = ref("");
	const classList = ref([]);
	const db = uniCloud.database();
	const goToo = ()=>{
		uni.navigateTo({
			url:"/pages/dome3/dome3"
		})
	}
	const goTo = () => {
		uni.navigateTo({
			url: "/pages/wallpaper/wallpaper"
		})
	}

	const getClassify = async () => {
		let res = await db.collection("dome3").orderBy("createTime desc").get();
		console.log(res);
		classList.value = res.result.data;

	}

	const switchChange = async (e,id) => {
		let status = e.detail.value;
		let res = await db.collection("dome3").where(`_id =="${id}"`).update({
			status
		});
		console.log("e",e.detail);
		console.log("id",id);
		
	}


	const handleRemove = async (e) => {
		// let res = await db.collection("dome3").where(`_id =="${e}"`).remove();
		uni.showLoading({
			title: "删除ing"
		})
		uni.showModal({
			title: "提示",
			content: "确定删除吗？",
			success: async (res) => {
				console.log(res, "shiw");
				if (res.confirm) {
					let res = await db.collection("dome3").doc(e).remove();
					console.log(res);
					getClassify();
					uni.showToast({
						title: "删除成功",
						icon: "none"
					})
				} else if (res.cancel) {
					uni.showToast({
						title: "取消删除",
						icon: "none"
					})
				}
			}
		})
		// let res = await db.collection("dome3").doc(e).remove();
		// console.log(res);
		// getClassify();
		// console.log(e);
	}

	const onConfirm = async () => {
		uni.showLoading({
			title: "加载中"
		})
		if (!classname.value) {
			uni.showToast({
				title: "请输入分类名称",
				icon: "none"
			})
			return;
		}
		let res = await db.collection("dome3").add({
			name: classname.value
		})
		uni.showToast({
			title: "添加成功",
			icon: "none"
		})
		console.log(res);
		getClassify()
		classname.value = "";
	}

	getClassify();
</script>

<style lang="scss" scoped>
	.layout {
		padding: 30rpx;

		.add {
			margin-bottom: 30rpx;
			display: flex;
			gap: 20rpx;

			.ipt {
				width: 100%;
				border: 1px solid #eee;
				height: 70rpx;
				padding: 0 20rpx;
				box-sizing: border-box;
			}
		}

		.item {
			display: flex;
			padding: 10rpx 0;
			align-items: center;
			justify-content: space-between;

			.name {
				color: #007AFF;
				flex: 1;
			}

			.status {
				width: 100rpx;
			}

			.remove {
				width: 100rpx;
				display: flex;
				justify-content: flex-end;
			}
		}
	}
</style>