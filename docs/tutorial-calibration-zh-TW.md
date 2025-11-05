# 校準教學

本文件說明 libsurvive 中校準的實作方式，特別針對配備 2 個 HTC Vive Base Station 2.0 和 1 個 HTC Vive Tracker 3.0 的設定。

## 概述

校準是確定 lighthouse 基地台相對於追蹤物件位置和方向的過程。這是實現精確 6DOF 追蹤的關鍵步驟。校準過程包含幾個階段：

1. **OOTX 數據收集**：從基地台收集校準參數
2. **光線數據處理**：處理來自追蹤器的感應器激活數據
3. **位置求解**：計算基地台與追蹤器之間的相對位置
4. **世界座標系建立**：將相對位置轉換為固定的世界座標系
5. **配置持久化**：保存校準結果以供未來使用

## 用戶指南：如何執行校準

本節提供逐步說明，指導您使用 HTC Vive Base Station 2.0 和 Tracker 3.0 設定進行校準。

### 系統限制

**⚠️ 重要限制：**

- **最大基地台數量**：libsurvive 最多支援 **16 個基地台**（由 `NUM_GEN2_LIGHTHOUSES` 定義）。此限制適用於 Base Station 2.0（Lighthouse 2.0）系統。對於 Base Station 1.0 系統，限制為 2 個基地台。
- **最大追蹤器數量**：追蹤器數量沒有硬編碼限制，但實際限制取決於：
  - USB 頻寬和連接埠可用性
  - CPU 處理能力
  - 系統記憶體
  - 實際上，通常使用 10-20 個追蹤器，但在硬體充足的情況下可以使用更多

**注意**：如果您超過最大基地台數量，系統只會識別並校準前 16 個檢測到的基地台。額外的基地台將被忽略。

### 先決條件

在開始校準之前，請確保您擁有：

- 2-16 × HTC Vive Base Station 2.0 已開機並處於追蹤模式（或 2 × Base Station 1.0）
- 1 個或多個 × HTC Vive Tracker 3.0（或其他相容的追蹤裝置）通過 USB 連接到您的電腦
- 已安裝並編譯 libsurvive（請參閱主 README.md 了解安裝說明）
- 已關閉 SteamVR（libsurvive 可能會與 SteamVR 競爭裝置存取權限）

### 不同基地台配置的推薦房間大小

**Base Station 2.0 追蹤範圍：**
- **有效範圍**：從基地台到追蹤器最多 7 公尺（23 英尺）
- **最佳範圍**：2-5 公尺（6.5-16 英尺）以獲得最佳追蹤品質
- **視野角度**：每個基地台約 120 度

**推薦房間大小：**

| 基地台數量 | 最小房間大小 | 最大房間大小 | 典型使用場景 |
|----------|------------|------------|------------|
| **2 個基地台** | 2m × 1.5m (6.5' × 5') | 7m × 7m (23' × 23') | 中小型房間、住宅 VR 設定 |
| **3-4 個基地台** | 3m × 3m (10' × 10') | 10m × 10m (33' × 33') | 大型房間、專業 VR 設定、遊戲廳 |
| **5-8 個基地台** | 5m × 5m (16' × 16') | 15m × 15m (50' × 50') | 非常大的空間、倉庫、活動場地 |
| **9-16 個基地台** | 10m × 10m (33' × 33') | 20m+ × 20m+ (65'+ × 65'+) | 工業應用、研究實驗室、動作捕捉工作室 |

**重要考量：**
- **追蹤品質**：更多基地台提供更好的追蹤覆蓋，特別是在追蹤空間邊緣
- **遮擋**：額外的基地台有助於減少當使用者或物件阻擋視線時的遮擋問題
- **冗餘性**：更多基地台提供冗餘性，如果一個故障或暫時被阻擋
- **校準複雜度**：更多基地台需要更仔細的校準，特別是在大型空間中

**基地台之間的距離：**
- **最小距離**：基地台之間 2 公尺（6.5 英尺）
- **最大距離**：10 公尺（33 英尺）以獲得最佳覆蓋重疊
- **推薦距離**：相距 3-5 公尺（10-16 英尺）以獲得平衡的覆蓋

### 多房間設置

**可以在多個房間使用基地台嗎？**

**簡短答案**：技術上可行，但不建議用於單一統一的追蹤空間。

**詳細說明：**

1. **單一統一追蹤空間（不推薦）**：
   - 基地台設計為在單一、連續的追蹤空間內工作
   - 房間之間的牆壁和障礙物會阻擋視線，導致追蹤失敗
   - 在房間之間移動時，追蹤品質會顯著下降
   - 由於追蹤器無法同時看到所有基地台，校準變得非常困難

2. **具有獨立追蹤空間的獨立房間（可行）**：
   - **設置**：每個房間都有自己的基地台組（建議每間 2-4 個）
   - **配置**：每個房間維護自己的獨立 `config.json` 檔案
   - **切換**：使用者在房間之間移動時必須重新校準或重新載入配置
   - **使用場景**：不同房間中的不同應用程式或使用者
   - **限制**：追蹤器無法在房間之間無縫移動，需要重新校準

3. **具有多個區域的大型開放空間（推薦）**：
   - **設置**：在大型開放空間中分布 4-8 個基地台
   - **配置**：所有基地台一起校準的單一統一追蹤空間
   - **優勢**：在整個空間中無縫追蹤
   - **使用場景**：大型倉庫、活動大廳或開放商業空間

**多房間場景的最佳實踐：**

- **選項 A - 獨立配置**：
   - 為每個房間維護獨立的校準檔案
   - 使用具有不同配置檔案的 libsurvive 實例：`./bin/survive-cli --config <room_config.json>`
   - 在房間之間移動時手動切換配置

- **選項 B - 大型開放空間**：
   - 移除或最小化區域之間的牆壁/障礙物
   - 將基地台安裝在足夠高的位置以看到障礙物上方
   - 使用更多基地台（6-12 個）以獲得更好的覆蓋
   - 確保所有基地台可以相互看到或具有重疊覆蓋

