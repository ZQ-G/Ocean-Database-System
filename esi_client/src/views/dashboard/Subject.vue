<template>
    <div>
        <n-tabs type="line" animated>
            <n-tab-pane name="add" tab="物种信息添加">
                <n-form ref="formRef" :rules="rules" :model="addSubject" style="margin-top:10px;">
                    <n-form-item path="creatureName" label="物种名称">
                        <n-input v-model:value="addSubject.creatureName" placeholder="请输入物种名称" clearable style="width: 500px;" />
                    </n-form-item>
                    <n-form-item label="物种类别">
                        <n-select v-model:value="addSubject.subType" :options="options" style="width: 500px;" />
                    </n-form-item>
                    <n-form-item label="">
                        <n-button @click="toCreate">
                            <template #icon><n-icon><AddCircleOutline /></n-icon></template>
                            添加&#8194;
                        </n-button>
                        <n-button @click="refresh" style="margin-left: 30px;">
                            <template #icon><n-icon><RefreshCircleOutline /></n-icon></template>                            
                            重置
                        </n-button>
                    </n-form-item>
                </n-form>               
            </n-tab-pane>
            <n-tab-pane name="maintain" tab="物种信息维护">
                <div style="display: flex; margin-top: 10px;">                  
                    <n-input v-model:value="pageInfo.creatureName" placeholder="请输入物种名称关键字" clearable />
                    <n-select v-model:value="pageInfo.subType" :options="options" style="width: 300px; margin-left: 20px;" placeholder="请选择物种类别" filterable />
                    <n-button @click="search" type="success" style="margin-left: 20px;">
                        <template #icon><n-icon><SearchOutline /></n-icon></template>
                        查询&#8194;
                    </n-button>
                </div>
                    <div style="display: flex; ">
                        <n-table :bordered="true" :single-line="false" style="margin-top:20px;">
                            <thead>
                                <tr>
                                <th>物种编号</th>
                                <th>物种名称</th>
                                <th>物种类别</th>
                                <th>操作</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="(subject,index) in subjectList">
                                <td>{{subject.ID}}</td>
                                <td>{{subject.CreatureName}}</td>
                                <td>{{subject.SubType}}</td>
                                <td>             
                                    <n-button @click="updateValue(subject)">
                                        <template #icon><n-icon><CreateOutline /></n-icon></template>                            
                                        修改&#8194;
                                    </n-button>
                                    <n-button @click="toDelete(subject)" style="margin-left:20px;">
                                        <template #icon><n-icon><TrashOutline /></n-icon></template>                            
                                        删除&#8194;
                                    </n-button>
                                </td>
                                </tr>
                            </tbody>
                        </n-table>
                        <n-button @click="clean" style="margin-top:20px;margin-left: 20px;">
                            <template #icon><n-icon><RefreshCircleOutline /></n-icon></template>
                            重置&#8194;
                        </n-button>
                        <n-button @click="exportExcel" style="margin-top:20px;margin-left: 20px;">
                            <template #icon><n-icon><DownloadOutline /></n-icon></template>
                            导出&#8194;
                        </n-button>
                        </div>
                        <n-pagination @update:page="loadSubject" v-model:page="pageInfo.pageNum" :page-count="pageInfo.pageCount" style="margin-top: 20px;"/>  
                        
                        <n-modal v-model:show="showUpdateModel" preset="dialog" title="Dialog">
                            <template #header>
                                <div>修改</div>
                            </template>
                            <n-form ref="formRef" :rules="rules" :model="updateSubject">
                                <n-form-item label="物种编号" style="margin-top: 20px;">
                                    <n-input v-model:value="updateSubject.id" :disabled="!active" />
                                </n-form-item>
                                <n-form-item path="creatureName" label="物种名称">
                                    <n-input v-model:value="updateSubject.creatureName" placeholder="请输入大学名称" clearable />
                                </n-form-item>
                                <n-form-item label="物种类别">
                                    <n-select v-model:value="updateSubject.subType" :options="options" />
                                </n-form-item>
                                <n-form-item label="" >
                                    <n-button @click="closeModal">取消</n-button>
                                    <n-button @click="toUpdate" type="success" style="margin-left: 20px;">确认</n-button>
                                </n-form-item>
                            </n-form>   
                        </n-modal>
            </n-tab-pane>
        </n-tabs>
    </div>
</template>

<script setup>
import {ref,reactive,inject, onMounted} from 'vue'
import {AddCircleOutline, RefreshCircleOutline, SearchOutline, TrashOutline, CreateOutline, DownloadOutline} from "@vicons/ionicons5"

import {useRouter, useRoute} from 'vue-router'
const router = useRouter()
const route = useRoute()

const axios = inject("axios")
const message = inject("message")
const serverUrl = inject("serverUrl")
const dialog = inject("dialog")
const fileDownload = inject("fileDownload")

