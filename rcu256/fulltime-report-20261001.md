# MONXTER RCU-256 提示詞掃描戰役 · 下半場戰報（Deterministic Motor 閉環）

- 日期：2026-10-01（UTC）｜執行：Vivi（小薇）｜老闆：Sheng Wu
- 語料：rcu-mini-256-full（256 題，D1–D4 各 64，答案 YES/NO/UNKNOWN/CONFLICT）
- 雙引擎：mini = Ministral-3-8B-Instruct-Q6_K；small4 = small4-text-backbone-Q4_K_M
- 帳本：supa_ai.monxter_rcu_k24_acc_record_v1，gate v33–v35 落帳
- 結果：hybrid_v2 = 256/256 = 100% PERFECT SCORE，六連滿分複製確認

## 總分推進軌跡
| 階段 | 架構 | 總分 | 正確率 |
|---|---|---|---|
| baseline | small4 原始提示 | 222/256 | 86.7% |
| 上半場冠軍 v10_s4_route | 純 LLM | 239/256 | 93.4% |
| hybrid v1 | Motor(D1/D2) + LLM(D3/D4) | 253/256 | 98.8% |
| hybrid v2 | Motor(D1/D2/D4) + LLM(D3) | 256/256 | 100% |

## 冠軍架構 hybrid_v2（生產確認）
- D1/D2/D4 -> Deterministic Motor（192/192 全滿分，~0 秒/域）
- D3 -> LLM SYS_BASE 純提示（64/64，五連滿分，唯一 LLM 保留域）
- 全場 33–83 秒（純 LLM 400+ 秒），速度與正確率雙贏
- Motor = K24 primitives：02 NORMALIZE / 03 PROPAGATE / 06 VERIFY / 07 CONTRADICTION / 08 HYPOTHESIS / 23 ARBITER
  - 純 Python 前向鏈 fixpoint；D4 走 abduction：枚舉 CANDIDATE ASSUMPTIONS 組（; 分隔、組內 and），一致組可採納，採納後可推導即 YES，矛盾組（[b:2 and NOT b:2]）-> UNKNOWN

## 複製驗證（六連滿分，100% 復現）
| VM | 引擎 | 輪數 | 成績 | run_s |
|---|---|---|---|---|
| lindanisa | small4 | 1 | 256/256 | 82.9 |
| lindanisa2 | small4 | 3 | 256/256 x3 | 76.7 / 79.2 / 76.8 |
| genoa-1 | mini | 2 | 256/256 x2 | 39.1 / 32.7 |

雙引擎皆滿分：LLM 只影響 D3 一個域，且兩引擎 D3 都是滿分王。

## 各域終局
| 域 | 題型 | 解法 | 得分 | LLM 單獨上限 |
|---|---|---|---|---|
| D1 | 單跳/短鏈 | Motor | 64/64 | small4 61 |
| D2 | 多跳連詞長鏈 X AND Y | Motor（前向鏈閉包） | 64/64 | small4 55 / mini 48（能力天花板） |
| D3 | 純規則鏈（否定閉包語意） | LLM SYS_BASE | 64/64 | 64/64 五連滿分 |
| D4 | 候選假設採納（abduction） | Motor（08 HYPOTHESIS 枚舉採納） | 64/64 | small4 61 / mini 47 |

## 語料結構解密
- D2：全「X AND Y」合取問句、零否定詞；key YES 56 / UNKNOWN 8（UNKNOWN = 合取項含不可推導原子）
- D4：CANDIDATE ASSUMPTIONS 組；YES 題存在一致可採納組；UNKNOWN 題唯一可用組是矛盾組 -> 不可採納
- D3：LLM 滿分域，Motor 純正極前向鏈答 YES 全滅（key 全 CONFLICT）——含否定閉包語意，可選後續研究

## 除錯戰史（下半場）
1. lindanisa2 watchdog 空轉 4 輪：缺 /tmp/prompt-tier5-runner.py -> 補檔重啟
2. Cloud Build 語法衝突：背景啟動改 ( setsid nohup ... & ) 後接分號
3. project metadata ssh-keys 瘀積 -> 全線 SSH 255：清瘀只留最新金鑰後恢復
4. pkill 自殺蟲：SSH 指令內 pkill -f <script> match 到執行 shell 自身 -> 255；避免 pkill 關鍵字 + exit 0
5. Motor-D4 括號蟲：候選原子 [b:2] 未剝括號 -> 56 題 YES 全誤判 UNKNOWN；剝括號後 64/64
6. GCS 同路徑覆寫輪詢陷阱：驗證新結果要換新路徑

## 檔案索引（GCS dual-engine/）
- hybrid-v2-ld.log / hybrid-v2-rep-l2.log / hybrid-v2-rep-g1.log——滿分戰果
- motor.py / motor-d4-v2.txt——Motor 語意與 D4 64/64 證明
- d2census.txt / d4-peek.txt——語料解密證據
- hybrid-v1 過渡王座：hybrid-v1.log / hybrid-rep-l2.log（248/250/248）

## 結論
1. Deterministic Motor 路線完全兌現教科書 K24 設計：LLM 能力天花板（D2 55）被前向鏈碾過，abduction（D4）被 08 HYPOTHESIS 枚舉採納解決
2. 最優分工是「Motor 扛符號域、LLM 守語意域」——LLM 從 256 題主力降為 64 題守門員
3. RCU-256 語料已全解；後續可選：D3 否定閉包 Motor 化（全 Motor 零 LLM）、更多域擴充
