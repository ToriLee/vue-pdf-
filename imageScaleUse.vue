<template>
    <div class="document-box">
        <div class="document-title">
            <div class="receipt-title">{{receipt.title}}</div>
            <div class="document-desc">
                <span>部门：{{receipt.namepath}}</span>
            </div>
            <div class="document-desc">
                <span>{{receipt.dealusername}}</span>
                <span>{{receipt.dealtime}}</span>
            </div>
        </div>
        <div class="document-tabs">
            <div :class="{'tabs-select':true,'selected':selectIdex === 0}" @click="select(0)"><img class="icon_file" src="../../assets/images/xiangqing.png"/>办文</div>
            <div class="line"></div>
            <div :class="{'tabs-select':true,'selected':selectIdex === 1}" @click="select(1)"><img class="icon_file" src="../../assets/images/zhengwen.png"/>正文</div>
            <div class="line"></div>
            <div :class="{'tabs-select':true,'selected':selectIdex === 2}" @click="select(2)"><img class="icon_file" src="../../assets/images/fujian.png"/>附件</div>
            <div class="line"></div>
            <div :class="{'tabs-select':true,'selected':selectIdex === 3}" @click="select(3)"><img class="icon_file" src="../../assets/images/liucheng.png"/>流程</div>
        </div>

        <div v-show="detailsShow">
            <group-cell title="处理流程" v-if="arr.length > 0" class="document-filelist">
                <cell-item-link :arr="arr" @itemClick="itemClick" :showStatus="false" :receipt="receipt"></cell-item-link>
            </group-cell>
        </div>

        <div class="document-filelist" v-show="fileListShow">
            <div class="document-filelist-show" v-if="filelistNull">暂无数据</div>
            <div class="document-filelist-a" v-for="item in fileList" :key="item.fileId">
                <a :title="item.fileName" v-if="item.filetype==='.pdf'" @click="fnFileClick(item)">
                    <img class="folder" src="../../../flow/assets/icon/icon-file.png" />{{item.fileName}}
                </a>
                <!--<a v-else :title="item.fileName" :href="basePath+'/oaFile/download?fileid='+item.fileId">-->
                <a v-else :title="item.fileName" @click="openFile(item)">
                    <img class="folder" src="../../../flow/assets/icon/icon-file.png" />{{item.fileName}}
                </a>
            </div>
        </div>
        <div class="pdf-box">
            <div v-show="isBackToBottomNow && isBackToBottom && isBackToBottomScroll" class="backBottom" @click="backBottom">
                <van-icon name="down" />
            </div>
            <div class="preview-box" id="pdf_demo" v-show="pdfShow"></div>
            <div v-if="imgShow">
                <img v-for="(img,index) in imgUrl" :src="img" :key="index" @click="scaleImg(index,imgUrl)"  style="width: 100%;" alt="Loading..."  >
                <!-- <img :src="imgUrl"  style="width: 100%;" alt="Loading..."  v-if="imgShow"> -->
            </div>
            <div v-if="imgShow1">
                <img v-for="(img,index) in imgUrl1" :key="index" :src="img" @click="scaleImg(index,imgUrl1)"  style="width: 100%;" alt="Loading..."  >
                <!-- <img :src="imgUrl1"  style="width: 100%;" alt="Loading..."  v-if="imgShow1"> -->
            </div>
        </div>

        <div v-if="maskShow" style="position: absolute;z-index:9999;top: 0;height: 100%;width: 100%;background-color: rgba(0,0,0,0.5);">

        </div>
        <photo-swipe ref="photowb" :imgsList="imgUrl"></photo-swipe>
    </div>
</template>

