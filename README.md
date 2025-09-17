
<!doctype html>
<html lang="ja">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>家計簿 </title>
<!-- シンプルなスタイル。淡いピンク基調。個人は色分け -->
<style>
:root{
--bg:#fff7f8; /* 淡いピンク */
--card:#fff;
--hikari:#ffd7e0; /* ピンク */
--akito:#d7f0ff; /* 水色 */
--ayumi:#ffc6c6; /* 赤寄り */
--text:#333;
--muted:#666;
--accent:#ff7aa2;
--radius:12px;
--glass: rgba(255,255,255,0.65);
}
*{box-sizing:border-box}
body{font-family: system-ui, -apple-system, "Hiragino Kaku Gothic ProN", "Meiryo", sans-serif; margin:0; background:linear-gradient(180deg,var(--bg), #fff); color:var(--text); padding:18px}
header{display:flex;gap:12px;align-items:center}
h1{margin:0;font-size:1.4rem}
.container{display:grid;grid-template-columns:320px 1fr;gap:18px;margin-top:16px}
.panel{background:var(--card);border-radius:var(--radius);padding:12px;box-shadow:0 6px 18px rgba(0,0,0,0.07)}
.small{font-size:0.85rem;color:var(--muted)}
label{display:block;margin-top:8px;font-size:0.9rem}
input, select, button{padding:8px;border-radius:8px;border:1px solid #e6e6e6;background:transparent}
button{cursor:pointer}
.owners{display:flex;gap:8px;flex-wrap:wrap;margin-top:8px}
.owner-pill{padding:6px 8px;border-radius:999px;font-weight:600}
.owner-hikari{background:var(--hikari)}
.owner-akito{background:var(--akito)}
.owner-ayumi{background:var(--ayumi)}
.owner-pet{background:#fff3d9}
.records{max-height:42vh;overflow:auto;margin-top:8px}
ul.record-list{list-style:none;padding:0;margin:0}
li.record{display:flex;justify-content:space-between;padding:8px;border-radius:8px;margin-bottom:6px}
.record .left{display:flex;gap:8px;align-items:center}
.badge{font-size:0.75rem;padding:4px 6px;border-radius:6px}
.summary{margin-top:12px}
table{width:100%;border-collapse:collapse}
th,td{padding:8px;border-bottom:1px dashed #eee;text-align:left}
.chart-wrap{height:260px}
.pets{display:flex;gap:8px}
.pet-card{width:60px;height:60px;border-radius:10px;display:flex;align-items:center;justify-content:center;font-weight:700}
.pet-edel{background:#ffeef6}
.pet-aster{background:#fff0e0}
.pet-lew{background:#f0f7ff}
.controls{display:flex;gap:8px;flex-wrap:wrap}
.danger{color:#b00}
footer{margin-top:18px;font-size:0.8rem;color:var(--muted)}
/* responsive */
@media(max-width:900px){.container{grid-template-columns:1fr}.chart-wrap{height:220px}}
</style>
</head>
<body>
<header>
<h1>やさしい家計簿 — ひかり家計</h1>
</header>


<div class="container">
<!-- left panel: controls + input -->
<aside class="panel">
<h3>アカウント & セキュリティ</h3>
<div class="small">クライアント側保存 / ローカルのみ（ブラウザの localStorage）</div>
<label>アカウント名</label>
<input id="accountName" placeholder="例: hikari_house" />
<label>パスワード（初回登録）</label>
<input id="password" type="password" placeholder="パスワード" />
<div style="display:flex;gap:8px;margin-top:8px">
<button id="createAccount">アカウント作成</button>
<button id="login">ログイン</button>
<button id="logout">ログアウト</button>
</div>
<div id="acctMsg" class="small" style="margin-top:8px"></div>

import React, { useState } from "react";
import { PieChart, Pie, Cell, Tooltip, Legend } from "recharts";

export default function BudgetApp() {
  const [view, setView] = useState("収入");

  // 収入項目
  const [income, setIncome] = useState({
    基本給: 0,
    固定残業代: 0,
    通勤手当: 0,
    ボーナス: 0,
    TikTok_ギフト: 0,
    TikTok_サブスク: 0,
    YouTube収益: 0,
    メルカリ収益: 0,
    BASE収益: 0,
  });

  // 支出項目
  const [expense, setExpense] = useState({
    家賃: 0,
    健康保険料: 0,
    厚生年金: 0,
    雇用保険: 0,
    所得税: 0,
    住民税: 0,
    立替金: 0,
    欠勤控除: 0,
    クレジットカード: 0,
    現金: 0,
    サブスク: 0,
    保険: 0,
    食材費: 0,
    外食: 0,
    車: 0,
    交通費: 0,
    医療費: 0,
    日用品: 0,
    消耗品: 0,
    活動経費: 0,
    旅行費: 0,
    交際費: 0,
    猫費用: 0,
  });

  // 合計値
  const totalIncome = Object.values(income).reduce((a, b) => a + Number(b), 0);
  const totalExpense = Object.values(expense).reduce((a, b) => a + Number(b), 0);
  const balance = totalIncome - totalExpense;

  // 円グラフ用データ
  const pieData = Object.entries(expense)
    .filter(([_, v]) => Number(v) > 0)
    .map(([k, v]) => ({ name: k, value: Number(v) }));

  const COLORS = [
    "#FFB6C1", "#87CEFA", "#FF6347", "#FFD700", "#ADFF2F",
    "#20B2AA", "#9370DB", "#FF69B4", "#FFA500", "#40E0D0"
  ];

  return (
    <div className="p-6 max-w-2xl mx-auto bg-pink-50 rounded-xl shadow">
      {/* ナビゲーション */}
      <div className="flex gap-2 mb-6">
        {["収入", "支出", "集計"].map((tab) => (
          <button
            key={tab}
            onClick={() => setView(tab)}
            className={`px-4 py-2 rounded-full ${
              view === tab ? "bg-pink-400 text-white" : "bg-gray-200"
            }`}
          >
            {tab}
          </button>
        ))}
      </div>

      {/* 収入タブ */}
      {view === "収入" && (
        <div>
          <h2 className="text-xl font-bold mb-4 text-pink-600">収入入力</h2>
          {Object.keys(income).map((key) => (
            <div key={key} className="mb-3">
              <label className="block mb-1">{key}</label>
              <input
                type="number"
                value={income[key]}
                onChange={(e) =>
                  setIncome({ ...income, [key]: e.target.value })
                }
                className="border p-2 w-full rounded"
              />
            </div>
          ))}
        </div>
      )}

      {/* 支出タブ */}
      {view === "支出" && (
        <div>
          <h2 className="text-xl font-bold mb-4 text-pink-600">支出入力</h2>
          {Object.keys(expense).map((key) => (
            <div key={key} className="mb-3">
              <label className="block mb-1">{key}</label>
              <input
                type="number"
                value={expense[key]}
                onChange={(e) =>
                  setExpense({ ...expense, [key]: e.target.value })
                }
                className="border p-2 w-full rounded"
              />
            </div>
          ))}
        </div>
      )}

      {/* 集計タブ */}
      {view === "集計" && (
        <div>
          <h2 className="text-xl font-bold mb-4 text-pink-600">集計結果</h2>
          <p className="mb-2">収入合計: {totalIncome} 円</p>
          <p className="mb-2">支出合計: {totalExpense} 円</p>
          <p className="mb-4 font-bold">
            残高:{" "}
            <span className={balance >= 0 ? "text-green-600" : "text-red-600"}>
              {balance} 円
            </span>
          </p>

          {/* 円グラフ */}
          {pieData.length > 0 ? (
            <PieChart width={400} height={300}>
              <Pie
                data={pieData}
                cx="50%"
                cy="50%"
                outerRadius={100}
                dataKey="value"
                label
              >
                {pieData.map((_, index) => (
                  <Cell
                    key={`cell-${index}`}
                    fill={COLORS[index % COLORS.length]}
                  />
                ))}
              </Pie>
              <Tooltip />
              <Legend />
            </PieChart>
          ) : (
            <p className="text-gray-500">支出データがありません</p>
          )}
        </div>
      )}
    </div>
  );
}
 import React, { useState } from "react";
import { PieChart, Pie, Cell, Tooltip, Legend } from "recharts";

export default function BudgetApp() {
  const [view, setView] = useState("収入");

  const owners = [
    { key: "hikari", label: "ひかり", color: "#FFB6C1" },
    { key: "akito", label: "あきと", color: "#87CEFA" },
    { key: "ayumi", label: "あゆみ", color: "#FF6347" },
    { key: "edel", label: "エーデル", color: "#FFD700" },
    { key: "aster", label: "アスター", color: "#ADFF2F" },
    { key: "lewisia", label: "レウィシア", color: "#20B2AA" },
  ];

  // 収入
  const [income, setIncome] = useState({
    基本給: { amount: 0, owner: "hikari" },
    固定残業代: { amount: 0, owner: "hikari" },
    通勤手当: { amount: 0, owner: "hikari" },
    ボーナス: { amount: 0, owner: "hikari" },
    TikTok_ギフト: { amount: 0, owner: "hikari" },
    TikTok_サブスク: { amount: 0, owner: "hikari" },
    YouTube収益: { amount: 0, owner: "hikari" },
    メルカリ収益: { amount: 0, owner: "hikari" },
    BASE収益: { amount: 0, owner: "hikari" },
  });

  // 支出
  const [expense, setExpense] = useState({
    家賃: { amount: 0, owner: "hikari" },
    健康保険料: { amount: 0, owner: "hikari" },
    厚生年金: { amount: 0, owner: "hikari" },
    光熱費: { amount: 0, owner: "hikari" },
    交通費: { amount: 0, owner: "hikari" },
    猫費用: { amount: 0, owner: "edel" },
    食材費: { amount: 0, owner: "hikari" },
    外食: { amount: 0, owner: "hikari" },
    クレジットカード: { amount: 0, owner: "hikari" },
  });

  // 合計計算
  const totalIncome = Object.values(income).reduce((acc, i) => acc + Number(i.amount), 0);
  const totalExpense = Object.values(expense).reduce((acc, e) => acc + Number(e.amount), 0);
  const balance = totalIncome - totalExpense;

  // 集計用データ（owner別）
  const totalByOwner = {};
  [...Object.values(income), ...Object.values(expense)].forEach(item => {
    totalByOwner[item.owner] = (totalByOwner[item.owner] || 0) + Number(item.amount);
  });

  // 円グラフ用
  const pieData = Object.entries(totalByOwner).map(([owner, value]) => ({
    name: owners.find(o => o.key === owner)?.label || owner,
    value,
    color: owners.find(o => o.key === owner)?.color || "#ccc",
  }));

  return (
    <div className="p-6 max-w-3xl mx-auto bg-pink-50 rounded-xl shadow">
      {/* タブ */}
      <div className="flex gap-2 mb-6">
        {["収入", "支出", "集計"].map(tab => (
          <button
            key={tab}
            onClick={() => setView(tab)}
            className={`px-4 py-2 rounded-full ${view === tab ? "bg-pink-400 text-white" : "bg-gray-200"}`}
          >
            {tab}
          </button>
        ))}
      </div>

      {/* 収入 */}
      {view === "収入" && (
        <div>
          <h2 className="text-xl font-bold mb-4 text-pink-600">収入入力</h2>
          {Object.keys(income).map(key => (
            <div key={key} className="mb-3 flex gap-2 items-center">
              <label className="w-40">{key}</label>
              <input
                type="number"
                value={income[key].amount}
                onChange={e =>
                  setIncome({ ...income, [key]: { ...income[key], amount: e.target.value } })
                }
                className="border p-2 rounded flex-1"
              />
              <select
                value={income[key].owner}
                onChange={e =>
                  setIncome({ ...income, [key]: { ...income[key], owner: e.target.value } })
                }
                className="border p-2 rounded"
              >
                {owners.slice(0, 3).map(o => <option key={o.key} value={o.key}>{o.label}</option>)}
              </select>
            </div>
          ))}
        </div>
      )}

      {/* 支出 */}
      {view === "支出" && (
        <div>
          <h2 className="text-xl font-bold mb-4 text-pink-600">支出入力</h2>
          {Object.keys(expense).map(key => (
            <div key={key} className="mb-3 flex gap-2 items-center">
              <label className="w-40">{key}</label>
              <input
                type="number"
                value={expense[key].amount}
                onChange={e =>
                  setExpense({ ...expense, [key]: { ...expense[key], amount: e.target.value } })
                }
                className="border p-2 rounded flex-1"
              />
              <select
                value={expense[key].owner}
                onChange={e =>
                  setExpense({ ...expense, [key]: { ...expense[key], owner: e.target.value } })
                }
                className="border p-2 rounded"
              >
                {owners.map(o => <option key={o.key} value={o.key}>{o.label}</option>)}
              </select>
            </div>
          ))}
        </div>
      )}

      {/* 集計 */}
      {view === "集計" && (
        <div>
          <h2 className="text-xl font-bold mb-4 text-pink-600">集計結果</h2>
          <p className="mb-1">収入合計: {totalIncome} 円</p>
          <p className="mb-1">支出合計: {totalExpense} 円</p>
          <p className="mb-4 font-bold">
            残高: <span className={balance >= 0 ? "text-green-600" : "text-red-600"}>{balance} 円</span>
          </p>

          <PieChart width={400} height={300}>
            <Pie
              data={pieData}
              cx="50%"
              cy="50%"
              outerRadius={100}
              dataKey="value"
              label
            >
              {pieData.map((entry, index) => (
                <Cell key={`cell-${index}`} fill={entry.color} />
              ))}
            </Pie>
            <Tooltip />
            <Legend />
          </PieChart>
        </div>
      )}
    </div>
  );
}       
