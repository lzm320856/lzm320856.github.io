<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>会员整体结构分析</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', sans-serif;
            background: #f5f7fa;
            color: #333;
            padding: 16px;
        }

        .header {
            background: #fff;
            padding: 16px 20px;
            border-radius: 8px;
            margin-bottom: 16px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
        }

        .header-top {
            display: flex;
            align-items: center;
            justify-content: space-between;
            flex-wrap: wrap;
            gap: 12px;
        }

        .header-title {
            font-size: 20px;
            font-weight: 600;
            color: #1a1a1a;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .header-title .logo {
            width: 28px;
            height: 28px;
            background: linear-gradient(135deg, #1890ff 0%, #096dd9 100%);
            border-radius: 4px;
        }

        .header-date { color: #8c8c8c; font-size: 13px; }

        .header-nav-row {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-top: 14px;
            border-bottom: 2px solid #e8e8e8;
        }

        .nav-tabs {
            display: flex;
            gap: 0;
            border-bottom: none;
        }

        .nav-tab {
            padding: 8px 18px;
            cursor: pointer;
            color: #595959;
            border-bottom: 2px solid transparent;
            margin-bottom: -2px;
            transition: all 0.2s;
            font-size: 14px;
        }

        .nav-tab.active { color: #1890ff; border-bottom-color: #1890ff; font-weight: 500; }

        .filter-row {
            display: flex;
            align-items: center;
            gap: 10px;
            margin-top: 14px;
            flex-wrap: wrap;
        }

        .date-filters { display: flex; gap: 8px; flex-wrap: wrap; }

        .date-filter {
            padding: 5px 14px;
            border: 1px solid #d9d9d9;
            border-radius: 4px;
            background: #fff;
            cursor: pointer;
            font-size: 13px;
            transition: all 0.2s;
        }

        .date-filter:hover { border-color: #1890ff; color: #1890ff; }
        .date-filter.active { background: #1890ff; color: #fff; border-color: #1890ff; }

        .custom-date-wrap {
            position: relative;
            display: inline-block;
        }

        .custom-date-panel {
            display: none;
            position: absolute;
            top: 100%;
            left: 0;
            margin-top: 6px;
            background: #fff;
            border: 1px solid #d9d9d9;
            border-radius: 6px;
            box-shadow: 0 6px 16px rgba(0,0,0,0.12);
            padding: 12px 16px;
            z-index: 100;
            min-width: 280px;
        }

        .custom-date-panel.show { display: block; }

        .custom-date-panel label {
            display: block;
            font-size: 12px;
            color: #595959;
            margin-bottom: 4px;
        }

        .custom-date-panel input[type="date"] {
            width: 100%;
            padding: 6px 10px;
            border: 1px solid #d9d9d9;
            border-radius: 4px;
            font-size: 13px;
            margin-bottom: 10px;
        }

        .custom-date-panel .btn-apply {
            width: 100%;
            padding: 6px;
            background: #1890ff;
            color: #fff;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 13px;
        }

        .org-select {
            padding: 5px 12px;
            border: 1px solid #d9d9d9;
            border-radius: 4px;
            font-size: 13px;
            color: #595959;
        }

        /* 原图比例：6个KPI等宽一排 */
        .kpi-grid {
            display: grid;
            grid-template-columns: repeat(6, 1fr);
            gap: 16px;
            margin-bottom: 16px;
        }

        .kpi-card {
            background: #fff;
            padding: 16px;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
        }

        .kpi-card .kpi-head {
            display: flex;
            align-items: center;
            gap: 8px;
            margin-bottom: 10px;
        }

        .kpi-card .kpi-icon {
            width: 36px;
            height: 36px;
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .kpi-card .kpi-icon.purple { background: #f0e6ff; color: #722ed1; }
        .kpi-card .kpi-icon.green { background: #e6f7e6; color: #52c41a; }
        .kpi-card .kpi-icon.orange { background: #fff7e6; color: #fa8c16; }
        .kpi-card .kpi-icon.yellow { background: #fffbe6; color: #faad14; }
        .kpi-card .kpi-icon.red { background: #fff1f0; color: #ff4d4f; }
        .kpi-card .kpi-icon.blue { background: #e6f4ff; color: #1890ff; }

        .kpi-label { font-size: 13px; color: #8c8c8c; }

        .kpi-value { font-size: 24px; font-weight: 600; color: #1a1a1a; margin-bottom: 6px; }

        .kpi-change { font-size: 12px; display: flex; gap: 12px; flex-wrap: wrap; }
        .kpi-change-item { display: flex; align-items: center; gap: 2px; }
        .kpi-change .up { color: #52c41a; }
        .kpi-change .down { color: #ff4d4f; }

        /* 三列布局（图一红字）：第一列 | 第二列 | 第三列，底部对齐 */
        .dashboard-cols {
            display: grid;
            grid-template-columns: 1fr 1fr 1fr;
            gap: 16px;
            align-items: stretch;
        }

        .dashboard-col {
            display: flex;
            flex-direction: column;
            gap: 16px;
            min-height: 0;
        }

        .dashboard-col > *:last-child {
            flex: 1;
            min-height: 0;
            display: flex;
            flex-direction: column;
        }

        .dashboard-col > *:last-child .chart-part,
        .dashboard-col > *:last-child .block-body {
            flex: 1;
            min-height: 0;
        }

        .dashboard-col > *:last-child .chart-part .chart-container {
            flex: 1;
            min-height: 180px;
        }
        /* 第一列有效会员分析、第二列留存会员分析：图表区域等高 */
        .dashboard-col-1 .analysis-block .chart-part .chart-container,
        .dashboard-col-2 .analysis-block .chart-part .chart-container {
            flex: none;
            height: 260px;
            min-height: 260px;
        }
        .dashboard-col-3 .cup-analysis-block .chart-part .chart-container {
            flex: 1;
            min-height: 180px;
        }

        .dashboard-col-2 .col2-row1 {
            display: flex;
            flex-direction: column;
            gap: 16px;
            flex-shrink: 0;
            /* 与第一列趋势图卡片等高：趋势图 380px + 标题/标签/内边距 ≈ 452px */
            height: 452px;
        }
        .dashboard-col-2 .col2-row1 .chart-card {
            flex: 1;
            display: flex;
            flex-direction: column;
            min-height: 0;
        }
        .dashboard-col-2 .col2-row1 .chart-card .chart-container {
            flex: 1;
            min-height: 120px;
        }

        .chart-card {
            background: #fff;
            padding: 16px;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
        }

        .chart-title { font-size: 15px; font-weight: 600; margin-bottom: 12px; color: #1a1a1a; }

        .chart-tabs { display: flex; gap: 8px; margin-bottom: 12px; }
        .chart-tab {
            padding: 4px 12px;
            border: 1px solid #d9d9d9;
            border-radius: 4px;
            background: #fff;
            cursor: pointer;
            font-size: 13px;
            transition: all 0.2s;
        }
        .chart-tab.active { background: #1890ff; color: #fff; border-color: #1890ff; }

        .chart-container { height: 280px; position: relative; }
        .chart-container-lg { height: 380px; position: relative; }
        /* 第一列趋势图卡片与第二列第一行等高，保证第一行底部对齐 */
        .dashboard-col-1 > .chart-card:first-child {
            height: 452px;
            display: flex;
            flex-direction: column;
        }
        .dashboard-col-1 > .chart-card:first-child .chart-container-lg {
            flex: 1;
            min-height: 0;
        }

        /* 日均指标分析：一个卡片包住两个平行指标（图3） */
        .daily-panel {
            background: #fff;
            padding: 16px;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
        }

        .daily-panel .chart-title {
            margin-bottom: 12px;
        }

        .daily-cards {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;
        }

        .daily-card {
            background: #fafafa;
            padding: 14px 16px;
            border-radius: 8px;
            border: 1px solid #f0f0f0;
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .daily-card .icon-wrap {
            width: 40px;
            height: 40px;
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .daily-card .icon-wrap.purple { background: #f0e6ff; color: #722ed1; }
        .daily-card .icon-wrap.blue { background: #e6f4ff; color: #1890ff; }

        .daily-card .label { font-size: 13px; color: #8c8c8c; }
        .daily-card .value { font-size: 20px; font-weight: 600; color: #1a1a1a; }

        /* 第二行：会员结构 + 杯量杯单价 */
        .chart-row-2 {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 16px;
            margin-bottom: 16px;
        }

        .metric-grid {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 16px;
            margin-bottom: 16px;
        }

        .metric-card {
            background: #fff;
            padding: 16px;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
        }

        .metric-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px 0;
            border-bottom: 1px solid #f0f0f0;
        }
        .metric-item:last-child { border-bottom: none; }
        .metric-label { color: #8c8c8c; font-size: 13px; }
        .metric-value { font-size: 16px; font-weight: 600; color: #1a1a1a; }

        .metric-card .card-head {
            display: flex;
            align-items: center;
            gap: 8px;
            margin-bottom: 8px;
        }

        .metric-card .card-icon {
            width: 32px;
            height: 32px;
            border-radius: 6px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        /* 单栏分析块：左侧摘要卡 + 右侧图表（图3样式） */
        .analysis-block {
            background: #fff;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
            overflow: hidden;
        }
        .dashboard-col .analysis-block { margin-bottom: 0; }

        .analysis-block .block-title {
            font-size: 15px;
            font-weight: 600;
            color: #1a1a1a;
            padding: 16px 16px 0;
            margin-bottom: 12px;
        }

        .analysis-block .block-body {
            display: flex;
            align-items: stretch;
            gap: 0;
            padding: 0 16px 16px;
        }

        .analysis-block .summary-card {
            width: 220px;
            min-width: 220px;
            border-radius: 8px;
            padding: 16px;
            display: flex;
            flex-direction: column;
            align-items: center;
            text-align: center;
        }

        .analysis-block .summary-card.purple { background: #f9f0ff; }
        .analysis-block .summary-card.blue { background: #e6f4ff; }

        .analysis-block .summary-card .summary-icon {
            width: 48px;
            height: 48px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            margin-bottom: 10px;
        }

        .analysis-block .summary-card.purple .summary-icon { background: #efdbff; color: #722ed1; }
        .analysis-block .summary-card.blue .summary-icon { background: #bae0ff; color: #1890ff; }

        .analysis-block .summary-card .summary-label {
            font-size: 13px;
            color: #595959;
            margin-bottom: 6px;
        }

        .analysis-block .summary-card .summary-value {
            font-size: 24px;
            font-weight: 600;
            color: #1a1a1a;
            margin-bottom: 8px;
        }

        .analysis-block .summary-card .summary-extra {
            font-size: 12px;
            color: #8c8c8c;
            margin-bottom: 4px;
        }

        .analysis-block .summary-card .summary-trend {
            font-size: 12px;
            color: #52c41a;
            margin-top: 6px;
        }

        .analysis-block .chart-part {
            flex: 1;
            min-width: 0;
            padding-left: 16px;
        }

        .analysis-block .chart-part .chart-container {
            height: 260px;
        }

        /* 杯量与杯单价分析（图二）：标题 + 2x2 灰色卡片 + 图表 */
        .cup-analysis-block .block-body {
            flex-direction: column;
        }

        .cup-analysis-block .cup-metrics {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;
            margin-bottom: 16px;
        }

        .cup-analysis-block .cup-metrics .cup-card {
            background: #fafafa;
            border-radius: 8px;
            padding: 14px 16px;
            border: 1px solid #f0f0f0;
        }

        .cup-analysis-block .cup-metrics .cup-card .metric-label {
            font-size: 13px;
            color: #8c8c8c;
            margin-bottom: 6px;
        }

        .cup-analysis-block .cup-metrics .cup-card .metric-value {
            font-size: 22px;
            font-weight: 600;
            color: #1a1a1a;
        }

        .cup-analysis-block .chart-part {
            padding-left: 0;
        }

        .cup-analysis-block .chart-part .chart-container {
            height: 280px;
        }

        .chart-card.cup-analysis-block .chart-title {
            margin-bottom: 12px;
        }

        @media (max-width: 900px) {
            .analysis-block .block-body {
                flex-direction: column;
            }
            .analysis-block .summary-card {
                width: 100%;
                min-width: 0;
            }
        }

        @media (max-width: 1200px) {
            .kpi-grid { grid-template-columns: repeat(3, 1fr); }
            .dashboard-cols { grid-template-columns: 1fr 1fr; }
            .dashboard-col-3 { grid-column: 1 / -1; }
        }

        @media (max-width: 768px) {
            .kpi-grid { grid-template-columns: 1fr; }
            .dashboard-cols { grid-template-columns: 1fr; }
            .chart-row-2 { grid-template-columns: 1fr; }
            .metric-grid { grid-template-columns: 1fr; }
            .daily-cards { grid-template-columns: 1fr; }
        }

        @media (max-width: 480px) {
            .cup-analysis-block .cup-metrics { grid-template-columns: 1fr; }
        }
    </style>
</head>
<body>
    <div class="header">
        <div class="header-top">
            <div class="header-title">
                <div class="logo"></div>
                会员整体结构分析
            </div>
            <select class="org-select">
                <option value="">组织架构</option>
                <option value="store">店长</option>
                <option value="region">区域总监</option>
                <option value="admin">管理员</option>
            </select>
        </div>
        <div class="header-nav-row">
            <div class="nav-tabs">
                <div class="nav-tab active" data-dim="overview">整体概览</div>
                <div class="nav-tab" data-dim="new">新增会员</div>
                <div class="nav-tab" data-dim="repeat">复购会员</div>
            </div>
            <div class="header-date">更新日期: <span id="updateDate">2025/7/1</span></div>
        </div>
        <div class="filter-row">
            <div class="date-filters">
                <div class="date-filter" data-range="yesterday">昨日</div>
                <div class="date-filter" data-range="today">今日</div>
                <div class="date-filter" data-range="lastYear">去年</div>
                <div class="date-filter active" data-range="monthToDate">月至今</div>
                <div class="date-filter" data-range="yearToDate">年至今</div>
                <div class="custom-date-wrap">
                    <div class="date-filter" id="customDateBtn">自定义</div>
                    <div class="custom-date-panel" id="customDatePanel">
                        <label>开始日期</label>
                        <input type="date" id="customStart" value="2025-06-01">
                        <label>结束日期</label>
                        <input type="date" id="customEnd" value="2025-06-30">
                        <button type="button" class="btn-apply" id="customApply">确定</button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div class="kpi-grid">
        <div class="kpi-card">
            <div class="kpi-head">
                <div class="kpi-icon purple">
                    <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor"><path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/></svg>
                </div>
                <span class="kpi-label">会员累计总数</span>
            </div>
            <div class="kpi-value" id="kpiTotal">2,520</div>
            <div class="kpi-change"><span class="kpi-change-item up" id="kpiTotalYoy">同比 5.0%↑</span><span class="kpi-change-item up" id="kpiTotalMom">环比 2.9%↑</span></div>
        </div>
        <div class="kpi-card">
            <div class="kpi-head">
                <div class="kpi-icon green">
                    <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor"><path d="M7 18c-1.1 0-1.99.9-1.99 2S5.9 22 7 22s2-.9 2-2-.9-2-2-2zM1 2v2h2l3.6 7.59-1.35 2.45c-.16.28-.25.61-.25.96 0 1.1.9 2 2 2h12v-2H7.42c-.14 0-.25-.11-.25-.25l.03-.12.9-1.63h7.45c.75 0 1.41-.41 1.75-1.03l3.58-6.49c.08-.14.12-.31.12-.48 0-.55-.45-1-1-1H5.21l-.94-2H1zm16 16c-1.1 0-1.99.9-1.99 2s.89 2 1.99 2 2-.9 2-2-.9-2-2-2z"/></svg>
                </div>
                <span class="kpi-label">会员累计消费额</span>
            </div>
            <div class="kpi-value" id="kpiConsume">2.58万</div>
            <div class="kpi-change"><span class="kpi-change-item up" id="kpiConsumeYoy">同比 1.6%↑</span><span class="kpi-change-item down" id="kpiConsumeMom">环比 0.9%↓</span></div>
        </div>
        <div class="kpi-card">
            <div class="kpi-head">
                <div class="kpi-icon orange">
                    <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8zm.31-8.86c-1.77-.45-2.34-.94-2.34-1.67 0-.84.79-1.43 2.1-1.43 1.38 0 1.9.66 1.94 1.64h1.71c-.05-1.34-.87-2.57-2.49-2.97V5H10.9v1.69c-1.51.32-2.72 1.3-2.72 2.81 0 1.79 1.49 2.69 3.66 3.21 1.95.46 2.34 1.15 2.34 1.87 0 .53-.39 1.39-2.1 1.39-1.6 0-2.23-.72-2.32-1.64H8.04c.1 1.7 1.36 2.66 2.86 2.97V19h2.34v-1.67c1.52-.29 2.72-1.04 2.73-2.77-.01-2.2-1.9-2.96-3.66-3.42z"/></svg>
                </div>
                <span class="kpi-label">会员毛利额</span>
            </div>
            <div class="kpi-value" id="kpiProfit">0.90万</div>
            <div class="kpi-change"><span class="kpi-change-item up" id="kpiProfitYoy">同比 1.5%↑</span><span class="kpi-change-item down" id="kpiProfitMom">环比 0.7%↓</span></div>
        </div>
        <div class="kpi-card">
            <div class="kpi-head">
                <div class="kpi-icon yellow">
                    <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor"><path d="M11 7h2v2h-2zm0 4h2v6h-2zm1-9C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z"/></svg>
                </div>
                <span class="kpi-label">会员毛利率</span>
            </div>
            <div class="kpi-value" id="kpiRate">35.01%</div>
            <div class="kpi-change"><span class="kpi-change-item down" id="kpiRateYoy">同比 0.1%↓</span><span class="kpi-change-item up" id="kpiRateMom">环比 0.2%↑</span></div>
        </div>
        <div class="kpi-card">
            <div class="kpi-head">
                <div class="kpi-icon red">
                    <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor"><path d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z"/></svg>
                </div>
                <span class="kpi-label">会员复购率</span>
            </div>
            <div class="kpi-value" id="kpiRepurchase">95.24%</div>
            <div class="kpi-change"><span class="kpi-change-item down" id="kpiRepurchaseYoy">同比 4.8%↓</span><span class="kpi-change-item down" id="kpiRepurchaseMom">环比 2.8%↓</span></div>
        </div>
        <div class="kpi-card">
            <div class="kpi-head">
                <div class="kpi-icon red">
                    <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor"><path d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z"/></svg>
                </div>
                <span class="kpi-label">会员留存率</span>
            </div>
            <div class="kpi-value" id="kpiRetention">95.24%</div>
            <div class="kpi-change"><span class="kpi-change-item down" id="kpiRetentionYoy">同比 4.8%↓</span><span class="kpi-change-item down" id="kpiRetentionMom">环比 2.8%↓</span></div>
        </div>
    </div>

    <div class="dashboard-cols">
        <div class="dashboard-col dashboard-col-1">
            <div class="chart-card">
                <div class="chart-title">会员指标趋势分析</div>
                <div class="chart-tabs">
                    <div class="chart-tab active">消费金额</div>
                    <div class="chart-tab">毛利额</div>
                    <div class="chart-tab">订单数</div>
                </div>
                <div class="chart-container-lg"><canvas id="trendChart"></canvas></div>
            </div>
            <div class="analysis-block">
                <div class="block-title">有效会员分析</div>
                <div class="block-body">
                    <div class="summary-card purple">
                        <div class="summary-icon">
                            <svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor"><path d="M18 6h-2c0-2.21-1.79-4-4-4S8 3.79 8 6H6c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h12c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2z"/></svg>
                        </div>
                        <div class="summary-label">有效会员数</div>
                        <div class="summary-value" id="activeCount">2,400</div>
                        <div class="summary-extra">占比 <span id="activeRatio">95.24%</span></div>
                        <div class="summary-trend">同比 <span id="activeYoy">0.0%</span> ↑</div>
                        <div class="summary-trend">环比 <span id="activeMom">0.0%</span> ↑</div>
                    </div>
                    <div class="chart-part">
                        <div class="chart-container"><canvas id="activeChart"></canvas></div>
                    </div>
                </div>
            </div>
        </div>
        <div class="dashboard-col dashboard-col-2">
            <div class="col2-row1">
                <div class="chart-card">
                    <div class="chart-title">会员生命周期分布</div>
                    <div class="chart-container col2-chart"><canvas id="lifecycleChart"></canvas></div>
                </div>
                <div class="chart-card">
                    <div class="chart-title">会员结构分布</div>
                    <div class="chart-tabs">
                        <div class="chart-tab active">年龄</div>
                        <div class="chart-tab">性别</div>
                        <div class="chart-tab">等级</div>
                    </div>
                    <div class="chart-container col2-chart"><canvas id="structureChart"></canvas></div>
                </div>
            </div>
            <div class="analysis-block">
                <div class="block-title">留存会员分析</div>
                <div class="block-body">
                    <div class="summary-card blue">
                        <div class="summary-icon">
                            <svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/></svg>
                        </div>
                        <div class="summary-label">留存会员数</div>
                        <div class="summary-value" id="retainCount">2,400</div>
                        <div class="summary-trend">同比 <span id="retainYoy">0.0%</span> ↑</div>
                        <div class="summary-trend">环比 <span id="retainMom">0.0%</span> ↑</div>
                    </div>
                    <div class="chart-part">
                        <div class="chart-container"><canvas id="retentionChart"></canvas></div>
                    </div>
                </div>
            </div>
        </div>
        <div class="dashboard-col dashboard-col-3">
            <div class="daily-panel">
                <div class="chart-title">日均指标分析</div>
                <div class="daily-cards">
                    <div class="daily-card">
                        <div class="icon-wrap purple">
                    <svg width="22" height="22" viewBox="0 0 24 24" fill="currentColor"><path d="M18 6h-2c0-2.21-1.79-4-4-4S8 3.79 8 6H6c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h12c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2zm-6-2c1.1 0 2 .9 2 2h-4c0-1.1.9-2 2-2zm6 16H6V8h2v2c0 .55.45 1 1 1s1-.45 1-1V8h4v2c0 .55.45 1 1 1s1-.45 1-1V8h2v12z"/></svg>
                </div>
                        <div><div class="label">日均会员订单量</div><div class="value" id="dailyOrders">2,400</div></div>
                    </div>
                    <div class="daily-card">
                        <div class="icon-wrap blue">
                            <svg width="22" height="22" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/></svg>
                        </div>
                        <div><div class="label">日均会员消费金额</div><div class="value" id="dailyAmount">1.20万</div></div>
                    </div>
                </div>
            </div>
            <div class="chart-card cup-analysis-block">
                <div class="chart-title">杯量与杯单价分析</div>
                <div class="cup-metrics">
                    <div class="cup-card"><div class="metric-label">会员出杯量</div><div class="metric-value" id="cupVolume">3,940</div></div>
                    <div class="cup-card"><div class="metric-label">单会员出杯</div><div class="metric-value" id="cupPerMember">1.6</div></div>
                    <div class="cup-card"><div class="metric-label">会员杯单价</div><div class="metric-value" id="cupPrice">6.5</div></div>
                    <div class="cup-card"><div class="metric-label">会员杯毛利</div><div class="metric-value" id="cupProfit">2.3</div></div>
                </div>
                <div class="chart-part">
                    <div class="chart-container"><canvas id="cupChart"></canvas></div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 单会员消费金额：10±2 元随机
        function randomSingleMemberConsume() {
            return Math.round((10 + (Math.random() * 4 - 2)) * 100) / 100;
        }

        function genSingleMemberSeries(days) {
            const arr = [];
            for (let i = 0; i < days; i++) arr.push(randomSingleMemberConsume());
            return arr;
        }

        // Mock 数据：时间 × 维度
        // 时间：昨日 < 今日 < 月至今 < 年至今；去年 = 去年全年
        // 维度：整体概览 >= 新增会员、复购会员（子集）
        const TIME_KEYS = ['yesterday', 'today', 'lastYear', 'monthToDate', 'yearToDate'];
        const DIM_KEYS = ['overview', 'new', 'repeat'];

        const mockKPI = {
            yesterday: {
                overview: { total: 82, consume: 0.08, profit: 0.028, rate: 35, repurchase: 92, retention: 92, dailyOrders: 82, dailyAmount: 0.08, active: 78, activeRatio: 95.1, cupVol: 131, cupPer: 1.6, cupPrice: 6.1, cupProfit: 2.1 },
                new:      { total: 12, consume: 0.012, profit: 0.004, rate: 33, repurchase: 0, retention: 88, dailyOrders: 12, dailyAmount: 0.012, active: 11, activeRatio: 91.7, cupVol: 19, cupPer: 1.6, cupPrice: 6.2, cupProfit: 2.0 },
                repeat:   { total: 65, consume: 0.062, profit: 0.022, rate: 35.5, repurchase: 100, retention: 94, dailyOrders: 65, dailyAmount: 0.062, active: 62, activeRatio: 95.4, cupVol: 104, cupPer: 1.6, cupPrice: 6.0, cupProfit: 2.2 }
            },
            today: {
                overview: { total: 85, consume: 0.085, profit: 0.03, rate: 35.3, repurchase: 93, retention: 93, dailyOrders: 85, dailyAmount: 0.085, active: 81, activeRatio: 95.3, cupVol: 136, cupPer: 1.6, cupPrice: 6.2, cupProfit: 2.15 },
                new:      { total: 13, consume: 0.013, profit: 0.0045, rate: 34, repurchase: 0, retention: 90, dailyOrders: 13, dailyAmount: 0.013, active: 12, activeRatio: 92.3, cupVol: 21, cupPer: 1.6, cupPrice: 6.3, cupProfit: 2.1 },
                repeat:   { total: 68, consume: 0.068, profit: 0.024, rate: 35.3, repurchase: 100, retention: 95, dailyOrders: 68, dailyAmount: 0.068, active: 65, activeRatio: 95.6, cupVol: 109, cupPer: 1.6, cupPrice: 6.1, cupProfit: 2.2 }
            },
            monthToDate: {
                overview: { total: 2520, consume: 2.58, profit: 0.90, rate: 35.01, repurchase: 95.24, retention: 95.24, dailyOrders: 2400, dailyAmount: 1.20, active: 2400, activeRatio: 95.24, cupVol: 3940, cupPer: 1.6, cupPrice: 6.5, cupProfit: 2.3 },
                new:      { total: 380, consume: 0.38, profit: 0.133, rate: 35, repurchase: 0, retention: 92, dailyOrders: 360, dailyAmount: 0.36, active: 350, activeRatio: 92.1, cupVol: 576, cupPer: 1.6, cupPrice: 6.4, cupProfit: 2.25 },
                repeat:   { total: 2000, consume: 2.08, profit: 0.73, rate: 35.1, repurchase: 100, retention: 96, dailyOrders: 1900, dailyAmount: 0.99, active: 1920, activeRatio: 96, cupVol: 3040, cupPer: 1.6, cupPrice: 6.5, cupProfit: 2.3 }
            },
            yearToDate: {
                overview: { total: 18200, consume: 18.6, profit: 6.51, rate: 35, repurchase: 94.5, retention: 94.5, dailyOrders: 17300, dailyAmount: 17.7, active: 17200, activeRatio: 94.5, cupVol: 27680, cupPer: 1.6, cupPrice: 6.5, cupProfit: 2.28 },
                new:      { total: 2100, consume: 2.1, profit: 0.74, rate: 35.2, repurchase: 0, retention: 91, dailyOrders: 2000, dailyAmount: 2.0, active: 1900, activeRatio: 90.5, cupVol: 3200, cupPer: 1.6, cupPrice: 6.5, cupProfit: 2.3 },
                repeat:   { total: 14800, consume: 15.2, profit: 5.32, rate: 35, repurchase: 100, retention: 95, dailyOrders: 14100, dailyAmount: 14.5, active: 14080, activeRatio: 95.1, cupVol: 22560, cupPer: 1.6, cupPrice: 6.5, cupProfit: 2.3 }
            },
            lastYear: {
                overview: { total: 23800, consume: 24.2, profit: 8.47, rate: 35, repurchase: 95, retention: 95, dailyOrders: 22600, dailyAmount: 23.0, active: 22600, activeRatio: 95, cupVol: 36160, cupPer: 1.6, cupPrice: 6.35, cupProfit: 2.28 },
                new:      { total: 2800, consume: 2.8, profit: 0.98, rate: 35, repurchase: 0, retention: 90, dailyOrders: 2660, dailyAmount: 2.66, active: 2520, activeRatio: 94.7, cupVol: 4256, cupPer: 1.6, cupPrice: 6.4, cupProfit: 2.3 },
                repeat:   { total: 19200, consume: 19.6, profit: 6.86, rate: 35, repurchase: 100, retention: 96, dailyOrders: 18300, dailyAmount: 18.6, active: 18400, activeRatio: 95.8, cupVol: 29280, cupPer: 1.6, cupPrice: 6.4, cupProfit: 2.3 }
            }
        };

        const mockYoyMom = {
            yesterday: { overview: { yoy: [5, 2.9], mom: [1.6, -0.9], profit: [1.5, -0.7], rate: [-0.1, 0.2], rep: [-4.8, -2.8], ret: [-4.8, -2.8] } },
            today: { overview: { yoy: [5.2, 3], mom: [1.8, -0.5], profit: [1.6, -0.5], rate: [-0.05, 0.25], rep: [-4.5, -2.5], ret: [-4.5, -2.5] } },
            monthToDate: { overview: { yoy: [5.0, 2.9], mom: [1.6, -0.9], profit: [1.5, -0.7], rate: [-0.1, 0.2], rep: [-4.8, -2.8], ret: [-4.8, -2.8] } },
            yearToDate: { overview: { yoy: [4.2, 2.5], mom: [1.2, -0.6], profit: [1.3, -0.5], rate: [-0.2, 0.15], rep: [-4.2, -2.2], ret: [-4.2, -2.2] } },
            lastYear: { overview: { yoy: [3.5, 2], mom: [0.8, -0.4], profit: [1.0, -0.3], rate: [-0.3, 0.1], rep: [-3.8, -1.8], ret: [-3.8, -1.8] } }
        };

        function formatNum(v, isWan) {
            if (isWan) return v.toFixed(2) + '万';
            if (v >= 10000) return (v / 10000).toFixed(2) + '万';
            if (v >= 1000) return v.toLocaleString();
            return String(v);
        }

        let currentRange = 'monthToDate';
        let currentDim = 'overview';
        let trendChartInstance, lifecycleChartInstance, structureChartInstance, cupChartInstance, activeChartInstance, retentionChartInstance;

        function getData() {
            const k = mockKPI[currentRange][currentDim];
            const yoyMom = mockYoyMom[currentRange] && mockYoyMom[currentRange].overview ? mockYoyMom[currentRange].overview : { yoy: [0, 0], mom: [0, 0], profit: [0, 0], rate: [0, 0], rep: [0, 0], ret: [0, 0] };
            return { k, yoyMom };
        }

        function renderKPI() {
            const { k, yoyMom } = getData();
            document.getElementById('kpiTotal').textContent = formatNum(k.total, false);
            document.getElementById('kpiConsume').textContent = formatNum(k.consume, true);
            document.getElementById('kpiProfit').textContent = formatNum(k.profit, true);
            document.getElementById('kpiRate').textContent = k.rate + '%';
            document.getElementById('kpiRepurchase').textContent = k.repurchase + '%';
            document.getElementById('kpiRetention').textContent = k.retention + '%';

            document.getElementById('kpiTotalYoy').textContent = '同比 ' + yoyMom.yoy[0] + '%↑';
            document.getElementById('kpiTotalMom').textContent = '环比 ' + yoyMom.yoy[1] + '%↑';
            document.getElementById('kpiConsumeYoy').textContent = '同比 ' + yoyMom.mom[0] + '%↑';
            document.getElementById('kpiConsumeMom').textContent = '环比 ' + yoyMom.mom[1] + '%↓';
            document.getElementById('kpiProfitYoy').textContent = '同比 ' + yoyMom.profit[0] + '%↑';
            document.getElementById('kpiProfitMom').textContent = '环比 ' + yoyMom.profit[1] + '%↓';
            document.getElementById('kpiRateYoy').textContent = '同比 ' + yoyMom.rate[0] + '%↓';
            document.getElementById('kpiRateMom').textContent = '环比 ' + yoyMom.rate[1] + '%↑';
            document.getElementById('kpiRepurchaseYoy').textContent = '同比 ' + yoyMom.rep[0] + '%↓';
            document.getElementById('kpiRepurchaseMom').textContent = '环比 ' + yoyMom.rep[1] + '%↓';
            document.getElementById('kpiRetentionYoy').textContent = '同比 ' + yoyMom.ret[0] + '%↓';
            document.getElementById('kpiRetentionMom').textContent = '环比 ' + yoyMom.ret[1] + '%↓';

            document.getElementById('dailyOrders').textContent = formatNum(k.dailyOrders, false);
            document.getElementById('dailyAmount').textContent = formatNum(k.dailyAmount, true);
            document.getElementById('activeCount').textContent = formatNum(k.active, false);
            document.getElementById('activeRatio').textContent = k.activeRatio + '%';
            document.getElementById('activeYoy').textContent = '0.0%';
            document.getElementById('activeMom').textContent = '0.0%';
            document.getElementById('retainCount').textContent = formatNum(k.active, false);
            document.getElementById('retainYoy').textContent = '0.0%';
            document.getElementById('retainMom').textContent = '0.0%';
            document.getElementById('cupVolume').textContent = formatNum(k.cupVol, false);
            document.getElementById('cupPerMember').textContent = k.cupPer;
            document.getElementById('cupPrice').textContent = k.cupPrice;
            document.getElementById('cupProfit').textContent = k.cupProfit;
        }

        const dates5 = ['2025-06-26', '2025-06-27', '2025-06-28', '2025-06-29', '2025-06-30'];

        function buildTrendChart() {
            const consumeData = [2.5, 2.5, 2.5, 2.5, 2.5];
            const memberData = [0.1, 0.1, 0.1, 0.1, 0.1];
            const singleData = genSingleMemberSeries(5);
            if (trendChartInstance) trendChartInstance.destroy();
            const ctx = document.getElementById('trendChart').getContext('2d');
            trendChartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: dates5,
                    datasets: [
                        { label: '会员消费金额', data: consumeData, backgroundColor: '#1890ff', yAxisID: 'y' },
                        { label: '会员数', data: memberData, backgroundColor: '#52c41a', yAxisID: 'y' },
                        { label: '单会员消费金额', data: singleData, type: 'line', borderColor: '#ff7a00', backgroundColor: 'transparent', yAxisID: 'y1', tension: 0.2 }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: { position: 'left', max: 3 },
                        y1: { position: 'right', min: 6, max: 14, grid: { drawOnChartArea: false } }
                    },
                    plugins: { legend: { position: 'top' } }
                }
            });
        }

        function buildLifecycleChart() {
            const d = currentDim === 'overview' ? [600, 1200, 500, 220] : currentDim === 'new' ? [120, 180, 60, 20] : [480, 1000, 380, 140];
            if (lifecycleChartInstance) lifecycleChartInstance.destroy();
            const ctx = document.getElementById('lifecycleChart').getContext('2d');
            lifecycleChartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: ['会员生命周期分布'],
                    datasets: [
                        { label: '新会员', data: [d[0]], backgroundColor: '#87ceeb', stack: 'lifecycle' },
                        { label: '活跃会员', data: [d[1]], backgroundColor: '#52c41a', stack: 'lifecycle' },
                        { label: '沉睡会员', data: [d[2]], backgroundColor: '#722ed1', stack: 'lifecycle' },
                        { label: '流失会员', data: [d[3]], backgroundColor: '#fa8c16', stack: 'lifecycle' }
                    ]
                },
                options: {
                    indexAxis: 'y',
                    scales: {
                        x: { stacked: true, max: 2600 },
                        y: { stacked: true }
                    },
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: { legend: { display: true, position: 'top' } }
                }
            });
        }

        function buildStructureChart() {
            const data = [23.81, 26.19, 25, 25];
            if (structureChartInstance) structureChartInstance.destroy();
            const ctx = document.getElementById('structureChart').getContext('2d');
            structureChartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: ['18岁以下', '18-30岁', '30-50岁', '50岁以上'],
                    datasets: [{ data, backgroundColor: ['#87ceeb', '#52c41a', '#722ed1', '#1890ff'] }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: { legend: { position: 'right' } }
                }
            });
        }

        function buildCupChart() {
            const cupData = [0.4, 0.4, 0.4, 0.4, 0.4];
            const priceData = [6, 6.2, 5.9, 6.3, 6.5];
            if (cupChartInstance) cupChartInstance.destroy();
            const ctx = document.getElementById('cupChart').getContext('2d');
            cupChartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: dates5,
                    datasets: [
                        { label: '出杯量', data: cupData, backgroundColor: '#1890ff', yAxisID: 'y' },
                        { label: '杯单价', data: priceData, type: 'line', borderColor: '#ff7a00', backgroundColor: 'transparent', yAxisID: 'y1' }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: { position: 'left', min: 0, max: 0.5 },
                        y1: { position: 'right', min: 2, max: 7, grid: { drawOnChartArea: false } }
                    },
                    plugins: { legend: { position: 'top' } }
                }
            });
        }

        function buildActiveChart() {
            const activeData = [0.25, 0.25, 0.25, 0.25, 0.25];
            const ratioData = [1, 1, 1, 1, 1];
            const targetData = [1, 1, 1, 1, 1];
            if (activeChartInstance) activeChartInstance.destroy();
            const ctx = document.getElementById('activeChart').getContext('2d');
            activeChartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: dates5,
                    datasets: [
                        { label: '有效会员数', data: activeData, backgroundColor: '#1890ff', yAxisID: 'y' },
                        { label: '有效会员占比', data: ratioData, type: 'line', borderColor: '#52c41a', backgroundColor: 'transparent', yAxisID: 'y1' },
                        { label: '目标达成率', data: targetData, type: 'line', borderColor: '#ff7a00', backgroundColor: 'transparent', yAxisID: 'y1' }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: { position: 'left', max: 0.5 },
                        y1: { position: 'right', max: 1.2, grid: { drawOnChartArea: false } }
                    },
                    plugins: { legend: { position: 'top' } }
                }
            });
        }

        function buildRetentionChart() {
            const next = [2.5, 2.5, 2.5, 2.5, 2.5];
            const d7 = [2.2, 2.2, 2.2, 2.2, 2.2];
            const d30 = [1.8, 1.8, 1.8, 1.8, 1.8];
            if (retentionChartInstance) retentionChartInstance.destroy();
            const ctx = document.getElementById('retentionChart').getContext('2d');
            retentionChartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: dates5,
                    datasets: [
                        { label: '次日留存率', data: next, borderColor: '#1890ff', backgroundColor: 'transparent', tension: 0.2 },
                        { label: '7日留存率', data: d7, borderColor: '#52c41a', backgroundColor: 'transparent', tension: 0.2 },
                        { label: '30日留存率', data: d30, borderColor: '#ff7a00', backgroundColor: 'transparent', tension: 0.2 }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: { y: { beginAtZero: true, max: 3 } },
                    plugins: { legend: { position: 'top' } }
                }
            });
        }

        function refreshAll() {
            renderKPI();
            buildTrendChart();
            buildLifecycleChart();
            buildStructureChart();
            buildCupChart();
            buildActiveChart();
            buildRetentionChart();
        }

        document.getElementById('customDateBtn').addEventListener('click', function(e) {
            e.stopPropagation();
            document.querySelectorAll('.date-filter').forEach(f => f.classList.remove('active'));
            this.classList.add('active');
            document.getElementById('customDatePanel').classList.toggle('show');
        });

        document.getElementById('customApply').addEventListener('click', function() {
            const start = document.getElementById('customStart').value;
            const end = document.getElementById('customEnd').value;
            document.getElementById('updateDate').textContent = start + ' ~ ' + end;
            document.getElementById('customDatePanel').classList.remove('show');
            refreshAll();
        });

        document.addEventListener('click', function() {
            document.getElementById('customDatePanel').classList.remove('show');
        });

        document.getElementById('customDatePanel').addEventListener('click', function(e) { e.stopPropagation(); });

        document.querySelectorAll('.date-filter[data-range]').forEach(el => {
            el.addEventListener('click', function(e) {
                e.stopPropagation();
                document.querySelectorAll('.date-filter').forEach(f => f.classList.remove('active'));
                this.classList.add('active');
                currentRange = this.dataset.range;
                document.getElementById('customDatePanel').classList.remove('show');
                refreshAll();
            });
        });

        document.querySelectorAll('.nav-tab').forEach(el => {
            el.addEventListener('click', function() {
                document.querySelectorAll('.nav-tab').forEach(t => t.classList.remove('active'));
                this.classList.add('active');
                currentDim = this.dataset.dim;
                refreshAll();
            });
        });

        document.querySelectorAll('.chart-tab').forEach(tab => {
            tab.addEventListener('click', function() {
                this.parentElement.querySelectorAll('.chart-tab').forEach(t => t.classList.remove('active'));
                this.classList.add('active');
            });
        });

        refreshAll();
    </script>
</body>
</html>