<script>
    // import { Icon } from 'vant';
    import faceConfig from '@/faceConfig'
    import dataControl from '../../api/DataControl'
    import Pdfh5 from "pdfh5";
    import HttpCommon from '../../../../common/HttpCommon'
    const DataControl = dataControl.receiptRegister
    import cellItemLink from '../../components/CellItemLink'
    import GroupCell from '../../components/GroupCell';
    import PhotoSwipe from '../receipt/photoswipe'; //适用移动端的图片查看器，此文档是调用
    export default {
        components: {
            Icon: 'Icon',
            cellItemLink,
            GroupCell,
            PhotoSwipe
        },
        props: {
            receipt: {
                type: Object,
                default: {}
            },
            historicals: {
                type: Array,
                default: () => []
            },
            arr: {
                type: Array,
                default: () => []
            },
            commonLanShowInit: { //常用语在初始页面是否展示
                type: Boolean,
                default: true
            },
            processShowInit: { //处理方式流程在初始页面是否展示
                type: Boolean,
                default: true
            },
            isBackToBottom: {
                type: Boolean,
                default: false
            }
        },
        data () {
            return {
                pdfh5: null,
                dataControl: this.$dataControl.article,
                selectIdex: 0,
                src: "", // pdf文件地址
                viewerUrl: '/static/pdf.js/web/viewer.html',
                // 要访问的pdf的路径
                fileUrl: '',
                fileUrl1: '',
                fileListShow: false, //文件列表展示
                fileList: [], //正文、附件文件列表
                pdfShow:false,
                imgShow: false,
                imgShow1: false,
                detailsShow: false, //详情tab显示
                basePath: faceConfig.basePath,
                filelistNull:true, //文件列表是否为空
                isBackToBottomNow:true, //切换tab时是否显示该按钮
                isBackToBottomScroll:true, //滚动是否显示该按钮
                imgUrl: [], //处理笺url
                imgUrl1: [], //阅文笺url
                maskShow:true, //蒙层
                pageCount: '',
            }
        },
        mounted () {
            console.log("this.receipt55",this.receipt)
            let vm = this
            setTimeout(() => {
                // if(vm.receipt.missivedealtype!==undefined&&vm.receipt.missiveregisterid!=undefined){
                //     vm.getPagesNumber(vm.receipt.missivedealtype,vm.receipt.missiveregisterid)
                // }
                vm.select(0)
            },1000);
            window.addEventListener('scroll', function(){
                var scr = document.documentElement.scrollTop || document.body.scrollTop; // 向上滚动的那一部分高度
                var clientHeight = document.documentElement.clientHeight; // 屏幕高度也就是当前设备静态下你所看到的视觉高度
                var scrHeight = document.documentElement.scrollHeight || document.body.scrollHeight; // 整个网页的实际高度，兼容Pc端
                if(scr + clientHeight + 10 >= scrHeight){
                    vm.isBackToBottomScroll = false
                }else{
                    vm.isBackToBottomScroll = true
                }
            })
        },
        methods: {
            // 回到底部
            backBottom() {
                window.scrollTo(0, document.body.clientHeight)
            },
            scaleImg(index,imgArr){
                this.$refs.photowb.imagePreview(index,imgArr)
            },
            getPagesNumber(dealType,registerid,hisData){
                let vm = this
                let params = {
                    missivedealtype: dealType,
                    missiveregisterid: registerid,
                }
                let extra = { }
                if(dealType==='1'){
                    extra = { printFlag: '03',...params }
                }else{
                    extra = params
                }
                DataControl.dealFilePage(extra)
                  .then((res)=>{
                      vm.pageCount = res.data.pageCount
                      vm.filesDetails(hisData)
                  })
                  .catch((err)=>{
                      this.$toast(err.message)
                  })
            },
            filesDetails(hisData1){
                let vm = this
                let path = faceConfig.basePath;
                for(let i=0; i<vm.pageCount; i++){
                    if (hisData1.missivedealtype === '1') { //阅文签
                        // for(let i=1; i<=vm.pageCount; i++){
                            let url= `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${hisData1.missivedealtype}&missiveregisterid=${hisData1.missiveregisterid}&viewNumber=${i}&printFlag=03&jpgFlag=true`;
                            this.imgUrl1.push(url);
                            // HttpCommon.axiosRequest({
                            //     url: url
                            // }).then(res => {
                            //     return 'data:image/png;base64,' + btoa(
                            //         new Uint8Array(res.data).reduce((data, byte) => data + String.fromCharCode(byte), '')
                            //     );
                            // }).then(data => {
                            //     this.imgUrl1.unshift(data);
                            //     console.log(this.imgUrl1)
                            // });
                        // }
                        vm.imgShow1 = true;
                        vm.maskShow = false
                    }else { //处理签
                        // for(let i=1; i<=vm.pageCount; i++){
                            let url= `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${hisData1.missivedealtype}&viewNumber=${i}&missiveregisterid=${hisData1.missiveregisterid}&jpgFlag=true`;
                            this.imgUrl.push(url);
                            // HttpCommon.axiosRequest({
                            //     url: url
                            // }).then(res => {
                            //     return 'data:image/png;base64,' + btoa(
                            //         new Uint8Array(res.data).reduce((data, byte) => data + String.fromCharCode(byte), '')
                            //     );
                            // }).then(data => {
                            //     this.imgUrl.unshift(data);
                            //     console.log(this.imgUrl)
                            // });
                        // }
                        vm.imgShow = true;
                        vm.maskShow = false
                    }
                }
            },
            select (index) {
                let vm = this;
                vm.imgShow = false;
                vm.imgShow1 = false;
                vm.imgUrl1 = [];
                vm.imgUrl = [];
                vm.selectIdex = index;
                let path = faceConfig.basePath;
                if (index === 0) {
                    vm.maskShow = true;
                    //有历史签文记录时
                    let historiArr = [];
                    let obj = {};
                    let calsArr = vm.historicals.reduce((cur,next) => {
                            obj[next.missiveregisterid&&next.missivedealtype] ? "" : obj[next.missiveregisterid&&next.missivedealtype] = true && cur.push(next);
                            return cur;
                        },[]);
                    calsArr.forEach((item,index) => {
                        if(vm.receipt.missiveregisterid==item.missiveregisterid&&vm.receipt.missivedealtype==item.missivedealtype){
                            return
                        }else{
                            historiArr.push(item)
                        }
                    });
                    // if (vm.historicals !== undefined && vm.historicals !== null && vm.historicals.length > 0){
                    if (historiArr !== undefined && historiArr !== null && historiArr.length > 0){
                        /*for (let i = 0; i < vm.historicals.length; i++) {
                            let hisData1 = vm.historicals[i]
                            if (hisData1.missivedealtype === '1') {
                                vm.fileUrl1 = `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${vm.receipt.missivedealtype}&missiveregisterid=${vm.receipt.missiveregisterid}&printFlag=03`
                                vm.imgShow1 = true
                            }else {
                                vm.fileUrl = `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${vm.receipt.missivedealtype}&missiveregisterid=${vm.receipt.missiveregisterid}`
                                vm.imgShow = true
                            }
                        }*/
                        for (let i = 0; i < historiArr.length; i++) {
                            let hisData1 = historiArr[i];
                            vm.getPagesNumber(hisData1.missivedealtype,hisData1.missiveregisterid,hisData1)
                            // if (hisData1.missivedealtype === '1') { //阅文签
                            //     let url= `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${hisData1.missivedealtype}&missiveregisterid=${hisData1.missiveregisterid}&printFlag=03&jpgFlag=true`;
                            //     HttpCommon.axiosRequest({
                            //         url: url
                            //     }).then(res => {
                            //         return 'data:image/png;base64,' + btoa(
                            //             new Uint8Array(res.data).reduce((data, byte) => data + String.fromCharCode(byte), '')
                            //         );
                            //     }).then(data => {
                            //         this.imgUrl1 = data;
                            //         vm.imgShow1 = true;
                            //         vm.maskShow = false
                            //     });
                            // }else { //处理签
                            //     let url= `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${hisData1.missivedealtype}&missiveregisterid=${hisData1.missiveregisterid}&jpgFlag=true`;
                            //     HttpCommon.axiosRequest({
                            //         url: url
                            //     }).then(res => {
                            //         return 'data:image/png;base64,' + btoa(
                            //             new Uint8Array(res.data).reduce((data, byte) => data + String.fromCharCode(byte), '')
                            //         );
                            //     }).then(data => {
                            //         this.imgUrl = data;
                            //         vm.imgShow = true;
                            //         vm.maskShow = false
                            //     });
                            // }
                        }
                    }
                    if (vm.receipt.missivedealtype === '1') {
                        vm.getPagesNumber(vm.receipt.missivedealtype,vm.receipt.missiveregisterid,{
                            missivedealtype:vm.receipt.missivedealtype,
                            missiveregisterid:vm.receipt.missiveregisterid
                        })
                        /*// vm.fileUrl1 = `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${vm.receipt.missivedealtype}&missiveregisterid=${vm.receipt.missiveregisterid}&printFlag=03`
                        // vm.fileUrl1 = `../../pdfjs/web/viewer.html?encodeURIComponent(file=${path}/missivedeal/dealFilePrint2Version?missivedealtype=${vm.receipt.missivedealtype}&missiveregisterid=${vm.receipt.missiveregisterid}&printFlag=03&token=${token})`;
                        HttpCommon.axiosRequest({
                            url: `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${vm.receipt.missivedealtype}&missiveregisterid=${vm.receipt.missiveregisterid}&printFlag=03`
                            // url: `https://oa.scspc.gov.cn:8080/oa/oaFile/viewSwFile?&fileid=c803cfad463e435d9cb32ba0ed6b342b`
                        }).then(res => {
                            this.pdfh5 = new Pdfh5('#pdf_demo', {
                                data: res.data,
                                // logo:false
                                logo:{
                                    src: 'https://ss0.bdstatic.com/94oJfD_bAAcT8t7mm9GUKT-xh_/timg?image&quality=100&size=b4000_4000&sec=1594692099&di=dabc0f0a2acd90d7693730fa73a5ead6&src=http://www.juren2007.com/uploads/allimg/180711/3-1PG1160500a5.jpg',
                                }
                            });
                        });
                        // vm.imgShow1 = true
                        vm.imgShow = true*/
                        // let url = `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${vm.receipt.missivedealtype}&missiveregisterid=${vm.receipt.missiveregisterid}&printFlag=03&jpgFlag=true`;
                        // HttpCommon.axiosRequest({
                        //     url: url
                        // }).then(res => {
                        //     return 'data:image/png;base64,' + btoa(
                        //         new Uint8Array(res.data).reduce((data, byte) => data + String.fromCharCode(byte), '')
                        //     );
                        // }).then(data => {
                        //     this.imgUrl1 = data;
                        //     vm.imgShow1 = true;
                        //     vm.maskShow = false
                        // });
                    }else if(vm.receipt.missivedealtype === '2' || vm.receipt.missivedealtype === '3'){
                        // vm.fileUrl = `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${vm.receipt.missivedealtype}&missiveregisterid=${vm.receipt.missiveregisterid}`
                        // vm.fileUrl1 = `../../pdfjs/web/viewer.html?encodeURIComponent(file=${path}/missivedeal/dealFilePrint2Version?missivedealtype=${vm.receipt.missivedealtype}&missiveregisterid=${vm.receipt.missiveregisterid}&token=${token})`;
                        /*HttpCommon.axiosRequest({
                            url: `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${vm.receipt.missivedealtype}&missiveregisterid=${vm.receipt.missiveregisterid}`
                            // url: `https://oa.scspc.gov.cn:8080/oa/oaFile/viewSwFile?&fileid=c803cfad463e435d9cb32ba0ed6b342b`
                        }).then(res => {
                            this.pdfh5 = new Pdfh5('#pdf_demo', {
                                data: res.data,
                                // logo:false
                                logo:{
                                    src: 'https://ss0.bdstatic.com/94oJfD_bAAcT8t7mm9GUKT-xh_/timg?image&quality=100&size=b4000_4000&sec=1594692099&di=dabc0f0a2acd90d7693730fa73a5ead6&src=http://www.juren2007.com/uploads/allimg/180711/3-1PG1160500a5.jpg',
                                }
                            });
                        });
                        vm.imgShow = true*/

                        vm.getPagesNumber(vm.receipt.missivedealtype,vm.receipt.missiveregisterid,{
                            missivedealtype:vm.receipt.missivedealtype,
                            missiveregisterid:vm.receipt.missiveregisterid
                        })
                        // let url = `${path}/missivedeal/dealFilePrint2Version?missivedealtype=${vm.receipt.missivedealtype}&missiveregisterid=${vm.receipt.missiveregisterid}&jpgFlag=true`;
                        // HttpCommon.axiosRequest({
                        //     url: url
                        // }).then(res => {
                        //     return 'data:image/png;base64,' + btoa(
                        //         new Uint8Array(res.data).reduce((data, byte) => data + String.fromCharCode(byte), '')
                        //     );
                        // }).then(data => {
                        //     this.imgUrl = data;
                        //     vm.imgShow = true;
                        //     vm.maskShow = false
                        // });
                    }

                    vm.pdfShow = false;
                    vm.fileListShow = false;
                    vm.detailsShow = false;
                    vm.isBackToBottomNow = true;
                    //显示签收框等
                    vm.$emit('showBottm', {banwenShow:true,qianshouShow:true,commonLanShow:this.commonLanShowInit,processShow:this.processShowInit,buttonShow:true,fankuiShow:true});
                }else if(index ===1 || index ===2){  //正文、附件
                    vm.pdfShow = false;
                    this.detailsShow = false;
                    DataControl.getSomeFileUrl(index,vm.receipt.missiveregisterid,vm.receipt.missivedealid).then(filelist => {
                        vm.fileList = filelist;
                        if(filelist && filelist.length>0){
                            this.filelistNull = false
                            //正文时显示pdf预览
                            if(index ===1){
                                HttpCommon.axiosRequest({
                                    url: `${path}/oaFile/viewSwFile?&fileid=${vm.fileList[0].fileId}`
                                }).then(res => {
                                    this.pdfh5 = new Pdfh5('#pdf_demo', {
                                        data: res.data,
                                        logo:{
                                            src: 'https://ss0.bdstatic.com/94oJfD_bAAcT8t7mm9GUKT-xh_/timg?image&quality=100&size=b4000_4000&sec=1594692099&di=dabc0f0a2acd90d7693730fa73a5ead6&src=http://www.juren2007.com/uploads/allimg/180711/3-1PG1160500a5.jpg',
                                        }
                                    });
                                });
                                vm.fileListShow = false;//隐藏列表
                                vm.pdfShow = true;//显示pdf预览
                            }else{
                                vm.fileListShow = true;
                            }
                        }else{
                            vm.fileListShow = true;
                            this.filelistNull = true
                        }
                    });

                    this.isBackToBottomNow = false;
                    vm.imgShow = false;
                    vm.imgShow1 = false;
                    //隐藏签收框等
                    this.$emit('showBottm', {banwenShow:false,qianshouShow:false,commonLanShow:false,processShow:false,buttonShow:false,fankuiShow:false})
                }else if(index ===3){  //流程
                    this.detailsShow = true;
                    vm.fileListShow = false;
                    vm.pdfShow = false;
                    vm.imgShow = false;
                    vm.imgShow1 = false;

                    this.isBackToBottomNow = false;
                    //隐藏签收框等
                    this.$emit('showBottm', {banwenShow:false,qianshouShow:false,commonLanShow:false,processShow:false,buttonShow:false,fankuiShow:false})
                }
            },
            itemClick({ id = '', missivedealtype='' }) {
                this.$router.push({
                    path: '/receiptInfodetail',
                    params: { missivedealtype },
                    query: {
                        id,
                        receiptinfo:this.receipt
                    }
                })
            },
            fnFileClick (item) {
                let fileUrl = faceConfig.basePath + '/oaFile/viewSwFile?' + 'fileid=' + item.fileId;
                this.$router.push({
                    path: '/file-preview',
                    query: {
                        fileUrl : fileUrl
                    }
                })
            },
            //下载文件
            openFile({ fileId, fileName }) {
                this.$MdModule.openFileByMd(fileId, fileName)
            },
        }
    }
