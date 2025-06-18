<script>
export default {
  data() {
    return {
      msg: 'Hello World!',
      mode:'',
      health:100,
      energy:100,
      skill1CD:0,
      skill2CD:0,
      vulnStack:0,
      vulnTimer:0,
      nextBigAdd:35,
      selfExplosionCasting:false,
      bigAddCasting:false,
      gameInterval:null,
      smoothInterval:null,
      selfExplosionInterval:null,
      selfExplosionCastInterval:null,
      bigAddPrepareInterval:null,
      bigAddCastInterval:null,
      selfExplosionTimer:null,
      bigAddTimer:null,
    }
  },
  mounted() {
    document.addEventListener( "keydown", this.onKeydown );
  },
  methods: {
    startGame: function (selectedMode) {
      // `this` inside methods point to the Vue instance
      this.mode=selectedMode;
      document.getElementById("mode-selection").style.display = 'none';
      document.getElementById("game-ui").style.display = 'block';
      document.getElementById("restart-btn").style.display = 'none';
      document.getElementById("big-add-prepare").innerText = '';
      document.getElementById("stop-dps").style.display = 'none';
        if(this.mode === "advanced") document.getElementById("big-add-section").style.display = 'block';
        else document.getElementById("big-add-section").style.display = 'none';

        this.health = 100; this.energy = 100; this.skill1CD = 0; this.skill2CD = 0; this.vulnStack = 0; this.vulnTimer = 0; 
        this.nextBigAdd = 35 - this.getRndInteger(0,20);
        this.updateAll();
        this.log("游戏开始！");
        this.gameInterval = setInterval(this.gameLoop, 1000);
        this.smoothInterval = setInterval(this.updateAll, 50);
        this.startSelfExplosionLoop();
        this.startBigAddPrepare();
    },
    getRndInteger(min, max) {
      return Math.floor(Math.random() * (max - min + 1) ) + min;
    },
    gameLoop() {
      if (!this.selfExplosionCasting && !this.bigAddCasting) {
        if (this.mode !== "advanced") {
          const hpLoss = (this.health > 25 ? (this.mode === "dps_focus" ? 3 : 2) : (this.health > 20 ? 1 : 1));
          this.health = Math.max(this.health - hpLoss, 0);
          document.getElementById("stop-dps").style.display = (this.health <= 25) ? 'block' : 'none';
        }else{
          this.health = 20;
        }
        this.energy = Math.max(this.energy - 2, 0);
      }
      if (this.vulnTimer > 0) {
        this.vulnTimer--;
        if (this.vulnTimer === 0) this.vulnStack = 0;
      }
      if (this.health <= 0 || this.energy <= 0) this.gameOver("Game Over! 你已死亡！");
      if (this.skill1CD > 0) this.skill1CD--;
      if (this.skill2CD > 0) this.skill2CD--;
    },
    updateAll() {
      document.getElementById("skill4").style.display = 'inline-block';
      document.getElementById("health-bar").style.width = this.health + "%";
      document.getElementById("health-bar").innerText = Math.floor(this.health) + "%";
      document.getElementById("energy-bar").style.width = this.energy + "%";
      document.getElementById("energy-bar").innerText = Math.floor(this.energy).toString();
      document.getElementById("skill1-cd").style.background = `conic-gradient(rgba(0,0,0,0.5) ${this.skill1CD/6*360}deg, transparent 0deg)`;
      document.getElementById("skill2-cd").style.background = `conic-gradient(rgba(0,0,0,0.5) ${this.skill2CD/6*360}deg, transparent 0deg)`;
      document.getElementById("skill1").classList.toggle("disabled", this.skill1CD > 0);
      document.getElementById("skill2").classList.toggle("disabled", this.skill2CD > 0);
      document.getElementById("vuln-buff").innerText = `易伤: ${this.vulnStack*10}% (剩余时间: ${this.vulnTimer}s)`;
       document.getElementById("bidaddcdremain").innerText = ` (${this.nextBigAdd}s)`;
    },
    log(msg) { 
      document.getElementById("log").innerText = msg; 
    },
    gameOver(reason) {
      clearInterval(this.gameInterval); clearInterval(this.smoothInterval);
      clearInterval(this.selfExplosionInterval); clearInterval(this.bigAddPrepareInterval); clearInterval(this.bigAddCastInterval);
      clearInterval(this.selfExplosionCastInterval);
      this.log(reason);
      if(reason != '游戏胜利！恭喜你！'){
         document.getElementById("restart-btn").style.display = 'inline-block';
      }
      document.getElementById("skill4").style.display = 'none';
    },
    resetGame() {
      document.getElementById("game-ui").style.display = 'none';
      document.getElementById("mode-selection").style.display = 'block';
      document.getElementById("restart-btn").style.display = 'none';
    },
    startSelfExplosionLoop() {
      this.selfExplosionInterval = setInterval(() => { if (!this.selfExplosionCasting) this.startSelfExplosion(); }, 15000);
    },
    startSelfExplosion() {
      this.selfExplosionCasting = true;
      this.log("自身爆炸读条中！");
      this.selfExplosionTimer = 20;
      this.selfExplosionCastInterval = setInterval(() => {
        this.selfExplosionTimer--;
        document.getElementById("explosion-bar").style.width = (100 - this.selfExplosionTimer*5) + "%";
        if (this.selfExplosionTimer <= 0) {
          clearInterval(this.selfExplosionCastInterval);
          if (this.selfExplosionCasting) this.gameOver("Game Over! 队友团灭！");
        }
      }, 100);
    },
    startBigAddPrepare() {
      this.bigAddPrepareInterval = setInterval(() => {
        this.nextBigAdd--;
        document.getElementById("big-add-prepare").innerText = (this.nextBigAdd <= 10) ? `大怪即将读条：${this.nextBigAdd}s` : '';
        if (this.nextBigAdd <= 0) {
          clearInterval(this.bigAddPrepareInterval);
          this.startBigAdd();
        }
      }, 1000);
    },
    startBigAdd() {
      this.bigAddCasting = true;
      this.log("大怪爆炸读条中！");
      this.bigAddTimer = 20;
      this.bigAddCastInterval = setInterval(() => {
        this.bigAddTimer--;
        document.getElementById("big-add-bar").style.width = (100 - this.bigAddTimer*5) + "%";
        if (this.bigAddTimer <= 0) {
          clearInterval(this.bigAddCastInterval);
          if (this.bigAddCasting) this.gameOver("Game Over! 大怪爆炸引爆！");
        }
      }, 100);
    },
    skill1() {
      if (this.skill1CD > 0) return;
      this.vulnStack++; this.vulnTimer = 10;
      if (this.mode === "advanced" && this.bigAddCasting) {
        this.log("成功打断大怪爆炸！");
        this.bigAddCasting = false;
        clearInterval(this.bigAddCastInterval);
        document.getElementById("big-add-bar").style.width = "0%";
        document.getElementById("big-add-prepare").innerText = '';
        this.nextBigAdd = 35;
        this.startBigAddPrepare();
      } else {
        this.log("使用易伤！ 易伤已叠加至 " + (this.vulnStack*10) + "%");
      }
      this.skill1CD = 6;
    },
    skill2() {
      if (this.skill2CD > 0) return;
      if (this.energy < 8) { this.log("能量不足无法使用技能2"); return; }
      this.energy -= 8;
      if (this.selfExplosionCasting) {
        this.log("成功打断自身爆炸！");
        this.selfExplosionCasting = false;
        clearInterval(this.selfExplosionCastInterval);
        document.getElementById("explosion-bar").style.width = "0%";
      } else {
        this.log("使用打断技能！");
      }
      this.skill2CD = 6;
    },
    skill3() { this.health = Math.min(this.health + 10, 100); this.energy = Math.min(this.energy + 20, 100); this.log("吃水成功！"); },
    skill4() { 
      if (this.health > 20) { 
        this.log("血量大于20%，无法脱离！"); 
        return; 
      } 
      this.gameOver("游戏胜利！恭喜你！"); 
    },
    onKeydown( e ) {
      if (e.key === "1") this.skill1(); if (e.key === "2") this.skill2(); if (e.key === "3") this.skill3(); if (e.key === "4") this.skill4();
    },
  }
}



