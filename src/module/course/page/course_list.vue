<template>
  <section>
    <el-row >
      <el-col :span="8"  :offset=2 >
        <el-card shadow="hover" :body-style="{ padding: '10px' }">
          <img src="/static/images/add.jpg" class="image" height="150px">
          <div style="padding: 10px;">
            <span>Course Manager</span>
            <div class="bottom clearfix">
              <time class="time"></time>
              <router-link class="mui-tab-item" :to="{path:'/course/add/base'}">
                  <el-button type="text" class="button" >新增课程</el-button>
              </router-link>
            </div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="8" v-for="(course, index) in courses" :key="course.id" :offset="index > 0 ? 2 : 2">
        <el-card shadow="hover" :body-style="{ padding: '10px' }">
          <img :src="course.pic!=null?imgUrl+course.pic:'/static/images/nonepic.jpg'" class="image" height="150px">
          <div style="padding: 10px;">
            <span>{{course.name}}</span>
            <div class="bottom clearfix">
              <time class="time"></time>
              <el-button type="text" class="button" @click="handleManage(course.id)">管理课程</el-button>
            </div>
          </div>
        </el-card>
      </el-col>

      <!--分页-->
      <el-col :span="24" class="toolbar">
        <el-pagination @size-change="handleSizeChange" background layout="total, sizes, prev, pager, next, jumper" @current-change="handleCurrentChange" :page-size="size"
                       :total="total"  :current-page="page" :page-sizes="sizes"
                       style="float:right;">
        </el-pagination>
      </el-col>
    </el-row>
  </section>
</template>
<script>
  import * as courseApi from '../api/course';
  import utilApi from '../../../common/utils';
  let sysConfig = require('@/../config/sysConfig');
  export default {
    data() {
      return {
        page:1,
        size:7,
        total: 0,
        courses: [
          {
            id:'4028e581617f945f01617f9dabc40000',
            name:'Bootstrap开发框架',
            pic:''
          }
          ],
        sels: [],//列表选中列
        imgUrl:sysConfig.imgUrl,
        sizes: [7, 15, 21, 28, 35]
      }
    },
    methods: {
        //分页方法,分页回调该方法
      handleCurrentChange(val) {
        this.page = val;
        this.getCourse();
      },
      // 页面改变后查询
      handleSizeChange(val) {
        this.size = val;
        this.getCourse();
      },
      //获取课程列表
      getCourse() {
        courseApi.findCourseList(this.page,this.size,{}).then((res) => {
          console.log(res);
          if(res.success){
            this.total = res.queryResult.total;
            this.courses = res.queryResult.list;
          }

        });
      },
      handleManage: function (id) {
        this.$router.push({ path: '/course/manager/'+id})
      }

    },
    created(){

    },
    mounted() {
      //查询我的课程
      this.getCourse();
    }
  }
</script>
<style scoped>
  .el-col-8{
    width:20%
  }
  .el-col-offset-2{
    margin-left:2%
  }
  .time {
    font-size: 13px;
    color: #999;
  }

  .bottom {
    margin-top: 13px;
    line-height: 12px;
  }

  .button {
    padding: 0;
    float: right;
  }

  .image {
    width: 100%;
    display: block;
  }

  .clearfix:before,
  .clearfix:after {
    display: table;
    content: "";
  }

  .clearfix:after {
    clear: both
  }
</style>
