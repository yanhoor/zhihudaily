<template>
    <div class="daily">
        <div class="daily-menu">
            <div class="daily-menu-item"
                 @click="handleToRecommend"
                 :class="{on: type === 'recommend'}"><Icon type="clock" size="24"></Icon>每日推荐</div>
            <div class="daily-menu-item"
                 @click="handleToHotTheme"
                 :class="{on: type === 'hot'}"><Icon type="flame" size="24"></Icon>热门内容</div>
            <div class="daily-menu-item"
                 :class="{on: type === 'daily'}"
                 @click="showThemes = !showThemes"><Icon type="ios-book" size="24"></Icon>主题日报</div>
            <ul v-show="showThemes">
                <li v-for="item in themes">
                    <a :class="{on: item.id === themeId && type === 'daily'}"
                        @click="handleToTheme(item.id)">{{item.name}}</a>
                </li>
            </ul>
        </div>
        <div class="daily-list" ref="list">
            <template v-if="type === 'recommend'">
                <div v-for="list in recommendList">
                    <div class="daily-date">{{formatDay(list.date)}}</div>
                    <Item 
                        v-for="item in list.stories"
                        :data="item"
                        :key="item.id"
                        @click.native="handleClick(item.id)"></Item>
                </div>
            </template>
            <template v-if="type === 'daily'">
                <Item
                    v-for="item in list"
                    :data="item"
                    :key="item.id"
                    @click.native="handleClick(item.id)"></Item>
            </template>
            <template v-if="type === 'hot'">
                <Item
                    v-for="item in hotList"
                    :data="item"
                    :key="item.news_id"
                    @click.native="handleClick(item.news_id)"></Item>
            </template>
        </div>
        <daily-article :id="articleId"></daily-article>
    </div>
</template>
<script>
    import $ from './libs/util';
    import Item from './components/item.vue';
    import dailyArticle from './components/daily-article.vue';
    export default{
        components: { Item, dailyArticle },
        data(){
            return {
                themes: [],
                showThemes: false,
                type: 'recommend',
                recommendList: [],  //每日推荐文章列表
                dailyTime: $.getTodayTime(),
                isLoading: false,
                themeId: 0,
                articleId: 0,
                hotList: [],
                list: []    //主题日报某个主题文章列表
            }
        },
        methods: {
            getThemes(){
                $.ajax.get('themes')
                    .then(res => {
                        this.themes = res.others;
                    })
                    .catch(error => console.log(error));
            },
            getHotThemes(){
                $.ajax.get('news/hot')
                    .then(res => {
                        this.hotList = res.recent;
                    })
                    .catch(error => console.log(error));
            },
            handleToHotTheme(){
                this.type = 'hot';
                this.hotList = []
                this.getHotThemes();
            },
            handleToRecommend(){
                this.type = 'recommend';
                this.recommendList = [];
                this.dailyTime = $.getTodayTime();
                this.getRecommendList();
            },
            getRecommendList(){
                this.$Message.loading({
                    content: '加载中...',
                    duration: 0
                });
                this.isLoading = true;
                const prevDay = $.prevDay(this.dailyTime + 86400000);
                $.ajax.get('news/before/' + prevDay)
                    .then(res => {
                        this.recommendList.push(res);
                        this.$Message.destroy();
                        this.isLoading = false;
                    })
                    .catch(error => console.log(error));
            },
            handleToTheme(id){
                this.type = 'daily';
                this.themeId = id;
                this.list = [];
                $.ajax.get('theme/' + id)
                    .then(res => {
                        this.list = res.stories.filter(item => item.type !== 1);
                    })
                    .catch(error => console.log(error));
            },
            handleClick(id){
                this.articleId = id;
            },
            formatDay(date){
                let month = date.substr(4, 2);
                let day = date.substr(6, 2);
                if (month.substr(0, 1) === '0') month = month.substr(1, 1);
                if (day.substr(0, 1) === '0') day = day.substr(1, 1);
                return `${month} 月 ${day} 日`;
            }
        },
        mounted(){
            this.getThemes();
            this.getRecommendList();
            this.getHotThemes();
            const $list = this.$refs.list;
            $list.addEventListener('scroll', () => {
                if (this.type === 'daily' || this.isLoading) return;
                if (
                        $list.scrollTop
                        + document.body.clientHeight
                        >= $list.scrollHeight
                    ) {
                        this.dailyTime -= 86400000;
                        this.getRecommendList();
                }
            });
        }
    }
</script>