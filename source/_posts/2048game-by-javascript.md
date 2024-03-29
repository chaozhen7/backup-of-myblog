layout: post
title: 基于javascript实现的2048小游戏
date: 2015-11-02 11:55:22
tags: 
	- Javascript
	- Web 前端
comments: true
copyright: true
---
## 1. 游戏规则
- 用键盘上下左右键控制数字走向
- 当点击了一个方向时，格子中的数字会全部往那个方向移动，直到不能再移动，如果有相同的数字则会合并
- 当格子内的最大数字达到2048时获胜
- 当格子中不再有可移动和可合并的数字时，游戏结束

[演示地址](http://cighao.github.io/2048-game)
<!--more-->

## 2. 数据结构
主要是用一个4*4的二维数组来保存游戏当前的状态。
## 3. 主要的函数
因为游戏中主要是上下左右的移动和合并，所以整个实现过程主要还是要完成上下左右的移动操作。所以程序中使用了up(),down(),left(),right()四个函数来实现上下左右的移动和合并操作。每个函数的实现大致分为两步：
- 根据用户的操作，向上，下，左或右移动数据，
- 移动数据后看看是否达到合并的要求，如果达到了则执行合并操作，否则不执行任何操作。


## 4.程序实现的流程
step1: 初始化界面，整个棋盘只有一个随即生成的数
step2: 捕捉用户输入的按钮
step3: 根据用户输入的按钮做出移动和合并的操作
step4: 判断第三步中是否执行了合并操作，如果是，则在空白区域随机产生一个数，否则空操作。
step5: 判断游戏是否结束，没有结束则跳到第三步。

## 5.源码
``` javascript
<html>
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
		<title>2048 Game</title>
		<style>
			table{
				border-spacing:2;
				text-align:center;
				width:300px;
				height:300px;
			}
			td{
				width:25%;
				height:25%;
				text-align:center;
			}
			
		</style>
		<script>
			var table = new Array(new Array(),new Array(),new Array(),new Array(),new Array());  // 5*5 array(except 0th)
			var is_moved = false ;  // is there any data have been moved
			function get_score(){     // get the score
				var max = 0;
				var i,j;
				for(i=1;i<=4;i++)
					for(j=1;j<=4;j++){
						if(table[i][j]>max)
							max = table[i][j];
					}
				return max;
			}
			function left(){    // left and combination
				var i,j,k;
				var cur = new Array();
				for(i=1;i<=4;i++){
					k=1;
					for(j=1;j<=4;j++){
						if(table[i][j]!=0){
							cur[k++] = table[i][j]; 
						}
					}
					while(k<=4)	cur[k++] = 0;
					for(j=1;j<=3;j++){   //combination
						if(cur[j]==cur[j+1]){
							cur[j] = cur[j]+cur[j+1];
							k=j+1;
							while(k<=3){
								cur[k] = cur[k+1];
								k++;
							}
							cur[4]=0;
						}
					}
					if(table[i][1]!=cur[1]||table[i][2]!=cur[2]||table[i][3]!=cur[3]||table[i][4]!=cur[4])
						is_moved = true;     // some datas are moved
					for(j=1;j<=4;j++)	table[i][j] = cur[j];
				}
			}
			function right(){    //right and combination
				var i,j,k;
				var cur = new Array();
				for(i=1;i<=4;i++){
					k=4;
					for(j=4;j>=1;j--){
						if(table[i][j]!=0)
							cur[k--] = table[i][j];
					}
					while(k>=1)	cur[k--] = 0;
					for(j=4;j>=2;j--){     //combination
						if(cur[j]==cur[j-1]){
							cur[j] = cur[j]+cur[j-1];
							k=j-1;
							while(k>=2){
								cur[k] = cur[k-1];
								k--;
							}
							cur[1] = 0;
						}
					}
					if(table[i][1]!=cur[1]||table[i][2]!=cur[2]||table[i][3]!=cur[3]||table[i][4]!=cur[4])
						is_moved = true;     // some datas are moved
					for(j=1;j<=4;j++)	table[i][j] = cur[j];
				}
			}
			function up(){    //up and combination
				var i,j,k;
				var cur = new Array();
				for(j=1;j<=4;j++){
					k=1;
					for(i=1;i<=4;i++){
						if(table[i][j]!=0)
							cur[k++] = table[i][j];
					}
					while(k<=4)	cur[k++] = 0;
					
					for(i=1;i<=3;i++){   //combination
						if(cur[i]==cur[i+1]){
							cur[i] = cur[i]+cur[i+1];
							k=i+1;
							while(k<=3){
								cur[k] = cur[k+1];
								k++;
							}
							cur[4]=0;
						}
					}
					if(table[1][j]!=cur[1]||table[2][j]!=cur[2]||table[3][j]!=cur[3]||table[4][j]!=cur[4])
						is_moved = true;     // some datas are moved
					for(i=1;i<=4;i++)	table[i][j] = cur[i];
				}
			}
			
			function down(){    //down and combination
				var i,j,k;
				var cur = new Array();
				for(j=1;j<=4;j++){
					k=4;
					for(i=4;i>=1;i--){
						if(table[i][j]!=0)
							cur[k--] = table[i][j];
					}
					while(k>=1)	cur[k--] = 0;
					
					for(i=4;i>=2;i--){     //combination
						if(cur[i]==cur[i-1]){
							cur[i] = cur[i]+cur[i-1];
							k=i-1;
							while(k>=2){
								cur[k] = cur[k-1];
								k--;
							}
							cur[1] = 0;
						}
					}
					if(table[1][j]!=cur[1]||table[2][j]!=cur[2]||table[3][j]!=cur[3]||table[4][j]!=cur[4])
						is_moved = true;     // some datas are moved
					for(i=1;i<=4;i++)	table[i][j] = cur[i];
				}
			}
			
			function is_over(){   //judge the game is over or not
				var i,j;
				for(i=1;i<=4;i++)     //is there any  empty position
					for(j=1;j<=4;j++)
						if(table[i][j]==0)  
							return false;   
				for(i=1;i<=4;i++)    // is there any datas can be combinated
					for(j=1;j<=3;j++)
						if(table[i][j]==table[i][j+1])
							return false;  
				for(j=1;j<=4;j++)
					for(i=1;i<=3;i++)
						if(table[i][j]==table[i+1][j])
							return false;
				return true;
			}
			function is_win(){   //judge the game is win or not
				if(get_score()==2048)
					return true;
				return false;				
			}
			
			function init(){   //init
				var i,j;
				for(i=1;i<=4;i++)
					for(j=1;j<=4;j++)
						table[i][j]=0;
			}
			function display_table(){   // 
				var i,j,cur;
				for(i=1;i<=4;i++)
					for(j=1;j<=4;j++){
						cur = String(i)+String(j);
						if(table[i][j]==0)
							document.getElementById(cur).innerHTML = " ";
						else{
							document.getElementById(cur).innerHTML = table[i][j];
						}
						
					}
			}
			function display_score(){    // display score
				var score = get_score();
				document.getElementById("score").innerHTML = "score:"+score;
			}
			function display(){     //display
				display_table();
				display_score();
			}
			
			function produce_new_number(){  // produce a new number randomly
				var num=0; // the number of empty position
				var i,j;
				var value;  // the value of the new number
				for(i=1;i<=4;i++)
					for(j=1;j<=4;j++)
						if(table[i][j]==0)
							num++;
				var p = Math.round(Math.random()*num)+1; // the position of new number
				if(Math.random()<=0.5)   // the new number is 2 or 4
					value = 2;
				else 
					value = 4;
				num=0;
				for(i=1;i<=4;i++)   // update table with new number 
					for(j=1;j<=4;j++){
						if(table[i][j]==0){
							num++;
							if(num==p){
								table[i][j] = value;
								break;
							}
								
						}
					}
			}
			function over(){    // when the game is over
				alert("the game is over");
				location.reload(); //refresh
			}
			function run(){   //deal with the event when you press button
				var key = window.event.keyCode;
				//var key = window.event.keyCode||e.which;
				switch(key){
					case 37: left();	break;  // left is pressed;
					case 38: up();	    break;  // up is pressed;
					case 39: right();	break;  // right is pressed;
					case 40: down();            // down is pressed;
				}	
				if(is_moved){
					produce_new_number();
					is_moved = false;
				}
				display();
				if(is_over())
					over();
			}
			
			window.onload = function (){
				
				window.onkeyup = run;
				init();
				produce_new_number();
				display();
				
			}
		</script>
		
	</head>
	<body>
		<div id="score">
			
		</div>
		<div>
			<table border="2" align="center" width="300" height="300">
				<tr>
					<td id="11">	</td>    <td id="12">	</td>    <td id="13">	</td>    <td id="14">	</td>    
				</tr>
				<tr>
					<td id="21">	</td>    <td id="22">	</td>    <td id="23">	</td>    <td id="24">	</td>    
				</tr>
				<tr>
					<td id="31">	</td>    <td id="32">	</td>    <td id="33">	</td>    <td id="34">	</td>    
				</tr>
				<tr>
					<td id="41">	</td>    <td id="42">	</td>    <td id="43">	</td>    <td id="44">	</td>    
				</tr>
			</table>
		</div>
		<div id="rule">
			<p class="rule">玩法：</p>
			<p class="rule">1.用键盘上下左右键控制数字走向</p>
			<p class="rule">2.当点击了一个方向时，格子中的数字会全部往那个方向移动，直到不能再移动，如果有相同的数字则会合并</p>
			<p class="rule">3.当格子内的最大数字达到2048时获胜</p>
			
			<p class="rule">注意:</p>
			<p class="rule">1.请使用IE,google浏览器</p>
			
		</div>
	</body>

</html>
```