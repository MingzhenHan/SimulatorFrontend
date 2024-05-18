<template>
  <div>
  
    <div class="grid-content">
  <!-- 包含操作界面标题的容器 -->
  <t1>操作界面</t1>
</div>
<div class="grid-content bg-purple-light" style="border: solid 3px black;box-shadow: -10px 0px 5px black;">
  <!-- 上传配置文件提示框 -->
  <el-alert style="radius: 20px"
    title="上传配置文件"
    type="success"
    :closable="false">
  </el-alert>

  
  <el-form>
  <!-- 上传文件表单 -->
  <el-form-item >
    <!-- Host.xml上传按钮 -->
    <div class="buttonstyle">
      <el-upload
        class="upload-demo"
        ref="upload"
        action="fakeaction"
        :before-upload="false"
        :http-request="uploadHostFile"
        :file-list="fileList"
        :multiple="false"
        :limit="1"
        :on-exceed="handleExceed"
      >
        <el-button size="small">上传Host.xml</el-button>
      </el-upload>
      <!-- Fault.xml上传按钮 -->
      <el-upload
        class="upload-demo"
        action="http://localhost:8082/uploadFault"
        :on-preview="handlePreview"
        :on-remove="handleRemove"
        multiple
        :limit="1"
        :on-exceed="handleExceed"
        :file-list="fileList">
        <el-button size="small">上传Fault.xml</el-button>
      </el-upload>
    </div>
  </el-form-item>

  <!-- 上传Container和AppInfo.xml表单 -->
  <el-form-item >
    <div class="buttonstyle">
      <!-- Container上传按钮 -->
      <el-upload
        class="upload-demo"
        action="http://localhost:8082/uploadContainer"
        :on-preview="handlePreview"
        :on-remove="handleRemove"
        :on-success="saveName"
        multiple
        :limit="1"
        :on-exceed="handleExceed"
        :file-list="fileList">
        <el-button size="small">上传Container</el-button>
      </el-upload>
      <!-- AppInfo.xml上传按钮 -->
      <el-upload
        class="upload-demo"
        ref="upload"
        action="fakeaction"
        :http-request="uploadAppFile"
        :file-list="fileList"
        multiple
        :limit="1"
        :on-exceed="handleExceed">
        <el-button size="small">上传AppInfo.xml</el-button>
      </el-upload>
    </div>
  </el-form-item>

  <!-- 设置暂停容器表单 -->
  <el-form-item>
    <el-form :inline="true" :model="formInline">
      <!-- 容器ID输入框 -->
      <el-form-item label="容器">
        <el-input v-model="formInline.id" placeholder="请输入容器"></el-input>
      </el-form-item>
      <!-- 开始时间输入框 -->
      <el-form-item label="开始时间">
        <el-input v-model="formInline.start" placeholder="请输入开始时间"></el-input>
      </el-form-item>
      <!-- 结束时间输入框 -->
      <el-form-item label="结束时间">
        <el-input v-model="formInline.last" placeholder="请输入结束时间"></el-input>
      </el-form-item>
      <br/>
      <!-- 设置暂停按钮 -->
      <el-form-item>
        <el-button class="button" type="success" round @click="setPause">设置暂停</el-button>
      </el-form-item>
      <!-- 取消暂停按钮 -->
      <el-form-item>
        <el-button class="button" type="success" round @click="resetPause">取消暂停</el-button>
      </el-form-item>
    </el-form>
  </el-form-item>

  <!-- 调度算法选择表单 -->
  <el-form-item >
    <!-- 调度算法选择下拉框 -->
    <span>{{"请选择调度算法"}}</span>
    <el-select v-model="optvalue" placeholder="" size="big">
      <el-option
        v-for="item in options"
        :key="item.optvalue"
        :label="item.label"
        :value="item.optvalue">
      </el-option>
    </el-select>
    <!-- 调度容器按钮 -->
    <el-button class="button" type="success" round @click="placeContainer">调度容器</el-button>
  </el-form-item>

  <!-- 修改容器分配方案按钮 -->
  <el-form-item>
    <el-button type="text" @click="getAssignTable">修改容器分配方案</el-button>
    <!-- 修改容器分配方案对话框 -->
    <el-dialog title="动态修改容器分配" :visible.sync="dialogTableVisible" width="50%" @close="uploadModifyAssign">
      <el-table :data="gridData"  width="90%">
        <el-table-column property="app" label="应用"></el-table-column>
        <el-table-column label="主机" >
          <template scope="scope">
            <el-input size="small" v-model="scope.row.host" placeholder="请输入内容" @change="handleEdit(scope.$index, scope.row)"></el-input>
          </template>
        </el-table-column>
        <el-table-column property="name" label="容器"></el-table-column>
      </el-table>
    </el-dialog>
  </el-form-item>

  <!-- 上传Topo.xml表单 -->
  <el-form-item >
    <div class="buttonstyle">
      <!-- Topo.xml上传按钮 -->
      <el-upload
        class="upload-demo"
        action="http://localhost:8082/uploadtopo"
        :on-preview="handlePreview"
        :on-remove="handleRemove"
        :on-success="saveName"
        multiple
        :limit="1"
        :on-exceed="handleExceed"
        :file-list="fileList">
        <el-button size="small">上传Topo.xml</el-button>
      </el-upload>
    </div>
  </el-form-item>

  <!-- 输入仿真持续时间表单 -->
  <el-form-item>
    <!-- 仿真持续时间输入框 -->
    输入仿真持续时间(微秒)：<el-input style="width:100px;height:80%" type="number" v-model="lasttime" @change="handleSetSimulationTime()">
    </el-input>
  </el-form-item>

  <!-- 全半双工切换和开始仿真按钮表单 -->
  <el-form-item >
    <!-- 全半双工切换开关 -->
    <el-switch
      v-model="halfduplex"
      active-color="white"
      inactive-color="lightskyblue"
      active-text="全双工"
      inactive-text="半双工"
      @change="handleSwitch(halfduplex.valueOf())"
      style="margin-right: 20px"
    >
    </el-switch>
    <!-- 开始仿真按钮 -->
    <el-button class="button" type="success" round @click="run">开始仿真</el-button>
  </el-form-item>

  <!-- 输入要查看的消息名称表单 -->
  <el-form-item>
    <!-- 要查看的消息名称输入框 -->
    输入要查看的消息名称：<el-input style="width:80%;height:80%" v-model="specifiedMsgName" @change="handleSpecifyMsg()">
    </el-input>
  </el-form-item>

  <!-- 输入要查看的链路名称表单 -->
  <el-form-item>
    <!-- 要查看的链路名称输入框 -->
    输入要查看的链路名称：<el-input style="width:80%;height:80%" v-model="specifiedLinkName" @change="handleSpecifyLink()">
    </el-input>
  </el-form-item>

  <!-- 查看单台主机按钮表单 -->
  <el-form-item>
    <span>{{"主机名称"}}</span>
    <!-- 主机名称输入框 -->
    <el-input style="width:60%;height:80%" v-model="specifiedhostname"></el-input>
    <br>
    <!-- 查看单台主机按钮 -->
    <el-button class="button" type="success" round @click="paintSingleCPU">查看单台主机</el-button>
  </el-form-item>
