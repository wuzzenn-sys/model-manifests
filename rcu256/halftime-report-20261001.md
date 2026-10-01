# MONXTER RCU-256 提示詞掃描戰役 · 上半場戰報（Tier 1–10）

- 日期：2026-10-01（UTC）｜執行：Vivi（小薇）｜老闆：Sheng Wu
- 語料：rcu-mini-256-full（256 題邏輯推理，D1-D4 各 64 題，答案 YES/NO/UNKNOWN/CONFLICT）
- 雙引擎：mini = Ministral-3-8B-Instruct-Q6_K；small4 = small4-text-backbone-Q4_K_M
- 帳本：supa_ai.monxter_rcu_k24_acc_record_v1，gate v29–v32 已落帳

## 總分推進軌跡
| 階段 | 引擎 | 配置 | 總分 | 正確率 |
|---|---|---|---|---|
| baseline | small4 | 原始提示 | 222/256 | 86.7% |
| tier1 | small4 | SYS_BASE 單變體 | 228/256 | 89.1% |
| tier5 | small4 | v10_s4_route | 237/256 | 92.6% |
| tier6 複製 | small4 | v10_s4_route ×3 | 92.6 / 93.4 / 92.6 | 平均 92.9%，最佳 239/256 |

## 冠軍配置（生產確認）
### small4：v10_s4_route（92.6–93.4%，最佳 239/256）
- D3 → SYS_BASE 純提示（You are a precise logic engine. Answer with exactly one word...）
- D1/D2/D4 → SYS_ABD（CANDIDATE ASSUMPTIONS 可採納為額外事實，取最小一致集）+ user 尾綴 HINT
- 三次複製全穩定，正式定為生產配置

### mini：v11_d4_example（80.5–84.0%）
- 在 v10 基礎上，D4 額外加「範例教學」：展示「採納 b:2 → 推出 c:3 → 答 YES」完整推導鏈
- 三次複製 80.5 / 84.0 / 82.8，D4 從 8 → 41/47/46，突破模型天花板
- v10_route 複製兩次完全一致（68.4%，55/48/64/8），已被 v11 取代

## 各域戰況（small4 v10 最佳輪 60/55/64/60）
| 域 | 題型 | small4 | mini(v11) | 狀態 |
|---|---|---|---|---|
| D1 | 單跳/短鏈 | 60/64 | 56/64 | 基本攻下 |
| D2 | 多跳連詞長鏈（X AND Y） | 55/64 | 48/64 | 兩引擎最弱，見下 |
| D3 | 純規則鏈 | 64/64 | 64/64 | 滿分 ×5 連冠 |
| D4 | 候選假設採納（abduction） | 60/64 | 47/64 | 語意已破解 |

## D2 攻堅全紀錄（全部失敗，判定能力天花板）
| 變體 | 引擎 | D2 得分 | 結論 |
|---|---|---|---|
| v10/v11 基線 | small4 / mini | 53–55 / 46–48 | — |
| c13 CONFLICT 前置排序 | 雙引擎 | — | 均不如 v1_system |
| v16 連詞 HINT（檢查兩個合取項） | mini | 6 | 毒藥，崩盤 |
| v15 連詞 HINT | small4 | 51 | 無增益 |
| h1_sys_cot（CoT 逐步推理） | mini | 6（總分 51.6%） | CoT 對 mini 是毒藥 |
| v17_vote3（D2 三取樣多數決 t=0.7） | mini | 41 | 投票稀釋，無效 |
| v18_vote3 | small4 | 53 | 無效 |
| v19_d2example（D2 範例教學） | mini / small4 | 34 / 42 | 反效果 |

D2 判定：錯誤源於長鏈多跳推力不足，非語意誤解。提示工程已到頂，下一步應走教科書路線：Deterministic Motor（K24 primitive）處理 D2，或換更大模型。

## 毒藥清單（勿再試）
v6_restate（雙引擎 0% 全滅）、v7_caps（43–53%）、CoT 全系、溫度投票、D2 連詞/範例提示

## 基礎設施教訓（本場新增）
1. 健康檢查必須 curl -sf：curl -s 對 503（模型載入中）誤判成功，曾導致 256 題全滅（0.1 秒敗）
2. lindanisa 機型：模型在 /opt/monxter/models/（非 /mnt/lssd），llama.cpp 在 /opt/monxter/llama.cpp（非 /opt/llama.cpp）
3. VM 盤點：專案共 5 台——genoa-1、lindanisa、lindanisa2（asia-east1-a）、genoa-2（asia-southeast1-a）、xiongxiong（us-west1，僅 2 核，不適合推理）。lindanisa 曾閒置 9h42m 被抓包，已補工
4. setsid nohup 背景啟動存在時序競態，建議包一層 launcher 腳本上傳後再啟動
5. SSH key 生成+傳播 40–90 秒；Cloud Build 巡檢落地 bucket 需 60–100 秒，期間 404 正常

## 檔案索引（GCS）
- dual-engine/prompt-tier5-runner.py（v10 生產 runner）／prompt-tier6-runner.py（v11）／prompt-tier8-runner.py（v15/v16）／prompt-tier9-runner.py（vote3）／prompt-t10-runner.py（v19）
- dual-engine/s4-tier6/7/8/9.log、genoa/g2-tier6/7/8/9.log、genoa/ld-tier7/8.log、genoa/g1-t10.log——完整戰果
- genoa/d2-samples.txt——D2 題目取樣證據

## 下半場建議
1. D2 交給 Motor：用教科書 K24 deterministic primitive（PROPAGATE/VERIFY）做 D2 前向鏈，繞過 LLM 能力天花板
2. 小引擎分工：small4 全域主力（v10），mini 當低成本離線配置（v11）
3. lindanisa 常駐派工，不再空轉；xiongxiong 可考慮轉做輕量監控節點
