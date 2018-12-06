<template>
  <div class="page">
    <div class="data-title">
      <h2 class="page-title">爬虫服务监控</h2>
      <div class="search">
        <div class="s-left">
          <div class="s-dot">
            <span>修改时间: </span>
            <el-date-picker
              v-model="listQuery.startTime"
              type="date"
              placeholder= "开始时间"
              @change="changeStartTime"
              :picker-options="pickerOptions0">
            </el-date-picker>
            -
            <el-date-picker
              v-model="listQuery.endTime"
              type="date"
              placeholder="结束时间"
              @change="changeEndTime"
              :picker-options="pickerOptions1">
            </el-date-picker>
          </div>
          <div class="s-dot">
            <span></span>
            <button class="s-btn" @click="search">搜索</button>
          </div>
        </div>
        <div class="s-right">
          <div class="s-dot">
            <span class="big">过滤条件：</span>
            <el-input v-model="listQuery.searchKey" placeholder=" 订单号 / 模板名称"></el-input>
          </div>
        </div>
      </div>
    </div>
    <div class="page-content">
      <h2 class="data-title2">爬虫服务监控列表 <!-- rtorId-->
        <!--<span class="">-->
          <el-button class="small form-btn" type="warning" size="small" @click="removeBatch(sels)" :disabled="this.sels.length === 0||this.disabled">批量删除</el-button>
        <!--</span>-->
         <!--<span @click="delGroup"  :disabled="this.sels.length == 0">批量终结</span>-->
      </h2><!--@click="userGroupVisible = true"-->
      <div class="data-content">
        <div class="data-right">
          <!--<el-table
            :data="tableData" ref="table" @selection-change="selsChange" @row-click="handleCurrentChange"
            class="big-tab"
            width="100%"
            height="236"
            style="width: 100%">--><!--:tooltip-effect="dark"-->
            <el-table  @selection-change="selsChange" ref="table" :data="tableData"
                      class="big-tab"
                      width="100%"
                      height="236"
                      style="width: 100%">
            <el-table-column type="selection">
          </el-table-column>
            <el-table-column label="订单号" width="130px">
              
              <template scope="scope">
                <span> {{scope.row.rtorOrderId}}</span>
              </template>
            </el-table-column>
            <el-table-column label="源地址" width="350px" :show-overflow-tooltip="true">
              <template scope="scope">
                <span> {{scope.row.rtrtDataUrl}}</span>
              </template>
            </el-table-column>
            <el-table-column label="模板名称" width="150px" :show-overflow-tooltip="true">
              <template scope="scope">
                <span> {{scope.row.rtrtTemplateName}}</span>
              </template>
            </el-table-column>
            <el-table-column label="修改时间">
              <template scope="scope">
                <span> {{scope.row.modifyTime}}</span>
              </template>
            </el-table-column>
            <el-table-column label="结束时间">
              <template scope="scope">
                <span> {{scope.row.rtorFinishTime}}</span>
              </template>
            </el-table-column>
            <el-table-column label="状态" width="60px">
              <template scope="scope">
                <span> {{scope.row.rtorStatus=='y' ? "有效" : "无效"}}</span>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </div>
      <div class="pagination">
        <el-pagination
          small
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="listQuery.page"
          :page-size="listQuery.limit"
          :page-sizes="[5,10,20,30,50,70]"
          layout="total, sizes, prev, pager, next, jumper"
          :page-count="total"
          :total="total">
          
        </el-pagination>
      </div>
    </div>
   
  </div>
</template>