</el-form>


  <!--      <br>-->
      </div>
  
  
  
  </div>
  </template>
  
  <script>
  // 引入所需的库和模块
  import G6 from '@antv/g6'; // 引入G6图形库
  import axios from 'axios'; // 引入axios库用于发送HTTP请求
  import {Message} from "element-ui"; // 引入element-ui的Message组件
  import {data} from "autoprefixer"; // 引入autoprefixer库
  
  // 导出默认模块
  export default {
      data() {
        return {
          // 初始化数据
          lasttime: 1000, // 最后时间，默认为1000
          specifiedMsgName: "", // 指定消息名称
          specifiedLinkName: "", // 指定链路名称
          specifiedhostname: "", // 指定主机名
          specifiedcpuid: 0, // 指定CPU ID，默认为0
          gridData: [ // 表格数据，默认为空数组
            {
              app: "", // 应用名称
              name: "", // 名称
              host: "", // 主机名
            },
          ],
          dialogTableVisible: false, // 对话框显示状态，默认为隐藏
          formInline: { // 内联表单数据
            id: 1, // ID，默认为1
            start:  0.0, // 开始时间，默认为0.0
            last: 0.0, // 结束时间，默认为0.0
          },
          halfduplex: true, // 是否为半双工模式，默认为true
          fileList: [], // 文件列表，默认为空数组
          filename:"", // 文件名，默认为空
          radio:1, // 单选按钮，默认为1
          tableData:[], // 表格数据，默认为空数组
          newdata:{}, // 新数据对象，默认为空对象
          dialogFormVisible: false, // 对话框显示状态，默认为隐藏
          form: { // 表单数据对象
            hostnum: 0, // 主机数量，默认为0
            edgeports: 0, // 边缘端口数，默认为0
            edgebw: 0, // 边缘带宽，默认为0
            coreports: 0, // 核心端口数，默认为0
            corebw: 0, // 核心带宽，默认为0
            datacenters: [] // 数据中心列表，默认为空数组
          },
          formLabelWidth: '200px', // 表单标签宽度，默认为'200px'
          scoremsg:"", // 分数信息，默认为空字符串
  
          options: [ // 选项列表
             {
            optvalue: '2', // 选项值为2
            label: 'MAXMIN ' // 选项标签为MAXMIN
          }, {
            optvalue: '5', // 选项值为5
            label: 'HEFT ' // 选项标签为HEFT
          }, {
            optvalue: '6', // 选项值为6
            label: 'MIGRATE ' // 选项标签为MIGRATE
          }, {
            optvalue: '7', // 选项值为7
            label: 'K8S ' // 选项标签为K8S
          }],
          optvalue: '' // 选项值，默认为空字符串
        };
      },
      methods: {
        // 清空文件列表方法
        cleanFiles(){
          this.fileList = [];
        },
        // 处理当前行变化方法
        handleCurrentChange(row, event, column) {
          console.log(row, event, column, event.currentTarget)
        },
        // 处理编辑方法
        handleEdit(index, row) {
          console.log(index, row);
          console.log(this.gridData.at(0).app, this.gridData.at(0).host)
        },
        // 设置仿真时间方法
        handleSetSimulationTime(){
          console.log('setSimulationTime',this.lasttime);
          // 发送设置仿真时间的POST请求
          axios.post('http://localhost:8082/setsimulationtime', {
            "time":this.lasttime,
          }).then(response => {
            console.log(response.data.message);
          }).catch(error => {
            console.log(error);
          });
        },
        // 处理指定消息方法
        handleSpecifyMsg(){
          console.log('setSimulationTime',this.lasttime);
          // 发送指定消息的POST请求
          axios.post('http://localhost:8082/specifiedmsg', {
            "msgname":this.specifiedMsgName,
          }).then(response => {
            this.$message.info(response.data.message);
          }).catch(error => {
            console.log(error)
          });
        },
        // 处理指定链路方法
        handleSpecifyLink(){
          console.log('setSimulationTime',this.lasttime);
          // 发送指定链路的POST请求
          axios.post('http://localhost:8082/specifiedlink', {
            "linkname":this.specifiedLinkName,
          }).then(response => {
            this.$message.info(response.data.message);
          }).catch(error => {
            console.log(error)
          });
        },
        // 查看单个CPU方法
        paintSingleCPU(){
          console.log('paintSingleCPU', this.specifiedhostname, this.specifiedcpuid);
          // 发送查看单个CPU的POST请求
          axios.post('http://localhost:8082/specifiedCpu', {
            "hostname":this.specifiedhostname,
            "cpuid":this.specifiedcpuid,
          }).then(response => {
            this.$message.info(response.data.message);
          }).catch(error => {
            console.log(error)
          });
        },
        // 获取分配表方法
        getAssignTable(){
          // 发送获取分配表的POST请求
          axios.post('http://localhost:8082/getassign', this.gridData)
            .then(response => {
              console.log("拿到数据",response.data.message,response.data.data);
              this.gridData = JSON.parse(response.data.data);
            })
            .catch(error => {
              console.log(error);
            });
          this.dialogTableVisible = true; // 显示对话框
        },
        // 上传修改后的分配方法
        uploadModifyAssign(){
          // 发送上传修改后的分配表的POST请求
          axios.post('http://localhost:8082/modifyassign', this.gridData)
            .then(response => {
              // 发送静态仿真的POST请求
              axios.post('http://localhost:8082/staticSimulate', this.gridData)
                .then(response => {
                  // 显示成功消息
                  Message.success({
                    message: "修改成功",
                    duration: 500
                  });
                })
                .catch(error => {
                });
            })
            .catch(error => {
            });
        },
        // 设置暂停方法
        setPause() {
          console.log('setPause!',this.formInline);
          //
          // 发送设置暂停的POST请求
          axios.post('http://localhost:8082/pauseContainer', {
            "id":this.formInline.id,
            "start":this.formInline.start,
            "last":this.formInline.last,
          })
            .then(response => {
              console.log(response.data.message);
            })
            .catch(error => {
              console.log(error);
            });
        },
        // 取消暂停方法
        resetPause() {
          console.log('resetPause!',this.formInline);
          // 发送取消暂停的POST请求
          axios.post('http://localhost:8082/deletePause', {
            "id":this.formInline.id,
            "start":this.formInline.start,
            "last":this.formInline.last,
          })
            .then(response => {
              console.log(response.data.message);
            })
            .catch(error => {
              console.log(error);
            });
        },
        // 处理文件移除方法
        handleRemove(file, fileList) {
          console.log(file, fileList);
        },
        // 处理文件预览方法
        handlePreview(file) {
          console.log(file);
        },
        // 处理文件超出限制方法
        handleExceed(file, fileList) {
          this.$message.warning(`当前限制选择 1 个文件，本次选择了 ${file.length} 个文件，共选择了 ${file.length + fileList.length} 个文件`);
        },
        // 保存文件名方法
        saveName(response, file, fileList){
          this.filename=file.name;
          console.log(filelist);
        },
  
        // 运行方法
        async run(){
          var url= 'http://localhost:8082/run';
          console.log(url);
          // 发送运行的GET请求
          await axios.get(url)
            .then(response => {
            this.tableData=response.data.data;
          })
          .catch(error => {
            console.log(error);
          });
          this.fileList = []; // 清空文件列表
        },
  
        // 容器调度方法
        async placeContainer(){
          // 发送容器调度的POST请求
          await axios.post('http://localhost:8082/startSimulate',  {"arithmetic":this.optvalue})
              .then(response => {
                console.log(response.data.message);
                this.scoremsg = '容器分配成功<br>评分值:' + response.data.data.replace(/\n/g,'<br>');
                // 显示通知
                this.$notify({
                title: '提示',
                dangerouslyUseHTMLString: true,
                message: this.scoremsg,
                duration: 0
              });
                
              })
              .catch(error => {
                // 显示错误通知
                this.$notify({
                title: '提示',
                message: '容器分配出错，请查看日志',
                duration: 0
              });
              });
        },
  
        // 处理开关切换方法
        handleSwitch(switchstate){
          console.log("handleSwitch!", switchstate);
          // 发送半双工状态的POST请求
          axios.post('http://localhost:8082/halfduplex',  {switchstate})
            .then(response => {
              console.log(response.data.message);
            })
            .catch(error => {
              console.log(error);
            });
        },
        // 上传Host文件方法
        uploadHostFile(params) {
          const _file = params.file;
          const isOverride = this.checked == true ? 1 : 0
          const formData = new FormData();
          formData.append('file', _file);
          formData.append('isOverride', isOverride); // 自定义参数isOverride
          // 发送上传Host文件的POST请求
          axios.post('http://localhost:8082/uploadhost', formData)
            .then(response => {
              // 显示上传成功消息
              Message.success({
                message: "上传成功",
                duration: 1000
              });
            })
            .catch(error => {
              console.log(error);
            });
        },
  
        // 上传App文件方法
        uploadAppFile(params) {
          const _file = params.file;
          const isOverride = this.checked == true ? 1 : 0
          const formData = new FormData();
          formData.append('file', _file);
          formData.append('isOverride', isOverride); // 自定义参数isOverride
          // 发送上传App文件的POST请求
          axios.post('http://localhost:8082/uploadApp', formData)
            .then(response => {
              // 显示上传成功消息
              Message.success({
                message: "上传成功",
                duration: 1000
              });
            })
            .catch(error => {
              console.log(error);
            });
        },
  
      }
  }
