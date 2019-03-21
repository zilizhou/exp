<template>
    <div>
        <div class="crumbs">
            <!-- <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-lx-calendar"></i> 表单</el-breadcrumb-item>
                <el-breadcrumb-item>markdown编辑器</el-breadcrumb-item>
            </el-breadcrumb> -->
        </div>
        <div class="container">
            <div class="plugins-tips">
                <el-form ref="form" :model="form"  label-width="80px">
                
                <el-form-item label="单选框">
                        <el-radio-group v-model="expsel" v-for="(item,index) in form.expname" :key="item.name">
                            <el-radio  :options=item.sel  :label="index" border @change="radioChange" >{{form.expname[index]}}</el-radio>
                        </el-radio-group>
                </el-form-item>
                </el-form>
            </div>
             <el-button class="editor-btn" type="primary" @click="submit">提交</el-button>
            <mavon-editor v-model="content" ref="md" @imgAdd="$imgAdd" @change="change" style="min-height: 600px"/>
            <el-button class="editor-btn" type="primary" @click="submit">提交</el-button>
           
        </div>
    </div>
</template>

<script>
    import { mavonEditor } from 'mavon-editor'
    import 'mavon-editor/dist/css/index.css'
    export default {
        name: 'markdown',
        data: function(){
            return {
                expsel:0,
                expselbak:0,
                userno: localStorage.getItem('ms_username'),
                form: {
                    expname: ['实验一','实验二','实验三','实验四','实验五','实验六','实验七','实验八'],
                    name: '',
                    region: '',
                    date1: '',
                    date2: '',
                    delivery: true,
                    type: ['步步高'],
                    resource: '小天才',
                    desc: '',
                    options: []
                },
                content:'',
                html:'',
                configs: {
                }
            }
        },
        components: {
            mavonEditor
        },
        mounted(){
            let data={'userno':this.userno,'expidx':0}
                    this.$axios.post('http://39.104.151.240:9000/getexpdata',data).then((res)=>{
                        

                        this.content=res.data.expdata
                        
                        });

        },
        methods: {
            radioChange(){
                
                this.$confirm('切换实验前一定要先 提交 实验报告，否则会清空实验内容, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
                }).then(() => {
                    let data={'userno':this.userno,'expidx':this.expsel}
                    this.$axios.post('http://39.104.151.240:9000/getexpdata',data).then((res)=>{
                        

                        this.content=res.data.expdata
                        
                        });
                        this.expselbak=this.expsel;
                    this.$message({
                        type: 'success',
                        message: '实验切换成功!'
                        });
                }).catch(() => {
                    this.expsel=this.expselbak;

                this.$message({
                    type: 'info',
                    message: '已取消实验切换'
                    });          
                });

                
                console.log(this.expsel);

            },
        
            // 将图片上传到服务器，返回地址替换到md中
            $imgAdd(pos, $file){
                var formdata = new FormData();
                formdata.append('file', $file);
                formdata.append('userno',this.userno);
                formdata.append('expidx',this.expsel);
                console.log(formdata);
                // 这里没有服务器供大家尝试，可将下面上传接口替换为你自己的服务器接口
                this.$axios({
                    url: 'http://39.104.151.240:9000/recimg',
                    method: 'post',
                    data: formdata,
                    headers: { 'Content-Type': 'multipart/form-data' },
                }).then((url) => {
                    this.$refs.md.$img2Url(pos, url.data);
                    
                })
            },
            change(value, render){
                // render 为 markdown 解析后的结果
                this.html = render;
            },
            submit(){
                
                let data={'userno':this.userno,'expidx':this.expsel,'expdata':this.content};
                    console.log(data)
                    this.$axios.post('http://39.104.151.240:9000/recexpdata',data).then((res)=>{
                    

                    this.$message.success('提交成功！');
                    
                    });
                
                
            }
        }
    }
</script>
<style scoped>
    .editor-btn{
        margin-top: 20px;
    }
</style>