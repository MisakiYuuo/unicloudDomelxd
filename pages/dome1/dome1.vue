<template>
	<view class="layout">
<view v-for="item in userLsit" :key="item._di">
	{{item.username}}
</view>
	</view>
</template>

<script setup>
	import {ref} from 'vue';
	
	const userLsit = ref([]);
	const db = uniCloud.database();
	const collection = db.collection('user-dome2');
	function getList(){
		collection.get().then(res=>{
			userLsit.value = res.result.data;
			console.log(userLsit.value);
			// console.log(res);
		})
	}
	getList();
	// const res = await collection.get();
	// console.log(res);
</script>

<style lang="scss" scoped>

</style>
