<template>
  <div :style="{width:windowWidth+'px',height:conHeight+'px',backgroundColor: designCache.bgColor,
  backgroundImage: designCache.bgImg ? 'url('+fileUrl+designCache.bgImg+')':'none'}"
       style="background-size:100% 100%;background-color: #2b3340;overflow: hidden">
    <div style="position:relative;transform-origin:0 0;"
         :style="{width:windowWidth+'px',height:conHeight+'px',transform: 'scale('+containerScale+')'}">
      <transition-group appear name="bounce">
        <div v-for="item in designCache.components" :key="item.id"
             style="position: absolute;"
             :style="{width:Math.round(item.cptWidth)+'px',
                  height:Math.round(item.cptHeight)+'px',
                  top:Math.round(item.cptY)+'px',
                  left:Math.round(item.cptX)+'px',
                  zIndex:item.cptZ}">

          <component :is="item.cptKey" :width="Math.round(item.cptWidth)"
                   :height="Math.round(item.cptHeight)" @reload="loadCacheData"
                   :option="item.cptOption"/>
        </div>
      </transition-group>
    </div>

    <el-dialog title="请输入访问码" :visible.sync="authCodeDialogVisible" width="30%" center
               :show-close="false" :close-on-click-modal="false" :close-on-press-escape="false">
      <el-form>
        <el-form-item label="访问码">
          <el-input v-model="viewCode" autocomplete="off"/>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="authCode">提 交</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import {authViewCodeApi, getByIdApi} from "@/api/DesignerApi";
import {fileUrl} from "/env";

export default {
  name: "preview_index",
  data(){
    return{
      fileUrl:fileUrl,
      designCache:{},
      windowWidth: 0,
      windowHeight: 0,
      conHeight: 0,
      containerScale:1,
      authCodeDialogVisible:false,
      viewCode:''
    }
  },
  mounted() {
    const that = this;
    that.loadCacheData();
    window.onresize = () => {
      return (() => {
        that.loadSize()
      })();
    };
  },
  methods:{
    loadCacheData(){
      const path = this.$route.path;
      const that = this;
      const id = this.$route.query.id;
      if (path === '/preview'){

        let temp = {"id":"","title":"我的大屏","scaleX":1920,"scaleY":1080,"version":"1.9.0","bgColor":"#2B3340","simpleDesc":"","bgImg":"","viewCode":"","components":[{"cptTitle":"边框","icon":"border","cptKey":"cpt-dataV-border","cptOptionKey":"cpt-dataV-border-option","cptOption":{"attribute":{"borderType":"dv-border-box-1","borderColor1":"#409eff","borderColor2":"#f00","backgroundColor":"rgba(0, 0, 0, 0)","borderTitle":"标题1","titleWidth":250,"dur":3,"reverse":false}},"cptX":419,"cptY":222,"cptZ":100,"cptWidth":400,"cptHeight":300,"id":"71b207d0-b77a-11ee-9e56-8748b18b515a"},{"cptTitle":"文字框","icon":"text","cptKey":"cpt-text","cptOptionKey":"cpt-text-option","cptOption":{"cptDataForm":{"dataText":"{\"value\":\"普通文本\"}","dataSource":1,"pollTime":0,"apiUrl":"/design/test","sql":"select username from sys_user limit 1"},"attribute":{"url":"","textColor":"#4BB344","textSize":16,"fontWeight":"normal","textLineHeight":30,"textFamily":"微软雅黑","textAlign":"center","fontStyle":"normal","textDecoration":"none","bgColor":"rgba(255, 255, 255, 0)"}},"cptX":227,"cptY":191,"cptZ":100,"cptWidth":150,"cptHeight":40,"id":"7258cf20-b77a-11ee-9e56-8748b18b515a"},{"cptTitle":"图片","icon":"image","cptKey":"cpt-image","cptOptionKey":"cpt-image-option","cptOption":{"attribute":{"url":null,"fit":"fill","preview":false}},"cptX":951,"cptY":200,"cptZ":100,"cptWidth":400,"cptHeight":300,"id":"730a44d0-b77a-11ee-9e56-8748b18b515a"}]}

        // let designCache = JSON.parse(localStorage.getItem('designCache'));
        let designCache = temp
        this.loadDesign(designCache,false);
      }else if(path === '/view'){
        if (!id){
          this.$message.error('id错');
          return;
        }
        const viewCode = localStorage.getItem('code'+id);//如果已经输入过访问码就带着访问码一起请求
        getByIdApi(id,1, viewCode).then(res => {
          if (res.data === 'NEED_AUTH'){
            that.authCodeDialogVisible = true;
          }else{
            that.loadDesign(res.data, true);
          }
        })
      }
    },
    loadDesign(design, componentPares){
      if (componentPares){
        design.components = JSON.parse(design.components);
      }
      document.title = design.title;
      this.designCache = design;
      this.loadSize();
    },
    authCode(){
      if (!this.viewCode){
        this.$message.error('请输入访问码');
        return;
      }
      const id = this.$route.query.id;
      authViewCodeApi({id: id, viewCode:this.viewCode}).then(res => {
        this.authCodeDialogVisible = false;
        localStorage.setItem('code'+id, res.data.viewCode);//缓存访问码避免二次刷新需要再次输入
        this.loadDesign(res.data,true)
      })
    },
    loadSize(){
      this.windowWidth = document.documentElement.clientWidth;
      this.windowHeight = document.documentElement.clientHeight;
      this.containerScale = Math.round(this.windowWidth / this.designCache.scaleX * 100) / 100
      this.conHeight = this.designCache.scaleY
      console.log(this.windowWidth,this.containerScale)
    },
  }
}
</script>

<style scoped>
.bounce-enter-active{
  transition: all 1s;
  /*animation: bounce-in 1s;*/
}
.bounce-enter{
  opacity: 0;
  transform: scale(.5);
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  25% {
    transform: scale(0.5);
  }
  50% {
    transform: scale(1);
  }
  75% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
</style>
