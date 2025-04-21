---
title: "視力存摺"
date: 2025-04-20
draft: false
---

## Apr
| Mon | Tue | Wed | Thu | Fri | Sat | Sun |  
| --------- | --------- | --------- | --------- | --------- | --------- | --------- |
| 14 (1575) | 15 (1110) | 16 (1040) | 17 (1165) | 18 (XXXX) | 19 (1295) | 20 (XXXX) |
| 21 (1165) | 22 (XXXX) | 23 (XXXX) | 24 (XXXX) | 25 (XXXX) | 26 (XXXX) | 27 (XXXX) |
| 28 (XXXX) | 29 (XXXX) | 30 (XXXX) |

## May
| Mon | Tue | Wed | Thu | Fri | Sat | Sun |  
| --------- | --------- | --------- | --------- | --------- | --------- | --------- |
|           |           |           | 1 (XXXX)  | 2 (XXXX)  | 3 (XXXX)  | 4 (XXXX)  |
| 5 (XXXX)  | 6 (XXXX)  | 7 (XXXX)  | 8 (XXXX)  | 9 (XXXX)  | 10 (XXXX) | 11 (XXXX) |
| 12 (XXXX) | 13 (XXXX) | 14 (XXXX) | 15 (XXXX) | 16 (XXXX) | 17 (XXXX) | 18 (XXXX) |
| 19 (XXXX) | 20 (XXXX) | 21 (XXXX) | 22 (XXXX) | 23 (XXXX) | 24 (XXXX) | 25 (XXXX) |
| 26 (XXXX) | 27 (XXXX) | 28 (XXXX) | 29 (XXXX) | 30 (XXXX) | 31 (XXXX) |

# 護眼 RPG 小遊戲 🎮👀

## 今天你的護眼行為打卡✅

### 水果護眼打金
<input type="checkbox" id="blueberry"> 吃藍莓 (+100)<br>
<input type="checkbox" id="raspberry"> 吃覆盆莓 (+80)<br>
<input type="checkbox" id="kiwi"> 吃奇異果 (+70)<br>
<input type="checkbox" id="papaya"> 吃木瓜 (+70)<br>
<input type="checkbox" id="strawberry"> 吃草莓 (+65)<br>
<input type="checkbox" id="orange"> 吃柳橙 (+60)<br>
<input type="checkbox" id="grape"> 吃葡萄 (+50)<br>
<input type="checkbox" id="tomato"> 吃番茄 (+40)<br>
<input type="checkbox" id="banana"> 吃香蕉 (+30)<br>
<input type="checkbox" id="apple"> 吃蘋果 (+20)<br>
<input type="checkbox" id="pineapple"> 吃鳳梨 (+10)<br>

### 正常護眼行為
<input type="checkbox" id="look_far"> 遠眺5分鐘 I (+20)<br>
<input type="checkbox" id="look_farII"> 遠眺5分鐘 II (+20)<br>
<input type="checkbox" id="look_farIII"> 遠眺5分鐘 III (+20)<br>
<input type="checkbox" id="steam_eye_mask"> 使用蒸氣眼罩20分鐘 (+150)<br>

### 負面行為（扣分）
<input type="checkbox" id="watch_tv"> 看電視1小時 (-100)<br>
<input type="checkbox" id="gaming"> 打電動超過2小時 (-300)<br>
<input type="checkbox" id="dark_light"> 昏暗環境用眼30分鐘以上 (-200)<br>

### 睡眠時數
<select id="sleep_hours">
  <option value="0">0小時</option>
  <option value="1">1小時</option>
  <option value="2">2小時</option>
  <option value="3">3小時</option>
  <option value="4">4小時</option>
  <option value="5">5小時</option>
  <option value="6">6小時</option>
  <option value="7">7小時</option>
  <option value="8">8小時</option>
  <option value="9">9小時</option>
  <option value="10">10小時以上</option>
</select>

### 午睡時數
<select id="nap_minutes">
  <option value="0">沒午睡</option>
  <option value="15">15分鐘</option>
  <option value="30">30分鐘</option>
  <option value="45">45分鐘</option>
  <option value="60">60分鐘</option>
  <option value="90">90分鐘</option>
  <option value="120">120分鐘</option>
</select>

### 手沖咖啡杯數
<select id="coffee_cups">
  <option value="0">沒喝</option>
  <option value="1">1杯</option>
  <option value="2">2杯</option>
  <option value="3">3杯以上</option>
</select>