</script>
  <style>
    body { font-family: Arial, sans-serif; background: #f4f4f4; text-align: center; margin: 50px; }
    h1 { font-size: 32px; margin-bottom: 20px; }
    .bar-container { width: 400px; background: #ddd; border-radius: 10px; margin: 10px auto; height: 30px; position: relative; }
    .bar { height: 100%; border-radius: 10px; text-align: center; color: white; line-height: 30px; font-weight: bold; }
    #health-bar { background: #e74c3c; }
    #energy-bar { background: #3498db; }
    #explosion-bar, #big-add-bar { background: #f39c12; height: 100%; width: 0%; transition: width 0.05s linear; border-radius: 5px; }
    button { padding: 15px 30px; font-size: 18px; margin: 10px; border: none; border-radius: 10px; cursor: pointer; position: relative; overflow: hidden; }
    button.disabled { background: #999 !important; cursor: not-allowed; }
    .cooldown-ring { position: absolute; top: 0; left: 0; width: 100%; height: 100%; border-radius: 10px; transform: rotate(270deg); }
    #log { margin-top: 20px; font-size: 20px; font-weight: bold; }
    .mode-btn { font-size: 24px; padding: 20px 40px; margin: 20px; border-radius: 20px; }
    #restart-btn { font-size: 24px; padding: 20px 40px; margin-top: 40px; border-radius: 20px; background: #2ecc71; color: white; display: none; }
    #vuln-buff { margin-top: 20px; font-size: 20px; font-weight: bold; color: #e67e22; }
    #big-add-prepare { margin-top: 10px; font-size: 20px; color: orange; }
    #stop-dps { margin-top: 10px; font-size: 24px; color: red; font-weight: bold; display: none; }
  </style>
<template>

<h1>Mutated Construct Simulator</h1>

  <div id="mode-selection">
    <button class="mode-btn" @click="startGame('dps_focus')">DPS集火版</button>
    <button class="mode-btn" @click="startGame('dps_smooth')">DPS平缓版</button>
    <button class="mode-btn" @click="startGame('advanced')">高阶特训版</button>
  </div>

  <div id="game-ui" style="display:none;">
    <div>Health</div>
    <div class="bar-container"><div id="health-bar" class="bar">100%</div></div>
    <div id="stop-dps">DPS停手！</div>

    <div>Energy</div>
    <div class="bar-container"><div id="energy-bar" class="bar">100</div></div>

    <div>自身爆炸读条</div>
    <div class="bar-container" style="height: 20px;"><div id="explosion-bar"></div></div>

    <div id="big-add-section" style="display:none;">
      <div>大怪爆炸CD <span id="bidaddcdremain"></span></div>
      <div>大怪爆炸读条</div>
      <div class="bar-container" style="height: 20px;"><div id="big-add-bar"></div></div>
      <div id="big-add-prepare"></div>
    </div>

    <div id="vuln-buff">易伤: 0% (剩余时间: 0s)</div>

    <div>
      <button id="skill1" @click="skill1()" style="background:#e67e22;">1: 易伤<div class="cooldown-ring" id="skill1-cd"></div></button>
      <button id="skill2" @click="skill2()"  style="background:#c0392b;">2: 打断<div class="cooldown-ring" id="skill2-cd"></div></button>
      <button id="skill3" @click="skill3()"  style="background:#27ae60;">3: 吃水</button>
      <button id="skill4" @click="skill4()"  style="background:#f1c40f;">4: 脱离</button>
    </div>

    <div id="log"></div>
  </div>

  <button id="restart-btn" @click="resetGame()">重新开始</button>


</template>