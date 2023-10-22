<template>
  <div id="app">
    <button @click="change()">{{ up_btn }}</button>
    <div class="calculator" v-show="show1">
      <div class="result" style="grid-area: result">{{ equation }}</div>
      <button style="grid-area: s" @click="append('Math.sin(')">sin</button>
      <button style="grid-area: c" @click="append('Math.cos(')">cos</button>
      <button style="grid-area: t" @click="append('Math.tan(')">tan</button>
      <button style="grid-area: l" @click="append('(')">(</button>
      <button style="grid-area: r" @click="append(')')">)</button>
      <button style="grid-area: g" @click="append('Math.sqrt(')">√</button>
      <button style="grid-area: ans" @click="getLast()">ans</button>
      <button style="grid-area: del" @click="deleteLastCharacter()">DEL</button>
      <button style="grid-area: ac" @click="clear()">AC</button>
      <button style="grid-area: add" @click="append('+')">+</button>
      <button style="grid-area: subtract" @click="append('-')">-</button>
      <button style="grid-area: multiply" @click="append('*')">*</button>
      <button style="grid-area: divide" @click="append('/')">/</button>
      <button style="grid-area: equal" @click="calculate()">=</button>
      <button style="grid-area: number-1" @click="append('1')">1</button>
      <button style="grid-area: number-2" @click="append('2')">2</button>
      <button style="grid-area: number-3" @click="append('3')">3</button>
      <button style="grid-area: number-4" @click="append('4')">4</button>
      <button style="grid-area: number-5" @click="append('5')">5</button>
      <button style="grid-area: number-6" @click="append('6')">6</button>
      <button style="grid-area: number-7" @click="append('7')">7</button>
      <button style="grid-area: number-8" @click="append('8')">8</button>
      <button style="grid-area: number-9" @click="append('9')">9</button>
      <button style="grid-area: number-0" @click="append('0')">0</button>
      <button style="grid-area: dot" @click="append('.')">.</button>
    </div>
    <div class="rate_calculator" v-show="show2">
      <h3 class="text" style="grid-area: t1">输入存款金额</h3>
      <input class="textbox" style="grid-area: t2" type="number" placeholder="输入存款金额" v-model="dep_amount">
      <h3 class="text" style="grid-area: t3">输入存储时间（年）</h3>
      <input class="textbox" style="grid-area: t4" type="number" placeholder="输入存储时间（年）" v-model="dep_time">
      <button style="grid-area: t5" @click="cal1()">计算</button>
      <input class="textbox" style="grid-area: t6" type="number" placeholder="利息" v-model="dep_int">
      <h3 class="text" style="grid-area: t7">输入贷款金额</h3>
      <input class="textbox" style="grid-area: t8" type="number" placeholder="输入贷款金额" v-model="loan_amount">
      <h3 class="text" style="grid-area: t9">输入贷款时间（年）</h3>
      <input class="textbox" style="grid-area: t10" type="number" placeholder="输入贷款时间（年）" v-model="loan_time">
      <button style="grid-area: t11" @click="cal2()">计算</button>
      <input class="textbox" style="grid-area: t12" type="number" placeholder="贷款" v-model="loan_int">
    </div>
  </div>
