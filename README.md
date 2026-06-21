# 🎉 派對抽獎機

支援多種「情境」、跨裝置即時同步的隨機抽獎工具。資料儲存在 Firebase Firestore，純前端、無需後端伺服器，可直接部署到 GitHub Pages。

## 功能

- **抽獎**：先選擇情境（例如「尾牙大獎」「生日小遊戲」），再從該情境的選項中隨機抽出一個，可重複抽。
- **維護情境**：新增 / 刪除情境；情境名稱限制 10 字元內；選項以多行文字框維護，一行一個選項。
- **跨裝置同步**：所有資料存在 Firebase Firestore，任何裝置打開同一個網址，編輯後其他裝置即時看到最新資料。

## 部署到 GitHub Pages

1. 在 GitHub 建立一個新的 repository（公開或私人皆可，私人也能開 Pages）。
2. 把這個資料夾裡的 `index.html` 上傳到 repository 的根目錄（檔名務必是 `index.html`）。
   - 用網頁介面：repo 頁面點 **Add file → Upload files**，把 `index.html` 拖進去，Commit。
   - 或用 git 指令：
     ```bash
     git init
     git add index.html
     git commit -m "首次部署抽獎機"
     git branch -M main
     git remote add origin https://github.com/<你的帳號>/<repo名稱>.git
     git push -u origin main
     ```
3. 到 repo 的 **Settings → Pages**：
   - Source 選擇 **Deploy from a branch**
   - Branch 選 `main`，資料夾選 `/ (root)`，按 **Save**
4. 等 1～2 分鐘，頁面會顯示網址，格式大致是：
   `https://<你的帳號>.github.io/<repo名稱>/`
   打開即可使用，手機、電腦都能連同一個網址。

## 必做：設定 Firestore 安全性規則

Firebase 專案預設會擋掉所有讀寫，**沒設定規則的話頁面會一直顯示「連線異常」**。

這個專案 `sons-topic` 與其他 App 共用，抽獎機的資料全部鎖在 `lottery_data` 這個 collection 底下，不會碰到其他 App 的 collection。規則也只開放 `lottery_data`，不影響專案裡其他 App 的資料安全性。

1. 到 [Firebase Console](https://console.firebase.google.com/) → 選專案 `sons-topic`
2. 左側選單 **Firestore Database** → 如果還沒建立資料庫，先點「建立資料庫」（地區任選，例如 asia-east1）
3. 上方分頁切到 **規則 (Rules)**，加入以下內容並發布（如果已有其他 App 的規則，把這段加進同一個 `match /databases/{database}/documents { ... }` 區塊裡即可，不要整段覆蓋掉）：

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /lottery_data/{docId} {
         allow read, write: if true;
       }
       // 其他 App 的 match 規則維持原樣，放在同一層即可
     }
   }
   ```

> ⚠️ 這組規則代表「任何拿到網址的人都能讀寫 `lottery_data` 裡的資料」，適合內部小工具、朋友聚會、公司尾牙等輕量使用情境，且只限定在 `lottery_data`，不會開放其他 collection 的存取權限。如果之後想加上權限保護（例如只有特定人能編輯選項），可以再加 Firebase Authentication，需要時再告訴我，我可以幫你加上。

## 本機測試

直接用瀏覽器打開 `index.html` 即可（不需要安裝任何套件），但因為使用 ES Module 載入 Firebase SDK，部分瀏覽器在 `file://` 協定下可能會擋住模組載入，建議用簡單的本機伺服器測試，例如：

```bash
# 在這個資料夾裡執行
python3 -m http.server 8000
```

然後打開 `http://localhost:8000`。

## 檔案結構

```
.
├── index.html   # 整個應用程式（HTML + CSS + JS，單一檔案）
└── README.md    # 這份說明文件
```
