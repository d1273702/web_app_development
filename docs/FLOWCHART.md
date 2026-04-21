# 流程圖文件 — 食譜收藏系統

## 1. 使用者流程圖（User Flow）

描述使用者從開啟網頁到完成各項操作的路徑。

```mermaid
flowchart LR
    Start([使用者開啟網頁]) --> Home[首頁\n食譜列表]

    Home --> Action{要執行什麼操作？}

    Action -->|新增食譜| AddForm[填寫新增食譜表單\n名稱、食材、步驟、分類]
    AddForm --> Validate1{輸入是否合法？}
    Validate1 -->|否| AddForm
    Validate1 -->|是| SaveRecipe[儲存食譜至資料庫]
    SaveRecipe --> Home

    Action -->|查看食譜| Detail[食譜詳細頁\n顯示食材與步驟]
    Detail --> DetailAction{要做什麼？}
    DetailAction -->|編輯| EditForm[修改食譜表單]
    EditForm --> Validate2{輸入是否合法？}
    Validate2 -->|否| EditForm
    Validate2 -->|是| UpdateRecipe[更新資料庫]
    UpdateRecipe --> Detail
    DetailAction -->|刪除| Confirm{確認刪除？}
    Confirm -->|取消| Detail
    Confirm -->|確認| DeleteRecipe[從資料庫刪除]
    DeleteRecipe --> Home
    DetailAction -->|返回| Home

    Action -->|關鍵字搜尋| SearchForm[輸入搜尋關鍵字]
    SearchForm --> SearchResult[顯示搜尋結果列表]
    SearchResult -->|點擊食譜| Detail

    Action -->|食材推薦| IngredientForm[輸入手邊現有食材]
    IngredientForm --> RecommendResult[顯示符合食材的推薦食譜]
    RecommendResult -->|點擊食譜| Detail

    Action -->|依分類瀏覽| TagFilter[選擇分類 / 標籤]
    TagFilter --> FilterResult[顯示該分類的食譜列表]
    FilterResult -->|點擊食譜| Detail
```

---

## 2. 系統序列圖（Sequence Diagram）

### 2-A：新增食譜

```mermaid
sequenceDiagram
    actor User as 使用者
    participant Browser as 瀏覽器
    participant Flask as Flask Route
    participant Model as Recipe Model
    participant DB as SQLite

    User->>Browser: 點擊「新增食譜」
    Browser->>Flask: GET /recipes/new
    Flask->>Browser: 回傳新增食譜表單 (create.html)

    User->>Browser: 填寫名稱、食材、步驟並送出
    Browser->>Flask: POST /recipes
    Flask->>Flask: 驗證輸入資料
    Flask->>Model: recipe.create(name, ingredients, steps, tags)
    Model->>DB: INSERT INTO recipes, recipe_ingredients, recipe_tags
    DB-->>Model: 寫入成功，回傳新 recipe_id
    Model-->>Flask: 回傳 Recipe 物件
    Flask-->>Browser: 302 重導向至 /recipes/<id>
    Browser->>Flask: GET /recipes/<id>
    Flask-->>Browser: 回傳食譜詳細頁 (detail.html)
```

### 2-B：食材推薦食譜

```mermaid
sequenceDiagram
    actor User as 使用者
    participant Browser as 瀏覽器
    participant Flask as Flask Route
    participant Model as Recipe Model
    participant DB as SQLite

    User->>Browser: 輸入手邊食材並送出
    Browser->>Flask: POST /search/recommend
    Flask->>Model: recipe.find_by_ingredients(ingredient_list)
    Model->>DB: SELECT recipes WHERE ingredients MATCH
    DB-->>Model: 回傳符合的食譜列表
    Model-->>Flask: 回傳 Recipe 列表
    Flask-->>Browser: 回傳推薦結果頁 (recommend.html)
```

### 2-C：關鍵字搜尋

```mermaid
sequenceDiagram
    actor User as 使用者
    participant Browser as 瀏覽器
    participant Flask as Flask Route
    participant Model as Recipe Model
    participant DB as SQLite

    User->>Browser: 輸入關鍵字並按搜尋
    Browser->>Flask: GET /search?q=關鍵字
    Flask->>Model: recipe.search(keyword)
    Model->>DB: SELECT WHERE name LIKE '%keyword%'
    DB-->>Model: 回傳符合的食譜列表
    Model-->>Flask: 回傳 Recipe 列表
    Flask-->>Browser: 回傳搜尋結果頁 (results.html)
```

---

## 3. 功能清單對照表

| 功能 | URL 路徑 | HTTP 方法 | 說明 |
|------|----------|-----------|------|
| 首頁 / 食譜列表 | `/` | GET | 顯示所有食譜 |
| 食譜詳細頁 | `/recipes/<id>` | GET | 顯示單一食譜詳情 |
| 新增食譜表單 | `/recipes/new` | GET | 顯示新增表單 |
| 儲存新增食譜 | `/recipes` | POST | 接收表單、寫入資料庫 |
| 編輯食譜表單 | `/recipes/<id>/edit` | GET | 顯示編輯表單（帶入現有資料） |
| 更新食譜 | `/recipes/<id>` | POST | 接收修改後的資料、更新資料庫 |
| 刪除食譜 | `/recipes/<id>/delete` | POST | 從資料庫刪除指定食譜 |
| 關鍵字搜尋 | `/search` | GET | 接收 `q` 參數，搜尋食譜名稱與內容 |
| 食材推薦 | `/search/recommend` | GET / POST | 輸入食材清單，回傳推薦食譜 |
| 依標籤瀏覽 | `/tags/<tag_name>` | GET | 顯示特定分類的食譜列表 |