- **選項 C - 混合方法**：
   - 在中央「樞紐」區域使用 4-6 個基地台，具有良好覆蓋
   - 在相鄰房間中添加 2 個基地台以獲得部分覆蓋
   - 接受周邊房間的追蹤品質可能會降低

**技術限制：**

- **視線**：基地台需要清晰的視線到追蹤器以進行準確追蹤
- **校準**：所有基地台必須至少從一個校準點可見
- **同步**：基地台需要彼此同步（Gen 2 自動完成）
- **座標系**：在校準期間建立單一世界座標系

**高架安裝的多房間設置（無天花板或高天花板）：**

如果您的設置**沒有天花板**或**天花板非常高**，您可以將基地台安裝得**比牆壁更高**，以創建跨越多個房間的統一追蹤空間：

**優勢：**
- **統一追蹤空間**：安裝在牆壁高度以上的基地台可以看到多個房間
- **無縫移動**：追蹤器可以在房間之間移動而無需重新校準
- **單一座標系**：所有房間共享相同的世界座標系
- **更好的覆蓋**：基地台通過位於障礙物上方可以覆蓋更大的區域

**設置要求：**
- **安裝高度**：基地台必須安裝在**至少比最高牆壁高 0.5-1 公尺（1.5-3 英尺）**的位置，以確保清晰的視線
- **安裝方式**：將基地台安裝在高桿、建築結構或高安裝支架上
- **推薦高度**：3-5 公尺（10-16 英尺）或更高，取決於牆壁高度
- **傾斜角度**：將基地台向下傾斜 30-45 度以覆蓋下方的追蹤空間
- **分布**：均勻分布基地台以確保所有房間的覆蓋重疊

**配置：**
1. **初始設置**：將所有基地台安裝在足以看到牆壁上方的高度
2. **校準**：將追蹤器放置在可以看到所有基地台的中央位置
3. **測試**：在房間之間移動追蹤器以驗證追蹤品質
4. **調整**：如果某些區域的追蹤品質不佳，添加更多基地台或調整安裝位置

**最佳實踐：**
- **高度安全**：確保基地台牢固安裝且易於維護
- **覆蓋測試**：在每個房間測試追蹤品質，特別是在牆壁附近
- **電源供應**：考慮為高架安裝點的基地台進行電源分配
- **維護**：確保基地台易於存取，以便進行韌體更新和故障排除

**範例場景：**
- **開放式倉庫**：由隔板分隔的多個區域，基地台安裝在天花板橫樑上
- **高天花板大廳**：具有部分牆壁的多個房間，基地台安裝在牆壁高度以上
- **室內外混合**：具有部分牆壁的大型空間，基地台安裝在高桿上

**限制：**
- **牆壁高度**：非常高的牆壁（>3-4 公尺）可能仍會阻擋視線
- **障礙物**：家具、設備或其他高物體可能仍會造成遮擋
- **校準難度**：校準期間可能需要攀爬或使用升降設備來存取基地台
- **電源/線纜管理**：高架基地台的佈線更複雜

### 步驟 1：設置您的硬體

1. **安裝基地台**： 
   - **對於 2 個基地台**：將它們放置在追蹤區域的對角
   - **對於 3-4 個基地台**：在角落使用正方形或矩形模式
   - **對於 5-8 個基地台**：在追蹤空間周圍均勻分布以獲得最佳覆蓋
   - **對於 9-16 個基地台**：創建覆蓋整個追蹤空間的網格模式
   - 將所有基地台安裝在 2-3 公尺（6-10 英尺）的高度，向下傾斜約 30-45 度
   - 確保它們彼此之間以及與追蹤空間之間有清晰的視線
   - 對於大型空間，確保基地台間距 3-5 公尺以獲得最佳覆蓋重疊
   - Base Station 2.0 裝置會自動分配通道（B、C、D 等）以進行正確同步
   - **重要**：不要超過 16 個基地台 - 系統只會識別前 16 個

2. **開啟基地台**：
   - 將所有基地台連接到電源
   - 等待它們完全初始化（LED 指示燈會顯示其狀態）
   - 確保所有基地台都處於追蹤模式（非待機模式）
   - 驗證每個基地台都有唯一的通道分配（在 LED 指示燈中可見）

3. **連接追蹤器**：
   - 將您的 Vive Tracker 3.0（或多個追蹤器）通過 USB 連接到您的電腦
   - 每個追蹤器應該會自動開機
   - 系統會自動檢測所有連接的追蹤器
   - 如果您有多個追蹤器，可以同時連接它們 - 校準會使用每個基地台的最佳可用追蹤器

### 步驟 2：初始校準

1. **啟動 libsurvive**：
   ```bash
   ./bin/survive-cli
   ```
   
   或者如果您想看到更詳細的輸出：
   ```bash
   ./bin/survive-cli --v 100
   ```

2. **放置追蹤器**：
   - **對於 2 個基地台**：將追蹤器放置在可以清楚「看到」兩個基地台的位置
   - **對於 3+ 個基地台**：將追蹤器放置在中央位置，使其可以看到盡可能多的基地台
   - 如果使用多個追蹤器，您可以在校準期間保持一個或多個靜止
   - 追蹤器應放置在平坦、穩定的表面上
   - **⚠️ 重要：在校準期間保持追蹤器完全靜止 - 不需要也不建議移動**
   - 追蹤器的感應器應該對基地台有無阻礙的視野
   - **提示**：如果您有多個追蹤器，系統會自動使用感應器覆蓋最佳的那個進行校準

3. **等待校準**：
   - 系統會自動開始從所有基地台收集 OOTX 數據
   - **保持追蹤器靜止** - 在此階段不要移動它
   - 根據訊號品質，每個基地台通常需要 10-60 秒
   - 對於多個基地台，校準可能需要更長時間，因為系統需要從每個基地台收集 OOTX 數據
   - 您應該在控制台中看到指示 OOTX 封包正在接收的訊息
   - 例如：`Got OOTX packet 1 00000001`、`Got OOTX packet 2 00000002` 等
   - 每個基地台都會被分配一個在這些訊息中可見的唯一 ID

