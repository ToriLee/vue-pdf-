<template>
    <div class="pswp" ref="pswb" tabindex="-1" role="dialog" aria-hidden="true">
        <div class="pswp__bg"></div>
        <div class="pswp__scroll-wrap">
            <div class="pswp__container">
                <div class="pswp__item"></div>
                <div class="pswp__item"></div>
                <div class="pswp__item"></div>
            </div>

            <div class="pswp__ui pswp__ui--hidden">
                <div class="pswp__top-bar">
                    <div class="pswp__counter"></div>
                    <button class="pswp__button pswp__button--close" title="Close (Esc)"></button>
                    <button class="pswp__button pswp__button--share" title="Share"></button>
                    <button class="pswp__button pswp__button--fs" title="Toggle fullscreen"></button>
                    <button class="pswp__button pswp__button--zoom" title="Zoom in/out"></button>
                    <div class="pswp__preloader">
                        <div class="pswp__preloader__icn">
                        <div class="pswp__preloader__cut">
                            <div class="pswp__preloader__donut"></div>
                        </div>
                        </div>
                    </div>
                </div>

                <div class="pswp__share-modal pswp__share-modal--hidden pswp__single-tap">
                    <div class="pswp__share-tooltip"></div> 
                </div>
                <button class="pswp__button pswp__button--arrow--left" title="Previous (arrow left)">
                </button>
                <button class="pswp__button pswp__button--arrow--right" title="Next (arrow right)">
                </button>

                <div class="pswp__caption">
                    <div class="pswp__caption__center"></div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import PhotoSwipe from 'photoswipe/dist/photoswipe'
import UI from 'photoswipe/dist/photoswipe-ui-default'
import 'photoswipe/dist/photoswipe.css'
import 'photoswipe/dist/default-skin/default-skin.css'//定义图片查看器的地方
export default {
    data(){
        return {

        }
    },
    props: {
        imgsList: {
            type: Array,
            default: ()=>[]
        }
    },
    mounted(){
        //this.getInit()
        
    },
    methods: {
        getInit(){
            var pswpElement = document.querySelectorAll('.pswp')[0];
            var items = [
                {
                    src: 'https://placekitten.com/600/400',
                    w: 600,
                    h: 400
                },
                {
                    src: 'https://placekitten.com/1200/900',
                    w: 1200,
                    h: 900
                }
            ];
            var options = {
                index: 0 // start at first slide
            };
            var gallery = new PhotoSwipe( pswpElement, UI, items, options);
            gallery.init();
        },
        imagePreview (currentIndex, imageArray) {
            var items = []
            imageArray.map((item, index) => {
                items.push({
                src: item,
                w: 1200,
                h: 2000
                })
            })

            var options = {
                index: currentIndex
            }
            var pswpElement = this.$refs.pswb
            var gallery = new PhotoSwipe(pswpElement, UI, items, options)
            gallery.init()
        }
        
    }
}
</script>
<style scoped>

</style>
