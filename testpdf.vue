<template>
    <div>
        <div style="text-align:center">测试pdf预览</div>
        <div class="pdf">
            <PDFViewer 
            :src="pdfSrc"
            v-for="page in pages"
            :key="page"
            :page="page"
            ></PDFViewer>
        </div>
    </div>

</template>
<script>
import PDFViewer from 'vue-pdf'
import faceConfig from '@/faceConfig'
import HttpCommon from '../../../../common/HttpCommon'
export default {
    components: { PDFViewer },
    data(){
        return {
           pdfSrc:'',
           pages:1,
        }
    },
    mounted(){
        //this.getPdfUrl(2,477)
        this.getPdfInit(2,502).then((res)=>{
            console.log(res)
            this.pdfSrc = res
            res.promise.then((page)=>{
                this.pages = page.numPages
            })
        })
    },
    methods:{
        getPdfUrl(type,id){
            let vm = this
            let path = faceConfig.basePath;
            let baseurl = `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${type}&missiveregisterid=${id}`;
            HttpCommon.axiosRequestRes({
                url: baseurl,
                responseType: 'blob',
                'Content-Type': 'application/pdf'
            }).then(res => {
                let content = res.data;
                let url = null;
                if (window.createObjectURL != undefined) { // basic
                url = window.createObjectURL(content);
                } else if (window.webkitURL != undefined) { // webkit or chrome
                try {
                    url = window.webkitURL.createObjectURL(content);
                } catch (error) {

                }
                } else if (window.URL != undefined) { // mozilla(firefox)
                try {
                    url = window.URL.createObjectURL(content);
                } catch (error) {

                }
                }
                this.pdfSrc=url
            })
        },
        async getPdfInit(type,id){
            let vm = this
            let path = faceConfig.basePath;
            let baseurl = `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${type}&missiveregisterid=${id}`;
            return HttpCommon.axiosRequestRes({
                url: baseurl,
                responseType: 'blob',
                'Content-Type': 'application/pdf'
            }).then(res => {
                let content = res.data;
                let url = null;
                //将文件流通过window.URL.createObjectURL方法转为本地地址
                if (window.createObjectURL != undefined) { // basic
                url = window.createObjectURL(content);
                } else if (window.webkitURL != undefined) { // webkit or chrome
                try {
                    url = window.webkitURL.createObjectURL(content);
                } catch (error) {

                }
                } else if (window.URL != undefined) { // mozilla(firefox)
                try {
                    url = window.URL.createObjectURL(content);
                } catch (error) {

                }
                }
                //当转成本地地址后，利用PDFViewer的方法来获取pdf的页数等信息
                let loadingTask = PDFViewer.createLoadingTask(url)
                return loadingTask
            })
        },
      // 将返回的流数据转换为url
    //   getObjectURL(file) {
    //     let url = null;
    //     if (window.createObjectURL != undefined) { // basic
    //       url = window.createObjectURL(file);
    //     } else if (window.webkitURL != undefined) { // webkit or chrome
    //       try {
    //         url = window.webkitURL.createObjectURL(file);
    //       } catch (error) {

    //       }
    //     } else if (window.URL != undefined) { // mozilla(firefox)
    //       try {
    //         url = window.URL.createObjectURL(file);
    //       } catch (error) {

    //       }
    //     }
    //     return url;
    //   },
    }
}
</script>
<style scoped>

</style>
