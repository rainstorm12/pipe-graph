<template>
    <div class="monitor-box">
        <div class="content-box">
            <div class="left-content-box">
                <div class="weather-box">
                    <div class="weather-state">{{weather_state}}</div>
                    <div class="temperature-state">{{temperature_state}}</div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import * as d3 from 'd3'
import {mapState,mapGetters,mapMutations,mapActions} from 'vuex'
import axios from 'axios'
export default {
    name: 'MonitorPage',
    props: {},
    data () {
        return {
            weather_state:'',
            temperature_state:'',
        }
    },
    created() {},
    computed:{
        //借助mapState生成计算属性，从state中读取数据
        ...mapState(['server_ip']),
    },
    mounted(){
        this.mount_init()
    },
    methods: {
        //mounted函数
        mount_init(){
            this.timer = setInterval(() => {
                axios.get('http://'+this.server_ip+'/api/show_weather').then(
                    response=>{
                        console.log('请求成功',response.data)
                        this.weather_state = response.data['weather_state']
                        this.temperature_state = String(response.data['temperature_state'])+"°"
                    },
                    error=>{
                        console.log('请求失败',error.message)
                    }
                )
            }, 1000)  // 0.5 秒定时拉取
        },
    },
}
</script>

<style scoped>
.monitor-box{
    flex:1;/*撑满布局 */
    height:100vh;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    background-color:rgba(227,237,245,1);
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
.left-content-box{
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    height: 800px;
    border-right: 1px solid SkyBlue;
}
.weather-box{
    display: flex;
    flex-direction: row;
    padding:40px;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid SkyBlue;
    font-size:50px;
    color:DeepSkyBlue;
}
.weather-state{
    padding:20px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width:100px;
}
.temperature-state{
    padding:20px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width:100px;
}
</style>
   