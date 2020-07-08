<template>
  <div>
    <p v-if="showFlag">
      <button
        v-for="(item,index) in list"
        :key="'phoneHandle-'+index"
        class="phoneButton"
        :style="{background:item.backgroundColor}"
        @click="changeState(index)"
      >{{item.text}}</button>
    </p>
    <p>
      剩余地雷数:{{allBumbNum}},
      步数:{{stepNum}},
      时间:{{time}}
    </p>
    <div class="container" :style="{width:containerWith}">
      <div v-for="(y,yIndex) in showArr" :key="'y-'+yIndex" class="row">
        <div
          v-for="(x,xIndex) in y"
          :key="'x-'+xIndex"
          class="item"
          @click="showItem(yIndex,xIndex,'person')"
          @contextmenu.prevent="hereHaveBomb(yIndex,xIndex)"
        >{{x}}</div>
      </div>
    </div>
    <button @click="changeD(10)">10*10</button>
    <button @click="changeD(20)">20*20</button>
    <button @click="changeD(30)">30*30</button>
    <button @click="showRealArr" style="margin:20px;">显示全部雷</button>
    <button @click="init" style="margin:20px;">重玩</button>

    <div v-show="gameOver" class="game-over">
      <div class="game-over-close" @click="finishGame">X</div>
      <div>{{finishTest}}</div>
    </div>
  </div>
</template>

