<template>
  <div>
    <!--
      action 上传的地址
      list-type	上传文件的类型
      before-upload 上传之前后调用,用来设置上传的参数。
      on-success 上传成功后调用。上传成功可以修改页面显示的图片。
      file-list 要显示的文件列表,值为[]
      limit 允许最多上传的图片
      on-exceed 超出个数限制调用方法
      before-remove 文件删除之前调用方法，用来作请求从数据库真实删除图片。
      data 需要上传时出了图片带上的一些其他参数>
    -->
    <el-upload
      action="/api/filesystem/upload"
      list-type="picture-card"
      :before-upload="setuploaddata"
      :on-success="handleSuccess"
      :file-list="fileList"
      :limit="picmax"
      :on-exceed="rejectupload"
      :before-remove="handleRemove"
      :data="uploadval">
      <i class="el-icon-plus"></i>
    </el-upload>
  </div>
</template>
<script>
  import * as sysConfig from '@/../config/sysConfig';
  import * as courseApi from '../../api/course';
  import utilApi from '../../../../common/utils';
  import * as systemApi from '../../../../base/api/system';
  export default {
    data() {
      return {
        picmax:1,//最大上传文件的数量
        courseid:'',
        dialogImageUrl: '',
        dialogVisible: false,
        fileList:[],
        uploadval:{filetag:"course"},//上传提交的额外的数据 ，将uploadval转成key/value提交给服务器
        imgUrl:sysConfig.imgUrl
      }
    },
    methods: {
      //超出文件上传个数提示信息
      rejectupload(){
        this.$message.error("最多上传"+this.picmax+"个图片");
      },
      //在上传前设置上传请求的数据
      setuploaddata(){

      },
      //删除图片
      handleRemove(file, fileList) {
        console.log(file);
        //调用服务端去删除课程图片信息，如果返回false，前端停止删除
        //异步调用，返回一个Promise对象。
        // 因为是异步调用所以不能直接返回false，不会等到异步调用完毕之后返回false。
        // 会直接返回false。使用Promise异步，初始化Permission(异步操作)。回调then或者catch。
        // 设置resolve() 和 reject的调用。
        // 也就能让elementUi知道执行的结果。如果回调then说明删除成功，不用显示图片了。
        // 如果回调catch说明删除失败，继续显示图片。
        return new Promise((resolve,rejct)=>{
          courseApi.deleteCoursePic(this.courseid).then(res=>{
            if(res.success){
                //成功
              resolve();
            }else{
              this.$message.error("删除失败");
                //失败
              rejct();
            }
          })
        })
      },
      //上传成功的钩子方法
      handleSuccess(response, file, fileList){
        console.log(response);
//        alert('上传成功')
        //调用课程管理的保存图片接口，将图片信息保存到课程管理数据库course_pic中
        //从response得到新的图片文件的地址
        if(response.success){
          let fileId = response.fileSystem.fileId;
          courseApi.addCoursePic(this.courseid,fileId).then(res=>{
              if(res.success){
                  this.$message.success("上传图片成功！");
              }else{
                this.$message.error(res.message);
              }
          })
        }
      },
      //上传失败执行的钩子方法
      handleError(err, file, fileList){
        this.$message.error('上传失败');
        //清空文件队列
        this.fileList = [];
      },
      //promise 有三种状态:
      //进行中pending
      //执行成功 resolve
      //执行失败 reject
      testPromise(i){
          return new Promise((resolve,reject)=>{
              if(i<2){
                  //成功了
                resolve('成功了');
              }else{
                  //失败了
                reject('失败了');
              }

          })
      }
    },
    mounted(){
      alert(testPromise(1));
      //课程id
      this.courseid = this.$route.params.courseid;
      //查询课程
      courseApi.findCoursePicList(this.courseid).then(res=>{
          if(res && res.pic){
            // http://img.xuecheng.com/ + 图片路径。
            let imgUrl = this.imgUrl+res.pic;
            //先清空文件列表，再将图片放入文件列表
            this.fileList = [];
            this.fileList.push({name:'pic',url:imgUrl,fileId:res.pic});
          }
      })
      //测试调用promise方法，then中写的成功后的回调方法，
//      this.testPromise(3).then(res=>{
//          alert(res)
//      }).catch(res=>{//catch就是执行失败的回调方法
//          alert("失败了。。。。。")
//          alert(res)
//      })

      /*
      * 上传文件：
      *   1. 从导航栏进入上传页面，渲染页面后请求课程服务，查询所有的图片显示在页面。
      *   [{name:'pic',url:imgUrl,fileId:res.pic},{}]  是一个数组，用来显示图片列表在页面。
      *   push到数组中就可以了，因为一个课程对应的是一个课程图片。每次刷新页面push进去的也只会有一个。
      *   2. 点击上传文件，上传文件和文件的信息。请求通用文件服务，上传图片到服务器。记录图片信息在通用文件服务
      *   数据库,返回上传成功的的信息.
      *   3. 上传成功后使用上传成功的信息请求课程服务，将课程id和图片路径存储在课程对应的图片数据库。
      *   4. 点击图片删除图标，请求课程服务删除课程对应图片。
      *   通用文件服务不作服务和图片的关联，只作上传和记录上传图片的信息。图片地址和服务的关联，由需要上传文件
      *   的服务自己去处理。
      * */
    }
  }
</script>
<style>

</style>
