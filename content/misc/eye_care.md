---
title: "視力存摺"
date: 2025-04-11
draft: false
---

## 2025-04-11


# 護眼小遊戲

選擇今天吃了哪些水果：

<div>
  <input type="checkbox" id="blueberry"> 藍莓 (+100)<br>
  <input type="checkbox" id="raspberry"> 覆盆莓 (+80)<br>
  <input type="checkbox" id="kiwi"> 奇異果 (+70)<br>
  <input type="checkbox" id="papaya"> 木瓜 (+70)<br>
  <input type="checkbox" id="strawberry"> 草莓 (+65)<br>
  <input type="checkbox" id="orange"> 柳橙 (+60)<br>
  <input type="checkbox" id="grape"> 葡萄 (+50)<br>
  <input type="checkbox" id="tomato"> 番茄 (+40)<br>
  <input type="checkbox" id="banana"> 香蕉 (+30)<br>
  <input type="checkbox" id="apple"> 蘋果 (+20)<br>
  <input type="checkbox" id="pineapple"> 鳳梨 (+10)<br><br>
</div>

<div>
  <input type="checkbox" id="look_far"> 遠眺5分鐘 (+50)<br>
  <input type="checkbox" id="steam_eye_mask"> 使用蒸汽眼罩15分鐘 (+100)<br><br>
</div>

<div>
  <button onclick="calculateHP()">結算今日 HP</button>

  <p id="result"></p>
</div>

<script>
function calculateHP() {
  let hp = 1000;
  let fruitsEaten = 0;
  
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

  // 新加的護眼行為
  if (document.getElementById('look_far').checked) { hp += 50; }
  if (document.getElementById('steam_eye_mask').checked) { hp += 100; }

  if (fruitsEaten >= 3) {
    hp += 50; // Bonus!
  }

  document.getElementById('result').innerText =  hp;
}
</script>