4. **監控進度**：
   - **繼續保持追蹤器靜止**，同時監控進度
   - 觀看控制台輸出的校準狀態訊息
   - 系統會指示何時正在求解基地台位置
   - 您可能會看到類似 "Using LH 0 (...) as reference lighthouse" 的訊息
   - 對於多個基地台，您會看到每個的進度：
     - `Solving for X correspondents`（其中 X 是感應器測量數量）
     - 指示正在校準哪些基地台的訊息
   - 當所有基地台在日誌中顯示 `PositionSet = 1` 時，校準完成
   - **對於 3+ 個基地台**：如果最初並非所有基地台都校準：
     - **只有在那時**才將追蹤器移動到可以看到剩餘基地台的新位置
     - **在新位置再次保持靜止**，直到剩餘基地台校準完成
   - **注意**：校準不需要運動 - 系統只需要追蹤器保持靜止以收集足夠的光線數據

### 為什麼不需要移動：與相機校準的區別

您可能熟悉相機校準，它通常需要在空間中採樣多個點（例如，將校準圖案移動到不同位置）。**libsurvive 的校準方法根本不同**，不需要移動。原因如下：

**傳統相機校準：**
- 需要**已知的 3D 參考點**（例如，具有已知尺寸的棋盤圖案）
- 需要從不同位置和方向**多次查看**同一圖案
- 求解相機內參數（焦距、失真）和外參數（位置、方向）
- 需要**2D-3D 對應關係** - 將圖像點匹配到已知的 3D 世界座標

**libsurvive Lighthouse 校準（為什麼不同）：**

1. **追蹤器上已知的感應器位置**：
   - 追蹤器具有**工廠校準的感應器位置**，嵌入在其韌體中
   - 每個感應器相對於追蹤器的 3D 位置是精確已知的
   - 這消除了對外部參考點的需求

2. **主動照明系統**：
   - 基地台**主動掃描**雷射光束穿過追蹤空間
   - 追蹤器接收**角度測量**（不是 2D 圖像投影）
   - 每個感應器報告它接收雷射掃描的角度
   - 在**靜止追蹤器**上的多個感應器同時提供多個角度測量

3. **單一位置提供足夠的約束**：
   - 當追蹤器靜止時，**所有感應器同時從同一基地台接收光線**
   - 每個感應器提供**兩個角度測量**（一個用於 X 軸掃描，一個用於 Y 軸掃描）
   - 在典型追蹤器上約有 20-30 個感應器，這為每個基地台提供**40-60 個角度測量**
   - 這些測量**足以求解**基地台相對於追蹤器的位置和方向

4. **數學充分性**：
   - 問題是：給定 N 個已知感應器位置和 N 個角度測量，求解基地台位置
   - 這是一個**Perspective-n-Point (PnP) 問題**的變體
   - 使用 4-5 個感應器，問題變得可解；使用 20+ 個感應器，它是高度超定的
   - **不需要移動**，因為單一靜止位置提供了足夠的約束

5. **世界座標系建立**：
   - IMU（加速度計）提供**重力方向**以建立「向上」
   - 第一個基地台建立**參考方向**
   - 兩者結合，無需多個位置即可創建一致的世界座標系

**為什麼移動實際上是有害的：**
- 在校準期間移動追蹤器**會引入關於追蹤器位置的不確定性**
- 校準算法假設追蹤器是靜止的，以準確關聯感應器測量
- 移動將需要同時求解**追蹤器軌跡和基地台位置**，這更複雜且準確性較低
- 靜止數據允許系統從同一位置**累積許多測量**，提高準確性

**對比總結：**

| 方面 | 相機校準 | libsurvive 校準 |
|------|---------|----------------|
| **參考點** | 外部（棋盤、已知 3D 點） | 內部（追蹤器上的感應器位置） |
| **需要移動** | 是（多個位置） | 否（靜止） |
| **測量類型** | 2D 圖像座標 | 3D 角度測量 |
| **照明** | 被動（環境光） | 主動（雷射掃描） |
| **每個位置的約束** | 有限（一個視圖） | 許多（所有感應器同時） |
| **座標系** | 從參考點建立 | 從重力 + 第一個基地台建立 |

**總結**：libsurvive 的校準之所以有效，是因為追蹤器的感應器位置是已知的，靜止追蹤器上的多個感應器提供了足夠的角度測量來求解基地台位置。這與相機校準根本不同，相機校準需要外部參考點和多個視圖。

### 步驟 3：驗證校準

校準完成後，驗證是否成功：

1. **檢查配置檔案**：
   - 校準數據保存在 `XDG_CONFIG_HOME/libsurvive` 中的 `config.json`（在 Linux 上通常是 `~/.config/libsurvive/config.json`）
   - 打開檔案並驗證**所有**基地台都有：
     - `"OOTXSet": 1`（已收集 OOTX 數據）
     - `"PositionSet": 1`（已求解位置）
     - 有效的 `"pose"` 陣列，包含 7 個值（位置 x,y,z 和旋轉四元數 w,x,y,z）
     - 每個基地台唯一的 `"id"` 值
   - **對於多個基地台**：計算配置檔案中的基地台條目數量 - 應該與您擁有的基地台數量相符（最多 16 個）
   - **對於多個追蹤器**：配置檔案只包含基地台位置 - 追蹤器特定數據單獨儲存

2. **測試追蹤**：
   - 在追蹤空間中移動追蹤器
   - 系統現在應該為所有追蹤器提供準確的 6DOF 位置數據
   - **使用多個基地台**：您應該有更好的追蹤覆蓋，特別是在追蹤空間邊緣
   - **使用多個追蹤器**：所有追蹤器應該同時被追蹤
   - 使用可視化工具驗證追蹤：
     ```bash
     ./bin/survive-websocketd & xdg-open ./tools/viz/index.html
     ```
   - 可視化將顯示所有連接的追蹤器和所有已校準的基地台

### 步驟 4：如何更改基地台通道

Base Station 2.0 支援通道 1-16（內部表示為模式 B、C、D 等）。預設情況下，基地台會自動分配通道以避免衝突。但是，您可能想要為您的設定手動設定特定通道。

