<template>
	<div class="daily-article">
		<div class="daily-article-title">{{data.title}}</div>
		<div class="daily-article-content" v-html="data.body"></div>

		<div class="daily-comments" v-show="comments.length">
			<span @click="showComments = !showComments">评论（{{comments.length}}）</span>
			<div class="daily-comment" v-for="comment in comments"  v-show="showComments">
				<div class="daily-comment-avatar">
					<img :src="comment.avatar">
				</div>
				<div class="daily-comment-content">
					<div class="daily-comment-name">{{ comment.author }}</div>
					<div class="daily-comment-time" v-time="comment.time"></div>
					<div class="daily-comment-text">{{ comment.content }}</div>
				</div>
			</div>
		</div>
	</div>
</template>
<script type="text/javascript">
	import $ from '../libs/util';
	import Time from '../directives/time';
	export default {
		directives: { Time },
		props: {
			id: {
				type: Number,
				default: 0
			}
		},
		data(){
			return {
				data: {},
				comments: [],
				showComments: false
			}
		},
		methods: {
			getArticle(){
				$.ajax.get('news/' + this.id)
					.then( res => {
						res.body = res.body
							.replace(/src="http/g, 'src="' + $.imgPath + 'http');
						res.body = res.body
							.replace(/src="https/g, 'src="' + $.imgPath + 'https');
						this.data = res;
						window.scrollTo(0, 0);
						this.getComments();
					})
                    .catch(error => console.log(error));
			},
			getComments(){
				this.comments = [];
				$.ajax.get('story/' + this.id + '/short-comments')
					.then( res => {
						this.comments = res.comments.map( comment => {
							comment.avatar = $.imgPath + comment.avatar;
							return comment;
						});
					})
                    .catch(error => console.log(error));
			}
		},
		watch: {
			id(val){
				if (val) this.getArticle();
			}
		}
	};
</script>