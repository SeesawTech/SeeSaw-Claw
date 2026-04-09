# SeeSaw-Claw - Development Guide

Python 爬蟲/自動化服務，作為 OpenClaw Skill 運行。負責市場生成、持倉調整、獎勵領取、Oracle 斷言等自動化操作。

## 項目結構

```
SeeSaw-Claw/
├── skills/seesaw/
│   ├── SKILL.md           # OpenClaw Skill 定義（API 列表）
│   ├── config.json        # 風控配置（max_position_size, max_daily_loss）
│   └── scripts/
│       ├── seesaw.py          # 主 CLI 入口
│       ├── generate_topic.py  # 搜索新聞 → 生成市場提案 → 創建市場
│       ├── adjust_position.py # 監控持倉 → 調整交易 → 結算
│       ├── open_position.py   # 發現熱門市場 → 分析價值 → 開倉
│       ├── claim_rewards.py   # 自動領取挑戰獎勵
│       ├── auto_assert.py     # 市場管理：對已創建市場進行斷言/評論
│       └── utils.py           # 工具函數
└── README.md
```

## 開發環境

```bash
# 安裝依賴
pip install -r requirements.txt

# 配置環境變量
export SEESAW_BASE_URL=https://dev-api.seesaw.fun/v1  # 或 prod URL
export SEESAW_API_KEY=your_key
export SEESAW_API_SECRET=your_secret

# 可選（自動化腳本需要）
export OPENAI_API_KEY=xxx
export BRAVE_API_KEY=xxx
export SLACK_BOT_TOKEN=xxx
export SLACK_CHANNEL_ID=xxx

# 運行腳本
python skills/seesaw/scripts/seesaw.py
```

### 風控配置
`skills/seesaw/config.json`：
- `max_position_size`: 單次最大持倉（默認 100）
- `max_daily_loss`: 每日最大虧損（默認 500）
- `dry_run`: true 時只模擬不執行

## 測試

手動測試為主：
1. 設置 `dry_run: true` 後運行腳本，確認輸出符合預期
2. 用 Dev 環境測試用戶驗證 API 調用
3. 檢查 Slack 通知是否正常發送

## 部署

### Dev 環境
- **自動**：PR 合併到 main 後打包為 zip 上傳 S3
- **手動**：`gh workflow run deploy.yml --ref main`
- 部署產物：`s3://{bucket}/skills/seesaw-claw.zip`，設置 no-cache

### Prod 環境
目前無獨立 Prod 部署，通過 S3 上的 zip 包分發。

## PR 規範

### Reviewers
指定倉庫中所有 collaborator team，由 team 自動 assign 成員：
- `@SeesawTech/devs`

### 分支命名
- 功能：`feature/xxx`
- 修復：`bugfix/xxx`

### Commit 格式
```
feat(claw): add new market generation strategy
fix(claw): fix position size calculation
chore(claw): update API endpoints
```

## 編碼規範

- Python 3，依賴 requests 庫
- 所有 API 調用需處理錯誤和重試
- 風控參數必須從 config.json 讀取，不可硬編碼
- Slack 通知用於重要操作的審計追蹤
- 敏感信息（API Key/Secret）只通過環境變量傳入
