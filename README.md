<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>长租轻资产决策驾驶舱 - 越秀橙·全配套增强版</title>
    
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.4.3/dist/echarts.min.js"></script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        :root {
            /* 基础配色 (保持 v2.0) */
            --bg-main: #F0F2F5;
            --bg-card: #FFFFFF;
            --border: #E8E8E8;
            --primary: #F58220;
            --primary-light: #FFF7E6;
            --text-main: #1F1F1F;
            --text-sub: #8C8C8C;
            --danger: #FF4D4F;
            --success: #52C41A;
            --blue: #1890FF;
            --purple: #722ED1;
            --cyan: #13C2C2;
            
            --shadow: 0 2px 8px rgba(0,0,0,0.08);
        }

        body {
            margin: 0; padding: 0;
            font-family: "PingFang SC", "Microsoft YaHei", sans-serif;
            background-color: var(--bg-main);
            color: var(--text-main);
            height: 100vh;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        /* === 顶部导航 === */
        .header {
            height: 60px;
            background: #FFFFFF;
            box-shadow: 0 2px 8px rgba(0,0,0,0.05);
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 24px;
            z-index: 100;
        }
        .logo-area {
            font-size: 20px;
            font-weight: bold;
            color: var(--primary);
            display: flex;
            align-items: center;
            gap: 10px;
        }
        .logo-icon {
            width: 32px; height: 32px;
            background: var(--primary);
            color: white;
            border-radius: 6px;
            display: flex; align-items: center; justify-content: center;
            font-size: 18px;
        }
        
        .nav-tabs { display: flex; gap: 20px; height: 100%; }
        .nav-item {
            display: flex; align-items: center; height: 100%;
            font-size: 15px; font-weight: 500; color: var(--text-main);
            cursor: pointer; position: relative; transition: 0.3s; padding: 0 10px;
        }
        .nav-item:hover { color: var(--primary); }
        .nav-item.active { color: var(--primary); font-weight: bold; }
        .nav-item.active::after {
            content: ''; position: absolute; bottom: 0; left: 0; width: 100%;
            height: 3px; background: var(--primary); border-radius: 3px 3px 0 0;
        }

        /* === 主内容区 === */
        .main-container { flex: 1; position: relative; overflow: hidden; }
        
        .page-view {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            display: none; padding: 20px; box-sizing: border-box; overflow-y: auto;
        }
        .page-view.active { display: block; }
        #page-map.page-view { padding: 0; display: none; }
        #page-map.page-view.active { display: flex; }

        /* === 通用组件 === */
        .grid-dashboard { display: grid; grid-template-columns: repeat(4, 1fr); gap: 20px; }
        .card { background: var(--bg-card); border-radius: 8px; padding: 20px; box-shadow: var(--shadow); }
        .card-title {
            font-size: 16px; font-weight: bold; margin-bottom: 15px;
            display: flex; justify-content: space-between;
            border-left: 4px solid var(--primary); padding-left: 10px; line-height: 1;
        }
        .kpi-big { font-size: 32px; font-weight: bold; color: var(--text-main); margin: 10px 0; }
        .kpi-sub { font-size: 12px; color: var(--text-sub); }
        
        /* === 地图布局 === */
        .map-layout { width: 100%; height: 100%; display: flex; position: relative; }
        .sidebar-left, .sidebar-right {
            width: 340px; background: rgba(255,255,255,0.96); backdrop-filter: blur(5px);
            height: 100%; z-index: 10; box-shadow: 2px 0 10px rgba(0,0,0,0.05);
            display: flex; flex-direction: column; border-right: 1px solid var(--border);
        }
        .sidebar-right { border-left: 1px solid var(--border); border-right: none; box-shadow: -2px 0 10px rgba(0,0,0,0.05); }
        .sidebar-content { flex: 1; overflow-y: auto; padding: 15px; }
        #map-core { flex: 1; height: 100%; background: #F0F0F0; }

        /* 表单与按钮 */
        .form-group { margin-bottom: 15px; }
        .form-label { font-size: 12px; color: var(--text-sub); margin-bottom: 5px; display: block; }
        .form-input { 
            width: 100%; padding: 8px; border: 1px solid var(--border); border-radius: 4px; 
            font-size: 13px; box-sizing: border-box; background: #FAFAFA;
        }
        .btn-primary {
            width: 100%; padding: 10px; background: var(--primary); color: white;
            border: none; border-radius: 4px; cursor: pointer; font-weight: bold; transition: 0.2s;
        }
        .btn-primary:hover { background: #E07010; }

        /* === 配套详情页特有样式 (NEW) === */
        .amenity-grid {
            display: grid; grid-template-columns: 280px 1fr; gap: 20px; height: 100%;
        }
        .amenity-nav { background: white; border-radius: 8px; overflow: hidden; box-shadow: var(--shadow); }
        .amenity-cat-item {
            padding: 15px 20px; border-bottom: 1px solid #f0f0f0; cursor: pointer;
            display: flex; justify-content: space-between; align-items: center;
            transition: 0.2s; font-size: 14px;
        }
        .amenity-cat-item:hover { background: #FFF7E6; color: var(--primary); }
        .amenity-cat-item.active { background: var(--primary); color: white; }
        
        .amenity-content { display: flex; flex-direction: column; gap: 20px; height: 100%; overflow: hidden; }
        
        .detail-table-container { flex: 1; overflow-y: auto; background: white; border-radius: 8px; padding: 20px; box-shadow: var(--shadow); }
        
        /* 标签体系 */
        .tag { padding: 3px 8px; border-radius: 4px; font-size: 11px; display: inline-block; border: 1px solid transparent; margin-right: 5px; }
        .tag-trans { background: #E6F7FF; color: #1890FF; border-color: #91D5FF; } /* 交通 */
        .tag-comm { background: #FFF7E6; color: #FA8C16; border-color: #FFD591; } /* 商业 */
        .tag-edu { background: #F9F0FF; color: #722ED1; border-color: #D3ADF7; } /* 教育 */
        .tag-med { background: #F6FFED; color: #52C41A; border-color: #B7EB8F; } /* 医疗 */
        .tag-risk { background: #FFF1F0; color: #F5222D; border-color: #FFA39E; font-weight: bold; } /* 风险 */
        .tag-biz { background: #E6FFFB; color: #13C2C2; border-color: #87E8DE; } /* 商务 */
        .tag-land { background: #FCFFE6; color: #A0D911; border-color: #EAFF8F; } /* 景观 */

        .filter-bar {
            display: flex; gap: 10px; margin-bottom: 15px; flex-wrap: wrap;
        }
        .summary-badge {
            background: #f5f5f5; padding: 5px 10px; border-radius: 4px; font-size: 12px; color: #666;
        }

    </style>
</head>
<body>

    <div class="header">
        <div class="logo-area">
            <div class="logo-icon"><i class="fa-solid fa-building"></i></div>
            <div>越秀长租决策系统 <span style="font-size:12px; font-weight:normal; color:#999; margin-left:10px;">v3.0 Pro</span></div>
        </div>
        <div class="nav-tabs">
            <div class="nav-item" onclick="switchTab('overview')">总览页</div>
            <div class="nav-item active" onclick="switchTab('map')">地图研判 & 本体</div>
            <div class="nav-item" onclick="switchTab('amenity')"><i class="fa-solid fa-list-check" style="margin-right:5px;"></i>周边配套详情</div>
            <div class="nav-item" onclick="switchTab('customer')">客群分析</div>
            <div class="nav-item" onclick="switchTab('competitor')">竞品分析</div>
        </div>
        <div style="display:flex; gap:15px; font-size:14px; color:#666;">
            <span><i class="fa-solid fa-location-dot"></i> 广州</span>
            <span><i class="fa-solid fa-user"></i> 管理员</span>
        </div>
    </div>

    <div class="main-container">
        
        <div id="page-overview" class="page-view">
            <div class="grid-dashboard">
                <div class="card">
                    <div class="card-title">管理项目总数</div>
                    <div class="kpi-big" style="color:var(--primary)">12</div>
                    <div class="kpi-sub">拟拓展: 3个</div>
                </div>
                <div class="card">
                    <div class="card-title">总房源规模 (间)</div>
                    <div class="kpi-big">3,580</div>
                    <div class="kpi-sub">环比 <span style="color:red">↑ 5.2%</span></div>
                </div>
                <div class="card">
                    <div class="card-title">本月营收 (万元)</div>
                    <div class="kpi-big">845.2</div>
                    <div class="kpi-sub">目标达成率: 102%</div>
                </div>
                <div class="card">
                    <div class="card-title">整体出租率</div>
                    <div class="kpi-big">92.4%</div>
                    <div class="kpi-sub">行业基准: 88%</div>
                </div>
                <div class="card" style="grid-column: span 2; height: 350px;">
                    <div class="card-title">广州各区租赁热度地图</div>
                    <div id="chart-bar-district" style="width:100%; height:280px;"></div>
                </div>
                <div class="card" style="grid-column: span 2; height: 350px;">
                    <div class="card-title">集团项目运营效能漏斗</div>
                    <div id="chart-funnel" style="width:100%; height:280px;"></div>
                </div>
            </div>
        </div>

        <div id="page-map" class="page-view active">
            <div class="map-layout">
                <div class="sidebar-left">
                    <div style="padding:15px; border-bottom:1px solid var(--border); font-weight:bold; color:var(--primary);">
                        <i class="fa-solid fa-pen-to-square"></i> 项目本体参数配置
                    </div>
                    <div class="sidebar-content">
                        <div class="form-group">
                            <label class="form-label">项目/地块名称</label>
                            <input type="text" class="form-input" value="天河智汇园拓展点" id="input-name">
                        </div>
                        <div class="card-title" style="font-size:13px; border-left-width:3px; margin-top:20px;">技术指标</div>
                        <div style="display:grid; grid-template-columns: 1fr 1fr; gap:10px;">
                            <div class="form-group">
                                <label class="form-label">总建面 (㎡)</label>
                                <input type="number" class="form-input" value="12000">
                            </div>
                            <div class="form-group">
                                <label class="form-label">容积率</label>
                                <input type="number" class="form-input" value="2.5">
                            </div>
                        </div>
                        <div class="form-group">
                            <label class="form-label">房源规划 (间)</label>
                            <input type="number" class="form-input" value="280">
                        </div>
                        <div class="card-title" style="font-size:13px; border-left-width:3px; margin-top:10px;">小区配套</div>
                        <div class="form-group">
                            <label class="form-label">公区设施</label>
                            <input type="text" class="form-input" value="健身房, 书吧, 共享厨房">
                        </div>
                        <div style="padding:10px; background:#FFFbe6; border-radius:4px; font-size:12px; color:#888; margin-bottom:15px;">
                            <i class="fa-solid fa-circle-info"></i> 请在右侧地图点击选点，系统将自动关联周边数据生成报告。
                        </div>
                        <button class="btn-primary" onclick="alert('已锁定坐标，请切换至[周边配套详情]页查看深度报告')">
                            <i class="fa-solid fa-crosshairs"></i> 锁定当前选址
                        </button>
                    </div>
                </div>

                <div id="map-core"></div>
                
                <div class="sidebar-right">
                    <div style="padding:15px; border-bottom:1px solid var(--border); font-weight:bold;">
                        研判预览 (3KM)
                    </div>
                    <div class="sidebar-content">
                        <div style="background:var(--primary-light); padding:15px; border-radius:6px; text-align:center; margin-bottom:20px;">
                            <div style="font-size:12px; color:var(--primary);">AI预测租金 (元/月)</div>
                            <div style="font-size:28px; font-weight:bold; color:var(--primary);" id="res-rent">--</div>
                        </div>
                        <div class="card-title" style="font-size:14px;">配套综合评分</div>
                        <div id="chart-radar-small" style="width:100%; height:200px;"></div>
                        
                        <div style="margin-top:10px; font-size:13px; color:#666;">
                            <div><span class="tag tag-trans">交通</span> 地铁站: <b id="count-subway">--</b> 个</div>
                            <div style="margin-top:5px;"><span class="tag tag-comm">商业</span> 购物中心: <b id="count-mall">--</b> 个</div>
                            <div style="margin-top:5px;"><span class="tag tag-edu">教育</span> 学校: <b id="count-school">--</b> 所</div>
                            <div style="margin-top:5px;"><span class="tag tag-risk">风险</span> 劣势设施: <b id="count-risk">--</b> 个</div>
                        </div>

                        <button onclick="switchTab('amenity')" style="width:100%; margin-top:20px; padding:10px; background:white; border:1px solid var(--primary); color:var(--primary); border-radius:4px; cursor:pointer; font-weight:bold;">
                            查看完整配套清单 &gt;
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <div id="page-amenity" class="page-view">
            <div style="margin-bottom:15px; display:flex; justify-content:space-between; align-items:center;">
                <div style="font-size:18px; font-weight:bold;">
                    <span style="color:var(--primary);" id="amenity-project-name">未选点</span> - 周边3KM配套深度分析报告
                </div>
                <div style="font-size:12px; color:#666;">数据生成时间: 2025-10-29 14:00</div>
            </div>

            <div class="amenity-grid">
                <div class="amenity-nav">
                    <div class="amenity-cat-item active" onclick="filterAmenity('ALL')">
                        <span><i class="fa-solid fa-layer-group"></i> 全部配套</span>
                        <span class="summary-badge" id="badge-all">0</span>
                    </div>
                    <div class="amenity-cat-item" onclick="filterAmenity('交通')">
                        <span><i class="fa-solid fa-train-subway" style="color:#1890FF"></i> 交通设施</span>
                        <span class="summary-badge" id="badge-trans">0</span>
                    </div>
                    <div class="amenity-cat-item" onclick="filterAmenity('商业')">
                        <span><i class="fa-solid fa-bag-shopping" style="color:#FA8C16"></i> 商业配套</span>
                        <span class="summary-badge" id="badge-comm">0</span>
                    </div>
                    <div class="amenity-cat-item" onclick="filterAmenity('教育')">
                        <span><i class="fa-solid fa-graduation-cap" style="color:#722ED1"></i> 教育配套</span>
                        <span class="summary-badge" id="badge-edu">0</span>
                    </div>
                    <div class="amenity-cat-item" onclick="filterAmenity('医疗')">
                        <span><i class="fa-solid fa-hospital" style="color:#52C41A"></i> 医疗配套</span>
                        <span class="summary-badge" id="badge-med">0</span>
                    </div>
                    <div class="amenity-cat-item" onclick="filterAmenity('商务')">
                        <span><i class="fa-solid fa-briefcase" style="color:#13C2C2"></i> 商务办公</span>
                        <span class="summary-badge" id="badge-biz">0</span>
                    </div>
                    <div class="amenity-cat-item" onclick="filterAmenity('景观')">
                        <span><i class="fa-solid fa-tree" style="color:#A0D911"></i> 景观休闲</span>
                        <span class="summary-badge" id="badge-land">0</span>
                    </div>
                    <div class="amenity-cat-item" onclick="filterAmenity('劣势')">
                        <span><i class="fa-solid fa-triangle-exclamation" style="color:#F5222D"></i> 劣势/风险</span>
                        <span class="summary-badge" id="badge-risk">0</span>
                    </div>
                </div>

                <div class="amenity-content">
                    <div style="display:grid; grid-template-columns:repeat(4, 1fr); gap:15px;">
                        <div class="card" style="padding:15px; display:flex; align-items:center; gap:15px;">
                            <div style="width:40px; height:40px; background:#E6F7FF; border-radius:50%; display:flex; align-items:center; justify-content:center; color:#1890FF; font-size:20px;"><i class="fa-solid fa-train"></i></div>
                            <div><div style="font-size:12px; color:#888;">地铁站密度</div><div style="font-size:18px; font-weight:bold;" id="stat-subway-density">--</div></div>
                        </div>
                        <div class="card" style="padding:15px; display:flex; align-items:center; gap:15px;">
                            <div style="width:40px; height:40px; background:#FFF7E6; border-radius:50%; display:flex; align-items:center; justify-content:center; color:#FA8C16; font-size:20px;"><i class="fa-solid fa-shop"></i></div>
                            <div><div style="font-size:12px; color:#888;">大型商圈</div><div style="font-size:18px; font-weight:bold;" id="stat-mall-count">--</div></div>
                        </div>
                        <div class="card" style="padding:15px; display:flex; align-items:center; gap:15px;">
                            <div style="width:40px; height:40px; background:#F6FFED; border-radius:50%; display:flex; align-items:center; justify-content:center; color:#52C41A; font-size:20px;"><i class="fa-solid fa-user-doctor"></i></div>
                            <div><div style="font-size:12px; color:#888;">三甲医院</div><div style="font-size:18px; font-weight:bold;" id="stat-3a-hosp">--</div></div>
                        </div>
                         <div class="card" style="padding:15px; display:flex; align-items:center; gap:15px; border:1px solid #FFCCC7;">
                            <div style="width:40px; height:40px; background:#FFF1F0; border-radius:50%; display:flex; align-items:center; justify-content:center; color:#F5222D; font-size:20px;"><i class="fa-solid fa-ban"></i></div>
                            <div><div style="font-size:12px; color:#888;">严重劣势源</div><div style="font-size:18px; font-weight:bold; color:#F5222D;" id="stat-risk-major">--</div></div>
                        </div>
                    </div>

                    <div class="detail-table-container">
                        <div class="filter-bar" id="sub-filter-container">
                            </div>
                        <table class="data-table">
                            <thead>
                                <tr>
                                    <th width="150">类别</th>
                                    <th width="150">细分项</th>
                                    <th>配套名称</th>
                                    <th width="100">距离(km)</th>
                                    <th>标签/备注</th>
                                </tr>
                            </thead>
                            <tbody id="amenity-table-body">
                                <tr><td colspan="5" style="text-align:center; padding:50px; color:#999;">请先在地图页选址</td></tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <div id="page-customer" class="page-view">
            <div class="grid-dashboard" style="height:100%;">
                <div class="card" style="grid-column: span 2;">
                    <div class="card-title">人口热力与流入流出</div>
                    <div id="chart-pop-trend" style="width:100%; height:300px;"></div>
                </div>
                <div class="card" style="grid-column: span 2;">
                    <div class="card-title">客群职业画像</div>
                    <div id="chart-job" style="width:100%; height:300px;"></div>
                </div>
            </div>
        </div>

        <div id="page-competitor" class="page-view">
            <div class="card">
                <div class="card-title">周边竞品经营数据透视</div>
                <table class="data-table">
                    <thead>
                        <tr><th>竞品名称</th><th>距离</th><th>开业时间</th><th>主力户型</th><th>租金(元)</th><th>出租率</th><th>配置</th></tr>
                    </thead>
                    <tbody id="comp-table-body"></tbody>
                </table>
            </div>
        </div>

    </div>

    <script>
        // === 全局数据存储 ===
        let GLOBAL_AMENITIES = []; // 存储当前选点的所有配套数据

        // === 1. Tab 切换 ===
        function switchTab(pageId) {
            document.querySelectorAll('.nav-item').forEach(el => el.classList.remove('active'));
            event.currentTarget.classList.add('active');
            document.querySelectorAll('.page-view').forEach(el => el.classList.remove('active'));
            document.getElementById('page-' + pageId).classList.add('active');
            
            setTimeout(() => {
                resizeCharts();
                if(pageId === 'map') map.invalidateSize();
                if(pageId === 'amenity' && GLOBAL_AMENITIES.length === 0) {
                    alert("请先在[地图研判]页面点击选址点，以生成配套数据。");
                }
            }, 100);
        }

        // === 2. 地图引擎 ===
        const map = L.map('map-core', { center: [23.1291, 113.2644], zoom: 13, zoomControl: false });
        L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', { attribution: '&copy; OpenStreetMap &copy; CARTO' }).addTo(map);
        let currentLayerGroup = L.layerGroup().addTo(map);

        // 地图点击事件 - 核心触发器
        map.on('click', function(e) {
            const lat = e.latlng.lat;
            const lng = e.latlng.lng;
            
            currentLayerGroup.clearLayers();
            
            // 绘制项目点
            L.circleMarker([lat, lng], { color: '#F58220', fillColor: '#F58220', fillOpacity: 1, radius: 8 })
                .addTo(currentLayerGroup).bindPopup("<b>选址点</b>").openPopup();
            L.circle([lat, lng], { radius: 3000, color: '#F58220', weight: 1, dashArray: '5, 5', fillOpacity: 0.05 }).addTo(currentLayerGroup);

            // 生成全量配套数据
            generateAllAmenities(lat, lng);

            // 更新页面显示
            updateRightSidebar();
            updateAmenityPageHeader();
            
            // 默认过滤展示全部
            filterAmenity('ALL');
        });

        // === 3. 数据生成器 (核心逻辑) ===
        // 按照您的要求：交通、商业、教育、医疗、休闲、商务、景观、劣势
        function generateAllAmenities(lat, lng) {
            GLOBAL_AMENITIES = [];
            
            // 辅助函数：生成随机点
            const rndPt = () => ({
                lat: lat + (Math.random()-0.5)*0.04, // 约4km范围
                lng: lng + (Math.random()-0.5)*0.04,
                dist: (Math.random() * 3).toFixed(2)
            });

            // 1. 交通 (Transport)
            const transTypes = [
                {sub:'地铁站', name:['体育西路','珠江新城','客村','广州塔','杨箕','东山口'], count: 4, tag:'tag-trans'},
                {sub:'公交站', name:['天河路口','南方报社','五羊新城','大剧院南'], count: 12, tag:'tag-trans'},
                {sub:'对外交通', name:['广州东站','天河客运站'], count: 1, tag:'tag-trans'},
                {sub:'快速路口', name:['华南快速出口','新光快速入口'], count: 2, tag:'tag-trans'}
            ];
            generateCategory(transTypes, '交通');

            // 2. 商业 (Commercial)
            const commTypes = [
                {sub:'大型商场', name:['天河城','正佳广场','太古汇','K11','IGC天汇','万菱汇'], count: 3, tag:'tag-comm'},
                {sub:'超市', name:['沃尔玛','盒马鲜生','Ole精品超市','百佳永辉'], count: 5, tag:'tag-comm'}
            ];
            generateCategory(commTypes, '商业');

            // 3. 教育 (Education)
            const eduTypes = [
                {sub:'幼儿园', name:['机关幼儿园','省委幼儿园','启雅幼儿园'], count: 5, tag:'tag-edu'},
                {sub:'小学', name:['体育东路小学','先烈东小学','华阳小学'], count: 3, tag:'tag-edu'},
                {sub:'中学', name:['广州中学','省实天河','执信中学'], count: 2, tag:'tag-edu'},
                {sub:'大学', name:['暨南大学','华南师范大学'], count: 1, tag:'tag-edu'}
            ];
            generateCategory(eduTypes, '教育');

            // 4. 医疗 (Medical)
            const medTypes = [
                {sub:'三甲医院', name:['中山三院','中山一院','省妇幼','南方医院'], count: 2, tag:'tag-med'},
                {sub:'普通医院', name:['天河区中医院','社区卫生服务中心'], count: 4, tag:'tag-med'}
            ];
            generateCategory(medTypes, '医疗');

            // 5. 商务 (Business)
            const bizTypes = [
                {sub:'写字楼', name:['广州国际金融中心','周大福金融中心','利通广场'], count: 8, tag:'tag-biz'},
                {sub:'产业园', name:['羊城创意园','TIT创意园'], count: 2, tag:'tag-biz'},
                {sub:'酒店', name:['希尔顿酒店','W酒店','柏悦酒店'], count: 4, tag:'tag-biz'}
            ];
            generateCategory(bizTypes, '商务');

            // 6. 景观 (Landscape/Leisure)
            const landTypes = [
                {sub:'公园', name:['天河公园','珠江公园','花城广场'], count: 3, tag:'tag-land'},
                {sub:'体育馆', name:['天河体育中心','二沙岛体育公园'], count: 1, tag:'tag-land'}
            ];
            generateCategory(landTypes, '景观');

            // 7. 劣势 (Risk)
            const riskTypes = [
                {sub:'垃圾站', name:['垃圾压缩站','废品回收站'], count: 1, tag:'tag-risk'},
                {sub:'其他', name:['高压变电站','施工工地'], count: 1, tag:'tag-risk'}
            ];
            generateCategory(riskTypes, '劣势');

            // 排序：按距离
            GLOBAL_AMENITIES.sort((a, b) => parseFloat(a.dist) - parseFloat(b.dist));
        }

        function generateCategory(types, mainCat) {
            types.forEach(t => {
                for(let i=0; i<t.count; i++) {
                    const pt = {
                        mainCat: mainCat,
                        subCat: t.sub,
                        name: t.name[Math.floor(Math.random()*t.name.length)] + (i>0 ? " ("+(i+1)+")" : ""),
                        dist: (Math.random() * 3).toFixed(2),
                        tagClass: t.tag
                    };
                    GLOBAL_AMENITIES.push(pt);
                }
            });
        }

        // === 4. 页面更新逻辑 ===

        // 更新地图页右侧栏
        function updateRightSidebar() {
            // 预测租金
            document.getElementById('res-rent').innerText = 3200 + Math.floor(Math.random() * 500);
            
            // 简单统计
            const count = (cat, sub) => GLOBAL_AMENITIES.filter(a => a.mainCat === cat && (sub ? a.subCat === sub : true)).length;
            
            document.getElementById('count-subway').innerText = count('交通', '地铁站');
            document.getElementById('count-mall').innerText = count('商业', '大型商场');
            document.getElementById('count-school').innerText = count('教育');
            document.getElementById('count-risk').innerText = count('劣势');
            
            updateRadarChart();
        }

        // 更新配套详情页头部
        function updateAmenityPageHeader() {
            const projName = document.getElementById('input-name').value;
            document.getElementById('amenity-project-name').innerText = projName;
            
            // 更新左侧导航的数字徽章
            const updateBadge = (id, cat) => {
                const num = GLOBAL_AMENITIES.filter(a => cat === 'ALL' ? true : a.mainCat === cat).length;
                document.getElementById(id).innerText = num;
            };
            updateBadge('badge-all', 'ALL');
            updateBadge('badge-trans', '交通');
            updateBadge('badge-comm', '商业');
            updateBadge('badge-edu', '教育');
            updateBadge('badge-med', '医疗');
            updateBadge('badge-biz', '商务');
            updateBadge('badge-land', '景观');
            updateBadge('badge-risk', '劣势');
            
            // 更新顶部统计卡
            const count = (cat, sub) => GLOBAL_AMENITIES.filter(a => a.mainCat === cat && a.subCat === sub).length;
            document.getElementById('stat-subway-density').innerText = (count('交通','地铁站') / 9).toFixed(1) + " 个/km²"; // 3km半径面积约28
            document.getElementById('stat-mall-count').innerText = count('商业','大型商场') + " 个";
            document.getElementById('stat-3a-hosp').innerText = count('医疗','三甲医院') + " 个";
            document.getElementById('stat-risk-major').innerText = GLOBAL_AMENITIES.filter(a => a.mainCat==='劣势').length + " 个";
        }

        // 筛选并渲染表格
        function filterAmenity(category) {
            // 样式激活
            document.querySelectorAll('.amenity-cat-item').forEach(el => el.classList.remove('active'));
            event.currentTarget.classList.add('active');

            const filteredData = GLOBAL_AMENITIES.filter(a => category === 'ALL' ? true : a.mainCat === category);
            
            // 渲染表格
            const tbody = document.getElementById('amenity-table-body');
            if(filteredData.length === 0) {
                tbody.innerHTML = `<tr><td colspan="5" style="text-align:center; padding:30px; color:#999;">该分类下暂无数据</td></tr>`;
                return;
            }

            tbody.innerHTML = filteredData.map(d => `
                <tr>
                    <td><span class="tag ${d.tagClass}">${d.mainCat}</span></td>
                    <td>${d.subCat}</td>
                    <td><b>${d.name}</b></td>
                    <td>${d.dist}</td>
                    <td style="color:#888;">正常运营</td>
                </tr>
            `).join('');
        }

        // === 5. 初始化图表 ===
        const chartBarDist = echarts.init(document.getElementById('chart-bar-district'));
        chartBarDist.setOption({
            color: ['#F58220'], tooltip: {}, xAxis: {data:['天河','海珠','黄埔','越秀','番禺']}, yAxis:{}, series:[{type:'bar', data:[120,98,85,60,70]}]
        });

        const chartFunnel = echarts.init(document.getElementById('chart-funnel'));
        chartFunnel.setOption({
            color: ['#F58220','#FF9C6E','#FFC069','#FFD591'],
            series: [{
                type: 'funnel', left: '10%', width: '80%',
                data: [
                    {value: 100, name: '线索接触'}, {value: 60, name: '实地勘测'}, {value: 30, name: '立项通过'}, {value: 10, name: '签约落地'}
                ]
            }]
        });

        const chartRadarSmall = echarts.init(document.getElementById('chart-radar-small'));
        function updateRadarChart() {
            chartRadarSmall.setOption({
                radar: { indicator: [{name:'交通',max:10},{name:'商业',max:10},{name:'教育',max:10},{name:'医疗',max:5},{name:'商务',max:10}], radius:'65%' },
                series: [{ type: 'radar', data: [{ value:[Math.random()*5+5, Math.random()*5+5, Math.random()*5+5, Math.random()*3+2, Math.random()*5+5], itemStyle:{color:'#F58220'}, areaStyle:{opacity:0.3} }] }]
            });
        }
        updateRadarChart();

        // 客群图表
        const chartPopTrend = echarts.init(document.getElementById('chart-pop-trend'));
        chartPopTrend.setOption({
            tooltip: {trigger:'axis'}, legend:{}, xAxis:{data:['1月','2月','3月','4月','5月','6月']}, yAxis:{},
            series:[{name:'常住人口',type:'line',smooth:true,data:[15.2,15.3,15.5,15.6,15.8,16.0], color:'#F58220'}]
        });
        
        const chartJob = echarts.init(document.getElementById('chart-job'));
        chartJob.setOption({
            series:[{type:'pie',radius:['40%','70%'], data:[{value:40,name:'互联网'},{value:30,name:'金融'},{value:20,name:'教育'},{value:10,name:'其他'}], color:['#F58220','#1890FF','#52C41A','#FA8C16']}]
        });

        // 竞品数据填充
        const compList = [
            {name:'万科泊寓(天河店)',dist:'0.5km',open:'2021',type:'单间',price:2800,rate:'95%',conf:'全配'},
            {name:'龙湖冠寓(五山店)',dist:'1.2km',open:'2022',type:'一房',price:3100,rate:'92%',conf:'全配'},
            {name:'魔方公寓(珠江新城)',dist:'2.5km',open:'2020',type:'单间',price:3800,rate:'98%',conf:'高端'}
        ];
        document.getElementById('comp-table-body').innerHTML = compList.map(c => `
            <tr><td>${c.name}</td><td>${c.dist}</td><td>${c.open}</td><td>${c.type}</td><td style="color:#F58220;font-weight:bold;">${c.price}</td><td>${c.rate}</td><td>${c.conf}</td></tr>
        `).join('');

        function resizeCharts() {
            chartBarDist.resize(); chartFunnel.resize(); chartRadarSmall.resize(); chartPopTrend.resize(); chartJob.resize();
        }
        window.onresize = resizeCharts;

        // 初始化
        filterAmenity('ALL'); // 初始空状态
    </script>
</body>
</html>