**先決條件：**
- 已安裝 Python 3 和 `bluepy` 庫：`pip3 install bluepy`
- 支援 Bluetooth 的 Linux 系統（GATT 通訊所需）
- root 權限或適當的 Bluetooth 權限

**方法 1：使用 bsd_ctrl.py 工具**

libsurvive 包含一個用於管理基地台通道的工具：

1. **導航到工具目錄**：
   ```bash
   cd tools/bsd_ctrl
   ```

2. **查看當前基地台通道**：
   ```bash
   python3 bsd_ctrl.py -t 4
   ```
   這會掃描基地台並顯示其當前的模式/通道分配。

3. **為基地台設定特定通道**：
   ```bash
   python3 bsd_ctrl.py -m 2 -d <MAC_ADDRESS>
   ```
   將 `<MAC_ADDRESS>` 替換為要配置的基地台的 Bluetooth MAC 位址。`-m` 參數設定模式（1-16）。

4. **自動分配通道以解決衝突**：
   ```bash
   python3 bsd_ctrl.py -t 4
   ```
   如果多個基地台具有相同通道，工具會自動將它們重新分配為唯一通道。

**方法 2：使用 libsurvive 的 GATT 驅動程式**

libsurvive 的 GATT 驅動程式在啟用時可以自動解決通道衝突：

1. **啟用 GATT 驅動程式**：
   ```bash
   ./bin/survive-cli --gatt
   ```
   GATT 驅動程式會通過 Bluetooth 檢測基地台，並在檢測到衝突時自動重新分配通道。

**重要注意事項：**

- **通道範圍**：Base Station 2.0 支援通道 1-16（分別映射到模式 B、C、D、E、F、G、H、I、J、K、L、M、N、O、P、Q）
- **持久性**：通道更改會保存到基地台，並在電源循環後保持
- **LED 指示燈**：基地台 LED 指示燈會顯示當前通道分配
- **電源循環**：更改通道後，您可能需要對基地台進行電源循環以使更改完全生效
- **驗證**：更改通道後，使用 `bsd_ctrl.py` 或檢查 libsurvive 日誌來驗證新的通道分配

**通道問題故障排除：**

- **無法連接到基地台**：確保 Bluetooth 已啟用且您擁有適當的權限
- **通道更改不持久**：嘗試在更改通道後對基地台進行電源循環
- **多個基地台使用相同通道**：使用不帶 `-m` 標誌的 `bsd_ctrl.py` 自動分配唯一通道
- **追蹤期間的通道衝突**：如果啟用，libsurvive 的 GATT 驅動程式會自動解決衝突

### 步驟 5：強制重新校準（如需要）

如果您需要重新校準（例如，基地台被移動），您有兩個選項：

**選項 1：使用 force-calibrate 標誌**（推薦）：
```bash
./bin/survive-cli --force-calibrate
```
這會重用現有的 OOTX 數據但重新計算位置，比完整重新校準更快。

**選項 2：刪除配置檔案**：
```bash
rm ~/.config/libsurvive/config.json
./bin/survive-cli
```
這會強制從頭開始完整重新校準。

### 最佳結果提示

1. **最佳放置**：
   - 在校準期間將追蹤器放置在追蹤空間的中心
   - 確保追蹤器距離每個基地台至少 1-2 公尺
   - 避免可能造成干擾的反射表面

2. **校準期間**：
   - **保持追蹤器完全靜止** - 不需要也不要求移動
   - 系統使用靜止的光線數據來求解基地台位置
   - 保持追蹤器靜止至少 30-60 秒（或直到校準完成）
   - 在看到校準完成的確認之前不要移動追蹤器
   - 追蹤器在整個過程中應該保持開機並連接
   - **重要**：在校準期間移動追蹤器可能會干擾位置求解過程

3. **多個追蹤器**：
   - 如果您有多個追蹤器，每個基地台設定只需校準一次
   - 校準數據儲存在配置檔案中，所有追蹤器共享
   - 後續追蹤器會自動使用現有校準
   - **多個追蹤器的校準過程**：
     - 通過 USB 將所有追蹤器連接到您的電腦
     - 在校準期間，系統會使用感應器最多的追蹤器（通常是 HMD）作為主要校準裝置
     - 如果存在多個追蹤器，系統會為每個基地台優先使用感應器覆蓋最佳的那個
     - 所有追蹤器都可以為校準過程貢獻數據，提高準確性
     - 校準完成後，所有連接的追蹤器將使用相同的基地台位置
   - **最佳實踐**：在校準期間至少保持一個追蹤器靜止以建立穩定的參考點

4. **多個基地台（3-16）**：
   - **設置**：以提供良好追蹤空間覆蓋的模式安裝基地台
     - 對於 3-4 個基地台：在角落使用正方形或矩形模式
     - 對於 5-8 個基地台：沿牆壁或中點添加基地台
     - 對於 9-16 個基地台：在大型追蹤空間周圍均勻分布
   - **通道分配**：Base Station 2.0 自動分配通道（模式 B、C、D 等）以避免衝突
     - 系統支援通道 0-15（共 16 個通道）
     - 每個基地台將使用唯一的通道/頻率
   - **校準策略**：
     - 將追蹤器放置在可以看到盡可能多基地台的中央位置
     - 系統會同時校準所有可見的基地台
     - 如果並非所有基地台都從一個位置可見：
       - 首先使用追蹤器看到盡可能多的基地台進行校準
       - 將追蹤器移動到另一個位置以看到額外的基地台
       - 保持靜止直到剩餘基地台校準完成
     - 參考基地台（第一個校準的）建立世界座標系
   - **驗證**：檢查所有基地台在日誌或配置檔案中顯示 `PositionSet = 1`
   - **性能**：更多基地台提供更好的追蹤覆蓋，但可能略微增加處理開銷

5. **大型空間**：
   - 如果您的追蹤空間很大，單個追蹤器無法看到所有基地台：
     - 首先使用追蹤器看到盡可能多的基地台進行校準
     - 然後將追蹤器移動到可以看到剩餘基地台的位置
     - 再次保持靜止直到剩餘基地台校準完成

### 常見問題和解決方案

