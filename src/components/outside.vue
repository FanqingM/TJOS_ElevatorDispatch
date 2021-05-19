<!-- 20层楼每层的外部电梯就是一个上一个下的案件 -->
<!-- 每一次在外部按下上楼和下楼健我们需要选择一个楼梯去响应他，然后将其加入这个楼梯的消息队列中 -->
<!-- 我们这里采用点击后给父组件发信号，父组件进行外调度算法的计算，将结果告知给elevator组件 -->
<!-- 关于外调度算法，如果采用单独的计时器进行弱实时调度很可能会造成饥饿问题，所以这里以用户响应优先（可能也不是最高响应比，但肯定不会饥饿） -->
<template>
    <div>
        <div>
            <el-tag type="" style="background-color:rgba(45, 133, 173, 0.178);">{{this.floorId}}</el-tag>
            <el-button-group>
                <el-button type="primary" icon="el-icon-top" size="small" @click="handleUpClick()"></el-button>
                <el-button type="primary" icon="el-icon-bottom" size="small" @click="handleDownClick()"></el-button>
            </el-button-group>
            <!-- 这里这个isopen对应着elevator到达了我们按的这一层，进行开门 -->
            <!-- 所以实际上是elevator中的isopen传值到父组件，父组件在传给outside，outside与这个接到的东西动态绑定 -->
            <!-- <el-switch style="display: block;margin-top:10px;"  active-color="#13ce66"
                inactive-color="#ff4949" active-text="开门" inactive-text="关门" class="switch">
            </el-switch> -->
        </div>
    </div>
</template>

<script>
    export default {
        name: "outside",
        props: {
            floorId: Number
        },
        data() {
            return {
                floorCount: 20,   //这是测试，后面会从父组件传值
                //floorId:1,       //同上
                isOpen: [],       //20层的每个门是否开
            }
        },
        methods: {
            handleUpClick() {
                
                if(this.floorId < 20)
                {
                    console.log("我这一层向上请求支援",this.floorId);
                    this.$emit("outsideCallFloorUp", this.floorId)
                }
                else
                {
                    alert("您已经到顶了啦🆘")
                }
            },
            handleDownClick() {
                if(this.floorId > 1)
                {
                    console.log("我这一层向下请求支援",this.floorId);
                    this.$emit("outsideCallFloorDown", this.floorId);
                }
                else
                {
                    alert("您已经到底了啦🆘")
                }
            },
        }
    }
</script>

<style>

</style>