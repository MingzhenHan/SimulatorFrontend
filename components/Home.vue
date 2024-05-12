<template>
<div>

  <div class="grid-content">
    <t1>操作界面</t1>
  </div>
  <div class="grid-content bg-purple-light"style="border: solid 3px black;box-shadow: -10px 0px 5px black;">
          <el-alert style="radius: 20px"
            title="上传配置文件"
            type="success"
            :closable="false">
          </el-alert>

          <el-form>
            <el-form-item >
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

            <el-form-item >
              <div class="buttonstyle">
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

            <el-form-item>
              <el-form :inline="true" :model="formInline">
<!--                :model="formInline"-->
                <el-form-item label="容器">
                  <el-input v-model="formInline.id" placeholder="请输入容器"></el-input>
                </el-form-item>
                <el-form-item label="开始时间">
                  <el-input v-model="formInline.start" placeholder="请输入开始时间"></el-input>
                </el-form-item>
                <el-form-item label="结束时间">
                  <el-input v-model="formInline.last" placeholder="请输入结束时间"></el-input>
                </el-form-item>
                <br/>
                <el-form-item>
                  <el-button class="button" type="success" round @click="setPause">设置暂停</el-button>
                </el-form-item>
                <el-form-item>
                  <el-button class="button" type="success" round @click="resetPause">取消暂停</el-button>
                </el-form-item>
              </el-form>
            </el-form-item>

            <el-form-item >
              <span>{{"请选择调度算法"}}</span>
              <el-select v-model="optvalue" placeholder="" size="big">
                <el-option
                  v-for="item in options"
                  :key="item.optvalue"
                  :label="item.label"
                  :value="item.optvalue">
                </el-option>
              </el-select>
              <el-button class="button" type="success" round @click="placeContainer">调度容器</el-button>
            </el-form-item>

            <el-form-item>
              <el-button type="text" @click="getAssignTable">修改容器分配方案</el-button>

              <el-dialog title="动态修改容器分配" :visible.sync="dialogTableVisible" width="50%" @close="uploadModifyAssign">
                <el-table :data="gridData"  width="90%">
                  <el-table-column property="app" label="应用"></el-table-column>
                  <el-table-column label="主机" >
                    <template scope="scope">
                      <el-input size="small" v-model="scope.row.host" placeholder="请输入内容" @change="handleEdit(scope.$index, scope.row)"></el-input>
<!--                      <span>{{scope.row.name}}</span>-->
                    </template>
                  </el-table-column>
                  <el-table-column property="name" label="容器"></el-table-column>
                </el-table>
              </el-dialog>
            </el-form-item>

            <el-form-item >
              <div class="buttonstyle">
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

            <el-form-item>
              输入仿真持续时间(微秒)：<el-input style="width:100px;height:80%" type="number" v-model="lasttime" @change="handleSetSimulationTime()">
              </el-input>
            </el-form-item>

            <el-form-item >
              <el-switch
                v-model="halfduplex"
                active-color="white"
                inactive-color="lightskyblue"
                active-text="全双工"
                inactive-text="半双工"
                @change = "handleSwitch(halfduplex.valueOf())"
                style="margin-right: 20px"
              >
              </el-switch>
              <el-button class="button" type="success" round @click="run">开始仿真</el-button>
            </el-form-item>

            <el-form-item>
              输入要查看的消息名称：<el-input style="width:80%;height:80%" v-model="specifiedMsgName" @change="handleSpecifyMsg()">
              </el-input>
            </el-form-item>

            <el-form-item>
              输入要查看的链路名称：<el-input style="width:80%;height:80%" v-model="specifiedLinkName" @change="handleSpecifyLink()">
            </el-input>
            </el-form-item>

            <el-form-item>
              <span>{{"主机名称"}}</span>
              <el-input style="width:60%;height:80%" v-model="specifiedhostname"></el-input>
              <br>
              <!-- <span>{{"CPU序号"}}</span>
              <el-input style="width:80px;height:80%" v-model="specifiedcpuid" type="number"></el-input>
              <br> -->
              <el-button class="button" type="success" round @click="paintSingleCPU">查看单台主机</el-button>
            </el-form-item>

          </el-form>