**問題：「OOTX not set」訊息**
- **解決方案**：確保基地台已開機並處於追蹤模式。等待更長時間（最多 60 秒）以接收 OOTX 數據。檢查追蹤器和基地台之間是否有障礙物。

**問題：「Can't solve for only X points」**
- **解決方案**：追蹤器需要從每個基地台看到至少 4-5 個感應器。將追蹤器移動到對兩個基地台都有更清晰視線的更好位置。

**問題：校準時間過長**
- **解決方案**：確保追蹤器和基地台之間有良好的視線。檢查反射或干擾。驗證基地台是否正確同步。

**問題：校準後追蹤不準確**
- **解決方案**：使用 `--force-calibrate` 強制重新校準。驗證基地台自上次校準後未被移動。檢查配置檔案是否包含所有基地台的有效位置數據。

**問題：並非所有基地台都在校準（3+ 個基地台）**
- **解決方案**：確保追蹤器可以看到所有基地台。您可能需要將追蹤器移動到不同位置以看到所有基地台。在每個位置保持靜止直到基地台校準完成。檢查您是否超過了 16 個基地台限制。

**問題：只有部分多個追蹤器被追蹤**
- **解決方案**：確保所有追蹤器都通過 USB 正確連接並已開機。檢查 USB 頻寬 - 一個 USB 控制器上連接太多裝置可能會導致問題。嘗試將追蹤器連接到不同的 USB 連接埠或控制器。通過運行 `./bin/survive-cli --v 100` 並檢查裝置列表來驗證所有追蹤器是否被檢測到。

**問題：系統在使用多個基地台和/或追蹤器時變慢**
- **解決方案**：這在大規模設定中是預期的。考慮：
  - 使用更強大的 CPU
  - 降低詳細程度（使用 `--v 10` 而不是 `--v 100`）
  - 關閉使用 CPU/GPU 資源的其他應用程式
  - 對於非常大的設定（10+ 個基地台 + 10+ 個追蹤器），您可能需要優化系統配置

### 在校準期間使用可視化

對於校準過程的可視化表示，您可以使用基於網頁的可視化工具：

```bash
# 終端 1：啟動 websocket 伺服器
./bin/survive-websocketd --v 100

# 終端 2：打開可視化（或使用瀏覽器）
xdg-open ./tools/viz/index.html
```

可視化將顯示：
- 基地台（lighthouse）在被檢測和校準時
- 追蹤器位置和方向的實時顯示
- 正在建立的座標系

這對於理解校準過程和驗證一切是否正常運作特別有幫助。

## 相關組件

### Base Station 2.0 (Lighthouse 2.0)

Base Station 2.0 使用帶有兩個軸（X 和 Y）的旋轉雷射系統，掃描追蹤空間。每個基地台都包含通過 OOTX（Out-of-TX）協議傳輸的工廠校準數據。

### Tracker 3.0

Vive Tracker 3.0 配備了多個光電二極體（感應器），在其表面以已知模式排列。當來自基地台的雷射掃描擊中感應器時，追蹤器會記錄時間資訊，用於計算角度。

## 校準過程

### 階段 1：OOTX 數據收集

**什麼是 OOTX？**

OOTX（Out-of-TX）是 lighthouse 基地台用於廣播其工廠校準參數的協議。此數據嵌入在同步脈衝中，必須解碼。

**實作細節：**

OOTX 解碼器從同步脈衝收集位元並重建校準封包：

```32:85:src/survive_process_gen2.c
STATIC_CONFIG_ITEM(SERIALIZE_OOTX, "serialize-ootx", 'b', "Serialize out ootx", 0)
static void ootx_packet_clbk_d_gen2(ootx_decoder_context *ct, ootx_packet *packet) {
	SurviveContext *ctx = ((SurviveObject *)(ct->user))->ctx;
	int id = ct->user1;

	lighthouse_info_v15 v15;
	init_lighthouse_info_v15(&v15, packet->data);

	if (survive_configi(ctx, SERIALIZE_OOTX_TAG, SC_GET, 0) == 1) {
		char filename[128];
		snprintf(filename, 128, "LH%02d_%08x.ootx", v15.mode_current & 0x7F, (unsigned)v15.id);
		FILE *f = fopen(filename, "w");
		fwrite(packet->data, packet->length, 1, f);
		fclose(f);
	}

	BaseStationData *b = &ctx->bsd[id];
	b->OOTXChecked |= true;
	FLT accel[3] = {v15.accel_dir[0], v15.accel_dir[1], v15.accel_dir[2]};
	bool upChanged = norm3d(b->accel) != 0.0 && dist3d(b->accel, accel) > 1e-3;

	if (upChanged) {
		SV_VERBOSE(10, "OOTX up direction changed for %x (%f)", b->BaseStationID, norm3d(b->accel));
	}
	bool doSave = b->BaseStationID != v15.id || b->OOTXSet == false || upChanged;
	b->OOTXSet = 1;

	if (doSave) {
	  SV_INFO("Got OOTX packet %d %08x", ctx->bsd[id].mode, (unsigned)v15.id);

		b->BaseStationID = v15.id;
		for (int i = 0; i < 2; i++) {
			b->fcal[i].phase = v15.fcal_phase[i];
			b->fcal[i].tilt = v15.fcal_tilt[i];
			b->fcal[i].curve = v15.fcal_curve[i];
			b->fcal[i].gibpha = v15.fcal_gibphase[i];
			b->fcal[i].gibmag = v15.fcal_gibmag[i];
			b->fcal[i].ogeephase = v15.fcal_ogeephase[i];
			b->fcal[i].ogeemag = v15.fcal_ogeemag[i];
		}

		for (int i = 0; i < 3; i++) {
			b->accel[i] = v15.accel_dir[i];
		}
		b->sys_unlock_count = v15.sys_unlock_count;

		// Although we know this already....
		b->mode = v15.mode_current & 0x7F;

		survive_reset_lighthouse_position(ctx, id);

		SURVIVE_INVOKE_HOOK(ootx_received, ctx, id);
	}
}
```

**提取的校準參數：**

對於 Base Station 2.0，每個基地台都有兩個軸（X 和 Y 轉子）的校準參數：