const formRef = ref(null)
const showUpdateModel = ref(false)
const addSubject = reactive({
    creatureName: "",
    subType: "鱼类"
})
const updateSubject = reactive({
    id: 0,
    creatureName: "",
    subType: ""
})
const subjectList = ref([])
const pageInfo = reactive({
    pageNum:1,
    pageSize:7,
    pageCount:0,
    count:0,
    creatureName:"",
    subType:null
})

onMounted(() => {
    loadSubject()
})

const loadSubject = async(pageNum = 0) => {
    if (pageNum != 0){
        pageInfo.pageNum = pageNum;
    }
    let res = await axios.post(`/subject/search?creatureName=${pageInfo.creatureName}&subType=${pageInfo.subType}&pageNum=${pageInfo.pageNum}&pageSize=${pageInfo.pageSize}`)
    console.log(res)
    if (res.data.code == 200) {
        subjectList.value = res.data.data.subject
    }
    pageInfo.count = res.data.data.count;
    pageInfo.pageCount = parseInt(pageInfo.count / pageInfo.pageSize) + (pageInfo.count % pageInfo.pageSize > 0 ? 1 : 0)
}

let rules = {
    creatureName: [
        { required: true, message: "请输入物种名称" },
        { max: 20, message: "物种名称长度在小于 20 个字符" },    
    ]
}

const options = ref([
    {value:"鱼类", label: "鱼类"},
    {value:"甲壳类", label: "甲壳类"},
    {value:"软体动物", label: "软体动物"},
    {value:"腔肠动物", label: "腔肠动物"},
    {value:"海藻类", label: "海藻类"},
    {value:"海绵类", label: "海绵类"},
    {value:"珊瑚类", label: "珊瑚类"},
    {value:"鲸类", label: "鲸类"},
    {value:"海鸟类", label: "海鸟类"},
    {value:"海龟类", label: "海龟类"},
    {value:"海豚类", label: "海豚类"},
    {value:"水母类", label: "水母类"},
    {value:"海星类", label: "海星类"},
    {value:"海胆类", label: "海胆类"},
    {value:"海葵类", label: "海葵类"},
    {value:"海马类", label: "海马类"},
    {value:"海螺类", label: "海螺类"},
    {value:"海参类", label: "海参类"},
    {value:"海蛇类", label: "海蛇类"},
    {value:"海蜘蛛类", label: "海蜘蛛类"},
    {value:"海蜇类", label: "海蜇类"},
    {value:"海螳螂类", label: "海螳螂类"},
]) 

const toCreate = async() => {
    formRef.value?.validate((errors) => {
        if (errors) {
            message.error("数据格式有误")
        } else {
            create();
        }
    })
}

const create = async() => {
    let res = await axios.post("/subject", {
        creatureName: addSubject.creatureName,
        subType: addSubject.subType,
    })
    console.log(res)
    if (res.data.code == 200) {
        message.success(res.data.msg)
        refresh()
        loadSubject()  
    } else {
        message.error(res.data.msg)
    }
}

const updateValue = async(subject) => {
    showUpdateModel.value = true
    updateSubject.id = subject.ID
    updateSubject.creatureName = subject.CreatureName
    updateSubject.subType = subject.SubType
}

const toUpdate = async() => {
    formRef.value?.validate((errors) => {
        console.log(errors)
        if (errors) {
            message.error("数据格式有误")
        } else {
            update();
        }
    })
}

const update = async() => {
    let res = await axios.put("/subject/"+ updateSubject.id, {
        creatureName: updateSubject.creatureName,
        subType: updateSubject.subType,
    })
    console.log(res)
    if (res.data.code == 200) {
        message.success(res.data.msg)
        loadSubject()  
        closeModal()
    } else {
        message.error(res.data.msg)
    }    
}

const toDelete = async(subject) => {
    dialog.warning({
        title: '删除',
        content: '是否要删除',
        positiveText: '确定',
        negativeText: '取消',
        onPositiveClick: async () => {
            let res = await axios.delete(`/subject/${subject.ID}`)
            if(res.data.code == 200){
                message.success(res.data.msg)
                loadSubject()
            }else{
                message.error(res.data.msg)
            }  
        },
        onNegativeClick: () => {}
    })    
}

const closeModal = () => {
    showUpdateModel.value = false;
}

const refresh = () => {
    addSubject.creatureName = ""
    addSubject.subType = "鱼类"
}

const search = () => {
    pageInfo.pageNum = 1
    loadSubject()
}

const clean = () => {
    pageInfo.creatureName = ""
    pageInfo.subType = null
    loadSubject()
}

const exportExcel = async() => {
    let res = await axios.get(`/exportSubject?creatureName=${pageInfo.creatureName}&subType=${pageInfo.subType}`, {responseType: "blob"})
    console.log(res)
    fileDownload(res.data, "result.xlsx")
}

</script>

<style lang="scss" scoped>

</style>