<script>
import { ref, watch, computed } from "vue";
export default {
  setup() {
    //大小 d * d;
    let d = ref(20);
    watch(d, () => {
      init();
    });
    // 雷的密度 (p = bumbP/1)
    let bumbP = 0.15;
    let realTime = ref(0); //显示在界面上的时间
    // computed格式化时间
    const time = computed(() => {
      let result = '';
      let m =Math.floor(realTime.value/60);
      let s = realTime.value - 60*m;
      m = 10 > m > 0 ? `0${m}` : m;
      s = 10 > s > 0 ? `0${s}` : s;
      result = m === '00' ? `${s}秒` : `${m}:${s}分`
      return result;
    });

    let allTimer = null; //定时器
    // 地雷总数
    let allBumbNum = ref(0);
    watch(allBumbNum, n => {
      if (n === 0) {
        gameOver.value = true;
        finishTest.value = "You Win!";
      }
    });
    // 游戏是否结束
    let gameOver = ref(false);
    let stepNum = ref(0);

    let finishTest = ref("");
    //是不是PC端
    let showFlag = ref(false);
    let isPC = () => {
      //是否为PC端
      let userAgentInfo = navigator.userAgent;
      let Agents = [
        "Android",
        "iPhone",
        "SymbianOS",
        "Windows Phone",
        "iPad",
        "iPod"
      ];
      let flag = true;
      for (var v = 0; v < Agents.length; v++) {
        if (userAgentInfo.indexOf(Agents[v]) > 0) {
          flag = false;
          break;
        }
      }
      return flag;
    };
    let list = ref([
      {
        text: "翻开",
        backgroundColor: "#E91E63"
      },
      {
        text: "插旗",
        backgroundColor: ""
      }
    ]);
    
    

    if (!isPC()) {
      showFlag.value = true;
      d.value = 10;
    }
    //翻开还是插旗
    let openOrFalg = '翻开'; 
    const changeState = index => {
       //改变状态
       openOrFalg = list.value[index].text;
       for(let i=0;i<list.value.length;i++){
          if(list.value[i].backgroundColor != ''){
            list.value[i].backgroundColor = ''
          }
       }
       list.value[index].backgroundColor = "#E91E63";
    };
    // 显示的空白数组
    let arr = e => {
      let arr = [];
      for (let i = 0; i < e; i++) {
        arr.push(new Array(e).fill(""));
      }
      return arr;
    };
    let showArr = ref(arr(d.value));
    //创建存储雷的数组
    let makeRealArr = () => {
      let tempArr = arr(d.value);
      allBumbNum.value = 0;
      for (let i = 0; i < tempArr.length; i++) {
        for (let j = 0; j < tempArr[0].length; j++) {
          if (Math.random() < bumbP) {
            tempArr[i][j] = "💣";
            allBumbNum.value++;
          }
        }
      }
      return tempArr;
    };
    let realArr = makeRealArr();
    // 坐标{findy,findx}处是否有雷
    let find = (findy, findx) => {
      if (realArr[findy][findx] === "💣") {
        return 1;
      } else {
        return 0;
      }
    };
    // {y,x}周围的八个格子
    const forArr = [
      [-1, -1],
      [-1, 0],
      [-1, 1],
      [0, -1],
      [0, 1],
      [1, -1],
      [1, 0],
      [1, 1]
    ];
    //  计算坐标{y,x}周围的雷的个数
    let aroundBombNum = (y, x) => {
      let sum = 0;
      for (let i = 0; i < forArr.length; i++) {
        let findY = y + forArr[i][0];
        let findX = x + forArr[i][1];
        if (findX >= 0 && findY >= 0 && findY < d.value && findX < d.value) {
          // console.log(find(findY, findX))
          if (find(findY, findX)) {
            sum++;
          }
        }
      }
      return sum;
    };
     // 右键插旗
    let hereHaveBomb = (y, x) => {
      stepNum.value++;
      if (realArr[y][x] === "💣") {
        showArr.value[y][x] = "⛳";
        allBumbNum.value--;
      } else {
        gameOver.value = true;
        finishTest.value = "失败！此处无雷";
      }
    };
    // 左键点击
    let showItem = (y, x, isPerson) => {
      // 如果是移动端的点击
      if(!isPC()){
        if(openOrFalg == '插旗'){
             hereHaveBomb(y,x);
             return
        }
      }
      // 如果第一次点击 翻开周围不是雷的
      // 开启一个定时器
      if (stepNum.value === 0) {
        allTimer = setInterval(() => {
          realTime.value ++;
        }, 1000);
        for (let i = 0; i < forArr.length; i++) {
          let findY = y + forArr[i][0];
          let findX = x + forArr[i][1];
          if (!find(findY, findX)) {
            showArr.value[findY][findX] = aroundBombNum(findY, findX);
          }
        }
      }
      // 是用户点击的 给步数加一
      if (isPerson) {
        stepNum.value++;
      }
      //{y,x}处是雷  游戏结束
      if (find(y, x) && isPerson === "person") {
        showArr.value[y][x] = "💣";
        allBumbNum.value--;
        gameOver.value = true;
        finishTest.value = "失败！";
        return;
      }

      // 不是雷 计算周围雷的个数

      showArr.value[y][x] = aroundBombNum(y, x);
      // 如果个数为0，则翻开周围，依次递归
      if (aroundBombNum(y, x) === 0) {
       
        for (let i = 0; i < forArr.length; i++) {
          let findY = y + forArr[i][0];
          let findX = x + forArr[i][1];
          //边界条件
          if (findX >= 0 && findY >= 0 && findY < d.value && findX < d.value) {
            // 格子为空再进行计算
            if (showArr.value[findY][findX] === "") {
              let aroundNum = aroundBombNum(findY, findX);
              showArr.value[findY][findX] = aroundNum;
              if (showArr.value[findY][findX] === 0) {
               
                showItem(findY, findX);
              }
            }
          }
        }
       
      } else {
        return;
      }
    };
    let timer = null;
    let showRealArr = () => {
      let tempShowArr = showArr.value;
      showArr.value = realArr;
      timer = setTimeout(() => {
        showArr.value = tempShowArr;
        clearTimeout(timer);
      }, 1000);
    };
   
    let containerWith = ref(`${21 * d.value}px`);
    //初始化
    let init = () => {
      showArr.value = arr(d.value); // 显示的数组
      clearInterval(allTimer); //清除定时器
      realTime.value = 0; // 时间归零
      containerWith.value = `${21 * d.value}px`; // 边长
      realArr = makeRealArr(); // 生成雷区
      stepNum.value = 0; // 步长清零
    };

    let finishGame = () => {
      gameOver.value = false;
      init();
    };
    let changeD = e => {
      d.value = e;
    };
    return {
      showFlag, //是PC端吗
      list, //移动端翻开还是插旗
      changeState, //改变翻开还是插旗状态
      d, //边长
      allBumbNum, // 地雷总数
      stepNum, //步数
      changeD, //改变边长
      showArr, // 显示在界面上的数组
      showItem, // 左键单击
      hereHaveBomb, //右键插旗
      showRealArr, // 显示全部的雷
      init, //重新初始化
      containerWith, // 棋盘总边长
      gameOver, //控制弹框
      finishTest, //游戏结束提示
      finishGame, //游戏结束
      time //游戏时长
    };
  }
};
</script>


<style scoped>
.container {
  /* width: 420px; */
  border-left: solid 1px #263238;
  border-top: solid 1px #263238;
  margin: 0 auto;
  background: #9e9e9e;
}
.row {
  height: 20px;
  display: flex;
}
.item {
  margin: 0;
  padding: 0;
  width: 20px;
  height: 20px;
  border-right: solid 1px #263238;
  border-bottom: solid 1px #263238;
  font-size: 10px;
  line-height: 21px;
  top: 0;
}
.game-over {
  position: fixed;
  top: 200px;
  width: 200px;
  left: 50%;
  background: #f5f5f5;
  height: 100px;
  transform: translate(-50%, -50%);
  font-weight: bold;
}
.game-over-close {
  position: relative;
  width: 95%;
  text-align: right;
  margin-right: 5%;
  height: 40px;
  line-height: 40px;
}
.phoneButton {
  outline: none;
  border: none;
}
</style>