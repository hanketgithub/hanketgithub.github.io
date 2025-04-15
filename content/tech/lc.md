---
title: "LeetCode 每日打卡 RPG 🎯💻"
date: 2025-04-14
draft: false
---

<a href="https://leetcode.com/" target="_blank">LeetCode</a>

## Apr
| Mon | Tue | Wed | Thu | Fri | Sat | Sun |  
| --------- | --------- | --------- | --------- | --------- | --------- | --------- |
| 14 (32)   | 15 (17)  | 16 (XXX)  | 17 (XXX)  | 18 (XXX)  | 19 (XXX)  | 20 (XXX)  |
| 21 (XXX)  | 22 (XXX)  | 23 (XXX)  | 24 (XXX)  | 25 (XXX)  | 26 (XXX)  | 27 (XXX)  |
| 28 (XXX)  | 29 (XXX)  | 30 (XXX)  |

## May
| Mon | Tue | Wed | Thu | Fri | Sat | Sun |  
| --------- | --------- | --------- | --------- | --------- | --------- | --------- |
|           |           |           | 1 (XXX)   | 2 (XXX)   | 3 (XXX)   | 4 (XXX)   |
| 5 (XXX)   | 6 (XXX)  | 7 (XXX)    | 8 (XXX)   | 9 (XXX)   | 10 (XXX)  | 11 (XXX) |
| 12 (XXX)  | 13 (XXX) | 14 (XXX)   | 15 (XXX)  | 16 (XXX)  | 17 (XXX)  | 18 (XXX) |
| 19 (XXX)  | 20 (XXX) | 21 (XXX)   | 22 (XXX)  | 23 (XXX)  | 24 (XXX)  | 25 (XXX) |
| 26 (XXX)  | 27 (XXX) | 28 (XXX)   | 29 (XXX)  | 30 (XXX)  | 31 (XXX)  |



#### <a href="https://leetcode.com/problems/maximum-total-damage-with-spell-casting/" target="_blank">3186 Maximum Total Damage With Spell Casting, 1840</a>
- 4/15
- Figure out why previous solution doesn't work. Lack consideration of 1 case. Pay attention to consider all!
- Follow up: what if power[i] - range?

## 今天你的刷題活動打卡 ✅

<form id="lc-activity-form">

### ✅ 每日任務

<input type="checkbox" id="daily_checkin"> 打卡 (+2)<br>
<input type="checkbox" id="easy_solved"> 解 Easy 題 (+5)<br>
<input type="checkbox" id="medium_solved"> 解 Medium 題 (+15)<br>
<input type="checkbox" id="hard_solved"> 解 Hard 題 (+60)<br>

### 🎯 今日比賽型活動

<input type="radio" name="contest_type" id="contest_official" value="official" checked> 正式 Weekly Contest<br>
<input type="radio" name="contest_type" id="contest_mock" value="mock"> 模擬考 Mock Contest (5折)<br>
<input type="radio" name="contest_type" id="contest_none" value="none"> 無比賽參加<br>

### 🏆 Weekly Contest 解題數

<select id="weekly_contest_solved">
  <option value="0">沒有解出</option>
  <option value="1">解出 1 題 (+15)</option>
  <option value="2">解出 2 題 (+50)</option>
  <option value="3">解出 3 題 (+200)</option>
  <option value="4">解出 4 題 (+500)</option>
</select>

<br>
<br>

<button type="button" onclick="calculateLCPoints()">結算今日成績</button>

</form>

<p id="lc-result" style="margin-top:20px; font-size: 18px; font-weight: bold;"></p>
<p id="lc-title" style="font-size: 16px;"></p>

<script>
function calculateLCPoints() {
  let points = 0;

  // 日常活動分數
  if (document.getElementById('daily_checkin').checked) { points += 2; }
  if (document.getElementById('easy_solved').checked) { points += 5; }
  if (document.getElementById('medium_solved').checked) { points += 15; }
  if (document.getElementById('hard_solved').checked) { points += 60; }

  // Weekly Contest 解題數
  const contestSolved = parseInt(document.getElementById('weekly_contest_solved').value);
  let contestPoints = 0;
  if (contestSolved === 1) { contestPoints = 15; }
  else if (contestSolved === 2) { contestPoints = 50; }
  else if (contestSolved === 3) { contestPoints = 200; }
  else if (contestSolved === 4) { contestPoints = 500; }

  // 判斷正式/模擬/無參加
  const contestType = document.querySelector('input[name="contest_type"]:checked').value;
  let contestMessage = "";
  
  if (contestType === "official") {
    contestMessage = "征戰正式戰場，氣勢如虹！🏆";
  } else if (contestType === "mock") {
    contestPoints = Math.floor(contestPoints * 0.5); // 模擬考打五折
    contestMessage = "模擬戰場訓練，未來主力！🛡️";
  } else if (contestType === "none") {
    // 沒有參加任何比賽，只顯示鼓勵訊息
    contestMessage = "休息是為了走更長遠的路！🌱";
    contestPoints = 0;  // **只有當 contest_type 是 none 時才硬設 0**
  }

  points += contestPoints;

  // 顯示總分
  document.getElementById('lc-result').innerText = "今日 LC 活動總分：" + points + " 分";

  // 決定頭銜
  let title = "";
  if (points >= 300) {
    title = "少林心經 📜🧘‍♂️";
  } else if (points >= 200) {
    title = "易筋經 🧘‍♂️";
  } else if (points >= 80) {
    title = "一拍兩散掌 🫳💥";
  } else if (points >= 30) {
    title = "大力金剛掌 💪";
  } else if (points > 0) {
    title = "羅漢拳 🥋";
  } else {
    title = "尚未入門的小師弟 🌱";
  }

  document.getElementById('lc-title').innerHTML = 
    "今日頭銜：" + title  + "<br>" + contestMessage;
}
</script>