</template>
<script>
import axios from "axios";
export default {
  data() {
    return {
      dep_amount:undefined,
      dep_time:undefined,
      dep_int:undefined,
      loan_amount:undefined,
      loan_time:undefined,
      loan_int:undefined,
      show1:true,
      show2:false,
      up_btn:"基本计算器",
      equation: "0",
      isDecimalAdded: false,
      isOperatorAdded: false,
      isStarted: false,
    };
  },
  methods: {
    cal2(){
      axios({
        method:'post',
        url:'/api/cal2',
        data:{
          loan_amount:this.loan_amount,
          loan_time:this.loan_time,
        }
      }).then(res=>{
        console.log(res.data);
        this.loan_int=res.data;
      })
    },
    cal1(){
      axios({
        method:'post',
        url:'/api/cal1',
        data:{
          dep_amount:this.dep_amount,
          dep_time:this.dep_time,
        }
      }).then(res=>{
        console.log(res.data);
        this.dep_int=res.data;
      })
    },
    change(){
      this.show1=!this.show1;
      this.show2=!this.show1;
        if(this.up_btn=="基本计算器"){
          this.up_btn="利率计算器";
        }
        else{
          this.up_btn="基本计算器";
        }
    },
    getLast(){
      console.log(1);
      axios({
        method:'get',
        url:'/api/getLast',
      }).then(res=>{
        console.log(res.data)
        if(this.equation=="0"){
          this.equation=res.data.data.result;
        }
        else if(res.data.data.result=="Error"){
          this.equation="Error";
        }
        else{
          this.equation+=res.data.data.result;
        }
        
      
      })
      
    },
    append(character) {
      if (this.isOperator(character)) {
        this.equation += character;
        this.isDecimalAdded = false;
        this.isOperatorAdded = true;
      } else {
        if (this.equation === "0" && character !== ".") {
          this.equation = "";
        }
        if (character === ".") {
          if (!this.isDecimalAdded) {
            this.equation += character;
            this.isDecimalAdded = true;
          }
        } else {
          this.equation += character;
        }
        this.isOperatorAdded = false;
      }
    },
    deleteLastCharacter() {
      if (this.equation.length > 1) {
        this.equation = this.equation.slice(0, -1);
      } else {
        this.equation = "0";
        this.isDecimalAdded = false;
        this.isOperatorAdded = false;
        this.isStarted = false;
      }
    },
    calculate() {
      try {
        const res = eval(this.equation);
        const result = res.toFixed(3);
        if (isNaN(result)) {
          throw new Error("Invalid expression");
        }

        axios({
          url:'http://localhost:8080/save',
          method:'post',
          data:{
            equation:this.equation,
            result:result.toString(),
          }
        }).then(res=>{
          console.log(res);
        })

        this.equation = result.toString();
      } catch (error) {
        console.error(error);
        axios({
          url:'http://localhost:8080/save',
          method:'post',
          data:{
            equation:this.equation,
            result:"Error",
          }
        }).then(res=>{
          console.log(res);
        })
        this.equation = "Error";
      }
    },
    clear() {
      this.equation = "0";
      this.isDecimalAdded = false;
      this.isOperatorAdded = false;
      this.isStarted = false;
    },
    isOperator(character) {
      return ["+", "-", "*", "/"].includes(character);
    },
  },
};
</script>


<style>
body {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;

  background-color: #eee;
}
.rate_calculator{
  display: grid;
  grid-template-areas:
    "t1 t2"
    "t3 t4"
    "t5 t6"
    "t7 t8"
    "t9 t10"
    "t11 t12";
}
.calculator {
  --button-width: 100px;
  --button-height: 80px;
  display: grid;
  grid-template-areas:
    "result result result result result"
    "s l r del ac"
    "c number-7 number-8 number-9 divide"
    "t number-4 number-5 number-6 multiply"
    "g number-1 number-2 number-3 subtract"
    "ans number-0 dot equal add";
  grid-template-columns: repeat(5, var(--button-width));
  grid-template-rows: repeat(6, var(--button-height));
  box-shadow: -8px -8px 16px -10px rgba(255, 255, 255, 1),
    8px 8px 16px -10px rgba(0, 0, 0, 0.15);
  padding: 24px;
  border-radius: 20px;
}
.calculator button {
  margin: 8px;
  padding: 0;
  border: 0;
  display: block;
  outline: none;
  border-radius: calc(var(--button-height) / 2);
  font-size: 24px;
  font-family: Helvetica;
  font-weight: normal;
  color: #999;
  background: linear-gradient(
    135deg,
    rgba(230, 230, 230, 1) 0%,
    rgba(246, 246, 246, 1) 100%
  );
  box-shadow: -4px -4px 10px -8px rgba(255, 255, 255, 1),
    4px 4px 10px -8px rgba(0, 0, 0, 0.3);
}
.calculator button:active {
  box-shadow: -4px -4px 10px -8px rgba(255, 255, 255, 1) inset,
    4px 4px 10px -8px rgba(0, 0, 0, 0.3) inset;
}
.result {
  text-align: right;
  line-height: var(--button-height);
  font-size: 48px;
  font-family: Helvetica;
  padding: 0 20px;
  color: #666;
}
</style>
