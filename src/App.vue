<template>
  <div id="app">
    <div id="outside-box">
      <div v-for="i in this.floorCount/2" :key="i">
        <outside :floorId="i" @outsideCallFloorUp="callUpHelp" @outsideCallFloorDown="callDownHelp"
          style="margin-left:20px" ref="outsideChild">
        </outside>
      </div>
    </div>
    <br>
    <br>
    <div id="outside-box">
      <div v-for="i in this.floorCount/2" :key="i">
        <outside :floorId="i+10" @outsideCallFloorUp="callUpHelp" @outsideCallFloorDown="callDownHelp"
          style="margin-left:14px" ref="outsideChild">
        </outside>
      </div>
    </div>
    <div id="inside-box">
      <div v-for="o in this.elevatorNum" :key="o">
        <elevator :elevatorId=o :floorCount="floorCount" :pressFloor="pressFloor" :isUp="isUp"
          @elevatorStatus="getElevatorStatus" ref="elevatorChild">
        </elevator>
      </div>
    </div>
  </div>
</template>
<!-- 从elevator的儿子传爸爸好像是完成了 -->
<!-- 从outside的儿子传爸爸好像也完成了 -->
<!-- 下面写Calculate函数咯 -->
<!-- Calculate结束，就是把press这一层加到某一elevator的消息对列里面，并且这里要注意避免重复添加，这就体现isselect【20】的优越性 -->
<script>
  import elevator from "./components/elevator"
  //  import card from "./components/card"
  import outside from "./components/outside"
  //import elevator from "./components/elevator"
  export default {
    name: 'App',
    components: {
      //elevator,
      //  card,
      elevator,
      outside,
    },
    data() {
      return {
        elevatorNum: 5,
        floorCount: 20,
        isAllNotOk: false,
        elevatorIsOK: [...Array(5)].map(() => true),
        cost: [...Array(5)].map(() => 0),
        aimFloor: [...Array(5)].map(() => 0),
        goingUp: [...Array(5)].map(() => 0),              //每个电梯的状态，1代表上升，-1代表下降，0代表禁止
        currentFloor: [...Array(5)].map(() => 0),        //每个电梯当前的楼层-1,需要收到请求
        CallElevatorId: 0,        //由哪个电梯来回应请求   
        pressFloor: -1,          //代表被按下的楼层-1（因为子组件中的isselect是从0开始的）
        isUp: false,             //这个只有在pressFloor不是-1时才有意义
      }
    },
    methods: {
      insertToElevator() {
        //把pressfloor加到指定的电梯的isselect里面
        //闲麻烦，直接ref了
        this.$refs.elevatorChild[this.CallElevatorId].insertFromOutside(
          this.pressFloor
        )
        console.log("传入第几个电梯（从0开始）：",
                this.CallElevatorId,
                "第几层楼",
                this.pressFloor)
      },
      calculateCallElevatorId() {
        //在callhelp后使用
        if (this.isUp) {
          //用户需要上楼
          for (let i = 0; i < this.elevatorNum; i++) {
            if (this.goingUp[i] === 0) {
              //如果此电梯静止
              this.cost[i] = Math.abs(this.currentFloor[i] - this.pressFloor);
            }
            else if (this.goingUp[i] === 1) {
              //如果电梯上升
              if (this.pressFloor >= this.currentFloor[i]) {
                //若某电梯正在向上运动并且用户选择楼层大于等于该电梯当前楼层
                this.cost[i] = this.pressFloor - this.currentFloor[i];
              }
              else {
                //若某电梯正在向上运动并且用户选择楼层小于该电梯当前楼层
                this.cost[i] = this.aimFloor[i] - this.currentFloor[i] + this.aimFloor[i] - this.pressFloor;
              }
            }
            else {
              //如果电梯下降
              if (this.pressFloor <= this.currentFloor[i]) {
                //若某电梯正在向下运动并且用户选择楼层小于等于该电梯当前楼层
                this.cost[i] = this.currentFloor[i] - this.aimFloor[i] + this.pressFloor - this.aimFloor[i];
              }
              else {
                //若某电梯正在向下运动并且用户选择楼层大于该电梯当前楼层
                this.cost[i] = this.currentFloor[i] - this.aimFloor[i] + this.pressFloor - this.aimFloor[i];
              }
            }
          }
        }
        else {
          //说明用户想向下走
          for (let i = 0; i < this.elevatorNum; i++) {
            if (this.goingUp[i] === 0) {
              //如果此电梯静止
              this.cost[i] = Math.abs(this.currentFloor[i] - this.pressFloor);
            }
            else if (this.goingUp[i] === 1) {
              //如果电梯上升
              if (this.pressFloor >= this.currentFloor[i]) {
                //若某电梯正在向上运动并且用户选择楼层大于等于该电梯当前楼层
                this.cost[i] = this.aimFloor[i] - this.currentFloor[i] + this.aimFloor[i] - this.pressFloor;
              }
              else {
                //若某电梯正在向上运动并且用户选择楼层小于该电梯当前楼层
                this.cost[i] = this.aimFloor[i] - this.currentFloor[i] + this.aimFloor[i] - this.pressFloor;
              }
            }
            else {
              //如果电梯下降
              if (this.pressFloor <= this.currentFloor[i]) {
                //若某电梯正在向下运动并且用户选择楼层小于等于该电梯当前楼层
                this.cost[i] = Math.abs(this.currentFloor[i] - this.pressFloor);
              }
              else {
                //若某电梯正在向下运动并且用户选择楼层大于该电梯当前楼层
                this.cost[i] = this.currentFloor[i] - this.aimFloor[i] + this.pressFloor - this.aimFloor[i];
              }
            }
          }
        }
        //以上是初始化cost数组，下面求出最小的
        //注意这里最小的是在没有故障的电梯中找的
        
        var min;
        var min_id;
        for(let i=0;i<5;i++)
        {
          if(this.elevatorIsOK[i] === true)
          {
            min = this.cost[i];
            min_id = i;
            break;
          }
        }
        for (let i = 0; i < 5; i++) {
          if(this.elevatorIsOK[i] === false)
          {
            continue;
          }
          else 
          {
              if (this.cost[i] < min) {
              min = this.cost[i];
              min_id = i;
            }
          }
        }
        this.CallElevatorId = min_id;
        console.log("让我康康谁来接我", this.CallElevatorId);
      },
      getElevatorStatus(
        elevator_id,
        elevator_currentFloor,
        elevator_isGoingUp,
        elevator_aimFloor,
        elevator_isOk,
      ) {
        //this.isOk[elevator_id - 1] = elevator_isOk;
        //有个问题currentFloor没有实时更新，所以我们在elevator里面currentfloor变也需要上报父组件
        var i = 0;
        this.elevatorIsOK[elevator_id - 1] = elevator_isOk;
        this.currentFloor[elevator_id - 1] = elevator_currentFloor;
        //有个问题goingUp在停下来的时候没有变零
        this.goingUp[elevator_id - 1] = elevator_isGoingUp;
        this.aimFloor[elevator_id - 1] = elevator_aimFloor;
        for(i=0;i<5;i++)
        {
          if(this.elevatorIsOK[i])
          {
            break;
          }
        }
        if(i==5)
        {
          this.isAllNotOk = true;
        }
      },
      callUpHelp(floor_id) {
        if(this.isAllNotOk === false)
        {
          //我们希望这个也从零开始，这样计算方便
          this.pressFloor = floor_id - 1;
          this.isUp = true;
          //下面经过计算函数，把这个需求分配给某一个电梯，即加入这个电梯的isselect
          //我们希望只在现在可以跑的电梯里面找
          this.calculateCallElevatorId()
          this.insertToElevator();
        }
        else
        {
          alert("所有电梯都坏啦🆘");
        }
      },
      callDownHelp(floor_id) {
        if(this.isAllNotOk === false)
        {
          this.pressFloor = floor_id - 1;
          this.isUp = false;
          this.calculateCallElevatorId()
          this.insertToElevator();
        }
        else
        {
          alert("所有电梯都坏啦🆘");
        }
      },
    }
  }
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }

  #inside-box {
    margin-top: 30px;
    display: flex;
    justify-content: flex-start;
  }

  #outside-box {
    display: flex;
    justify-content: center;
  }
</style>