</script>

  
  <!-- Add "scoped" attribute to limit CSS to this component only -->
  <!--<style src="../common/css/bodycss.css"></style>-->
  
  ```html
<!-- 定义 CSS 样式 -->
<style>
  /* 表单容器的样式 */
  .el-form{
    border: solid 1px #99a9bf;
  }
  /* 表单项的样式 */
  .el-form-item{
    margin-bottom: 0px;
    border: solid 1px #99a9bf;
    min-height: 40px;
  }
  /* 输入字段的样式 */
  .el-input{
    width: 100px;
    margin: 0px;
  }
  /* 对话框底部的样式 */
  .dialog-footer{
    margin-bottom: 10px;
    margin-top: 0px;
    //border: solid 2px black;
  }
  /* 按钮的样式 */
  .el-button{
    background-color: #99a9bf;
    color:revert;
    margin-top: 5px;
    margin-bottom: 5px;
  }
  /* 上传组件的样式 */
  .el-upload{
    display: flex;
    /* 改变主轴上项目的对齐方式 */
    justify-content: center;
  }
  /* 行的样式 */
  .el-row {
    margin-bottom: 20px;
    &:last-child {
      margin-bottom: 0;
    }
  }
  /* 列的样式 */
  .el-col {
    border-radius: 4px;
  }
  /* 深紫色背景的样式 */
  .bg-purple-dark {
    background: #99a9bf;
  }
  /* 紫色背景的样式 */
  .bg-purple {
    background: #d3dce6;
  }
  /* 浅紫色背景的样式 */
  .bg-purple-light {
    background: #e5e9f2;
  }
  /* 网格内容的样式 */
  .grid-content {
    border-radius: 4px;
    //min-height: 10%;
  }
  /* 行背景的样式 */
  .row-bg {
    padding: 10px 0;
    background-color: #f9fafc;
  }
  
  /* 上传演示的样式 */
  .upload-demo {
    margin-top: 5px;
    margin-bottom: 5px;
  }
  /* 自定义按钮的样式 */
  .button{
    margin-top: 10px;
    margin-bottom: 10px;
    background-color: white;
  }
  /* 容器样式 */
  #container {
    position: relative;
  }
  
  /* 按钮容器的样式 */
  .buttonstyle {
    display: flex;
    /* 改变主轴上项目的对齐方式 */
    justify-content: center;
    margin: 0px;
    //justify-content: space-evenly;
  }
  
  /* 图表提示框的样式 */
  .g6-tooltip {
      border-radius: 6px;
      font-size: 10px;
      color: #fff;
      background-color: #000;
      padding: 2px 8px;
      text-align: center;
  }
  /* 表格的样式 */
  .table {
    margin-left: 135px;
    margin-top: 25px;
  }
</style>
<!-- CSS 结束 -->
```