<!--      <br>-->
    </div>



</div>
</template>

<script>
import G6 from '@antv/g6';
import axios from 'axios';
import {Message} from "element-ui";
import {data} from "autoprefixer";
// axios.defaults.baseURL = 'http://localhost:8082';
export default {
    data() {
      return {
        lasttime: 1000,
        specifiedMsgName: "",
        specifiedLinkName: "",
        specifiedhostname: "",
        specifiedcpuid: 0,
        gridData: [
          {
            app: "",
            name: "",
            host: "",
          },
        ],
        dialogTableVisible: false,
        formInline: {
          id: 1,
          start:  0.0,
          last: 0.0,
        },
        halfduplex: true,
        fileList: [],
        filename:"",
        radio:1,
        tableData:[],
        newdata:{},
        dialogFormVisible: false,
        form: {
          hostnum: 0,
          edgeports: 0,
          edgebw: 0,
          coreports: 0,
          corebw: 0,
          datacenters: []
        },
        formLabelWidth: '200px',
        scoremsg:"",

        options: [
           {
          optvalue: '2',
          label: 'MAXMIN '
        }, {
          optvalue: '5',
          label: 'HEFT '
        }, {
          optvalue: '6',
          label: 'MIGRATE '
        }, {
          optvalue: '7',
          label: 'K8S '
        }],
        optvalue: ''
      };
    },
    methods: {
      cleanFiles(){
        this.fileList = [];
      },
      handleCurrentChange(row, event, column) {
        console.log(row, event, column, event.currentTarget)
      },
      handleEdit(index, row) {
        console.log(index, row);
        console.log(this.gridData.at(0).app, this.gridData.at(0).host)
      },
      handleSetSimulationTime(){
        console.log('setSimulationTime',this.lasttime);
        axios.post('http://localhost:8082/setsimulationtime', {
          "time":this.lasttime,
        }).then(response => {
          console.log(response.data.message);
        }).catch(error => {
          console.log(error);
        });
      },
      handleSpecifyMsg(){
        console.log('setSimulationTime',this.lasttime);
        axios.post('http://localhost:8082/specifiedmsg', {
          "msgname":this.specifiedMsgName,
        }).then(response => {
          this.$message.info(response.data.message);
        }).catch(error => {
          console.log(error)
        });
      },
      handleSpecifyLink(){
        console.log('setSimulationTime',this.lasttime);
        axios.post('http://localhost:8082/specifiedlink', {
          "linkname":this.specifiedLinkName,
        }).then(response => {
          this.$message.info(response.data.message);
        }).catch(error => {
          console.log(error)
        });
      },
      paintSingleCPU(){
        console.log('paintSingleCPU', this.specifiedhostname, this.specifiedcpuid);
        axios.post('http://localhost:8082/specifiedCpu', {
          "hostname":this.specifiedhostname,
          "cpuid":this.specifiedcpuid,
        }).then(response => {
          this.$message.info(response.data.message);
        }).catch(error => {
          console.log(error)
        });
      },
      getAssignTable(){
        axios.post('http://localhost:8082/getassign', this.gridData)
          .then(response => {
            console.log("拿到数据",response.data.message,response.data.data);
            this.gridData = JSON.parse(response.data.data);
            // console.log("data第一行为：",this.gridData.at(0).app, this.gridData.at(0).host)
          })
          .catch(error => {
            console.log(error);
          });
        this.dialogTableVisible = true;
      },
      uploadModifyAssign(){
        axios.post('http://localhost:8082/modifyassign', this.gridData)
          .then(response => {
            axios.post('http://localhost:8082/staticSimulate', this.gridData)
              .then(response => {
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

        // axios.post('http://localhost:8082/staticSimulate', this.gridData)
        //   .then(response => {
        //     Message.success({
        //       message: "修改成功",
        //       duration: 500
        //     });
        //   })
        //   .catch(error => {
        //   });
      },
      setPause() {
        console.log('setPause!',this.formInline);
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
      resetPause() {
        console.log('resetPause!',this.formInline);
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
      handleRemove(file, fileList) {
        console.log(file, fileList);
      },
      handlePreview(file) {
        console.log(file);
      },
      handleExceed(file, fileList) {
        this.$message.warning(`当前限制选择 1 个文件，本次选择了 ${file.length} 个文件，共选择了 ${file.length + fileList.length} 个文件`);
      },
      // beforeRemove(file, fileList) {
      //   return this.$confirm(`确定移除 ${ file.name }？`);
      // },
      saveName(response, file, fileList){
        this.filename=file.name;
        console.log(filelist);
      },

      async run(){
        // console.log(this.filename);
        var url= 'http://localhost:8082/run';
        console.log(url);
        await axios.get(url)
          .then(response => {
          this.tableData=response.data.data;
        })
        .catch(error => {
          console.log(error);
        });
        this.fileList = [];
      },

      async placeContainer(){
        await axios.post('http://localhost:8082/startSimulate',  {"arithmetic":this.optvalue})
            .then(response => {
              console.log(response.data.message);
              this.scoremsg = '容器分配成功<br>评分值:' + response.data.data.replace(/\n/g,'<br>');
              this.$notify({
              title: '提示',
              dangerouslyUseHTMLString: true,
              message: this.scoremsg,
              duration: 0
            });
              
            })
            .catch(error => {
              this.$notify({
              title: '提示',
              message: '容器分配出错，请查看日志',
              duration: 0
            });
            });
      },

      handleSwitch(switchstate){
        console.log("handleSwitch!", switchstate);
        axios.post('http://localhost:8082/halfduplex',  {switchstate})
          .then(response => {
            console.log(response.data.message);
          })
          .catch(error => {
            console.log(error);
          });
      },
      uploadHostFile(params) {
        const _file = params.file;
        const isOverride = this.checked == true ? 1 : 0
        const formData = new FormData();
        formData.append('file', _file);
        formData.append('isOverride', isOverride); // isOverride自定义的其他参数
        axios.post('http://localhost:8082/uploadhost', formData)
          .then(response => {
            Message.success({
              message: "上传成功",
              duration: 1000
            });
          })
          .catch(error => {
            console.log(error);
          });
      },

      uploadAppFile(params) {
        const _file = params.file;
        const isOverride = this.checked == true ? 1 : 0
        const formData = new FormData();
        formData.append('file', _file);
        formData.append('isOverride', isOverride); // isOverride自定义的其他参数
        axios.post('http://localhost:8082/uploadApp', formData)
          .then(response => {
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

<style>
.el-form{
  border: solid 1px #99a9bf;
}
.el-form-item{
  margin-bottom: 0px;
  border: solid 1px #99a9bf;
  min-height: 40px;
}
.el-input{
  width: 100px;
  margin: 0px;
}
.dialog-footer{
  margin-bottom: 10px;
  margin-top: 0px;
  //border: solid 2px black;
}
.el-button{
  background-color: #99a9bf;
  color:revert;
  margin-top: 5px;
  margin-bottom: 5px;
}
.el-upload{
  display: flex;
  /* 改变item在主轴上的对齐方式 */
  justify-content: center;
}
.el-row {
  margin-bottom: 20px;
  &:last-child {
    margin-bottom: 0;
  }
}
.el-col {
  border-radius: 4px;
}
.bg-purple-dark {
  background: #99a9bf;
}
.bg-purple {
  background: #d3dce6;
}
.bg-purple-light {
  background: #e5e9f2;
}
.grid-content {
  border-radius: 4px;
  //min-height: 10%;
}
.row-bg {
  padding: 10px 0;
  background-color: #f9fafc;
}


.upload-demo {
  margin-top: 5px;
  margin-bottom: 5px;
}
.button{
  margin-top: 10px;
  margin-bottom: 10px;
  background-color: white;
}
#container {
  position: relative;
}

.buttonstyle {
  display: flex;
  /* 改变item在主轴上的对齐方式 */
  justify-content: center;
  margin: 0px;
  //justify-content: space-evenly;
}

.g6-tooltip {
    border-radius: 6px;
    font-size: 10px;
    color: #fff;
    background-color: #000;
    padding: 2px 8px;
    text-align: center;
}
.table {
  margin-left: 135px;
  margin-top: 25px;
}
</style>
