<h1 align="center">
  Auto Maple
</h1>

Auto Maple 是一個智慧型 Python AI，能夠自動遊玩《楓之谷》（MapleStory）這款 2D 橫向捲軸 MMORPG 遊戲。程式透過模擬按鍵輸入、TensorFlow 機器學習、OpenCV 模板匹配以及其他電腦視覺技術來實現自動化操作。

社群所建立的資源，例如各職業的**指令書（Command Books）**以及各地圖的**路徑腳本（Routines）**，可在 **[資源庫](https://github.com/tanjeffreyz/auto-maple-resources)** 中找到。

<br>

## 專案架構

```
auto-maple/
├── main.py                 # 主程式入口，初始化並啟動所有模組
├── requirements.txt        # Python 依賴套件清單
├── setup.py               # 建立桌面捷徑的設定腳本
├── assets/                # 資源檔案（模型、模板圖片、警示音效）
├── resources/             # 子模組資源（指令書、路徑腳本）
└── src/
    ├── command_book/      # 指令書載入與管理
    ├── common/            # 共用工具（設定、介面、虛擬按鍵）
    ├── detection/         # TensorFlow 符文辨識模組
    ├── gui/               # 圖形介面（選單、檢視、編輯、設定）
    ├── modules/           # 核心模組（機器人、螢幕擷取、監聽器、通知器）
    └── routine/           # 路徑腳本編譯與執行
```

<br>

<h2 align="center">
  小地圖追蹤
</h2>

<table align="center" border="0">
  <tr>
    <td>
Auto Maple 使用 <b>OpenCV 模板匹配</b>技術來偵測小地圖的邊界以及其中的各種元素，從而精準追蹤玩家在遊戲中的位置。若將 <code>record_layout</code> 設定為 <code>True</code>，Auto Maple 會將玩家先前的位置記錄到一個基於<b>四叉樹（Quadtree）</b>的 Layout 物件中，並定期儲存到「layouts」目錄下的檔案中。每當載入新的路徑腳本時，若對應的 Layout 檔案存在，也會一併載入。這個 Layout 物件使用 <b>A* 搜尋演算法</b>來計算從玩家當前位置到任何目標位置的最短路徑，可以大幅提升路徑腳本執行的準確度與速度。
    </td>
    <td align="center" width="400px">
      <img align="center" src="https://user-images.githubusercontent.com/69165598/123177212-b16f0700-d439-11eb-8a21-8b414273f1e1.gif"/>
    </td>
  </tr>
</table>

<br>

<h2 align="center">
  指令書
</h2>

<p align="center">
  <img src="https://user-images.githubusercontent.com/69165598/123372905-502e5d00-d539-11eb-81c2-46b8bbf929cc.gif" width="100%"/>
  <br>
  <sub>
    上方影片展示 Auto Maple 持續執行高難度的技能組合。
  </sub>
</p>
  
<table align="center" border="0">
  <tr>
    <td width="100%">
Auto Maple 採用模組化設計，只要提供遊戲內動作的清單（即「指令書」），就能操控遊戲中的任何角色。指令書是一個 Python 檔案，包含多個類別，每個類別對應一個遊戲內的技能，告訴程式應該按什麼按鍵、何時按下。指令書載入後，其中的類別會自動編譯成一個字典，供 Auto Maple 在執行路徑腳本時解析指令。指令可以存取 Auto Maple 的所有全域變數，因此能根據玩家位置和遊戲狀態動態調整行為。

<br><br>
<b>指令書必須包含：</b>
<ul>
  <li><code>Key</code> 類別：定義所有技能的按鍵綁定</li>
  <li><code>Buff</code> 類別：定義角色的 Buff 技能</li>
  <li><code>Move</code> 和 <code>Adjust</code> 類別，或 <code>step</code> 函式：定義角色的移動方式</li>
</ul>
    </td>
  </tr>
</table>
  
<br>

<h2 align="center">
  路徑腳本
</h2>

<table align="center" border="0">
  <tr>
    <td width="350px">
      <p align="center">
        <img src="https://user-images.githubusercontent.com/69165598/150469699-d8a94ab4-7d70-49c3-8736-a9018996f39a.png"/>
        <br>
        <sub>
          點擊<a href="https://github.com/tanjeffreyz02/auto-maple/blob/f13d87c98e9344e0a4fa5c6f85ffb7e66860afc0/routines/dcup2.csv">這裡</a>查看完整的路徑腳本。
        </sub>
      </p>
    </td>
    <td>
路徑腳本是一個使用者建立的 CSV 檔案，告訴 Auto Maple 要移動到哪裡以及在每個位置使用什麼指令。Auto Maple 內建的編譯器會解析所選的路徑腳本，並將其轉換為一系列 <code>Component</code> 物件供程式執行。對於包含無效參數的每一行，都會印出錯誤訊息，並在轉換過程中忽略這些行。
<br><br>
以下是最常用的路徑腳本元件摘要：
<ul>
  <li>
    <b><code>Point</code></b>（位置點）：儲存其下方的指令，當角色進入 <code>move_tolerance</code> 範圍內時，會依序執行這些指令。也有幾個可選的關鍵字參數：
    <ul>
      <li>
        <code>adjust</code>：在執行任何指令之前，微調角色位置使其處於目標位置的 <code>adjust_tolerance</code> 範圍內。
      </li>
      <li>
        <code>frequency</code>：設定此 Point 多久執行一次。若設為 N，則每 N 次迭代執行一次。
      </li>
      <li>
        <code>skip</code>：設定此 Point 是否在第一次迭代時執行。若設為 True 且 frequency 為 N，則此 Point 會在第 N-1 次迭代時執行。
      </li>
    </ul>
  </li>
  <li>
    <b><code>Label</code></b>（標籤）：作為參考點，幫助將路徑腳本組織成多個區段，也可用於建立迴圈。
  </li>
  <li>
    <b><code>Jump</code></b>（跳轉）：從路徑腳本的任何位置跳轉到指定的標籤。
  </li>
  <li>
    <b><code>Setting</code></b>（設定）：將指定的設定更新為給定的值。可放置在路徑腳本的任何位置，因此同一個路徑腳本的不同部分可以有不同的設定。所有可編輯的設定可在 <a href="https://github.com/tanjeffreyz02/auto-maple/blob/v2/settings.py">settings.py</a> 底部找到。
  </li>
</ul>
    </td>
  </tr>
</table>

<br>

<h2 align="center">
  符文解謎
</h2>

<p align="center">
  <img src="https://user-images.githubusercontent.com/69165598/123479558-f61fad00-d5b5-11eb-914c-8f002a96dd62.gif" width="100%"/>
</p>

<table align="center" border="0">
  <tr>
    <td width="100%">
Auto Maple 具備自動解開「符文」（遊戲內的方向鍵謎題）的能力。它首先使用 OpenCV 的顏色過濾和 <b>Canny 邊緣偵測</b>演算法來分離方向鍵並盡可能減少背景雜訊。然後，使用自訂訓練的 <b>TensorFlow</b> 模型對預處理後的影像進行多次推論，直到兩次推論結果一致。由於這種預處理方式，Auto Maple 在各種（通常色彩繽紛且混亂的）環境中都能極為準確地解開符文。
    </td>
  </tr>
</table>

<br>

<h2 align="center">
  核心模組
</h2>

<table align="center" border="0">
  <tr>
    <td width="100%">
Auto Maple 由以下核心模組組成：

<ul>
  <li><b>Bot</b>：主要的機器人模組，負責解讀並執行使用者定義的路徑腳本、處理符文解謎、管理 Buff 和寵物餵食。</li>
  <li><b>Capture</b>：螢幕擷取模組，持續追蹤玩家位置和遊戲內事件，使用 MSS 進行高效能螢幕截圖。</li>
  <li><b>Listener</b>：鍵盤監聽模組，處理熱鍵和使用者輸入。</li>
  <li><b>Notifier</b>：通知模組，可在特定事件發生時發送警示。</li>
  <li><b>GUI</b>：圖形使用者介面，提供路徑腳本編輯、小地圖視覺化、設定管理等功能。</li>
</ul>
    </td>
  </tr>
</table>

<br>

<h2 align="center">
  設定參數
</h2>

<table align="center" border="0">
  <tr>
    <td width="100%">
以下是可在路徑腳本中動態調整的設定參數：

<ul>
  <li><code>move_tolerance</code>：移動到 Point 時允許的誤差範圍（預設：0.1）</li>
  <li><code>adjust_tolerance</code>：微調位置時允許的誤差範圍（預設：0.01）</li>
  <li><code>record_layout</code>：是否將新的玩家位置儲存到目前的 Layout（預設：False）</li>
  <li><code>buff_cooldown</code>：每次呼叫 buff 指令之間的等待時間（秒）（預設：180）</li>
</ul>
    </td>
  </tr>
</table>

<br>

<h2 align="center">
  示範影片
</h2>

<p align="center">
  <a href="https://youtu.be/iNj1CWW2--8?si=MA4n6EAHokI9FX8B"><b>點擊下方觀看完整影片</b></a>
</p>

<p align="center">
  <a href="https://youtu.be/iNj1CWW2--8?si=MA4n6EAHokI9FX8B">
    <img src="https://user-images.githubusercontent.com/69165598/123308656-c5b61100-d4d8-11eb-99ac-c465665474b5.gif" width="600px"/>
  </a>
</p>

<br>

<h2 align="center">
  安裝設定
</h2>

<ol>
  <li>
    下載並安裝 <a href="https://www.python.org/downloads/">Python3</a>。
  </li>
  <li>
    下載並安裝最新版本的 <a href="https://developer.nvidia.com/cuda-downloads">CUDA Toolkit</a>。
  </li>
  <li>
    下載並安裝 <a href="https://git-scm.com/download/win">Git</a>。
  </li>
  <li>
    下載並解壓縮最新的 <a href="https://github.com/tanjeffreyz02/auto-maple/releases">Auto Maple 發行版</a>。
  </li>
  <li>
    下載 <a href="https://drive.google.com/drive/folders/1SPdTNF4KZczoWyWTgfyTBRvLvy7WSGpu?usp=sharing">TensorFlow 模型</a>，並將「models」資料夾解壓縮到 Auto Maple 的「assets」目錄下。
  </li>
  <li>
    在 Auto Maple 的主目錄中，開啟命令提示字元並執行：
    <pre><code>python -m pip install -r requirements.txt</code></pre>
  </li>
  <li>
    最後，執行以下指令建立桌面捷徑：
    <pre><code>python setup.py</code></pre>
    此捷徑使用絕對路徑，因此可以隨意移動捷徑位置。但如果移動了 Auto Maple 的主目錄，則需要重新執行 <code>python setup.py</code> 來產生新的捷徑。若要在 Auto Maple 關閉後保持命令提示字元開啟，請在上述指令加上 <code>--stay</code> 參數。
  </li>
</ol>

<br>

<h2 align="center">
  依賴套件
</h2>

<table align="center" border="0">
  <tr>
    <td width="100%">
<ul>
  <li><b>GitPython</b>：用於子模組更新管理</li>
  <li><b>keyboard</b>：用於鍵盤監聽與熱鍵處理</li>
  <li><b>mss</b>：用於高效能螢幕擷取</li>
  <li><b>numpy</b>：用於數值運算</li>
  <li><b>opencv-python-headless</b>：用於電腦視覺處理</li>
  <li><b>Pillow</b>：用於影像處理</li>
  <li><b>pygame</b>：用於 GUI 介面</li>
  <li><b>pywin32</b>：用於 Windows API 呼叫</li>
  <li><b>tensorflow</b>：用於符文辨識機器學習模型</li>
</ul>
    </td>
  </tr>
</table>
