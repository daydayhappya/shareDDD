<style lang="less">
    @import '../../../styles/common.less';
    @import './case-edit.less';
</style>

<template>
    <div>
        <Row>
            <Col span="18">
                <Card>
                    <Form ref="formValidate" :model="formValidate" :rules="ruleValidate" :label-width="80">
                        <FormItem label="Desc" prop="desc">
                            <Input v-model="formValidate.desc" placeholder="Enter case desc" type="textarea" :autosize="{minRows: 2,maxRows: 5}"></Input>
                        </FormItem>
                        <FormItem label="Name" prop="name">
                            <Input v-model="formValidate.name" placeholder="Enter case name"></Input>
                        </FormItem>

                        <FormItem
                                v-for="(itemStep, indexStep) in formValidate.itemsSteps"
                                v-if="itemStep.statusStep"
                                :key="indexStep"
                                :label="'Step ' + itemStep.indexStep"
                                :prop="'itemsSteps.' + indexStep + '.value'"
                                >
                            <Row>
                                <Col span="18">
                                    <Input type="text" v-model="itemStep.value" placeholder="Enter something..."></Input>
                                </Col>
                                <Col span="4" offset="1">
                                    <Button type="ghost" @click="handleRemoveStep(indexStep)">Delete</Button>
                                </Col>
                            </Row>
                            <FormItem label="Type" prop="type">
                                <RadioGroup v-model="itemStep.type">
                                    <Radio label=0>API</Radio>
                                    <Radio label=1>SQL</Radio>
                                </RadioGroup>
                            </FormItem>
                            <FormItem label="Format" prop="format">
                                <RadioGroup v-model="itemStep.format">
                                    <Radio label=0>form-data</Radio>
                                    <Radio label=1>JSON</Radio>
                                </RadioGroup>
                            </FormItem>
                            <FormItem label="ResponseType" prop="response_type">
                                <RadioGroup v-model="itemStep.response_type">
                                    <Radio label=0>JSON</Radio>
                                    <Radio label=1>View</Radio>
                                </RadioGroup>
                            </FormItem>
                            <FormItem label="Url">
                                <Row :gutter="8">
                                    <Col span="2">
                                        <FormItem prop="method">
                                            <Select v-model="itemStep.method" placeholder="Get">
                                                <Option value=0>Get</Option>
                                                <Option value=1>Post</Option>
                                            </Select>
                                        </FormItem>
                                    </Col>
                                    <Col span="11">
                                        <FormItem prop="path">
                                            <Input v-model="itemStep.path" placeholder="Enter Request path"></Input>
                                        </FormItem>
                                    </Col>
                                    <Col span="5">
                                        <FormItem>
                                            <Button type="primary" @click="selectInterface" >select Interface </Button>
                                        </FormItem>
                                    </Col>
                                </Row>
                            </FormItem>
                            <FormItem label="Header" prop="header">
                                <Input v-model="itemStep.header" type="textarea" :autosize="{minRows: 1,maxRows: 10}"></Input>
                            </FormItem>
                            <FormItem label="Params" prop="params">
                                <Input v-model="itemStep.params" type="textarea" :autosize="{minRows: 2,maxRows: 100}"></Input>
                            </FormItem>
                            <FormItem label="SQL" prop="sql">
                                <Input v-model="itemStep.sql" type="textarea" :autosize="{minRows: 2,maxRows: 100}"></Input>
                            </FormItem>
                            <FormItem
                                    v-for="(itemAssert, index) in itemStep.itemsAsserts"
                                    v-if="itemAssert.statusAssert"
                                    :key="index"
                                    :label="'Assert ' + itemAssert.index"
                                    :prop="'itemStep.itemsAsserts.' + index + '.actual'"
                                    >
                                <Row :gutter="10">
                                    <Col span="8">
                                        <Input type="text" v-model="itemAssert.actual" placeholder="Enter actual..."></Input>
                                    </Col>
                                    <Col span="2">
                                        <FormItem prop="rules">
                                            <Select v-model="itemAssert.rules" placeholder="=">
                                                <Option value=0>=</Option>
                                                <Option value=1>!=</Option>
                                                <Option value=2><</Option>
                                                <Option value=3>></Option>
                                            </Select>
                                        </FormItem>
                                    </Col>
                                    <Col span="8">
                                        <Input type="text" v-model="itemAssert.expect" placeholder="Enter expect..."></Input>
                                    </Col>
                                    <Col span="2" offset="1">
                                        <Button type="ghost" @click="handleRemove(indexStep,index)">Delete</Button>
                                    </Col>
                                </Row>
                            </FormItem>
                            <FormItem>
                                <Row>
                                    <Col span="12">
                                        <Button type="dashed" long @click="handleAdd(indexStep)" icon="plus-round">Add Assert</Button>
                                    </Col>
                                </Row>
                            </FormItem>

                        </FormItem>

                        <FormItem>
                            <Row>
                                <Col span="12">
                                    <Button type="dashed" long @click="handleAddStep" icon="plus-round">Add Step</Button>
                                </Col>
                            </Row>
                        </FormItem>

                        <FormItem>
                            <Button type="primary" @click="handleSubmit('formValidate')">Submit</Button>
                        </FormItem>
                    </Form>
                </Card>
            </Col>
            <Col span="6" class="padding-left-10">
                <Card>
                    <p slot="title">
                        <Icon type="paper-airplane"></Icon>
                        设置
                    </p>
                    <p class="margin-top-10">
                        <Icon></Icon>&nbsp;&nbsp;状&nbsp;&nbsp;&nbsp; 态：
                        <Select size="small" style="width:90px" value=0>
                            <Option value=0>正常</Option>
                            <Option value=1>弃用</Option>
                        </Select>
                    </p>
                </Card>
                <div class="margin-top-10">
                    <Card>
                        <p slot="title">
                            <Icon type="navicon-round"></Icon>
                            选择分组
                        </p>
                        <p type="card">
                            <Tree :data="groupList" @on-check-change="setCaseInGroup" show-checkbox ref="tree"></Tree>
                        </p>
                    </Card>
                </div>
                <div class="margin-top-10">
                    <Card>
                        <p slot="title">
                            <Icon type="ios-pricetags-outline"></Icon>
                            调试
                        </p>
                        <Row>
                            <Col span="18">
                                <Select v-model="envSelected" @on-change="handleSelectEnv" placeholder="请选择调试环境，可为空">
                                    <Option v-for="item in envList" :value="item.value" :key="item.value">{{ item.value }}</Option>
                                </Select>
                            </Col>
                            <Col span="6" class="padding-left-10">
                                <Button  @click="handleCase" long type="ghost">调试</Button>
                            </Col>
                        </Row>
                    </Card>
                </div>
                <div class="margin-top-10">
                    <Card>
                        <p slot="title">
                            <Icon type="clipboard"></Icon>
                            返回信息
                        </p>
                        <p>
                        {{message}}
                        </p>
                    </Card>
                </div>
            </Col>
        </Row>
    </div>
