<template>
  <div class="picture">
    <el-upload
        ref="imgBroadcastUpload"
        :auto-upload="false" multiple
        :file-list="diaLogForm.imgBroadcastList"
        list-type="picture-card"
        :on-change="imgBroadcastChange"
        :on-remove="imgBroadcastRemove"
        accept="image/jpg,image/png,image/jpeg"
        action=""
    >
      <i class="el-icon-plus"></i>
    </el-upload>
    <div slot="tip" class="el-upload__tip">上传750*406的图片效果更佳</div>
    <el-button type="primary" @click="submitDialogData">提交</el-button>
  </div>
</template>

<script>
import { uploadImgToBase64 } from '@/lib/utils' // 导入本地图片转base64的方法

export default {
  name: 'imgUpload',
  data () {
    return {
      diaLogForm: {
        goodsName:'', // 商品名称字段
        imgBroadcastList:[], // 储存选中的图片列表
        imgsStr:''   // 后端需要的多张图base64字符串 , 分割
      }
    }
  },
  methods: {
    // 图片选择后 保存在 diaLogForm.imgBroadcastList 对象中
    imgBroadcastChange (file, fileList) {
      console.log(file);
      const isLt2M = file.size / 1024 / 1024 < 2 // 上传头像图片大小不能超过 2MB
      if (!isLt2M) {
        this.diaLogForm.imgBroadcastList = fileList.filter(v => v.uid !== file.uid)
        this.$message.error('图片选择失败，每张图片大小不能超过 2MB,请重新选择!')
      } else {
        this.diaLogForm.imgBroadcastList.push(file)
      }
    },
    // 有图片移除后 触发
    imgBroadcastRemove (file, fileList) {
      this.diaLogForm.imgBroadcastList = fileList
    },
    // 提交弹窗数据
    async submitDialogData () {
      const imgBroadcastListBase64 = []
      console.log('图片转base64开始...')
      // 并发 转码轮播图片list => base64
      const filePromises = this.diaLogForm.imgBroadcastList.map(async file => {
        const response = await uploadImgToBase64(file.raw)
        return response.result.replace(/.*;base64,/, '') // 去掉data:image/jpeg;base64,
      })
      // 按次序输出 base64图片
      for (const textPromise of filePromises) {
        imgBroadcastListBase64.push(await textPromise)
      }
      console.log('图片转base64结束..., ', imgBroadcastListBase64)
      this.diaLogForm.imgsStr = imgBroadcastListBase64.join()
      console.log(this.diaLogForm)
    },
  }
}
</script>
<style scoped lang="scss">
/*::v-deep .el-upload--picture-card{
  width: 232px;
  height: 172px;
  display: flex;
  align-items: center;
  justify-content: center;
}*/
.picture{
  margin-right: -10px;
}
::v-deep .el-upload{
  width: 100px;
  height: 100px;
  line-height: 100px;
}
::v-deep .el-upload-list--picture-card .el-upload-list__item{
  width: 750px;
  height: 406px;
  line-height: 100px;
}
::v-deep .el-upload-list--picture-card .el-upload-list__item-thumbnail{
  width: 750px;
  height: 406px;
  line-height: 100px;
}
::v-deep .avatar{
  width: 100px;
  height: 100px;
}
</style>