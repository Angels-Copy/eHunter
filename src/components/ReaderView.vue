<template>
<div class="reader-view">
    <!-- loading view -->
    <div class="loading-container" v-if="isloadingImgInfos">
        <span class="loading">loading...</span>
        <p class="tip">
            注意事项:<br>无<br>
        </p>
    </div>
    <!-- top bar view -->
    <top-bar class="top-bar" />
    <!-- panel view -->
    <div class="panel">
        <h4 class="location">{{ (curIndex + 1) + '/' + AlbumService.getPageCount() }}</h4>
        <img title="全屏" @click="fullscreen" class="focus icon" :src="image.fullScreen" />
    </div>
    <album-scroll-view class="scroll-mode" v-if="!isloadingImgInfos" :img-info-list="imgInfoList" :page-urls-obj="pageUrlsObj"></album-scroll-view>
</div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import AlbumScrollView from './AlbumScrollView.vue'
import TopBar from './TopBar.vue'
import ImgHtmlParser from 'src/service/parser/ImgHtmlParser.js'
import AlbumService from '../service/AlbumService'
import image from '../assets/img'
// import Logger from '../utils/Logger'

export default {
    name: 'reader-view',

    components: { AlbumScrollView, TopBar },

    data() {
        return {
            parser: new ImgHtmlParser(document.documentElement.innerHTML),
            imgInfoList: [],
            pageUrlsObj: {} // hash, for fast get page index by pagUrl
        };
    },

    async created() {
        await this.initImgInfoList();
    },

    computed: {
        ...mapGetters({
            curIndex: 'curIndex'
        }),
        isloadingImgInfos() {
            return this.imgInfoList.length === 0;
        },
        AlbumService: () => AlbumService,
        image: () => image
    },

    methods: {
        ...mapActions([]),
        async initImgInfoList() {
            this.imgInfoList = await AlbumService.getImgInfos();
            // init pageUrlsObj
            for (let i = 0; i < this.imgInfoList.length; i++) {
                this.pageUrlsObj[this.imgInfoList[i].pageUrl] = i;
            }
        },

        fullscreen() {
            // hack for crossing chrome and firefox
            const elem = document.querySelector('.vue-container');
            if (document.webkitCurrentFullScreenElement || document.mozFullScreenElement) {
                document.webkitExitFullscreen ? document.webkitExitFullscreen() : '';
                document.mozCancelFullScreen ? document.mozCancelFullScreen() : '';
            } else {
                elem.mozRequestFullScreen ? elem.mozRequestFullScreen() : '';
                elem.webkitRequestFullScreen ? elem.webkitRequestFullScreen() : '';
            }
        }
    }
}
</script>

<style lang="scss" scoped>
@import "~style/_responsive";
@import "~style/_variables";

div {
    display: flex;
}

.reader-view {
    position: relative;
    flex-direction: column;
    align-items: center;
        > .loading-container {
        position: absolute;
        flex-direction: column;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        color: $img_container_color;
        > .loading {
            display: block;
            font-size: 24px;
            font-weight: bolder;
        }
        > .tip {
            padding: 0;
            margin: 10px 0;
            font-size: 16px;
        }
    }
    > .top-bar {
        position: absolute;
        z-index: 10;
        left: 0;
        top: 0;
        width: 100%;
    }
    > .panel {
        position: absolute;
        bottom: 5px;
        right: 23px;
        z-index: 10;
        color: rgba(white, 0.2);
        display: flex;
        align-items: center;
        justify-content: center;
        > .location {
            font-size: 14px;
            display: inline-block;
            line-height: 16px;
            height: 16px;
        }
        .icon-container {
            position: relative;
            display: inline-block;
        }
        .focus.icon {
            width: 16px;
            display: inline-block;
            height: 16px;
            cursor: pointer;
            margin: 18px 0 18px 10px;
        }
    }
    > .scroll-mode {
        align-self: stretch;
    }
}
</style>