</template>

<script>
import axios  from 'axios';
export default {
    name: 'case-edit',
    data () {
        return {
            envSelected: [], // 选中的环境
            envList:[], // 所有环境列表
            indexStep: 1,
            message:"",
            formValidate: {
                    desc: '',
                    name: '',
                    itemsSteps: [
                        {
                            type: 0,
                            method: 0,
                            path: '',
                            header: '',
                            params: '',
                            sql:'',
                            response_type: 0,
                            format: 1,
                            statusStep:1,
                            indexStep: 1,
                            value:'',
                            itemsAsserts: [
                                {
                                    statusAssert: 1,
                                    index: 1,
                                    actual:'',
                                    rules:'',
                                    expect:''
                                }
                            ]
                        }
                    ],
                },
            ruleValidate: {
                name: [
                        { required: true, message: 'The name cannot be empty', trigger: 'blur' }
                    ],
                desc: [
                        { required: true, message: 'Please enter a personal introduction', trigger: 'blur' },
                        { type: 'string', min: 10, message: 'Introduce no less than 10 words', trigger: 'blur' }
                    ]
                },
            groupList:[],
            selectInterfaceData:{
                name:''
            },
            selectedGroupId: 0,
        };
    },
    methods: {
        getData(){
            axios.get("/v1/case/getCaseInfosById",{params:{caseId:this.$route.query.caseId}}
                ).then((res)=>{
                console.log(res)
                if(res.data.success){
                    this.formValidate = res.data.message;
                    this.selectedGroupId = res.data.message.groupId;

                }else{
                    this.$Message.error("获取数据失败")
                }
            });
        },
        getGroupList(){
            axios.get("/v1/group/getGroupByProjectId",{params:{projectId:this.$route.query.projectId,type:1}}).then((res)=>{
                console.log(res);
                if(res.data.success){
                    const that =this;
                    res.data.message.forEach(function(item){
                        console.log(item);
                        that.groupList.push(item);
                    });
                }else{
                    this.$Message.error("获取数据失败")
                }
            })
        },
        getEnv(){
            axios.get("/v1/env/getEnvironmentInfoByUserId",{params:{userId:1}}
                ).then((res)=>{
                console.log(res)
                if(res.data.success){
                    const that = this
                    res.data.message.forEach(function(item){
                        const tmpJson ={}
                        tmpJson["value"]=item["name"]
                        that.envList.push(tmpJson);
                    });
                }else{
                    this.$Message.error("获取数据失败")
                }
            });
        },
        setCaseInGroup(){
            const groupCheckedData=this.$refs.tree.getCheckedNodes()
            const that = this
            groupCheckedData.forEach(function(item){
                if(item.parentGroupId !=0){
                    that.selectedGroupId = item.groupId
                }
                return
            });
        },
        handleCase(){
            if(this.$route.query.caseId == ""){
                this.$Message.error('暂时不能发起调试接，请先保存用例');
                return ;
            }
            const data={}
            data["caseId"]=this.$route.query.caseId
            axios.post("/v1/case/startTaskBySingleCase",data).then((res)=>{
                if(res.data.success){
                    this.$Message.success("触发成功");
                    this.message = res.data.message;
                }else{
                    this.$Message.error("触发失败");
                }
            });
        },
        handleAddStep () {
            this.indexStep++;
            this.formValidate.itemsSteps.push({
                type: 0,
                method: 0,
                path: '',
                header: '',
                params: '',
                sql:'',
                value: '',
                response_type: 0,
                format: 1,
                indexStep: this.indexStep,
                statusStep: 1,
                itemsAsserts: [
                    {
                        statusAssert: 1,
                        index: 1,
                        actual:'',
                        rules:'',
                        expect:''
                    }
                ]
            });
        },
        handleRemoveStep (indexStep) {
            this.formValidate.itemsSteps[indexStep].statusStep = 0;
        },
        handleAdd (indexStep) {
            const index = this.formValidate.itemsSteps[indexStep].itemsAsserts.length + 1;
            alert(index)
            this.formValidate.itemsSteps[indexStep].itemsAsserts.push({
                actual:'',
                rules:'',
                expect:'',
                index: index,
                statusAssert: 1
            });
        },
        handleRemove (indexStep,index) {
            this.formValidate.itemsSteps[indexStep].itemsAsserts[index].statusAssert = 0;
        },

        handleSelectEnv () {
            localStorage.tagsList = JSON.stringify(this.envSelected);
        },

        handleSubmit (name) {
            this.$refs[name].validate((valid) => {
                if (valid) {
                    if(this.$route.query.caseId != ""){
                        this.$Message.error('暂时没有编辑接口');
                        return ;
                    }
                    const data= this.formValidate;
                    data["projectId"] = this.$route.query.projectId;
                    //for test
                    data["userId"]=1
                    data["userName"]="jessica"
                    if(this.selectedGroupId ==0){
                        this.$Message.error('请选择分组');
                        return ;
                    }
                    data["groupId"]=this.selectedGroupId
                    axios.post("/v1/case/addTestCase",data).then((res)=>{
                        if(res.data.success){
                            this.$Message.success("添加成功");
                            alert(JSON.stringify(res.data.message));
                        }else{
                            this.$Message.error("添加数据失败");
                        }
                    });
                } else {
                    this.$Message.error('Fail!');
                }
            })
        },
        selectInterface(){
//            this.$Modal.confirm({
//                onOk: () => {
//                       this.selectInterfaceNet();
//                    },
//                    render: (h) => {
//                    return h('div',[
//                        h('Input', {
//                            props: {
//                                value: this.selectInterfaceData.name,
//                                autofocus: true,
//                                placeholder: '接口名称'
//                            },
//                            on: {
//                                input: (val) => {
//                                    this.selectInterfaceData.name = val;
//                                }
//                            }
//                        }),
//                    ])
//                }
//            })
        },
        selectInterfaceNet(){
        },
    },
    created () {
        this.getData();
        this.getEnv();
        this.getGroupList();
    },
};
</script>
