<template>
    
    <div id="MainArea" :style="workspaceDivClass">
        <div :style="iconDivClass">
            <span>
                <i class="el-icon-zoom-in" @click="zoomIn"></i>
            </span>
            <el-divider direction="vertical"></el-divider>
            <span>
                <i class="el-icon-zoom-out" @click="zoomOut"></i>
            </span>
        </div>
        <!--生成所有网格-->
        
        <div v-for="index in rowGridNum*rowGridNum"
            :key="index" :style="gridStyle[index-1]"
            :id="index-1"
            @dragenter="ondragenter" @dragover="ondragover($event)" 
            @dragleave="ondragleave" @drop="ondrop($event, index)"
            @click="dialogFormVisible = true"
        ></div>
        <el-dialog title="设置属性" :visible.sync="dialogFormVisible">
            <el-form :model="form">
                <el-form-item label="活动名称" :label-width="formLabelWidth">
                    <el-input v-model="form.name" autocomplete="off"></el-input>
                </el-form-item>
            <el-form-item label="活动区域" :label-width="formLabelWidth">
                <el-select v-model="form.region" placeholder="请选择活动区域">
                    <el-option label="区域一" value="shanghai"></el-option>
                    <el-option label="区域二" value="beijing"></el-option>
                </el-select>
            </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">取 消</el-button>
                <el-button type="primary" @click="dialogFormVisible = false">确 定</el-button>
            </div>
        </el-dialog>
        <br />
        <el-button type="primary" @click="testAxios"> 保存 </el-button>
    </div>
</template>

<script>

