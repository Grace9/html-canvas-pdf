<template>
  <div class="flowTemp">
    <!-- 左边菜单 -->
    <div class="costTypeLeft">
      <div class="costTypeTitle">
        <span>审批流程</span>
        <div class="addBtnDiv">
          <el-button type="primary" size="mini" @click="addTemplate">添加</el-button>
        </div>
      </div>
       <div class="treeDiv">
         <div class="searchDiv">
          <el-input placeholder="请输入流程名称" v-model="searchContext" class="input-with-select" size="small" clearable >
            <el-button slot="append" icon="el-icon-search" @click="getFlowData"></el-button>
          </el-input>
        </div>
        <div class="tree">
          <div class=".el-table" style="margin-top: 8px">
            <div class="el-table__body-wrapper is-scrolling-none">
              <table cellspacing="0" cellpadding="0" border="0" class="el-table__body">
                <colgroup><col name="el-table_3_column_12" width="298"></colgroup>
                <tbody>
                  <tr :class="currentTempIndex==index ? 'el-table__row current-row': 'el-table__row'" v-for="(item,index) in tempList" :key="index">
                    <td class="el-table_5_column_14" @click="getCurrentTemple(index)">
                      <div class="tree-item">
                        <span :title="item.flowTemplateName">{{item.flowTemplateName}}</span>
                        <i data-v-a8dfd0ac="" class="el-icon-circle-close-outline" @click.stop="deleteFlowTemplate(item)"></i>
                      </div>
                    </td>
                  </tr>
                </tbody>
              </table>
              </div>
          </div>
        </div>
        <!-- 底部操作 -->
        <div class="switchDiv">
          <el-checkbox v-model="isDelete" class="left">显示停用</el-checkbox>
          <!-- <div class="right">
            <el-button plain size="mini">导 入</el-button>
            <el-button plain size="mini">导 出</el-button>
          </div>-->
        </div>
      </div>

    </div>
    <!-- 右边内容 -->
    <div class="costTypeRight">
      <fullscreen ref="fullscreen" @change="fullscreenChange">
      <el-tabs v-model="activeName"  @before-leave="changeTab">
        <!-- 1基本设置 -->
        <el-tab-pane label="基本设置" name="first">
          <div class="custom-fieldSetting">
            <div class="setting-content">
              <el-form label-width="120px" :model="template" :rules="rules" ref="customSetting" >
                <el-form-item label="流程模板名称：" prop="flowTemplateName">
                  <el-input v-model="template.flowTemplateName" maxlength="64" class="width80"></el-input>
                  <!-- <div class="remarks">流程模板名称不可重名，包括已停用流程模板。</div> -->
                </el-form-item>
                <el-form-item label="流程模板ID:" prop="flowTemplateId">
                  <el-input v-model="template.flowTemplateId" class="width80" disabled=""></el-input>
                </el-form-item>
                <el-form-item label="用途说明：" >
                  <el-input type="textarea" v-model="template.flowTemplateDesc" maxlength="128" class="width80"></el-input>
                </el-form-item>
              </el-form>
            </div>
            <!-- 底部操作 -->
            <div class="setting-oper">
              <el-button type="primary" size="small" @click="saveAllData">保存</el-button>
              <el-button type="primary" size="small" >停用</el-button>
              <el-button type="primary" size="small" @click="toggleFull">{{fullscreen ? '关闭全屏' : '全屏'}}</el-button>
            </div>
          </div>
        </el-tab-pane>
        <!-- 2流程设置 -->
        <el-tab-pane label="流程设置" name="two">
          <div class="custom-fieldSetting">
            <div class="setting-content">
              <div class="setting-content-child">
                <!-- 左侧发起审批 -->
                <div class="content-left">
                  <div v-for="(item,index) in nodeList" :key="index">
                    <div class="flow-item" :style="item.nodeCode=='start_node' ? 'margin-top: 20px;' : ''">
                      <div class="item-node" :class="currNodeIndex == index ? 'curr-item-node' : '' " v-if="item.nodeCode=='start_node'" @click="clickNode(index)">
                        <span>发起审批</span>
                      </div>
                    </div>
                    <!-- 动态添加 -->
                    <el-row v-if="item.nodeCode!='start_node' && item.nodeCode!='end_node'">
                      <div class="item-node-add">
                        <span @click="andNode(index)">+</span>
                      </div>
                      <div class="flow-item">
                        <div class="item-node-middle" :class="currNodeIndex == index ? 'curr-item-node' : '' " @click="clickNode(index)">
                          <span>{{item.nodeName}}</span>
                          <i class="el-icon-circle-close-outline icon" @click.stop="deleteNode(index)"></i>
                        </div>
                      </div>
                    </el-row>
                    <el-row v-if="item.nodeCode=='end_node'">
                      <div class="item-node-add">
                        <span @click="andNode(index)">+</span>
                      </div>
                      <div class="flow-item">
                        <div class="item-node" :class="currNodeIndex == index ? 'curr-item-node' : '' "  @click="clickNode(index)">
                          <span>结束审批</span>
                        </div>
                      </div>
                    </el-row>
                  </div>
                </div>
                <!-- 右侧节点展示 -->
                <div class="content-right">
                  <el-form label-width="120px" :model="currNode" :rules="rules" ref="flowSettingRef">
                    <el-form-item label="节点名称：" prop="nodeName">
                      <el-input
                      v-model="currNode.nodeName"
                      maxlength="20"
                      class="width80"
                      :disabled="currNode.nodeCode == 'start_node' || currNode.nodeCode == 'end_node'">
                      </el-input>
                    </el-form-item>
                    <!-- 非开始和结束节点 -->
                    <el-row v-if="currNode.nodeCode != 'start_node' && currNode.nodeCode != 'end_node'">
                      <el-form-item label="环节名称："  prop="flowCode">
                        <el-select v-model="currNode.flowCode" placeholder="请选择" class="width80">
                          <el-option
                            v-for="item in stepList"
                            :key="item.value"
                            :label="item.label"
                            :value="item.value"
                          ></el-option>
                        </el-select>
                      </el-form-item>

                      <el-form-item label="设置审批人：" prop="nodeRuleType" >
                      <!--  <el-radio :label="3">发起人自选</el-radio> -->
                        <!-- <el-radio label="1" v-model="currNode.nodeRuleType">发起人自选</el-radio> -->
                        <el-radio-group v-model="currNode.nodeRuleType" @change="nodeRuleTypeChange">
                          <el-radio :label="'2'">部门主管</el-radio>
                          <el-radio :label="'3'">外部流程引擎</el-radio>
                        </el-radio-group>
                      </el-form-item>

                      <el-form-item label="审批人为空时：" v-if="currNode.nodeRuleType == '2'">
                        <!--  <div style="text-align: left"><el-radio disable="true" label="1" v-model="nodeRule.nodeRuleDeal">自动通过</el-radio> </div>
                        <div style="text-align: left"><el-radio label="2" disable="true" v-model="nodeRule.nodeRuleDeal">由通讯录上部门主管的上级主管代审批</el-radio> </div>-->
                        <el-radio-group v-model="currNode.nodeRuleType2">
                          <el-radio :label="'3'">不允许发起人提交</el-radio>
                        </el-radio-group>
                      </el-form-item>

                      <el-form-item label="外部流程系统：" v-if="currNode.nodeRuleType == '3'">
                        <el-select v-model="currNode.nodeRuleType3" placeholder="请选择" class="width80">
                          <el-option
                            v-for="item in flowSystemList"
                            :key="item.processId"
                            :label="item.systemName"
                            :value="item.processId"
                          ></el-option>
                        </el-select>
                        <el-row>
                          <a @click="openOtherSysFlowPage" v-if= "currNode.nodeRuleType == '3'" style="cursor:pointer;text-decoration: underline">点击设置外部流程系统</a>
                        </el-row>
                      </el-form-item>
                    </el-row>
                  </el-form>
                </div>
              </div>
            </div>
            <!-- 底部操作行 -->
            <div class="setting-oper">
              <el-button type="primary" size="small" @click="saveAllData" >保存</el-button>
              <el-button type="primary" size="small" >停用</el-button>
              <el-button type="primary" size="small" @click="toggleFull">{{fullscreen ? '关闭全屏' : '全屏'}}</el-button>
            </div>
          </div>
        </el-tab-pane>

      </el-tabs>
      </fullscreen>
    </div>
    <!-- 添加流程模板弹框 -->
    <el-dialog title="添加流程模板" :visible.sync="addDialogShow" width="45%">
          <el-form label-width="120px" :model="addTemplateData" :rules="rules" ref="addCustomSetting">
            <el-form-item label="流程模板名称：" prop="flowTemplateName">
              <el-input size="small" v-model="addTemplateData.flowTemplateName" maxlength="64" class="width80"></el-input>
              <!-- <div class="remarks">流程模板名称不可重名，包括已停用流程模板。</div> -->
            </el-form-item>
            <!-- 暂时不显示 -->
            <!-- <el-form-item label="流程模板ID:">
              <el-input
                v-model="addTemplateData.flowTemplateId"
                placeholder="由系统后台自动生成"
                size="small"
                class="width80"
                :disabled="true"
              ></el-input>
            </el-form-item> -->
            <el-form-item label="用途说明：">
              <el-input size="small" type="textarea" maxlength="128" v-model="addTemplateData.flowTemplateDesc" class="width80"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button size="small" @click="addDialogShow = false">取 消</el-button>
            <el-button size="small" type="primary" @click="saveTemplate('add')">保 存</el-button>
          </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    var checkRepeatValue = function(value, list, index) {
      let checkValue = false
        for(let i = 0; i< list.length; i++) {
          let tmp =list[i]
          if(tmp.flowTemplateName == value && i != index) {
            checkValue = true
          }
        }
        return checkValue
    };
    var checkTmpName = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('请输入模板名称'));
        } else {
          let index = this.addDialogShow ? null : this.currentTempIndex
          let checkValue = checkRepeatValue(value, this.tempList, index)
          if(checkValue) {
            return callback(new Error('流程模板名称不可重名，包括已停用流程模板。'));
          } else {
            return callback();
          }
        }
    };
    return {
      addNodeSaveStatus: true, // 新增节点是否保存
      fullscreen: false,
      template: {}, //模板对象
      addTemplateData: {
        flowTemplateName: '',
        flowTemplateId: '',
        flowTemplateDesc: '',
      },
      rules: {
        nodeName: [
          { required: true, message: "请输入节点名称", trigger: "blur" }
        ],
        flowCode: [
          { required: true, message: "请输入环节名称", trigger: "blur" }
        ],
        nodeRuleType: [
          { required: true, message: "请设置审批人", trigger: "blur" }
        ],
        flowTemplateName: [
          { required: true, validator: checkTmpName, trigger: 'blur' }
        ],
        flowTemplateId: [
          { required: true, message: "请输入流程模板ID", trigger: "blur" }
        ],
        flowTemplateDesc: [
          { required: true, message: "请输入编码", trigger: "blur" }
        ]
      },
      currentTempIndex: 0, //当前选中的流程模板
      addDialogShow: false,
      nodeRule: {"nodeRuleId":''},   //当前节点规则
      addTemplateShow: false,
      tempList: [],
      nodeList: [], //当前流程的节点列表
      nodeArr: [{ nodeCode: "", nodeName: "" }],
      stepList: [{value:'new_process',label:'单据编辑'},{value:'business_process',label:'业务审批'},{value:'print_process',label:'打印环节'},{value:'financial_process',label:'财务审批'}],   //环节列表
      currNodeIndex: 0, //当前正在编辑的节点
      currNode: {templateNodeId:'', nodeRuleType2: '1'},   //当前节点
      flowSystemList: [],
      searchContext: "",
      index: "",
      formId: "",
      formType: "",
      defaultFileds: {},
      fileds: [],
      formEntity: {},
      tree: [],
      firstTree: [],
      defaultProps: {
        children: "children",
        label: "formName",
        id: "formId"
      },
      parentFormId: "",
      isDelete: false, //是否显示停用

      activeName: "first",
      ruleForm: {},
      fileds: [],
      formId: "",
      formType: "",
      checkItem: {
        name: "",
        name2: "",
        costId: ""
      },
      dialogForm: { formType: "" },
      dialogFormVisible: false,
      showAddfield: false, //添加字段弹窗
      fieldList: [], //字段列表
      showCustom: false, //是否为自定义列表
      customForm: {
        //自定义弹窗表单
        filedType: "",
        filedLable: "",
        filedRef: ""
      },
      filedRefShow: false
    };
  },
  mounted() {
    this.getFlowData();
    // this.addCostType();
  },
  methods: {
    toggleFull () {
      this.$refs['fullscreen'].toggle() // recommended
      // this.fullscreen = !this.fullscreen // deprecated
    },
    fullscreenChange (fullscreen) {
      this.fullscreen = fullscreen
    },
    saveAllData() {
      // 基本设置
      let customValid = false
      let flowValid = false
      let _this = this
      this.$refs['customSetting'].validate((valid) => {
        if (valid) {
          customValid = true
        } else {
          _this.activeName = 'first'
          return false
        }
      })
      //流程设置
      this.$refs['flowSettingRef'].validate((valid) => {
        if (valid) {
          flowValid = true
        } else {
          _this.activeName = 'two'
          return false
        }
        if(customValid && flowValid) {
          _this.saveTemplate('template')
          _this.saveCurrNode()
        }
      })

    },
    // 新增或更新模板
    saveTemplate(type) {
      let formName = (type == 'add') ? "addCustomSetting" : "customSetting"
      this.$refs[formName].validate((valid) => {
          if (valid) { //表单校验通过
            let params = {}
            let _this = this
            if(type == 'add') { // 新增模板
              params = JSON.parse(JSON.stringify(_this.addTemplateData))
              params.bootUrl = "/ers-expense/addErsFlowTemplateCfg"
            } else {
              params = JSON.parse(JSON.stringify(_this.template))
              params.bootUrl = "/ers-expense/updateErsFlowTemplateCfg"
            }
            this.$httpExt()
              .post("/approvalData", params)
              .then(
                function(res) {
                  _this.addDialogShow = false;
                  if(type == 'add') {
                    _this.currentTempIndex = 0
                    _this.currNodeIndex = 0
                  }
                  _this.getFlowData();
                },
                function(res) {
                  _this.$message({
                    message: res.msg,
                    type: "warning"
                  });
                }
              );

          } else {  //表单校验未通过
            return false;
          }
      });
    },
    // 保存当前节点信息
    saveCurrNode() {
      this.$refs['flowSettingRef'].validate((valid) => {
        if (valid) { //表单校验通过
          var that = this;
          if(this.currNode.nodeCode == 'start_node' || this.currNode.nodeCode == 'end_node') {
            console.log('流程设置开始节点或结束节点不提交保存')
            return false
          }
          var params = JSON.parse(JSON.stringify(this.currNode));
          params.flowTemplateId = this.template.flowTemplateId;
          if(!this.currNode.templateNodeId) {
            params.bootUrl = "/ers-expense/addErsflowTemplateNodeCfg";
          }else{
            params.bootUrl = "/ers-expense/updateErsflowTemplateNodeCfg";
          }

          this.$httpExt()
            .post("/approvalData", params)
            .then(
              function(res) {
                that.nodeRule.templateNodeId = res.msg;
                that.nodeRule.nodeRuleType = that.currNode.nodeRuleType
                that.nodeRule.nodeRuleDeal = that.currNode['nodeRuleType' + that.currNode.nodeRuleType]
                that.saveNodeRule()
              },
              function(res) {
                that.$message({
                  message: res.msg,
                  type: "warning"
                });
              }
            );
        } else {
          return false
        }
      })

    },
    // 保存流程规则
    saveNodeRule(nodeRule) {
      var that = this
      var params = JSON.parse(JSON.stringify(this.nodeRule));
      if(!this.nodeRule.nodeRuleId) {
        params.bootUrl = "/ers-expense/saveErsflowTemplateNodeRuleCfg";
      }else{
        params.bootUrl = "/ers-expense/updateErsflowTemplateNodeRuleCfg";
      }
      this.$httpExt()
        .post("/approvalData", params)
        .then(
          function(res) {
            that.addNodeSaveStatus = true
            that.$message({
              message:"保存成功",
              type: "success"
            });
          },
          function(res) {
            that.$message({
              message: res.msg,
              type: "warning"
            });
          }
        );
    },
    // 设置审批人切换事件
    nodeRuleTypeChange(value) {
      if(value == '2') {
         this.currNode = Object.assign({},this.currNode, {nodeRuleType2: '3'})
      } else {
        this.currNode = Object.assign({},this.currNode, {nodeRuleType3: this.flowSystemList[0].processId})
      }
    },
    changeTab(tab, event) {
      console.log(tab, event);
      if(!this.template.flowTemplateId && this.activeName == 'two') {
        this.activeName = 'first';
        this.$message({
          message: "请先设置流程模板",
          type: "warning"
        });
        return false;
      }
    },
    // 删除模板
    deleteFlowTemplate(row) {
      this.$confirm("此操作将永久删除, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$httpExt()
            .post("/approvalData", {
              templateId: row.templateId,
              flowTemplateId: row.flowTemplateId,
              bootUrl: "/ers-expense/deleteErsFlowTemplateCfg"
            })
            .then(
              response => {
                this.getFlowData();
                this.$message({
                  type: "success",
                  message: "删除成功!"
                });
              },
              response => {
              	if('exists' ==response.msg){
              		this.$message({
	                  type: "warning",
	                  message: "已被单据关联的审批流程不允许删除。"
	                });
              	}else{
              		Message({
	                  message: "服务器请求失败，请重试！",
	                  type: "warning"
	                });
              	}
              })
        })
    },
    // 删除节点
    deleteNode(index) {
      let node = this.nodeList[index]
      var that = this
      console.log(node.flowTemplateId)
      if(!node.flowTemplateId) { // 新增节点
        that.nodeList.splice(index,1)
        that.addNodeSaveStatus = true
      } else { // 删除已保存节点
        let hasSaveNum = that.nodeList.filter((item) => {
          return item.flowTemplateId
        })
        console.log(hasSaveNum.length)
        if(hasSaveNum.length <= 3) {
          that.$message({message: "已保存审批节点数量至少为一个", type: "warning"})
          return false
        }
        var params = JSON.parse(JSON.stringify(this.nodeList[index]));
        params.bootUrl = "/ers-expense/deleteErsflowTemplateNodeCfg";
        this.$httpExt().post("/approvalData", params).then(function(res) {
            that.nodeList.splice(index,1);
          },function(res) {
            that.$message({ message: res.msg, type: "warning" });
          }
        );
      }
    },
    // 新增节点
    andNode(index) {
      if(!this.addNodeSaveStatus) {
        this.$message({ message:"存在未保存的新增节点", type: "warning"})
        return false
      }
      this.addNodeSaveStatus = false
      this.currNodeIndex = index;
      var node = {
        nodeName: "新增审批节点",
        flowCode: this.stepList[0].value || '',
        nodeRuleType: '2',
        nodeRuleType2: '3',
      };
      node.preNodeCode = this.nodeList[index-1].nodeCode;
      node.nodeSort = this.nodeList[index-1].nodeSort + 1;
      this.nodeList.splice(index, 0, node);
      for(var i= index+1;i<this.nodeList.length-1;i++) {
        this.nodeList[i]['nodeSort'] = this.nodeList[i]['nodeSort'] + 1;
      }
      this.clickNode(index);
    },
    // 根据模板获取节点信息
    getNodeList() {
      var params = {};
      var that = this;
      params.flowTemplateId = this.template.flowTemplateId;
      params.bootUrl = "/ers-expense/getErsFlowTemplateNodeCfgList";
      this.$httpExt()
        .post("/approvalData", params)
        .then(
          function(res) {
            that.nodeList = res.result.list;
            that.clickNode(that.currNodeIndex);   //默认选中第一个节点
          },
          function(res) {
            that.$message({
              message: res.msg,
              type: "warning"
            });
          }
        );
    },
    // 添加模板
    addTemplate() {
      this.addDialogShow = true;
    },
    // 点击节点
    clickNode(index) {
      this.currNode = JSON.parse(JSON.stringify(this.nodeList[index]))
      this.nodeRule = {templateNodeId: this.currNode.templateNodeId}
      if(!this.currNode.nodeRuleType) {
        this.currNode.nodeRuleType = '2'
        this.currNode.nodeRuleType2 = '3'
      }
      this.currNodeIndex = index;
      this.getNodeRule(this.currNode.templateNodeId);
      this.getOtherSys();
    },
    // 获取费用树形结构
    append(data) {
      this.parentFormId = data.formId;
      this.addCostTypeFn();
    },
    //新建单据模板
    add() {
      this.formId = "";
      this.dialogFormVisible = false;
      this.formType = this.dialogForm.formType;
      this.get();
    },
    getId() {},
    // 获取流程模板数据
    getFlowData() {
      var that = this;
      this.$httpExt()
        .post("/approvalData", {
          flowTemplateName: this.searchContext,
          bootUrl: "/ers-expense/flow-template/getAllFlowTemplates"
        })
        .then(
          function(res) {
            that.tempList = res.result;
            that.getCurrentTemple(that.currentTempIndex); //默认选中第一个
          },
          function(res) {
            that.$message({
              message: res.msg,
              type: "warning"
            });
          }
        );
    },
    //添加字段弹窗
    addfield() {
      this.showAddfield = true;
    },
    //新建自定义字段
    addCustom() {
      this.showCustom = true;
    },
    getCurrentTemple(index) {
      this.currentTempIndex = index;
      this.template = JSON.parse(JSON.stringify(this.tempList[index]))
      // this.currNodeIndex = 0;
      this.nodeRule = {};
      this.getNodeList();
    },
    //保存字段自定义字段
    saveFlowTemplateCfg(formName) {
      var that = this;
      this.$refs[formName].validate(valid => {
        valid = true;
        if (valid) {
          var params = JSON.parse(JSON.stringify(this.template));
          params.bootUrl = "/ers-expense/addErsFlowTemplateCfg";

          this.$httpExt()
            .post("/form_expense", params)
            .then(
              function(res) {
                that.$message({
                  message: "保存成功",
                  type: "success"
                });

                that.getFlowData();
              },
              function(res) {
                that.$message({
                  message: res.msg,
                  type: "warning"
                });
              }
            );
        } else {
          return false;
        }
      });
    },
    openOtherSysFlowPage() {
      this.$emit("page", '外部流程系统', "otherSystem");
    },
    getOtherSys() {
      var params = {};
      params.pageSize = "100";
      params.currentPage = "0";
      params.bootUrl = "/ers-expense/getOtherFlowCfgList";

      /*查询接口*/
      this.$httpExt()
        .post("/approvalData", params)
        .then(
          response => {
            this.flowSystemList = response.result;
          },
          response => {
            Message({
              message: "服务器请求失败，请重试！",
              type: "warning"
            });
          }
        );
    },
    // 查询节点规则
    getNodeRule(templateNodeId) {
      if(!templateNodeId) {
        return;
      }
      var that = this;
      var params = {templateNodeId: templateNodeId};
      params.bootUrl = "/ers-expense/getErsFlowTemplateNodeRuleCfgList";
      this.$httpExt()
        .post("/approvalData", params)
        .then(
          response => {
            let data = response.result.list[0]
            that.nodeRule = data
            if(data) {
              that.currNode.nodeRuleType = data.nodeRuleType
              let nodeRuleDeal = data.nodeRuleDeal
              if(!data.nodeRuleType) {
                that.currNode.nodeRuleType = '2'
                that.currNode.nodeRuleType2 = '3'
              } else {
                if(data.nodeRuleType == '2') {
                  nodeRuleDeal = data.nodeRuleDeal ? data.nodeRuleDeal : '3'
                  that.currNode = Object.assign({},that.currNode, {nodeRuleType2: nodeRuleDeal})
                } else {
                	// 外部流程Id
                	nodeRuleDeal = nodeRuleDeal ? nodeRuleDeal : '1';
                  that.currNode = Object.assign({},that.currNode, {nodeRuleType3: parseInt(nodeRuleDeal)})
                }
              }
            }
            console.log(that.currNode)
          },
          response => {
            Message({
              message: "服务器请求失败，请重试！",
              type: "warning"
            });
          }
        );
    }

  }
};
</script>
<!--
// 防止less预处理
<style scoped>
  .flowTemp {
    height: calc(100vh - 117px);

  .costTypeRight {
    width: calc(100% - 310px);
  }
  .treeDiv {
    height: calc(100vh - 159px);
  }
  .custom-fieldSetting {
    height: calc(100vh - 171px);
  }
  .setting-content-child {
    height: calc(100vh - 212px);
  }
</style>
-->
<style lang="less" scoped>
// 全屏样式 
/deep/ .fullscreen {
  background-color: #ffffff !important;
  .custom-fieldSetting {
    height: calc(~"100vh - 55px") !important;
  }
  .setting-content-child {
    height: calc(~"100vh - 98px") !important;
  }
}
.flowTemp {
  height: calc(~"100vh - 117px"); //防止less预处理
  background-color: #F1F2F7;
  // 左侧样式
  .costTypeLeft {
    float: left;
    height: 100%;
    width: 300px;
    margin-right: 4px;
    background: white;
    .costTypeTitle {
      line-height: 36px;
      border: 1px solid #c5c5c5;
      padding: 0 15px;
      font-size: 14px;
      font-weight: 600;
      .addBtnDiv {
        display: flex;
        height: 36px;
        align-items: center;
        float: right;
      }
    }
    .treeDiv {
      height: calc(~"100vh - 159px");
      border: 1px solid #c5c5c5;
      margin-top: 4px;
      display: flex;
      flex-direction: column;
        .searchDiv {
          padding: 10px 15px 15px 15px;
          /deep/ .el-input--small {
            font-size: 12px;
          }
        }
        .tree {
          flex: 1;
          overflow: auto;
          .el-table_5_column_14 {
              padding: 5px 10px;
              cursor: pointer;
              border-bottom: 1px solid #ebeef5;
          }
          .tree-item {
            padding: 8px;
            position: relative;
            > span {
              color: #409EFF;
              font-size: 12px;
              font-weight: 600;
              width: 240px;
              display: inline-block;
              white-space: nowrap;
              overflow: hidden;
              text-overflow: ellipsis;
            }
            .el-icon-circle-close-outline {
              font-size: 16px;
              position: absolute;
              top: 25%;
              right: -8px;
            }
            :after {
              clear: both;
              content: "";
            }
          }
        }
        .switchDiv {
          border-top: 1px solid #c5c5c5;
          line-height: 40px;
          padding: 0 15px;
        }
    }
  }
  //右侧内容样式
  .costTypeRight {
    width: calc(~"100% - 310px");
    height: 100%;
    margin-left: 304px;
    border: 1px solid #c5c5c5;
    overflow: auto;
    background-color: #ffffff !important;
    /deep/ .el-tabs__content{
      background-color: #ffffff !important;
      padding: 0px;
    }
    .custom-fieldSetting {
      height: calc(~"100vh - 171px");
      display: flex;
      flex-direction: column;
      .setting-content {
        flex: 1;
        overflow: auto;
        padding: 0px 10px;
        // 流程设置样式
        .setting-content-child {
          height: calc(~"100vh - 212px");
          display: flex;
          flex-direction: row;
          font-size: 12px;
          .content-left {
            flex: 4;
            overflow: auto;
            border: 1px solid #c5c5c5;
            margin-right: 5px;
            margin-bottom: 10px;
            .flow-item {
              display: flex;
              align-content: center;
              justify-content: center;
              .item-node {
                cursor: pointer;
                width: 100px;
                text-align: center;
                background: #72A2D4;
                border: 2px solid #72A2D4;
                padding: 10px;
                border-radius: 20px;
                font-weight: 600;
                color: #fff;
              }
              .curr-item-node {
                border-color:#F56C6C !important;
              }
              .item-node-middle {
                cursor: pointer;
                width: 180px;
                text-align: center;
                background: #72A2D4;
                border: 2px solid #72A2D4;
                padding: 15px;
                font-weight: 600;
                color: #fff;
                position: relative;
                .icon {
                  position: absolute;
                  top: -15px;
                  right: -20px;
                  z-index: 1;
                }
              }

            }
            .item-node-add {
                  text-align: center;
                  margin: 5px 0;
                  cursor: pointer;
                  > span {
                    font-size: 20px;
                    font-weight: 600;
                    color:#409EFF;
                  }
                }
          }
          .content-right {
            flex: 7;
            overflow: auto;
            border: 1px solid #c5c5c5;
            margin-left: 5px;
            margin-bottom: 10px;
            padding: 0px 10px;
            > form {
              margin-top: 20px;
            }
          }
        }
      }
      // 底部操作行样式
      .setting-oper {
        border-top: 1px solid #c5c5c5;
        line-height: 40px;
        padding: 0 15px;
      }
    }
  }
  /deep/ .el-form-item__label, /deep/ .el-input, /deep/ .el-textarea, /deep/ .el-radio__label,
  /deep/ .el-select {
    font-size: 12px;
  }
  .el-icon-circle-close-outline {
    // float: right;
     font-size: 20px;
    color: rgb(245, 108, 108);
  }
  :after {
    clear: both;
    content: "";
  }
}
.remarks {
  font-size: 12px;
  color: #b2b2b2;
  line-height: 24px;
  text-align: left;
}
.width80 {
  width: 80%;
}
.el-checkbox + .el-checkbox {
  margin-left: 0px;
  margin-right: 30px;
}
.el-checkbox {
  margin-right: 30px;
}
.el-tabs {
  width: 100%;
  height: auto;
}
</style>
