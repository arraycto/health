<template>

  <a-drawer
          :title="title"
          :maskClosable="true"
          :width="drawerWidth"
          placement="right"
          :closable="true"
          @close="handleCancel"
          :visible="visible"
          style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <template slot="title">
      <div style="width: 100%;">
        <span>{{ title }}</span>
      </div>

    </template>

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item label="可选择的领导" :labelCol="labelCol" :wrapperCol="wrapperCol" >

          <a-tree
                  checkable
                  :treeData="groupList"
                  :checkedKeys="leaderIds"
                  @check="this.onCheck"
          >
          </a-tree>

          <!--<a-select-->
          <!--mode="multiple"-->
          <!--style="width: 100%;height:100%;"-->
          <!--placeholder="请选择分组"-->
          <!--v-model="selectedGroups">-->
          <!--<a-select-option v-for="(group,groupindex) in groupList" :key="groupindex.toString()" :value="group.id">-->
          <!--{{ group.groupName }}({{group.description}})-->
          <!--</a-select-option>-->
          <!--</a-select>-->
        </a-form-item>

      </a-form>
    </a-spin>
    <div class="drawer-bootom-button" v-show="!disableSubmit">
      <a-popconfirm title="确定放弃编辑？" @confirm="handleCancel" okText="确定" cancelText="取消">
        <a-button style="margin-right: .8rem">取消</a-button>
      </a-popconfirm>
      <a-button @click="handleSubmit" type="primary" :loading="confirmLoading">提交</a-button>
    </div>
  </a-drawer>
</template>
<script>
  import {queryTreeListForLeaderGroup,saveSysUserLeader
    // queryLeaderGroupLeaders,
    // saveLeaderGroupLeaders
  } from '@/api/api'

  export default {
    name: "SysUserLeaderModal",
    components:{
    },
    data(){
      return {
        modalWidth:800,
        drawerWidth:700,
        modaltoggleFlag:true,
        confirmDirty: false,
        userId:"", //保存单位id
        disableSubmit:false,
        title:"领导配置",
        visible: false,
        model: {},
        groupList:[],
        leaderIds:[],
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        uploadLoading:false,
        confirmLoading: false,
        form:this.$form.createForm(this),
      }
    },
    methods: {

      onCheck (leaderIds) {
        this.leaderIds = [];
        this.leaderIds.push(leaderIds[leaderIds.length-1])
      },
      initialGroupList(){
        queryTreeListForLeaderGroup().then((res)=>{
          if(res.success){
            this.groupList = res.result;
            //this.loadUserGroups(this.departId);
          }else{
            console.log(res.message);
          }
        });
      },
      edit (userid,leaderid) {

        let that = this;
        that.form.resetFields();
        that.userId = userid;
        that.leaderIds.push(leaderid);
        console.log("leaderIds+", that.leaderIds);
        that.initialGroupList();
        that.visible = true;
      },
      close () {
        this.$emit('close');
        this.visible = false;
        this.disableSubmit = false;
        this.leaderIds = [];
      },
      handleSubmit () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let formData = Object.assign(this.model, values);
            formData.userId = this.userId;
            formData.leaderIds = this.leaderIds.length>0?this.leaderIds.join(","):'';
            let obj;
            obj=saveSysUserLeader(formData);
            obj.then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })

          }

        })
        this.$emit("userLeaderModalOk")
      },
      handleCancel () {
        this.close()
      },
    },
  }

</script>
<style lang="scss" scoped>
  .drawer-bootom-button {
    position: absolute;
    bottom: 0;
    width: 100%;
    border-top: 1px solid #e8e8e8;
    padding: 10px 16px;
    text-align: right;
    left: 0;
    background: #fff;
    border-radius: 0 0 2px 2px;
  }

</style>