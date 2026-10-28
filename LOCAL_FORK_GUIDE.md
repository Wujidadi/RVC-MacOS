# 本地 fork 傻瓜式使用指南

2026-10-28T14:46:29+08:00

本指南適用於 `Wujidadi/RVC-MacOS` fork，專案目錄為 `~/Documents/Workspaces/AI/RVC/RVC-MacOS`。
首次安裝（虛擬環境、依賴套件、基礎模型與聲音模型）請依 `CLAUDE.md` 步驟 3～7 完成。

## 兩種介面怎麼選

| 介面   | 程式     | 用途                                 |
| ------ | -------- | ------------------------------------ |
| GUI    | `gui.py` | 即時變聲：對著麥克風說話，即時輸出   |
| Web UI | `web.py` | 轉換錄音檔：上傳音檔，輸出轉換後音檔 |

## 啟動 GUI（即時變聲）

1. 開啟終端機，執行：

   ```zsh
   cd ~/Documents/Workspaces/AI/RVC/RVC-MacOS
   source .venv/bin/activate
   python gui.py
   ```

2. 「选择音色」選一個模型。
3. Input device 選麥克風；Output device 測試時選耳機，通話時選 BlackHole 2ch。
4. 按 `Start audio conversion` 開始，按 `Stop audio conversion` 停止。
5. 關閉視窗或在終端機按 `Ctrl+C` 結束程式。

GUI 會記住上次的設定，存放在 `configs/inuse/config.json`；刪除這個檔案即可回到 `configs/config.json` 的預設值。

## 啟動 Web UI（轉換錄音檔）

1. 開啟終端機，執行：

   ```zsh
   cd ~/Documents/Workspaces/AI/RVC/RVC-MacOS
   source .venv/bin/activate
   python web.py
   ```

2. 瀏覽器會自動開啟 `http://localhost:7865`；沒有開啟時請手動輸入這個網址。
3. 在「Model Inference」分頁的 Inferencing voice 選模型。
4. 在 Single inference 上傳音檔，按 `Convert`。
5. 轉換完成後，在 Export audio 右下角的選單下載結果。
6. 整個資料夾的音檔可改用 Batch inference 分頁。
7. 在終端機按 `Ctrl+C` 結束程式。

常用啟動參數：

- `--port 8000`：改用其他連接埠。
- `--noautoopen`：不自動開啟瀏覽器。

## 推薦參數

GUI 與 Web UI 的預設值都已調成下列建議值，一般不需更動。

| 參數                    | GUI    | Web UI |
| ----------------------- | ------ | ------ |
| Transpose／Pitch        | 12     | 12     |
| 音高演算法              | fcpe   | rmvpe  |
| Feature searching ratio | 0.3    | 0.5    |
| Response threshold      | -45    | 不適用 |
| Sample length           | 0.25   | 不適用 |
| Extra inference time    | 1.0    | 不適用 |
| Protect                 | 不適用 | 0.33   |

Transpose 依性別調整：

- 男聲轉女聲：12，太尖可降到 8～10。
- 女聲轉男聲：-12。
- 同性別：0。

## 介面語言

啟動指令前加上 `LANG=zh_TW.UTF-8` 可切換為繁體中文介面，例如 `LANG=zh_TW.UTF-8 python web.py`。
本指南的欄位名稱以英文介面為準。

## 這個 fork 的修改

- index 維度與模型版本不符時自動略過 index，不再出現 `index mistatch`。
  上游模型倉庫中，丁真、卢本伟、女声-OI、女声-佩佩、女声-石榴的 index 屬於這種情況，對這 5 個模型調整 Feature searching ratio 不會有效果。
- 在 Apple Silicon 上，Web UI 的聲碼器改在 CPU 執行，音訊並切成較短片段，以避開 MPS 卷積輸出長度上限 65536。
- 修正 torch 與 faiss 各自內含 libomp 時，faiss 搜尋造成的程式崩潰。
- GUI 啟動轉換前先預先編譯 numba 函式，避免 CoreAudio 執行緒崩潰。

## 常見問題

- **聲音有機械感**：先確認 Transpose 符合性別；錄音有背景音樂或殘響時，先在「Vocals/Accompaniment Separation」分頁分離人聲再轉換。
- **GUI 聲音斷斷續續**：轉換中的 Inference time 大於 Sample length 的毫秒數時，加大 Sample length。
- **終端出現 `[error with torchfcpe.mel_extractor.MelModule]`**：輸入音量超過 ±1 的提示，不影響轉換。
- **終端出現 `Index dim ... skip index`**：模型的 index 不符，已自動略過，轉換仍會完成。

## 同步上游更新

```zsh
git fetch upstream
git merge upstream/main
```

合併時若 `infer/modules/vc/pipeline.py`、`configs/config.py`、`web.py`、`gui.py` 或 `infer/lib/rtrvc.py` 發生衝突，需保留本 fork 的修改。
