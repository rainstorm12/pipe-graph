<template>
    <div class="kb-box">
        <div class="tool-box">
            <el-button style="margin: 0px; " :style="add_button_style" type="success" icon="el-icon-plus" @click="add_button_click" @mouseenter.native="add_button_enter" @mouseleave.native="add_button_leave">添加知识</el-button>
        </div>
        <div class="content-box">
            <el-table :data="KBData" style="width: 100% ;" height="800" border>
                <el-table-column label="知识" min-width="80">
                    <template slot-scope="scope"> {{ scope.row.knowledge }} </template>
                </el-table-column>
                <el-table-column label="内容" min-width="120">
                    <template slot-scope="scope"> {{ scope.row.content }} </template>
                </el-table-column>
                <el-table-column label="操作" min-width="75">
                    <template slot-scope="scope">
                        <el-button class="edit-button" type="primary" icon="el-icon-edit" @click="StartEdit(scope.$index, scope.row)" circle></el-button>
                        <el-button class="delete-button" type="danger" icon="el-icon-delete" @click="StartDel(scope.$index,scope.row)" circle></el-button>
                    </template>
                </el-table-column>
            </el-table>
        </div>
        <!-- 编辑框 -->
        <el-dialog :visible.sync="isOpenDialog">
            <el-form class="dialog-form">
                <el-form-item>
                    <span>知识:</span>
                    <el-input v-model="edit_tuple.knowledge"/>
                </el-form-item>
                <el-form-item>
                    <span>内容:</span>
                    <el-input v-model="edit_tuple.content"/>
                </el-form-item>
            </el-form>
            <div slot="footer">
                <el-button @click="StopEdit">取消</el-button>
                <el-button type="primary" @click="ActEdit">确定</el-button>
            </div>
        </el-dialog>
        <!-- 添加编辑框 -->
        <el-dialog :visible.sync="isOpenAddDialog">
            <el-form class="dialog-form">
                <el-form-item>
                    <span>知识:</span>
                    <el-input v-model="add_tuple.knowledge"/>
                </el-form-item>
                <el-form-item>
                    <span>内容:</span>
                    <el-input v-model="add_tuple.content"/>
                </el-form-item>
            </el-form>
            <div slot="footer">
                <el-button @click="StopAdd">取消</el-button>
                <el-button type="primary" @click="ActAdd">确定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
import * as d3 from 'd3'
import {mapState,mapGetters,mapMutations,mapActions} from 'vuex'
import axios from 'axios'
//深拷贝对象函数
var cloneObj = function (obj) {
    var newObj = {};
    if (obj instanceof Array) {
        newObj = [];
    }
    for (var key in obj) {
        var val = obj[key];
        newObj[key] = typeof val === 'object' ? cloneObj(val) : val;
    }
    return newObj;
}
export default {
    name: 'KBPage',
    props: {},
    data () {
        return {
            //knowledge data
            KBData:[],
            //dialog state
            isOpenDialog: false, //是否编辑对话框
            isOpenAddDialog:false,//是否编辑添加对话框
            edit_tuple:{},
            add_tuple:{},
            //button style
            add_button_style:{
                background:"#67C23A"
            },
        }
    },
    created() {},
    computed:{
        //借助mapState生成计算属性，从state中读取数据
        ...mapState(['server_ip']),
    },
    mounted(){
        // this.KBData=[{knowledge:"1",content:"好"},{knowledge:"2",content:"不好"}]

        this.datainit()
    },
    methods: {
        //数据初始化
        datainit(event){
            axios.get('http://'+this.server_ip+'/api/ShowKnowledgeData').then(
                response=>{
                    console.log('请求成功',response.data)
                    for(let j=0; j<response.data["KnowledgeData"].length;j++){
                        let knowledge = response.data["KnowledgeData"][j].fields.Knowledge
                        let content = response.data["KnowledgeData"][j].fields.Content
                        this.KBData.push({knowledge:knowledge,content:content})
                    }
                },
                error=>{
                    console.log('请求失败',error.message)
                }
            )
        },
        /*el-button按键的动态效果，css好像被模板绑定了，所以只能通过js实现 */
        add_button_enter(event){
            this.add_button_style = { background:"#85ce61" }
        },
        add_button_leave(event){
            this.add_button_style = { background:"#67C23A" }
        },
        /*edit button*/
        StartEdit(index,row){//打开编辑框
            this.edit_tuple = cloneObj(this.KBData[index])//深拷贝
            this.isOpenDialog = true
        },
        StopEdit(){//关闭编辑框
            this.isOpenDialog = false
        },
        ActEdit(){//执行编辑操作
            this.isOpenDialog = false
            // this.$store.commit('UPDATE_NODE_MODEL',this.edit_tuple)
            // this.datainit()
            this.$message({
                message: '编辑成功',
                type: 'success'
            })
        },
        /*del button*/
        StartDel(index,row){
            // this.$store.commit('DEL_RELATION_MODEL',del_tuple)
            this.$confirm('此操作将永久删除该知识, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                })
                .then(() => {
                    let del_tuple = this.KBData[index]
                    this.$message({
                        type: 'success',
                        message: '删除成功!'
                    });
                    // this.DeleteNode()
                    // this.datainit()
                })
                .catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });       
                });
        },
        /*add button */
        add_button_click(){
            // this.add_tuple={}
            this.isOpenAddDialog = true
        },
        StopAdd(){//关闭编辑框
            this.isOpenAddDialog = false
        },
        ActAdd(){//执行编辑操作
            this.isOpenAddDialog = false
            if(this.add_tuple.knowledge===undefined||this.add_tuple.content===undefined||
                this.add_tuple.knowledge===''||this.add_tuple.content===''){
                    this.$message({
                        message: '输入不能为空',
                        type: 'warning'
                    })
                }
            else{
                // let result = this.$store.commit('ADD_RELATION_MODEL',this.add_tuple)
                // this.datainit()
                this.$message({
                        message: '添加成功',
                        type: 'success'
                    })
            }
        },
    },
}
</script>

<style scoped>
/* 总体布局 */
.kb-box{
    flex:1;/*撑满布局 */
    height:100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background-color:rgba(227,237,245,1);
}
.tool-box{
    width:1190px;
    display: flex;
    flex-direction: row;
    justify-content: end;
    align-items: center;
    background-color:white;
    padding:5px;
    border: 1px solid #dcdfe6;
}
.content-box{
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    background-color: white;
    width: 1200px;
    border: 1px solid #dcdfe6;
    border-radius: 10px ;
}
/* 按键 */
.edit-button.is-circle{
    margin:0;
    background: #409EFF;
}
.edit-button.is-circle:hover{
    margin:0;
    background: #66b1ff;
}
.delete-button.is-circle{
    margin:0 3px 0 3px;
    background:#F56C6C;
}
.delete-button.is-circle:hover{
    background: #f78989;
}
/* 编辑框 */
.dialog-form{
    width: 500px; 
    margin-left:50px;
}
</style>
   