</script>

<style scoped lang="scss">
    @import "pdfh5.css";
    .receipt-title {
    width: 100%;
    text-align: justify;
    word-break: break-all;
    }
    .document-title{
        width: 100%;
        background-color: #e9eefb;
        font-size: 18px;
        padding: 12px 16px;
        box-sizing: border-box;
    }
    .document-desc{
        padding: 10px 0 0;
        box-sizing: border-box;
        color: rgba(0,0,0,.6);
        font-size: 14px;
    }
    .document-desc :nth-child(1){
        padding-right: 25px;
        box-sizing: border-box;
    }

    .document-tabs {
        display: flex;
        align-items: center;
        background: #ffffff;
        height: 48px;
        font-family: PingFang-SC-Medium;
        font-size: 16px;
        color: #0764c4;
        padding: 0 10px;
    }
    .document-tabs .tabs-select {
        justify-content: center;
        width: 33.33%;
        height: 100%;
        line-height: 100%;
        display: flex;
        align-items: center;
        position: relative;
    }
    /*.document-tabs .selected {*/
    /*font-size: 1rem;*/
    /*color: #333333;*/
    /*}*/
    .document-tabs .selected::after {
        content: '';
        position: absolute;
        left: 50%;
        transform: translateX(-50%);
        bottom: 0;
        right: 0;
        width: 50px;
        height: 3px;
        background: url('../../assets/images/line-bottom.png') no-repeat;
        background-size: cover;
    }
    .document-tabs .line {
        width: 2px;
        height: 26px;
        background: url('../../assets/images/line.png') no-repeat;
        background-size: cover;
    }
    .preview-box{
        width: 100%;
        min-height: 540px;
        margin-top: 16px;
        background-color: #fff;
    }
    /deep/.pinch-zoom-container{
       height: auto!important;
    }

    .document-filelist{
        margin-top: 2px;
    }

    .document-filelist-a{
        position: relative;
        display: -webkit-box;
        display: -webkit-flex;
        display: flex;
        box-sizing: border-box;
        width:100%;
        padding:10px 16px;
        overflow:hidden;
        color: #323233;
        font-size: 14px;
        line-height: 24px;
        background-color: #fff;
        border-bottom: 1px solid #ebedf0;
    }

    .document-filelist-a a{
        color: #d71313;
    }
    .document-filelist-a a:visited{
        color: #d71313;
    }
    .document-filelist-show{
        background-color: #fff;
        line-height: 46px;
        text-align: center;
        color: #dddddd;
    }

    .folder {
        width: 15px;
        height: 13px;
        margin-right: 6px;
    }
    .icon_file {
        width: 20px;
        height: 18px;
        margin-right: 6px;
    }
    /deep/.pdfjs .pdfViewer .pageContainer img.pdfLogo{
        display: none;
    }

    .pdf-box {
        position: relative;
        margin-top: 2px;
        .backBottom{
            position: fixed;
            z-index: 1000;
            right: 20px;
            bottom: 80px;
            display: flex;
            align-items: center;
            justify-content: center;
            width: 35px;
            height: 35px;
            background: rgb(238, 10, 36);
            color: rgb(255, 255, 255);
            box-shadow: 0 2px 7px rgba(0, 0, 0, .35);
            border-radius: 50%;
            font-size: 25px;
            img {
                width: 20px;
                height: 20px;
            }
        }
    }
</style>
