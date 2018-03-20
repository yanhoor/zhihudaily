<template>
	<a class="daily-item">
		<div class="daily-img" v-if="hasImg">
			<img :src="imgSrc">
		</div>
		<div
			class="daily-title"
			:class="{noImg: !hasImg}">{{data.title}}</div>
	</a>
</template>
<script>
	import $ from '../libs/util';
	export default {
		props: {
			data: {
				type: Object
			}
		},
		data(){
			return {
				imgPath: $.imgPath,
				imgSrc: '',
			}
		},
		computed: {
			hasImg(){
				let hasImg = false;
				this.data.images
				? this.imgSrc = this.imgPath + this.data.images[0]
				: this.imgSrc = this.imgPath + this.data.thumbnail;

				hasImg = this.data.images || this.data.thumbnail;
				return hasImg;
			}
		}
	};
</script>