- **phase**：轉子的相位偏移
- **tilt**：轉子平面的傾斜角度
- **curve**：曲率校正參數
- **gibpha/gibmag**：凸月相位和幅度校正
- **ogeephase/ogeemag**：S 形相位和幅度校正（Gen 2 專用）
- **accel_dir**：重力方向向量（用於建立「向上」方向）

這些參數儲存在 `BaseStationCal` 結構中：

```257:267:include/libsurvive/survive.h
typedef struct BaseStationCal {
	FLT phase;
	FLT tilt;
	FLT curve;
	FLT gibpha;
	FLT gibmag;

	// Gen 2 specific cal params
	FLT ogeephase;
	FLT ogeemag;
} BaseStationCal;
```

**為什麼 OOTX 很重要：**

工廠校準參數對於準確的重投影至關重要。沒有這些參數，系統無法正確解釋來自雷射掃描的角度測量，導致嚴重的追蹤錯誤。

### 階段 2：光線數據收集

在收集 OOTX 數據的同時，追蹤器也收集光線激活數據。當雷射掃描擊中追蹤器上的感應器時，它會記錄：

- 哪個感應器被激活
- 哪個基地台（通過通道/頻率識別）
- 哪個軸（X 或 Y）
- 激活的時間

這些數據被處理以提取每個感應器-基地台-軸組合的角度測量。

### 階段 3：位置求解

一旦收集到足夠的光線數據和 OOTX 數據，poser（位置求解器）會嘗試確定基地台的相對位置。

**問題：**

給定：
- 追蹤器上已知的感應器位置（來自裝置配置）
- 從多個感應器到每個基地台的角度測量
- 基地台校準參數

求解：
- 每個基地台相對於追蹤器的位置和方向

**解決方法：**

libsurvive 使用幾種 poser 算法：

1. **Barycentric SVD Poser**：不需要初始估計的種子 poser，在啟動時使用
2. **EPnP Poser**：基於 Efficient Perspective-n-Point 算法
3. **MPFit Poser**：使用 Levenberg-Marquardt 優化進行非線性最小二乘

poser 求解每個基地台的相對位置。對於配備 2 個基地台和 1 個追蹤器的設定，poser 將：

1. 求解 Base Station 0 相對於追蹤器的位置
2. 求解 Base Station 1 相對於追蹤器的位置

**來自 EPnP Poser 的範例：**

```62:104:src/poser_epnp.c
static int opencv_solver_fullscene(SurviveObject *so, PoserDataFullScene *pdfs) {
	SurvivePose lh2object[NUM_GEN2_LIGHTHOUSES] = { 0 };
	for (int lh = 0; lh < so->ctx->activeLighthouses; lh++) {
		epnp pnp = {.fu = 1, .fv = 1};
		epnp_set_maximum_number_of_correspondences(&pnp, so->sensor_ct);

		for (size_t i = 0; i < so->sensor_ct; i++) {
			FLT *_ang = pdfs->angles[i][lh];
			if (isnan(_ang[0]) || isnan(_ang[1]))
				continue;

			FLT ang[2];
			survive_apply_bsd_calibration(so->ctx, lh, _ang, ang);

			epnp_add_correspondence(&pnp, so->sensor_locations[i * 3 + 0], so->sensor_locations[i * 3 + 1],
									so->sensor_locations[i * 3 + 2], get_u(ang), get_v(ang));
		}

		SurviveContext *ctx = so->ctx;
		SV_INFO("Solving for %d correspondents", pnp.number_of_correspondences);
		if (pnp.number_of_correspondences <= 4) {
			SV_INFO("Can't solve for only %d points on lh %d\n", pnp.number_of_correspondences, lh);
			continue;
		}

		lh2object[lh] = solve_correspondence(so, &pnp, true);

		epnp_dtor(&pnp);
	}

	PoserData_lighthouse_poses_func(&pdfs->hdr, so, lh2object, so->ctx->activeLighthouses, 0);

	return 0;
}
```

### 階段 4：世界座標系建立

階段 3 中求解的位置相對於追蹤器的座標系，這是任意的。為了建立一致的世界座標系，libsurvive 執行幾種轉換：

**步驟 1：重力對齊**

系統使用 IMU（加速度計）數據確定「向上」方向。這確保 Z 軸指向上方，無論追蹤器在校準期間如何定向。

**步驟 2：參考基地台**

第一個基地台（或由 `reference-basestation` 配置指定的）用作參考。通過繞 Z 軸旋轉將其放置在 X 軸上。

**步驟 3：座標系標準化**

世界座標系的建立方式：
- 原點：最初在追蹤器位置（或可選地在第一個基地台）
- Z 軸：指向上方（與重力對齊）
- X 軸：指向參考基地台（投影到 XY 平面）
- Y 軸：完成右手座標系

**實作：**