import axios from 'axios';
    // var db = openDatabase('BSHdb', '1.0', 'Test DB', 2 * 1024 * 1024);
    export default {
        name: 'templateDemo2',
        data() {
            return {
                 //弹窗默认关闭
                dialogFormVisible: false,
                totalWidth: 800,
                // 每行网格个数
                rowGridNum: 35,
                // 每个网格的边长，mounted时将被重新计算
                gridWidth: 30,
                // 网格绑定的样式，在mounted中初始化
                gridStyle: [],
                button: [],
                led: [],
                // 每个网格所包含的元素, 设为二维数组，在mounted中初始化
                /*
                    {
                        type
                        id
                    }
                */
                form: {
                    name: '',
                    region: '',
                },
                formLabelWidth: '120px',

                // 分别定义：背景颜色，单button颜色，单led颜色，组件叠加颜色
                backgroundColor: '#FFFFFF',
                buttonColor: "#8CFE90",
                ledColor: "#00EFEF",
                mixColor: "#000080",

                // 微调放大/缩小图标的布局
                iconDivClass: {
                    textAlign: 'right',
                    marginRight: '2%',
                    marginBottom: '1%',
                    marginTop: '1%'
                },
                // 微调网格的布局
                // offsetWidth会随此处的width改变
                workspaceDivClass: {
                    width: '96%',
                    height: '730px',
                }

            }
        },
        
        mounted: function() {
            // 获取中间工作区的页面宽度
            this.totalWidth = document.getElementById("MainArea").offsetWidth;
            // 计算一个网格的边长, 减去0.01防止四舍五入后总宽度不够，在浏览器允许的缩放范围内这个误差够了
            // 1+this.rowGridNum用于抵消滚动条和一些玄学误差
            this.gridWidth = this.totalWidth / (1 + this.rowGridNum) - 0.01

            //console.log(this.totalWidth, this.gridWidth)
            // 初始化所有网格的样式
            for (let i = 0 ; i < this.rowGridNum * this.rowGridNum ; ++i) {
                this.gridStyle.push(
                    {
                        width: this.gridWidth + 'px',
                        height: this.gridWidth + 'px',
                        display: 'inline-block',
                        border: '1px solid',
                        'box-sizing': 'border-box',
                        'font-size': '1px',
                        float: 'left',
                        background: this.backgroundColor,
                    }
                )
            }

            // 监听改变页面大小的事件
            let _this = this
            window.onresize = function() {
                _this.alterGridWidth()
            }

            // 利用elemet-resize-detector, 监听放大/缩小事件
            var elementResizeDetectorMaker = require("element-resize-detector");
            let erd = elementResizeDetectorMaker()
            erd.listenTo(document.getElementById("MainArea"), function() {
                _this.alterGridWidth()
            });
        },
        methods: {
            testAxios() {
                // 向后端传递组件参数
                axios ({
                    method: 'post',
                    url: '/save',
                    data: {
                        button: this.button,
                        led: this.led
                    }
                    }).then(function (response) {
                        console.log(response.data)
                });
            },
            ondragenter() {
                //this.style.borderColor='red';
            },
            ondragover(event) {
                // this.style.borderColor='red';
                event.preventDefault()
            },
            ondragleave() {
                //this.style.borderColor='#aaaaaa'
            },
            ondrop(event, index) {
                //this.style.borderColor='#aaaaaa';
                event.preventDefault()
                let data = JSON.parse(sessionStorage.getItem("element"));
                //console.log(data)
                //this.style.borderColor='#aaaaaa';
                switch(data.type){
                    // 放置一个Button
                    case 1:{
                        // button覆盖的index
                        let rangeIndex = [index-1, index, index+this.rowGridNum-1, index+this.rowGridNum]
                        for(let i = 0 ; i < rangeIndex.length ; ++i) {
                            // 如果覆盖位置上已存在其它组件，就将背景颜色设为mixColor，否则设为buttonColor
                            let curBackground = this.gridStyle[rangeIndex[i]].background
                            if (curBackground == this.backgroundColor) {
                                this.gridStyle[rangeIndex[i]].background = this.buttonColor
                            } else {
                                this.gridStyle[rangeIndex[i]].background = this.mixColor
                            }
                            this.gridStyle[rangeIndex[i]].cursor="pointer"
                        }
                        /*
                        this.gridStyle[index-1].background = "#67C23A";
                        this.gridStyle[index].background = "#67C23A";
                        this.gridStyle[index+this.rowGridNum-1].background = "#67C23A";
                        this.gridStyle[index+this.rowGridNum].background = "#67C23A";
                        //拖动按钮松开后绿色代表button
                        this.gridStyle[index-1].cursor = "pointer";
                        this.gridStyle[index].cursor = "pointer";
                        this.gridStyle[index+this.rowGridNum-1].cursor = "pointer";
                        this.gridStyle[index+this.rowGridNum].cursor = "pointer";  
                        */
                        //已有元素在上面的格子变成可点击的手型 
                        this.button.push({
                            id: data.id,
                            name: data.name,
                            hwId: data.hwId
                        })
                        break;
                    }
                    // 放置一个led
                    case 2:{
                        // 如果覆盖位置上已存在其它组件，就将背景颜色设为mixColor，否则设为ledColor
                        let curBackground = this.gridStyle[index-1].background;
                        if (curBackground == this.backgroundColor) {
                            this.gridStyle[index-1].background = this.ledColor
                        } else {
                            this.gridStyle[index-1].background = this.mixColor
                        }
                        //拖动按钮松开后蓝色代表LED
                        this.gridStyle[index-1].cursor = "pointer";
                        this.led.push({
                            id: data.id,
                            name: data.name,
                            hwId: data.hwId
                        })
                        break;
                    }
                }
            },

            // 当页面状态改变时调用该函数重新计算网格边长
            alterGridWidth() {
                this.totalWidth = document.getElementById("MainArea").offsetWidth;
                // 重新计算一个网格的边长
                this.gridWidth = this.totalWidth / (1+this.rowGridNum) - 0.01
                
                // 改变网格宽度
                for (let i = 0 ; i < this.rowGridNum * this.rowGridNum ; ++i) {
                    this.gridStyle[i].width = this.gridWidth + 'px'
                    this.gridStyle[i].height = this.gridWidth + 'px'
                }
            },

            // 放大页面
            zoomIn() {
                // 解析百分制的width参数
                let width = parseInt(this.workspaceDivClass.width.replace('%', ''))
                if (width >= 300) {
                    this.$message({
                        type: 'error',
                        duration: 1500,
                        message: '不可放大'
                    });
                } else {
                    width += 10
                    this.workspaceDivClass.width = width + '%'
                    // 在这里读取offsetWidth仍会得到旧值
                    // 所以选择在mount里定义监听事件而非显式调用alterGridWidth()函数
                }
            },

            // 缩小页面
            zoomOut() {
                // 解析百分制的width参数
                let width = parseInt(this.workspaceDivClass.width.replace('%', ''))
                if (width <= 30) {
                    this.$message({
                        type: 'error',
                        duration: 1500,
                        message: '不可缩小'
                    });
                } else {
                    this.workspaceDivClass.width = width - 10 + '%'
                }
            },
            /*
            loadDiv() {
                // 获得中间区域的窗口参数
                let mainDiv = document.getElementById("div2");
                let totalWidth = mainDiv.offsetWidth;
                let totalHeight = mainDiv.offsetHeight;
                let totalCount = 0;

                //生成40*40的网格
                for(let w = 1;w*40<totalWidth;w++){
                    for(let h = 1;h*40<totalHeight;h++){
                        totalCount++;
                        let newDiv = this.createDiv(totalCount);
                        mainDiv.append(newDiv);
                    }  
                }
            },
            createDiv(id) {
                let basicdiv = document.createElement("div");
                basicdiv.id = id;
                basicdiv.style.width = "38px";
                basicdiv.style.height = "38px";
                basicdiv.className = "Area";
                // basicdiv.innerHTML = id;
                basicdiv.style.float = "left";
                basicdiv.style.border = "1px solid #aaaaaa";
                basicdiv.drop = "drop($event)";
                basicdiv.ondragenter=function(){
                    this.style.borderColor='red';
                }
                basicdiv.ondragover=function(ev){
                    ev.preventDefault()
                }
                basicdiv.ondragleave=function(){
                    this.style.borderColor='#aaaaaa'
                }
                basicdiv.ondrop=function(){
                    // location = this.id;
                    this.style.borderColor='#aaaaaa';
                    // this.innerHTML = this.innerHTML+"<p>"+msg.name;
                    //    targetLi.parentNode.removeChild(targetLi);
                }
                return basicdiv;
            }
            */
        }
    }
</script>

<style scoped>
    #MainArea {
        height:730px;
        border:1px solid #aaaaaa;
        overflow-y:auto
    }
    .el-icon-zoom-in {
        font-size: 30px;
        cursor: pointer
    }
    .el-icon-zoom-out {
        font-size: 30px;
        cursor: pointer
    }

</style>