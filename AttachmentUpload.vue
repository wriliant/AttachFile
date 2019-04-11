<template>
    <div  uploader="uploader" filters="queueLimit, customFilter">
        <table class="table" style="width:100%">
            <thead>
                <tr class="table_header">
                    <th width="10%">序号</th>
                    <th width="45%">文件名&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(最多显示或上传30个文件)</th>
                    <th width="10%">大小</th>
                    <th width="10%">进度</th>
                    <th width="10%">状态</th>
                    <th width="15%">动作</th>
                </tr>
            </thead>
            <tbody>
                <tr class="filelist" v-for="item in fileList" :key="item.id" :class="item.status == 'Y' ? 'success_bgc':'' ">
                    <td  width="5%" class="center"><strong>{{item.id}}</strong></td>
                    <td width="50%" class="center"><strong>{{item.name}}</strong></td>
                    <td width="10%" class="center" nowrap>{{Math.ceil((item.size/1024)/1024) }}MB</td>
                    <!-- 进度 -->
                    <td width="10%" class="center"> 
                        <div class="progress" style="margin-bottom: 0;">
                            <div v-if="item.status == 'Y' " class="progress-bar" style="transition: width 2s ease;width:100%" role="progressbar"></div>
                            <div v-if="item.status == 'N' " class="progress-bar" role="progressbar"></div>
                        </div>
                    </td>
                    <!-- 状态 -->
                    <td width="10%" class="center">
                        <span v-if="item.status == 'Y' "><i class="el-icon-check"></i></span>
                        <span v-if="item.status == 'N' "><i class="el-icon-close"></i></span>
                    </td>
                    <!-- 动作 -->
                    <td width="15%" class="center" nowrap>
                        <el-button  size="small" :disabled="item.status == 'Y'" icon="el-icon-upload2" @click="uploadOneToServe(item.id)" type="success">上传</el-button>
                        <el-button  size="small" :disabled="item.status == 'N' || item.status == '' " @click="downLoad(item.attachId,item.name)" icon="el-icon-download" type="primary">下载</el-button>
                        <el-button  size="small" icon="el-icon-delete" @click="delfileListOne(item.id)" type="danger">删除</el-button>
                    </td> 
                </tr>
            </tbody>
        </table>
        <div style="margin-top:20px;">
            <div v-if="fileList.length > 0">总进度{{width}}
                <div class="progress">
                    <div class="progress-bar-total" role="progressbar"></div>
                </div>
            </div>
            <input type="file" @change="uploadFileSystem" name = "file" id="fileId" multiple style="margin-right: 0px"/>
            <el-button  size="small" icon="el-icon-upload2" :disabled="fileList.length == 0 " @click="uploadAllToServe" type="success">上传全部</el-button>
            <el-button  size="small" icon="el-icon-delete" :disabled="fileList.length == 0 " @click="delfileListAll" type="danger">删除全部</el-button>
        </div>
    </div>
</template>
<style>
    @import url("../AttachmentUpload.css");
</style>
<script>
import { uploadAttachFilAPI} from "@/api/api.js";
export default {
    data(){
        return{
            fileList:[],
            formdata:new FormData(),
        }
    },
    computed:{
        width(){
            let count = 0;
            for (let i = 0; i < this.fileList.length; i++) {
                if (this.fileList[i].status == 'Y') {
                    count ++;
                }
            }
            let width = (count/this.fileList.length)*100;
            this.$nextTick(()=>{
                $(".progress-bar-total").width(width + '%');
            })
            return '';
        }
    },
    methods:{
        uploadFileSystem(){
            let files = $('#fileId')[0].files[0];
            files.status = '';            
            if (this.fileList.length > 0) {
                for (let i = 1; i <= this.fileList.length; i++) {
                    files.id = i+1;
                }
            }else{
                files.id = 1;
            }
            this.fileList.push(files);
        },
        // 删除单条
        delfileListOne(id){
            this.$confirm("确认删除吗？", "提示", {}).then(() => {
                for (let i = 0; i < this.fileList.length; i++) {
                    if (this.fileList[i].id == id) {
                        this.fileList.splice(i,1);
                    }
                }
                this.$message({ type: 'success', message: '删除成功!' });
                // 删除成功后重新为文件列表序号赋值（本逻辑代码可优化）
                if (this.fileList.length > 0) {
                    for (let i = 1; i <= this.fileList.length; i++) {
                        this.fileList[i].id = i+1;
                    }
                }else{
                    this.fileLis[0].id = 1;
                }
            })
        },
        // 全部删除
        delfileListAll(){
            if (this.fileList.length > 0) {
                this.$confirm("确认删除全部吗？", "提示", {}).then(() => {
                    this.fileList = [];
                 })
            }else{
                this.$message.warning("列表中无文件可删除");
            }
        },
        // 上传单个文件
        uploadOneToServe(id){
            for (let i = 0; i < this.fileList.length; i++) {
                if (this.fileList[i].id == id) {
                    this.formdata.append("file",this.fileList[i]);
                    this.sendRequest();
                } 
            }
        },
        // 上传所有文件
        uploadAllToServe(){
            let count = 0;
            if (!this.fileList.length == 0) {
                for (let i = 0; i < this.fileList.length; i++) {
                    if (this.fileList[i].status == '') { 
                        count++;                     
                        this.formdata.append("file",this.fileList[i]);
                    }
                }
                count > 0 ? this.sendRequest() : this.$message.warning("列表中无文件可上传");
            }else{
                this.$message.warning("列表中无文件可上传");
            }
        },
        //下载文件
        downLoad(val,filename){
            this.formdata.append("attachId",val);
            this.$http({
                method: 'post',
                url: '/sysAttachFile/downloadAttachFile', // 请求地址
                data: this.formdata, // 参数
                responseType: 'blob', // 表明返回服务器返回的数据类型
            }).then(res=>{
                const blob = new Blob([res.data]);
                if ('download' in document.createElement('a')) { // 非IE下载
                    const elink = document.createElement('a');
                    elink.download = filename;
                    elink.style.display = 'none';
                    elink.href = URL.createObjectURL(blob);
                    document.body.appendChild(elink);
                    elink.click();
                    URL.revokeObjectURL(elink.href); // 释放URL 对象
                    document.body.removeChild(elink);
                } else { // IE10+下载
                    navigator.msSaveBlob(blob, filename);
                }
            })
        },
        sendRequest(){
            this.formdata.append("uploadFolder","D:/abc");//上传到目标路径
            this.formdata.append("fileCoverInd","Y");//是否覆盖
            uploadAttachFilAPI(this.formdata).then(res=>{
                if (res.data.code == 0) {
                    for (let i = 0; i < res.data.resultValue.length; i++) {
                        for (let y = 0; y < this.fileList.length; y++) {
                            if (res.data.resultValue[i].attachName == this.fileList[y].name) {
                                this.fileList[y].status = 'Y';
                                this.fileList[y].attachId = res.data.resultValue[i].attachId;
                                // 解决视图不更新问题
                                this.$set(this.fileList,'aa','aa');
                                this.$delete(this.fileList,'aa','aa');
                            }
                        }
                    }
                    console.log(this.fileList);
                }else{
                    this.$message.error("文件上传出错");
                }
            })
        }
    }
}
</script>