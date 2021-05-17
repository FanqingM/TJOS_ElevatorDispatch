<template>
  <!-- 初始同时点击下和上有问题，但是上和下没问题 -->
  <!-- 原因是aimfloor在改变方向的时候没有更新，上下都有这个问题 -->
  <!-- 所以我们在update那里做了点处理 -->
  <!-- 还有点击后的css样式没搞好 -->
  <div id="lay">
    <el-card class="box-card">
      <el-card :class="{map1:isGoingUp===0, map2:!(isGoingUp===0)}" shadow="always">
        <div>
          <i class="el-icon-caret-top" v-show="this.isGoingUp===1"></i>
          <i class="el-icon-caret-top disabled" v-show="this.isGoingUp!==1"></i>
          {{ this.currentFloor+1}}F
          <i class="el-icon-caret-bottom" v-show="this.isGoingUp===-1"></i>
          <i class="el-icon-caret-bottom disabled" v-show="this.isGoingUp!==-1"></i>
        </div>
        <div>{{msg}}</div>
      </el-card>
      <div class="button-group">
        <div v-for="i in this.floorCount" :key="i" class="button-place">
        <el-button type="primary" :class="{button:buttonClicked[i-1] === false, button2:buttonClicked[i-1] === true}" @click="handleInsideButtonClick(i)">
            {{ i }}
          </el-button>
        </div>
      </div>
      <div class="control">
        <el-switch style="display: block" v-model="isOpen" active-color="#13ce66" inactive-color="#ff4949"
          active-text="开门" inactive-text="关门" class="switch" @click.native="doorAlert()">
        </el-switch>
      </div>
      <div class="accident">
        <el-button type="primary" icon="el-icon-phone" style="margin-left:-5px;background-color:#11d82b;border:none"  @click = "askForCommunicate()">
        </el-button>
        <el-button type="primary" icon="el-icon-message-solid"
          style="margin-left:80px;background-color:#e7370a;border:none" @click = "askForHelp()"></el-button>
      </div>
    </el-card>
    <el-card>
      <div v-for="i in this.floorCount" :key="i" class="floor-place">
        <el-button type="primary"  :class = "{elevatorFloor: (20-i) === currentFloor, elevatorFloor2:(20-i) != currentFloor}" >
            {{ 21-i }}
          </el-button>
        </div>
    </el-card>
  </div>
</template>