```142:201:src/poser.c
		// Assume that the space solved for is valid but completely arbitrary. We are going to do a few things:
		// a) Using the gyro data, normalize it so that gravity is pushing straight down along Z
		// c) Assume the object is at origin
		// b) Place the first lighthouse on the X axis by rotating around Z
		//
		// This calibration setup has the benefit that as long as the user is calibrating on the same flat surface,
		// calibration results will be roughly identical between all posers no matter the orientation the object is
		// lying
		// in.
		//
		// We might want to go a step further and affix the first lighthouse in a given pose that preserves up so that
		// it doesn't matter where on that surface the object is.
		bool worldEstablished = quatmagnitude(object_pose->Rot) != 0;
		SurvivePose object2arb = {.Rot = {1.}};
		if (object_pose && !quatiszero(object_pose->Rot))
			object2arb = *object_pose;
		SurvivePose lighthouse2arb = *lighthouse_pose;

		SurvivePose obj2world, lighthouse2world;
		// Purposefully only set this once. It should only depend on the first (calculated) lighthouse
		if (!worldEstablished) {
			bool centerOnLh0 =

			// Start by just moving from whatever arbitrary space into object space.
			SurvivePose arb2object;
			InvertPose(&arb2object, &object2arb);

			SurvivePose lighthouse2obj;
			ApplyPoseToPose(&lighthouse2obj, &arb2object, &lighthouse2arb);
			SurvivePose arb2world = arb2object;

			// Find the poses that map to the above

			// Find the space with the same origin, but rotated so that gravity is up
			SurvivePose lighthouse2objUp = {0}, object2objUp = {0};
			FLT accel_mag = norm3d(so->activations.accel);
			if (accel_mag != 0.0 && !isnan(accel_mag)) {
				quatfrom2vectors(object2objUp.Rot, so->activations.accel, up);
			} else {
				SV_WARN("Calibration didn't have valid IMU data for %s; couldn't establish 'up' vector.", so->codename);
				object2objUp.Rot[0] = 1.0;
			}

			// Calculate the pose of the lighthouse in this space
			ApplyPoseToPose(&lighthouse2objUp, &object2objUp, &lighthouse2obj);
			ApplyPoseToPose(&arb2world, &object2objUp, &arb2world);

			// Find what angle we need to rotate about Z by to get to 90 degrees.
			FLT ang = atan2(lighthouse2objUp.Pos[1], lighthouse2objUp.Pos[0]);
			FLT ang_target = M_PI / 2.;
			FLT euler[3] = {0, 0, ang_target - ang};
			SurvivePose objUp2World = {0};
			quatfromeuler(objUp2World.Rot, euler);

			ApplyPoseToPose(&arb2world, &objUp2World, &arb2world);
			ApplyPoseToPose(&obj2world, &arb2world, &object2arb);
			ApplyPoseToPose(&lighthouse2world, &arb2world, &lighthouse2arb);

			if (centerOnLh0) {
				sub3d(obj2world.Pos, obj2world.Pos, lighthouse2world.Pos)
```

**多個基地台：**

使用 2 個基地台進行校準時，系統：

1. 首先校準 Base Station 0（參考基地台）
2. 然後使用已建立的世界座標系校準 Base Station 1
3. 兩個基地台的位置都儲存在世界座標系中

參考基地台的選擇在這裡處理：

```337:359:src/poser.c
		uint32_t lh_indices[NUM_GEN2_LIGHTHOUSES] = {0};
		uint32_t cnt = 0;

		uint32_t reference_basestation = survive_configi(so->ctx, "reference-basestation", SC_GET, 0);

		for (int lh = 0; lh < lighthouse_count; lh++) {
			SurvivePose lh2object = lighthouse_pose[lh];
			if (quatmagnitude(lh2object.Rot) != 0.0) {
				lh_indices[cnt] = lh;
				uint32_t lh0 = lh_indices[0];
				bool preferThisBSD = reference_basestation == 0
										 ? (so->ctx->bsd[lh].BaseStationID < so->ctx->bsd[lh0].BaseStationID)
										 : reference_basestation == so->ctx->bsd[lh].BaseStationID;
				if (preferThisBSD) {
					lh_indices[0] = lh;
					lh_indices[cnt] = lh0;
				}
				cnt++;
			}
		}

		struct SurviveContext *ctx = so->ctx;
		SV_INFO("Using LH %d (%08x) as reference lighthouse", lh_indices[0], so->ctx->bsd[lh_indices[0]].BaseStationID);
```

### 階段 5：配置持久化

一旦校準完成，結果會保存到配置檔案（通常是 `XDG_CONFIG_HOME/libsurvive` 中的 `config.json`）。這包括：

- 基地台位置（位置：位置 + 旋轉四元數）
- 基地台校準參數（OOTX 數據）
- 基地台 ID 和模式
- 置信度值

**保存配置：**

```443:476:src/survive_config.c
void config_set_lighthouse(config_group *lh_config, BaseStationData *bsd,
```

配置會在後續運行中自動載入，如果基地台沒有移動，系統可以跳過校準。

**載入配置：**

```385:441:src/survive_config.c
bool config_read_lighthouse(config_group *lh_config, BaseStationData *bsd, uint8_t idx) {
	config_group *cg = lh_config + idx;
	uint8_t found = 0;
	for (int i = 0; i < NUM_GEN2_LIGHTHOUSES; i++) {
		uint32_t tmpIdx = 0xffffffff;
		cg = lh_config + idx;

		tmpIdx = config_read_uint32(cg, "index", 0xffffffff);

		if (tmpIdx == idx && i == idx) // assumes that lighthouses are stored in the config in order.
		{
			found = 1;
			break;
		}
	}

	//	assert(found); // throw an assertion if we didn't find it...  Is this good?  not necessarily?
	if (!found) {
		return false;
	}

	FLT defaults[7] = {0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0};

	bsd->BaseStationID = config_read_uint32(cg, "id", 0);
	bsd->mode = config_read_uint32(cg, "mode", 0);

	config_read_float_array(cg, "pose", &bsd->Pose.Pos[0], defaults, 7);
	config_read_float_array(cg, "variance", &bsd->variance.Pos[0], defaults, 6);

	FLT accel[3] = {0};
	config_read_float_array(cg, "accel", accel, defaults, 3);
	for (int i = 0; i < 3; i++)
		bsd->accel[i] = accel[i];

	FLT cal[sizeof(bsd->fcal)] = { 0 };
	config_read_float_array(cg, "fcalphase", cal, defaults, 2);
	config_read_float_array(cg, "fcaltilt", cal + 2, defaults, 2);
	config_read_float_array(cg, "fcalcurve", cal + 4, defaults, 2);
	config_read_float_array(cg, "fcalgibpha", cal + 6, defaults, 2);
	config_read_float_array(cg, "fcalgibmag", cal + 8, defaults, 2);
	config_read_float_array(cg, "fcalogeephase", cal + 10, defaults, 2);
	config_read_float_array(cg, "fcalogeemag", cal + 12, defaults, 2);

	for (size_t i = 0; i < 2; i++) {
		bsd->fcal[i].phase = cal[0 + i];
		bsd->fcal[i].tilt = cal[2 + i];
		bsd->fcal[i].curve = cal[4 + i];
		bsd->fcal[i].gibpha = cal[6 + i];
		bsd->fcal[i].gibmag = cal[8 + i];
		bsd->fcal[i].ogeephase = cal[10 + i];
		bsd->fcal[i].ogeemag = cal[12 + i];
	}

	bsd->OOTXSet = config_read_uint32(cg, "OOTXSet", 0);
	bsd->PositionSet = config_read_uint32(cg, "PositionSet", 0);
	return true;
}
```