### 電腦使用時數
<select id="pc_hours">
  <option value="0">0小時</option>
  <option value="1">1小時</option>
  <option value="2">2小時</option>
  <option value="3">3小時</option>
  <option value="4">4小時</option>
  <option value="5">5小時</option>
  <option value="6">6小時</option>
  <option value="7">7小時</option>
  <option value="8">8小時</option>
  <option value="9">9小時</option>
  <option value="10">10小時以上</option>
</select>

### 手機使用時數
<select id="phone_hours">
  <option value="0">0小時</option>
  <option value="1">1小時</option>
  <option value="2">2小時</option>
  <option value="3">3小時</option>
  <option value="4">4小時</option>
  <option value="5">5小時以上</option>
</select>

<br><br>
<button onclick="calculateHP()">結算今天分數</button>

<p id="result"></p>

<script>
function calculateHP() {
  let hp = 1000;
  let fruitsEaten = 0;

  // 水果加分
  if (document.getElementById('blueberry').checked) { hp += 100; fruitsEaten++; }
  if (document.getElementById('raspberry').checked) { hp += 80; fruitsEaten++; }
  if (document.getElementById('kiwi').checked) { hp += 70; fruitsEaten++; }
  if (document.getElementById('papaya').checked) { hp += 70; fruitsEaten++; }
  if (document.getElementById('strawberry').checked) { hp += 65; fruitsEaten++; }
  if (document.getElementById('orange').checked) { hp += 60; fruitsEaten++; }
  if (document.getElementById('grape').checked) { hp += 50; fruitsEaten++; }
  if (document.getElementById('tomato').checked) { hp += 40; fruitsEaten++; }
  if (document.getElementById('banana').checked) { hp += 30; fruitsEaten++; }
  if (document.getElementById('apple').checked) { hp += 20; fruitsEaten++; }
  if (document.getElementById('pineapple').checked) { hp += 10; fruitsEaten++; }
  
  if (fruitsEaten >= 3) { hp += 50; } // 水果大餐加分

  // 護眼行為加分
  if (document.getElementById('look_far').checked) { hp += 20; }
  if (document.getElementById('look_farII').checked) { hp += 20; }
  if (document.getElementById('look_farIII').checked) { hp += 20; }
  if (document.getElementById('steam_eye_mask').checked) { hp += 150; }

  // 負面行為扣分
  if (document.getElementById('watch_tv').checked) { hp -= 100; }
  if (document.getElementById('gaming').checked) { hp -= 300; }
  if (document.getElementById('dark_light').checked) { hp -= 200; }

  // 睡眠時數
  let sleep = parseInt(document.getElementById('sleep_hours').value);
  if (sleep <= 4) { hp -= 300; }
  else if (sleep == 5) { hp -= 150; }
  else if (sleep == 6) { hp -= 50; }
  else if (sleep == 7 || sleep == 8) { hp += 100; }
  else if (sleep == 9) { hp += 30; }
  else if (sleep >= 10) { hp -= 30; }

  // 午睡
  let nap = parseInt(document.getElementById('nap_minutes').value);
  if (nap > 0 && nap <= 30) { hp += 30; }
  else if (nap > 60) { hp -= 50; }

  // 咖啡
  let coffee = parseInt(document.getElementById('coffee_cups').value);
  if (coffee == 1) { hp += 10; }
  else if (coffee >= 3) { hp -= 50; }

  // 電腦/手機使用
  let pc = parseInt(document.getElementById('pc_hours').value);
  let phone = parseInt(document.getElementById('phone_hours').value);
  let totalScreenTime = pc + phone;

  if (totalScreenTime <= 4) { hp += 50; }
  else if (totalScreenTime <= 6) { hp += 20; }
  else if (totalScreenTime <= 8) { hp -= 50; }
  else { hp -= 100; }

  // 決定護眼等級
  let level = "";
  if (hp >= 1200) {
    level = "護眼聖騎士（S+）";
  }
  else if (hp >= 1100) {
    level = "護眼勇者（A）";
  }
  else if (hp >= 1000) {
    level = "普通防衛者（B）";
  }
  else if (hp >= 900) {
    level = "受損旅人（C）";
  }
  else if (hp >= 800) {
    level = "重傷流浪者（D）";
  }
  else {
    level = "眼睛崩壞王（F）";
  }

  document.getElementById('result').innerHTML = 
    "今日視力貨幣：" + hp + " Gold<br>" + 
    "頭銜：" + level;
}
</script>