<script>
  import {
    getCrawlerService,
    delCrawler
  } from 'api/dataMonitor'

  export default {
    data() {
      return {
//        arr: ['装船', '重箱进场', '申报', 'VGM回执','重箱进场', '申报', 'VGM回执','VGM回执','重箱进场', '申报', 'VGM回执'],
        tableData: [],
        pickerOptions0: {
          disabledDate: (time) => {
            if (this.listQuery.endTime != "") {
              return time.getTime() > Date.now() || time.getTime() > this.listQuery.endTime;
            } else {
              return time.getTime() > Date.now();
            }
          },
        }, //超时当前时间后，禁用时间
        pickerOptions1: {
          disabledDate: (time) => {
            return time.getTime() < this.listQuery.startTime || time.getTime() > Date.now();
          }
        },
        listQuery: {
          page: 1,
          limit: 5,
          startTime: '',
          endTime: '',
          searchKey: ''
        },
        total: null,
        sels: [],//选中的值显示
        disabled:true,
        sort: 'createAt',
        order: 'descending',
      }
    },
    created() {
    
    },
    mounted(){
    },
    methods: {
      selsChange(sels) {
        //被选中的行组成数组
        this.sels = sels;
        //遍历被选中行数所组成的数组
        for(let element of this.sels){
          //如果视频正在转码或者等待转码,禁用按钮,结束方法
          if(element.status == 'waiting'||element.progress){
            this.disabled = true;
            return;
          }
          //如果没有视频正在转码或者等待转码,按钮可用
          this.disabled = false;
        }
      },
      removeBatch(rows){
        let ids = [];
        rows.forEach(element =>{
          ids.push(element.rtorId) ;
        })
       let msg = `您将终结${ids.length}个订单的爬虫服务。`;
        this.$confirm(msg,'提示',{
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        
        }).then(() =>{
          ids = ids.join(',');
          delCrawler(ids).then(() => {
            this.tableData.splice( this.row, ids.length)
          }).catch(()=>{})
        }).catch(()=>{});
      },
      
        //计算相隔天数
      timeFn(dateBegin, dateEnd){

//        let dateBegin = new Date(dateBegin);//获取当前时间
//        let dateEnd = new Date(dateEnd);
        let dateDiff = new Date(dateEnd).getTime() - new Date(dateBegin).getTime();//时间差的毫秒数
        let dayDiff = Math.floor(dateDiff / (24 * 3600 * 1000));//计算出相差天数
//        let leave1=dateDiff%(24*3600*1000)    //计算天数后剩余的毫秒数
//        let hours=Math.floor(leave1/(3600*1000))//计算出小时数
//        //计算相差分钟数
//        let leave2=leave1%(3600*1000)    //计算小时数后剩余的毫秒数
//        let minutes=Math.floor(leave2/(60*1000))//计算相差分钟数
//        //计算相差秒数
//        let leave3=leave2%(60*1000)      //计算分钟数后剩余的毫秒数
//        let seconds=Math.round(leave3/1000)
        if(dayDiff < 0){
          return '0'
        }else if(dayDiff > 31){
          return '1';
        }
      },

      //	选择开始时间
      changeStartTime(options) {
        let _this =this;
        if(options) {
          _this.listQuery.startTime = options;
        } else {
          _this.listQuery.startTime = ''
        }
//        _this.timeFormat(options);
        /*if(new Date(_this.listQuery.startTime).getTime() > new Date(_this.listQuery.endTime).getTime()) {
        return '0';
      } else if(months >= 12) {
        return '1';
      } else {}*/
//        console.log(_this.listQuery.startTime , _this.listQuery.endTime, "时间查询---" );
//        let isshow = CF.dateMethod(_this.listQuery.startTime, _this.listQuery.endTime);
        /*if(isshow == 0) {
          _this.$alert('开始时间不能晚于结束时间', '提示', {
            confirmButtonText: '确定',
          });
          _this.listQuery.startTime = '';
        } else {}*/
      },
      //选择结束时间
      changeEndTime(options) {
        let _this =this;
        if(options) {
          _this.listQuery.endTime = options;
        } else {
          _this.listQuery.endTime = '';
        }
      },
      //调用接口
      getCrawlerService(){
        this.tableData=[];
        let obj = {
          "page": this.listQuery.page,
          "limit": this.listQuery.limit,
          "startTime": this.listQuery.startTime,
          "endTime": this.listQuery.endTime,
          "searchKey": this.listQuery.searchKey
        }
        getCrawlerService(obj).then(res => {
            let data = res.data;
            if(res.status == 200) {
              this.tableData = data.rows;
              this.total = data.total;
            }
        }).catch()
      },
      //批量删除
      handleDelete(row) {
//        console.log(id, "传入的id值");
        let ids = [];
        this.$confirm("您将终结{{}}个订单的爬虫服务。", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }).then(() => {
          delCrawler(row.ids).then(() => {
            this.$notify({
              title: "成功",
              message: "删除成功",
              type: "success",
              duration: 2000
            });
            const index = this.tableData.indexOf(row);
            console.log(index, "id值=====")
            this.tableData.splice(index, 1);
          });
        });
      },
      // 通知 —— 悬浮出现在页面右上角，显示全局的通知提醒消息。
      unDatanotification() {
        this.$notify.info({
          title: '消息',
          message: '您好，暂无数据！'
        });
//        this.tableData = [];
      },
      
      handleSizeChange(val) {
        this.listQuery.limit = val;
        this.getCrawlerService();
      },
      handleCurrentChange(val) {
        this.listQuery.page = val;
        this.getCrawlerService();
      },
      search() {
       
        let isTime = this.timeFn(this.listQuery.startTime, this.listQuery.endTime)
        if(isTime == 0){
          this.$alert('开始时间不能晚于结束时间', '提示', {
            confirmButtonText: '确定',
          });
          this.listQuery.endTime = '';
          return false;
        }else if(isTime == 1){
          this.$alert('时间范围不能超过31天', '提示', {
            confirmButtonText: '确定',
          })
          this.listQuery.endTime = '';
          return false;
        }else{}
        if(!this.listQuery.startTime && !this.listQuery.endTime){
          this.$alert('修改时间不能为空', '提示', {
            confirmButtonText: '确定',
          });
          return false;
        }
        if(!this.listQuery.endTime && this.listQuery.startTime){
          this.listQuery.endTime = this.listQuery.startTime
        }
        if(!this.listQuery.startTime && this.listQuery.endTime){
          this.listQuery.startTime = this.listQuery.endTime
        }
        this.getCrawlerService();
        setTimeout(() => {
          if(this.tableData.length==0 || this.tableData == 'undefined'){
            this.unDatanotification()
          }else{}
        },1000)
       
        
  
      },
    }
  };
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  .pagination{
    margin-top: -40px;
  }
  .el-input{
    width: 200px !important;
  }
  .data-title{
    height: 190px;
    background: #fff;
    h2{
      height: 65px;
      line-height: 65px;
      font-weight: 400;
    }
  }
  .search{
    padding-top: 15px;
    padding-left: 20px;
  }
  .s-left, .s-right{
    float: left;
    width: 55%;
  }
  .s-right{
    width: 45%;
  }
  .s-dot{
    padding-bottom: 22px;
    span{
      display: inline-block;
      width: 70px;
      text-align: right;
    }
    span.big{
      width: 120px;
    }
  }
  .page-content{
   
    border: 1px solid #D9D9D9;
    padding: 0;
  }
  .data-title2{
    width: 100%;
    height: 40px;
    line-height: 40px;
    background: #FBFBFB;
    border-bottom: 1px solid #D9D9D9;
    font-size: 16px;
    padding-left: 20px;
    /*float: left;*/
  }
  .form-btn{
    float: right;
    text-align: right;
    height: 80%;
    line-height: 1.5em;
    margin: 5px;
    box-sizing: border-box;
  }
  .data-right{
    margin-left: 20px;
    margin-top: -20px;
  }
  .data-content{
    display: flex;
    display: -webkit-flex;
    height: 286px;
    width: 100%;
  }
  .data-left{
    width: 376px;
    position: relative;
  }
  .data-right{
    flex: 1;
    -webkit-flex: 1;
    padding-top: 29px;
    padding-right: 20px;
  }
  #stateEC1_1, #stateEC1_2{
     width: 100%;
     height: 100%;
  }
  .echarts-tooltip{
    position: absolute;
    left: 50%;
    bottom: 10px;
    font-size: 10px;
    margin-left: -78px;
    span:nth-of-type(1), span:nth-of-type(3){
      display: inline-block;
      width: 4px;
      height: 4px;
      background: #314552;
      vertical-align: middle;
      margin-bottom: 2px;
    }
    span:nth-of-type(3){
      margin-left: 10px;
      background: #C8843A;
      
    }
  }
</style>
