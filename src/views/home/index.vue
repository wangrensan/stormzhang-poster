<template>
  <div class="home-container">
    <div class="common-operate">
      <div class="common-title"></div>
      <div class="common-btns">
      <el-upload ref = 'upload' accept = '.xls,.xlsx'
        action = '' :auto-upload = false
        :on-change = handleUpload :show-file-list = false>
        <el-button type="primary" slot = 'trigger' native-type = 'file' >导入</el-button>
      </el-upload>
      <el-button type="primary" @click="downloadPoster">导出</el-button>
      </div>
    </div>
    <div v-if="status === 1">当前下载进度：<span :style="{color: progress === '100.00%' ? '#67C23A' : '#409EFF'}">{{progress}}</span></div>
    <div class="poster-list" v-if="list.length">
      <div class="poster-item" v-for="(item, index) in list" :key="index">
        <img class="poster-background" :src="require(`../../assets/images/template_${item.template}.png`)" alt="" srcset="">
        <div class="poster-no">星球编号：{{item.no}}</div>
        <div class="poster-name" :style="{'font-size': `${Math.min((190 / item.name.length), 27)}px`}">{{item.name}}</div>
        <div class="poster-image">
          <img :src="item.image" width="100%" height="100%" crossorigin="anonymous">
        </div>
      </div>
    </div>
    <div v-else>
      <div style="margin-bottom: 10px">请上传excel数据</div>
      <el-upload ref = 'upload' accept = '.xls,.xlsx'
        action = '' :auto-upload = false
        :on-change = handleUpload :show-file-list = false>
        <el-button type="primary" slot = 'trigger' native-type = 'file' >导入</el-button>
      </el-upload>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import html2canvas from 'html2canvas'
import jszip from 'jszip'
import { saveAs } from 'file-saver'
import XLSX from 'xlsx'

export default {
  name: 'Home',
  data () {
    return {
      list: [
        {
          image: "https://dss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=3202386446,3057899623&fm=26&gp=0.jpg",
          name: "帅张一号帅张一号帅",
          no: 1101,
          template: 1,
        },
      ],
      fileLength: 0, // 当前下载的文件数量
      status: 0, // 是否开始下载
    }
  },
  computed: {
    ...mapGetters([
      'name'
    ]),
    progress () {
      return (this.fileLength / this.list.length * 100).toFixed(2) + '%'
    }
  },
  methods: {
    async downloadPoster() {
      this.fileLength = 0
      this.status = 1
      this.zip = new jszip()
      const items = document.getElementsByClassName('poster-item')
      for (let i = 0; i < items.length; i++) {
        await this.addToZip(items[i], this.zip, `${this.list[i].name}.png`)
      }
      this.createZip();
    },
    fileDownload(downloadUrl) {
        let aLink = document.createElement("a");
        aLink.style.display = "none";
        aLink.href = downloadUrl;
        aLink.download = "海报.png";
        // 触发点击-然后移除
        document.body.appendChild(aLink);
        aLink.click();
        document.body.removeChild(aLink);
    },
    addToZip(ele, zip, name) {
        return new Promise(async (resolve, reject) => {
          let scaleCanvas = document.createElement("canvas"); 
          const context = scaleCanvas.getContext("2d");
          const scale = 2; 
          const width = ele.clientWidth
          const height = ele.clientHeight

          scaleCanvas.width = width * scale
          scaleCanvas.height = height * scale
          scaleCanvas.getContext("2d").scale(scale, scale)

          const canvas = await html2canvas(ele, {
            logging: false, //日志开关，便于查看html2canvas的内部执行流程
            width, //dom 原始宽度
            height,
            scrollY: 0, 
            scrollX: 0,
            canvas,
            scale,
            dpi: window.devicePixelRatio * 4,
            useCORS: true // 【重要】开启跨域配置
          })

          console.log(canvas)
          canvas.toBlob((blob) => {
              zip.file(name, blob);  // 将每次不同的canvas数据添加到zip文件中
              this.fileLength++
              resolve();
          });
        })
    },
    createZip() {
      try {
        this.zip.generateAsync({ type: 'blob' }).then((content) => {
            saveAs(content, '帅张星球海报.zip');
            this.$message({ message: '下载成功', type: 'success' })
        });
      } catch (error) {
        this.$message({ message: '下载失败', type: 'error' })
      }
    },
    async handleUpload (file,fileList) {
      const [data] = await this.readExcel(file);
      this.list = data.sheet
    },
    readExcel (file) {
      return new Promise((resolve, reject) => {
        const types = file.name.split('.')[1];
        const fileType = [
          'xlsx', 'xlc', 'xlm', 'xls', 'xlt', 'xlw', 'csv'
        ].some(item => item == types);
        if (!fileType) {
          alert('格式错误！请重新选择');
          return
        }

        const reader = new FileReader();
        let result = [];
        reader.onload = function(e) {
          const data = e.target.result;
          const wb = XLSX.read(data, {
            type: 'binary'
          });
          wb.SheetNames.forEach((sheetName) => {
            result.push({
              sheetName: sheetName,
              sheet: XLSX.utils.sheet_to_json(wb.Sheets[sheetName])
            })
          });
          resolve(result)
        };
        reader.readAsBinaryString(file.raw)

        return result;
      })
    }
  }
}
</script>

<style lang="scss" scoped>
// 通用样式
.common-operate {
  display: flex;
  justify-content: space-between;
}
.common-btns {
  display: flex;
}
.common-btns > * {
  margin-left: 10px;
}

.home-container {
  padding: 10px;
}
.home {
  padding: 10px;
}

.poster-list {
  display: flex;
  flex-wrap: wrap;
  margin-top: 10px;
}
.poster-item {
  width: 320px;
  height: 500px;
  background-repeat: no-repeat;
  background-position: center;
  background-size: cover;
  margin-right: 10px;
  margin-bottom: 10px;
  position: relative;
}
.poster-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
.poster-no {
  position: absolute;
  bottom: 210px;
  color: rgb(253, 253, 253);
  font-size: 17px;
  left: 50%;
  transform: translateX(-50%);
}
.poster-name {
  position: absolute;
  top: 70px;
  left: 50%;
  transform: translateX(-50%);
  color: rgb(253, 253, 253);
  font-size: 27px;
  font-weight: 600;
  width: 190px;
  white-space:nowrap;
  text-align: center;
  overflow: hidden;
}
.poster-image {
  width: 120px;
  height: 120px;
  position: absolute;
  top: 132px;
  border-radius: 50%;
  left: 50%;
  transform: translateX(-50%);
  background-size: cover;
  overflow: hidden;
}
.poster-desc span {
  font-size: 26px;
  font-weight: 1000;
}
</style>