<script>
  export default {
    name: "card",
    props: {
          elevatorId: Number,                //电梯编号
          floorCount: Number,                //这里是20
          pressFloor: Number,                //父组件的按下楼层（注意，这里是外调度后选择的电梯）
          isUp: Boolean,                     //看看父组件按下的楼层是向上还是向下
          //当外部用户按下楼层后，父组件就向子组件传递pressFLoor和isUp
      },
    data() {
      return {
        buttonClicked: [...Array(20)].map(() => false),    // 内部请求队列,防止重复点击加入队列多个相同值，所以这么设置了
        currentFloor: 0,    //当前楼层-1
        isOpen: false,      //指门开了没
        aimFloor: 0,
        msg: "空闲中",
        isGoingUp: 0,
        isSelect: [...Array(20)].map(() => false),
        timer: null,    // 内部请求队列
        cnt: 0,
        selfColor:[{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'},{backgroundColor:'hsl(220, 8%, 93%)'}],
      //   selfcolor: {
      //       backgroundColor:'hsl(220, 8%, 93%)',
      //   },
      }
    },
    methods: {
      askForCommunicate()
      {
        alert("请求与电梯管理员通话☎️");
      },
      askForHelp()
      {
        alert("请求电梯管理员援助🔥")
      },
      isEmpty() {
        for (let i = 0; i < 20; i++) {
          if (this.isSelect[i]) {
            return false;
          }
        }
        return true;
      },
      updateStateToFather()
      {
          //当每次更新状态后都向父组件传递最新情况
          this.$emit(
              "elevatorStatus",
              this.elevatorId,
              this.currentFloor,
              this.isGoingUp,
              this.aimFloor,
          )
      },
      openDoor() {
        this.isOpen = true;
        this.msg = "快进出电梯哇";
        this.isSelect[this.currentFloor] = false;
        //改变了数组元素状态，我们要重新更新电梯状态
        this.buttonClicked[this.currentFloor] = false;
        this.updateState();
        if(this.isEmpty())
        {
            this.isGoingUp = 0;
        }
        this.updateStateToFather();
        console.log("此时目标楼层  " + this.aimFloor + "当前楼层  " + this.currentFloor);
        //我开门了之后是肯定要关门的
        console.log("此时电梯状态", this.isGoingUp);
        //有时我们到顶了，aim为设为了-1，isgoingup为-1了，但是后面关门时候检测空又改为0了
        setTimeout(() => {   //设置延迟执行
          this.closeDoor();
        }, 3000);
      },
      //我们规定只有关门才能按按纽
      closeDoor() {
        //这里是到达该层后
        this.isOpen = false;
        //注意在关门之前我们执行了pop操作
        //这里我们关门后需要判断还需不需要继续run
        //第一种情况是电梯上升或者下降中途停下，继续run，并且goingup维持原来
        //第二种情况是电梯上升或者下降到了aimFloor了，队列里面没东西了，就隔着听着，直接goingup为0，不run了
        //第三种情况是电梯上升到了aimFloor，但是还有下面楼层需要响应，改goingup再update在run
        if (this.isEmpty()) {
          this.msg = "空闲中";
          this.isGoingUp = 0;
          this.cnt = 0;
          //this.cnt = 0;
        }
        else {
          this.msg = "忙碌中";
          setTimeout(this.run, 1000);
        }

      },
      checkFromBottom() {
        for (let i = 0; i <= this.currentFloor; i++) {
          if (this.isSelect[i]) {
            return i;
          }
        }
        return -1;
      },
      checkFromTop() {
        for (let i = 19; i >= this.currentFloor; --i) {
          if (this.isSelect[i]) {
            return i;
          }
        }
        return -1;
      },
      updateState() {
        //再更改动态指标
        if (this.isGoingUp === 1) {
          //上升
          this.aimFloor = this.checkFromTop();
          if (this.aimFloor == -1) {
            //否则上升转下降aim就一直是-1
            this.aimFloor = this.checkFromBottom();
          }
        }
        else if (this.isGoingUp === -1) {
          //下降
          this.aimFloor = this.checkFromBottom();
          if (this.aimFloor == -1) {
            //否则下降转上升aim就一直是-1
            this.aimFloor = this.checkFromTop();
          }
        }
        else {
          //当电梯此时停下是，我们先从上向下
          //如果找不到在冲下岛上
          this.aimFloor = this.checkFromTop();
          if (this.aimFloor === -1) {
            this.aimFloor = this.checkFromBottom();
          }
        }
        if (this.currentFloor === this.aimFloor) {
          this.isGoingUp = 0;
        }
        else if (this.currentFloor > this.aimFloor) {
          this.isGoingUp = -1;
        }
        else {
          this.isGoingUp = 1;
        }
      },
      insertFromOutside(pressFloor) {
          //这里收到的pressFloor是父组件传过来的
          console.log("父组件触发我了,我是电梯(从1开始）", this.elevatorId)
          console.log("现在按下第几层？", pressFloor)
          this.isSelect[pressFloor] = true;
          this.updateState();
          if (this.cnt < 1) {
              //这里与内部点击处理是相同的
          this.run();
        }
      },
      handleInsideButtonClick(floor) {
        //先改数组信息
        this.$set(this.buttonClicked, floor - 1, true);
        console.log("内部队列我康康  ", this.isSelect);
        console.log("当前点击楼层（从1开始） " + floor);
        this.$set(this.isSelect, floor - 1, true);
        this.updateState(floor);
        this.updateStateToFather();
        //再进行run的逻辑
        console.log("当前电梯目标楼层  " + this.aimFloor);
        console.log("当前电梯状态 " + this.isGoingUp);
        if (this.cnt < 1) {
          //我们一个电梯只创建一个运行活动，不然计时器会冲突
          //但是我们当电梯停下来时就等价与进入初始状态，会把cnt重新为0
          console.log("cnt  " + this.cnt);
          this.run();
        }
      },
      //先下后上有问题
      //先上后下暂时未出现大bug
      run() {
        //let _this = this
        //run和update在很多情况下好像是连体婴儿
        //运行逻辑
        this.msg = "忙碌中";
        if (this.isGoingUp !== 0) {
          console.log("1目标楼层  " + this.aimFloor + "当前楼层  " + this.currentFloor);
          if (this.isGoingUp === 1) {
            //向上运动
            console.log("我的当前楼层要加一拉")
            //不加这个if会多一层
            if (!this.isSelect[this.currentFloor]) {
              setTimeout(() => {   //设置延迟执行
                this.currentFloor++;
                this.updateStateToFather()
              }, 1000);
            }
            console.log("目标楼层  " + this.aimFloor + "当前楼层  " + this.currentFloor);
            //如果这层是内部请求或者外部请求，外部请求先不考虑
            //需要在这里停下开门，再关门，再继续跑
            console.log(this.currentFloor);
            if (this.isSelect[this.currentFloor] === true) {
              setTimeout(() => {   //设置延迟执行
                this.openDoor();
              }, 1000);
            }
            else {
              //clearInterval(this.timer)
              //我们注意这里我们不采用interval来进行run，而是延迟递归调用
              //这样当else不会一路飞上去，到了aim后就停了，又回到了前面开门关门的逻辑
              setTimeout(this.run, 1000);
              this.cnt++;
              console.log("创建了计时器")
            }
          }
          else if (this.isGoingUp === -1) {
            //向上运动
            console.log("我的当前楼层要减一拉")
            //不加这个if会多一层
            if (!this.isSelect[this.currentFloor]) {
              setTimeout(() => {   //设置延迟执行
                this.currentFloor--;
                this.updateStateToFather()
              }, 1000);
            }
            console.log("目标楼层  " + this.aimFloor + "当前楼层  " + this.currentFloor);
            //如果这层是内部请求或者外部请求，外部请求先不考虑
            //需要在这里停下开门，再关门，再继续跑
            console.log(this.currentFloor);
            if (this.isSelect[this.currentFloor] === true) {
              setTimeout(() => {   //设置延迟执行
                this.openDoor();
              }, 1000);
            }
            else {
              //clearInterval(this.timer)
              //我们注意这里我们不采用interval来进行run，而是延迟递归调用
              //这样当else不会一路飞上去，到了aim后就停了，又回到了前面开门关门的逻辑
              setTimeout(this.run, 1000);
              this.cnt++;
              console.log("创建了计时器")
            }
          }
        }
      },
      doorAlert()
      {
        if(this.isGoingUp != 0)
        {
          alert("电梯正在运行不要开门呀🆘 ");
          this.isOpen = false;
        }
      },
    }
  }

</script>

<style>
  .button-group {
    width: 100%;
    height: 0;
    display: flex;
    flex-wrap: wrap;
  }

  .button-group .button-place {
    width: 50%;
    display: flex;
    justify-content: center;
    margin-bottom: 10px
  }

  .button-group .button-place .button .button-click {
    color: #ffffff;
    background: purple;
  }

  .button-group .button-place .button {
    width: 120px;
    color:black;
    background: #E8F2FF;
    
  }
  .button-group .button-place .button2 {
    width: 120px;
    color:black;
    background: #409eff;
  }

  #lay {
    display: flex;
    justify-content: flex-start;
  }


  .el-icon-caret-top {
    margin-right: 40px;
  }

  .el-icon-caret-bottom {
    margin-left: 40px;
  }

  .disabled {
    color: #152959;
  }

  .text {
    font-size: 14px;
  }

  .item {
    margin-bottom: 18px;
  }

  .clearfix:before,
  .clearfix:after {
    display: table;
    content: "";
  }

  .clearfix:after {
    clear: both
  }

  .box-card {
    width: 340px;
    height: 750px;
    padding: 10px;
  }

  .control {
    display: flex;
    justify-content: space-around;
    align-items: center;
    margin-top: 500px;
    margin-left: -5px;
  }

  .accident {
    margin-top: 30px;
  }

  .box-card .map1 {
    background-color: #409eff;
    text-align: center;
    font-size: 150%;
    color: #ffffff;
    margin-bottom: 20px;
  }

  .box-card .map2 {
    background: linear-gradient(to bottom left, #EF8D9C 40%, #FFC39E 100%);
    text-align: center;
    font-size: 150%;
    color: #ffffff;
    margin-bottom: 20px;
  }
  .floor-place .elevatorFloor {
    width:80px;
    height:35px;
    color:black;
    background: #409eff;
    pointer-events: none;
  }
  .floor-place .elevatorFloor2 {
    width:80px;
    height:35px;
    color:black;
    background: #E8F2FF;
    pointer-events: none;
  }
  /* .donghua
  {
    animation: move 10s linear;
    width:50px;
    height:50px;
    margin-top: 0px;
  }
  @keyframes move
  {
  0%   {margin-top:0px;}
  25%  {margin-top:200px;}
  50%  {margin-top:400px;}
  75%  {margin-top:600px;}
  100% {margin-top:0px;}
  } */
</style>