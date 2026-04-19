‎<!DOCTYPE html>
‎<html lang="ru">
‎<head>
‎<meta charset="UTF-8">
‎<meta name="viewport" content="width=device-width, initial-scale=1.0">
‎<title>Zombie Boss Game</title>
‎
‎<style>
‎html,body{
‎margin:0;
‎padding:0;
‎height:100%;
‎overflow:hidden;
‎font-family:Arial;
‎background:linear-gradient(135deg,#0f2027,#203a43,#2c5364);
‎color:white;
‎}
‎
‎canvas{
‎position:fixed;
‎top:0;
‎left:0;
‎}
‎
‎#quiz{
‎position:fixed;
‎inset:0;
‎display:flex;
‎justify-content:center;
‎align-items:center;
‎flex-direction:column;
‎gap:10px;
‎z-index:10;
‎background:rgba(0,0,0,0.6);
‎}
‎
‎input,button{
‎padding:12px;
‎font-size:18px;
‎border:none;
‎border-radius:10px;
‎}
‎
‎button{
‎background:#00ff88;
‎cursor:pointer;
‎}
‎
‎#timerQuiz{
‎color:gray;
‎}
‎
‎#ui{
‎position:fixed;
‎top:10px;
‎left:10px;
‎z-index:5;
‎font-size:18px;
‎}
‎</style>
‎</head>
‎
‎<body>
‎
‎<div id="quiz">
‎<h1>1000 - 500</h1>
‎<input id="answer">
‎<div id="timerQuiz">10</div>
‎<button onclick="check()">Старт</button>
‎</div>
‎
‎<div id="ui"></div>
‎<canvas id="c"></canvas>
‎
‎<script>
‎const c=document.getElementById("c");
‎const ctx=c.getContext("2d");
‎c.width=innerWidth;
‎c.height=innerHeight;
‎
‎let game=false;
‎let zombies=[];
‎let bullets=[];
‎let boss=null;
‎
‎let score=0;
‎let weapon=1;
‎
‎let bossTime=80;
‎let bossHitCooldown=0;
‎
‎/* QUIZ TIMER */
‎let quizTime=10;
‎let quizInt=setInterval(()=>{
‎quizTime--;
‎document.getElementById("timerQuiz").innerText=quizTime;
‎if(quizTime<=0){
‎location.reload();
‎}
‎},1000);
‎
‎const weapons=[
‎"Пистолет",
‎"Дробовик",
‎"Автомат",
‎"Винтовка",
‎"Снайперка",
‎"Миниган"
‎];
‎
‎function startGame(){
‎game=true;
‎spawnZombie();
‎loop();
‎}
‎
‎/* ZOMBIES */
‎function spawnZombie(){
‎zombies.push({
‎x:Math.random()*c.width,
‎y:Math.random()*c.height,
‎r:25,
‎hp:2,
‎vx:(Math.random()-0.5)*4,
‎vy:(Math.random()-0.5)*4,
‎holes:[]
‎});
‎
‎setTimeout(()=>{
‎if(game) spawnZombie();
‎},500);
‎}
‎
‎function updateWeapon(){
‎weapon=Math.min(6,Math.floor(score/60)+1);
‎}
‎
‎/* SHOOT */
‎function shoot(x,y){
‎let sx=c.width/2;
‎let sy=c.height;
‎let angle=Math.atan2(y-sy,x-sx);
‎
‎if(weapon===1){
‎bullets.push({x:sx,y:sy,vx:Math.cos(angle)*14,vy:Math.sin(angle)*14});
‎}
‎
‎if(weapon===2){
‎for(let i=0;i<3;i++){
‎let spread=(Math.random()-0.5)*0.4;
‎bullets.push({
‎x:sx,y:sy,
‎vx:Math.cos(angle+spread)*12,
‎vy:Math.sin(angle+spread)*12
‎});
‎}
‎}
‎
‎if(weapon===3){
‎for(let i=0;i<3;i++){
‎setTimeout(()=>{
‎bullets.push({x:sx,y:sy,vx:Math.cos(angle)*16,vy:Math.sin(angle)*16});
‎},i*50);
‎}
‎}
‎
‎if(weapon===4){
‎bullets.push({x:sx,y:sy,vx:Math.cos(angle)*22,vy:Math.sin(angle)*22});
‎}
‎
‎if(weapon===5){
‎bullets.push({x:sx,y:sy,vx:Math.cos(angle)*35,vy:Math.sin(angle)*35});
‎}
‎
‎if(weapon===6){
‎for(let i=0;i<6;i++){
‎bullets.push({
‎x:sx,y:sy,
‎vx:Math.cos(angle+(Math.random()-0.5)*0.3)*20,
‎vy:Math.sin(angle+(Math.random()-0.5)*0.3)*20
‎});
‎}
‎}
‎}
‎
‎c.addEventListener("click",(e)=>{
‎if(!game)return;
‎const r=c.getBoundingClientRect();
‎shoot(e.clientX-r.left,e.clientY-r.top);
‎});
‎
‎/* BOSS */
‎function spawnBoss(){
‎boss={
‎x:c.width/2,
‎y:150,
‎r:100,
‎hp:300,
‎damage:0,
‎vx:2,
‎holes:[],
‎text:"ну заплачь"
‎};
‎
‎bossTime=80;
‎}
‎
‎/* LOOP */
‎function loop(){
‎if(!game)return;
‎ctx.clearRect(0,0,c.width,c.height);
‎
‎if(bossHitCooldown>0) bossHitCooldown--;
‎
‎/* bullets */
‎for(let i=bullets.length-1;i>=0;i--){
‎let b=bullets[i];
‎b.x+=b.vx;
‎b.y+=b.vy;
‎
‎ctx.fillStyle="yellow";
‎ctx.beginPath();
‎ctx.arc(b.x,b.y,3,0,Math.PI*2);
‎ctx.fill();
‎}
‎
‎/* zombies */
‎for(let i=zombies.length-1;i>=0;i--){
‎let z=zombies[i];
‎
‎z.x+=z.vx;
‎z.y+=z.vy;
‎
‎if(z.x<0||z.x>c.width)z.vx*=-1;
‎if(z.y<0||z.y>c.height)z.vy*=-1;
‎
‎ctx.fillStyle="green";
‎ctx.beginPath();
‎ctx.arc(z.x,z.y,z.r,0,Math.PI*2);
‎ctx.fill();
‎
‎/* eyes */
‎ctx.fillStyle="black";
‎ctx.beginPath();ctx.arc(z.x-6,z.y-5,3,0,Math.PI*2);ctx.fill();
‎ctx.beginPath();ctx.arc(z.x+6,z.y-5,3,0,Math.PI*2);ctx.fill();
‎
‎/* 🧟 HOLES ZOMBIES */
‎for(let h of z.holes){
‎ctx.fillStyle="black";
‎ctx.beginPath();
‎ctx.arc(z.x+h.x,z.y+h.y,3,0,Math.PI*2);
‎ctx.fill();
‎}
‎
‎/* hit */
‎for(let j=bullets.length-1;j>=0;j--){
‎let b=bullets[j];
‎
‎if(Math.hypot(z.x-b.x,z.y-b.y)<z.r){
‎z.holes.push({x:b.x-z.x,y:b.y-z.y});
‎z.hp--;
‎bullets.splice(j,1);
‎
‎if(z.hp<=0){
‎zombies.splice(i,1);
‎score+=5;
‎updateWeapon();
‎
‎if(score>=300 && !boss)spawnBoss();
‎}
‎break;
‎}
‎}
‎}
‎
‎/* BOSS */
‎if(boss){
‎
‎boss.x+=boss.vx;
‎if(boss.x<boss.r||boss.x>c.width-boss.r)boss.vx*=-1;
‎
‎bossTime-=1/60;
‎
‎if(bossTime<=0){
‎game=false;
‎alert("💀 Время вышло!");
‎location.reload();
‎}
‎
‎/* body */
‎ctx.fillStyle="darkred";
‎ctx.beginPath();
‎ctx.arc(boss.x,boss.y,boss.r,0,Math.PI*2);
‎ctx.fill();
‎
‎/* 👹 HOLES BOSS */
‎for(let h of boss.holes){
‎ctx.fillStyle="black";
‎ctx.beginPath();
‎ctx.arc(boss.x+h.x,boss.y+h.y,4,0,Math.PI*2);
‎ctx.fill();
‎}
‎
‎/* eyes */
‎ctx.fillStyle="black";
‎ctx.beginPath();ctx.arc(boss.x-20,boss.y-20,8,0,Math.PI*2);ctx.fill();
‎ctx.beginPath();ctx.arc(boss.x+20,boss.y-20,8,0,Math.PI*2);ctx.fill();
‎
‎/* text */
‎let t=Math.max(0,bossTime);
‎let sec=Math.floor(t%60);
‎let min=Math.floor(t/60);
‎
‎ctx.fillStyle="white";
‎ctx.textAlign="center";
‎
‎ctx.font="bold 26px Arial";
‎ctx.fillText(boss.text,boss.x,boss.y-boss.r-120);
‎
‎ctx.font="22px Arial";
‎ctx.fillText("HP: "+boss.hp,boss.x,boss.y-boss.r-60);
‎ctx.fillText("⏱ "+String(min).padStart(2,"0")+":"+String(sec).padStart(2,"0"),boss.x,boss.y-boss.r-90);
‎
‎/* hit boss */
‎for(let i=bullets.length-1;i>=0;i--){
‎let b=bullets[i];
‎
‎if(Math.hypot(boss.x-b.x,boss.y-b.y)<boss.r){
‎
‎if(bossHitCooldown<=0){
‎
‎let dmg=1;
‎if(weapon===5) dmg=6;
‎if(weapon===4) dmg=3;
‎if(weapon===1) dmg=2;
‎
‎boss.hp-=dmg;
‎boss.damage+=dmg;
‎
‎bossHitCooldown=4;
‎}
‎
‎bullets.splice(i,1);
‎
‎/* win */
‎if(boss.damage>=400){
‎game=false;
‎
‎document.body.innerHTML+=`
‎<div style="position:fixed;inset:0;background:black;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:20px;">
‎<h1>👹 БОСС ПОБЕЖДЁН</h1>
‎
‎<a href="https://t.me/mewon1x" target="_blank">
‎<button>📲 Telegram</button>
‎</a>
‎
‎<button onclick="location.reload()">🔁 заново</button>
‎</div>`;
‎}
‎}
‎}
‎}
‎
‎/* UI */
‎document.getElementById("ui").innerHTML=
‎"⭐ "+score+
‎"<br>🔫 "+weapons[weapon-1]+
‎(boss?`<br>👹 HP:${boss.hp}<br>💥 Урон:${boss.damage}/400<br>⏱ ${Math.floor(bossTime)}`:"");
‎
‎requestAnimationFrame(loop);
‎}
‎
‎/* QUIZ */
‎function check(){
‎if(answer.value==500){
‎clearInterval(quizInt);
‎document.getElementById("quiz").style.display="none";
‎startGame();
‎}
‎}
‎</script>
‎
‎</body>
‎</html>
‎