## 重投影模型

校準參數用於重投影模型，將 3D 位置轉換為角度測量。對於 Base Station 2.0，這涉及複雜的非線性模型：

```62:103:src/survive_reproject_gen2.c
static inline FLT survive_reproject_axis_gen2(const BaseStationCal *bcal, FLT X, FLT Y, FLT Z, bool axis) {
	const FLT phase = bcal->phase;
	const FLT curve = bcal->curve;
	const FLT tilt = bcal->tilt;
	const FLT gibPhase = bcal->gibpha;
	const FLT gibMag = bcal->gibmag;
	const FLT ogeePhase = bcal->ogeephase;
	const FLT ogeeMag = bcal->ogeemag;

	FLT B = atan2(Z, X);

	FLT Ydeg = tilt + (axis ? -1 : 1) * LINMATHPI / 6.;
	FLT tanA = FLT_TAN(Ydeg);
	FLT normXZ = FLT_SQRT(X * X + Z * Z);

	FLT asinArg = tanA * Y / normXZ;
	FLT asinArg_sanitized = linmath_enforce_range(asinArg, -1, 1);

	FLT sinYdeg = FLT_SIN(Ydeg);
	FLT cosYdeg = FLT_COS(Ydeg);

	FLT sinPart = FLT_SIN(B - FLT_ASIN(asinArg_sanitized) + ogeePhase) * ogeeMag;

	FLT normXYZ = FLT_SQRT(X * X + Y * Y + Z * Z);

	FLT modAsinArg = linmath_enforce_range(Y / normXYZ / cosYdeg, -1, 1);

	FLT asinOut = FLT_ASIN(modAsinArg);

	FLT mod, acc;
	calc_cal_series(asinOut, &mod, &acc);

	FLT BcalCurved = sinPart + curve;
	FLT asinArg2 = linmath_enforce_range(asinArg + mod * BcalCurved / (cosYdeg - acc * BcalCurved * sinYdeg), -1, 1);

	FLT asinOut2 = FLT_ASIN(asinArg2);
	FLT sinOut2 = sin(B - asinOut2 + gibPhase);

	FLT rtn = B - asinOut2 + sinOut2 * gibMag - phase - LINMATHPI_2;
	assert(!isnan(rtn));
	return rtn;
}
```

此模型考慮了基地台雷射系統中的各種光學缺陷，包括：
- 相位偏移
- 傾斜校正
- 曲率校正
- 凸月相位/幅度校正
- S 形相位/幅度校正（Gen 2 專用）

## 校準要求

為了成功校準：

1. **OOTX 數據**：所有基地台必須廣播其 OOTX 數據。根據訊號品質，每個基地台通常需要 10-60 秒。對於有多個基地台的設定，這可能需要幾分鐘。

2. **光線覆蓋**： 
   - **對於 2 個基地台**：追蹤器必須同時或順序接收來自兩個基地台的光線。
   - **對於 3+ 個基地台**：追蹤器應放置在可以看到盡可能多基地台的位置。您可能需要將追蹤器移動到不同位置以看到所有基地台。
   - 追蹤器應放置在可以「看到」基地台且視線清晰的位置。

3. **靜止期間**：在校準期間，追蹤器**必須保持完全靜止**。系統使用來自多個感應器的靜止光線測量來求解基地台位置。**不需要也不建議移動** - 校準算法在靜止數據下效果最佳。對於多個基地台，如果並非所有基地台都從一個位置可見，您可能需要將追蹤器移動到新位置並在那裡保持靜止，直到剩餘基地台校準完成。

4. **感應器可見性**：追蹤器上的多個感應器必須對每個基地台可見。poser 需要每個基地台至少 4-5 個感應器對應關係才能可靠求解。使用多個追蹤器時，系統會為每個基地台使用最佳可用追蹤器。

5. **基地台限制**：不要超過 16 個基地台。系統只會識別並校準檢測到的前 16 個基地台。如果您需要超過 16 個基地台，您可能需要使用多個 libsurvive 實例或修改源代碼（在 `include/libsurvive/survive_types.h` 中更改 `NUM_GEN2_LIGHTHOUSES`）。

## 校準時序

校準過程通常按以下方式進行：

1. **0-10 秒**：開始收集 OOTX 數據。系統嘗試從所有基地台解碼 OOTX 封包。

2. **10-30 秒**：收集 OOTX 數據後，開始累積光線數據。系統從多個感應器收集角度測量。

3. **30-60 秒**：當有足夠數據時，poser 運行並求解基地台位置。隨著更多數據的可用，這可能會發生多次。

4. **60+ 秒**：校準穩定。系統繼續優化位置，但主要校準已完成。

## 故障排除

**基地台未校準：**

- 確保所有基地台都已開機並處於追蹤模式
- 檢查追蹤器是否可以看到所有基地台（無障礙物）
- 驗證是否正在接收 OOTX 數據（使用 `--v 100` 檢查日誌）
- 嘗試將追蹤器移動到具有更好可見性的不同位置

**校準時間過長：**

- 確保追蹤器和基地台之間有良好的視線
- 檢查反射或干擾
- 驗證基地台是否正確同步（Gen 2 應該是自動的）

**校準後追蹤不準確：**

- 使用 `--force-calibrate` 強制重新校準
- 驗證基地台自上次校準後未被移動
- 檢查 OOTX 數據是否成功收集（在配置檔案中）

## 總結

libsurvive 中的校準過程是一個複雜的多階段過程，它：

1. 從基地台收集工廠校準數據（OOTX）
2. 處理來自追蹤器的光線激活數據
3. 使用幾何算法求解相對位置
4. 建立一致的世界座標系
5. 持久化結果以供未來使用

對於配備 2 個 Base Station 2.0 和 1 個 Tracker 3.0 的設定，系統將相對於追蹤器校準兩個基地台，基於重力和參考基地台建立世界座標系，並保存此資訊以供未來的追蹤會話使用。

