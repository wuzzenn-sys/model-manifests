# MONXTER RCU-256 全 Motor 零 LLM 戰報（2026-10-01）

## 最終戰果：motor-full-v2 = 256/256 = 100%，三機複製確認

| 機器 | 區 | 分數 | parse_fail | 證據 log |
|---|---|---|---|---|
| lindanisa | asia-east1-a | 256/256 | 0 | dual-engine/motor-full-v2.log |
| lindanisa2 | asia-east1-a | 256/256 | 0 | dual-engine/motor-full-v2-rep-l2-b.log |
| genoa-1 | asia-east1-a | 256/256 | 0 | dual-engine/motor-full-v2-rep-g1-j.log |

## 各域終局
- D1 64/64、D2 64/64、D3 64/64、D4 64/64——全部純 Python K24 Motor，LLM 全場零呼叫
- 推進軌跡：hybrid v2 256/256（D3 仍靠 LLM）→ 全 Motor 零 LLM 256/256（D3 否定閉包 Motor 化）

## 關鍵除蟲戰史
1. D3 大寫否定蟲：舊 motor.py lit() 是 case-sensitive startswith("not ")，D3 語料否定是大寫 NOT w:t → 前向鏈閉包缺否定 → 64 題全誤答 YES。v2 改 .lower() 全域 case-insensitive 即修。
2. D3 語意解密（d3-census.txt）：全 64 題 key=CONFLICT，鏈尾中繼原子 X 同掛 (X→w:t) 與 (X→NOT w:t)，問句全問 w:t。
3. D4 abduction 枚舉：cands 以 ; 分組、組內 and、矛盾組跳過、採納後前向鏈推得即 YES，否則 UNKNOWN。

## 基礎設施教訓
- genoa-1 實際在 asia-east1-a（舊戰報 us-west1-a 有誤），跨機操作前務必 list_instances 核實 zone。
- Cloud Build 容器 /tmp 每次全新，build 內引用 /tmp 檔必須先在 build 內拉取，否則重試全是空跑且 || true 靜默吞掉 → build 假 SUCCESS。
- ssh --command="cat" 會混入 SSH host key 雜訊（SHA256:... root@xxx），抓檔一律用 scp 兩參數。
- VM → build /tmp → VM 中轉傳檔可行。

## 檔案索引
- dual-engine/motor-full-v2.py——冠軍腳本（全 Motor 256/256 證明）
- dual-engine/motor-full-v2.log / -rep-l2-b.log / -rep-g1-j.log——三機滿分 log
- dual-engine/d3-census.txt——D3 語意普查
- dual-engine/fulltime-report-20261001.md——上半場+下半場戰報

戰役完全收官。MONXTER Motor 引擎從此 256 題全自動、零 LLM、可跨機復現。
