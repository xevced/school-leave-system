<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>校园请假系统（分页版）</title>
    <style>
        /* 保持原有样式不变 */
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            background-color: #f7f7f7;
            margin: 0;
            padding: 12px;
        }
        .container {
            background: #ffffff;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
            padding: 16px;
            overflow-x: auto;
        }
        .header {
            font-size: 18px;
            color: #333;
            margin-bottom: 16px;
            padding-bottom: 12px;
            border-bottom: 1px solid #e8e8e8;
        }
        .data-table {
            width: 100%;
            min-width: 500px;
            border-collapse: collapse;
            margin-bottom: 16px;
        }
        .data-table th,
        .data-table td {
            border: 1px solid #e8e8e8;
            padding: 10px;
            font-size: 13px;
        }
        .data-table th {
            background-color: #fafafa;
            font-weight: 600;
        }
        .status-approved {
            color: #52c41a;
            background-color: #f6ffed;
            padding: 2px 8px;
            border-radius: 4px;
            display: inline-block;
            border: 1px solid #b7eb8f;
        }
        .pagination {
            display: flex;
            justify-content: center;
            gap: 8px;
            margin-top: 20px;
        }
        .page-item {
            padding: 6px 12px;
            border-radius: 4px;
            color: #666;
            background: #f5f5f5;
            cursor: pointer;
        }
        .page-item.active {
            background: #1890ff;
            color: white;
        }
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0,0,0,0.5);
            align-items: center;
            padding: 16px;
        }
        .modal-content {
            background: white;
            width: 100%;
            max-width: 400px;
            margin: 0 auto;
            padding: 20px;
            border-radius: 8px;
            position: relative;
        }
        .modal-close {
            position: absolute;
            right: 20px;
            top: 20px;
            cursor: pointer;
            font-size: 20px;
            color: #999;
        }
        .detail-row {
            display: flex;
            margin: 14px 0;
            font-size: 14px;
            flex-wrap: wrap;
        }
        .detail-label {
            width: 80px;
            color: #666;
            margin-bottom: 4px;
        }
        .detail-value {
            color: #333;
            flex: 1;
            min-width: 60%;
        }
        .clickable {
            color: #1890ff;
            cursor: pointer;
            text-decoration: none;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">校园平台(学生请假)</div>
        
        <table class="data-table">
            <thead>
                <tr>
                    <th>序号</th>
                    <th>姓名</th>
                    <th>请假事由</th>
                    <th>审核状态</th>
                </tr>
            </thead>
            <tbody id="tableBody"></tbody>
        </table>

        <div class="pagination" id="pagination"></div>
    </div>

    <!-- 弹窗 -->
    <div id="detailModal" class="modal">
        <div class="modal-content">
            <span class="modal-close" onclick="closeModal()">×</span>
            <div class="header" style="margin-bottom: 16px;">请假详情</div>
            <div id="modalContent"></div>
        </div>
    </div>

    <script>
        // 模拟数据（可替换为真实数据）
        const allRecords = Array.from({length: 25}, (_, i) => ({
            id: i + 1,
            name: "张思平",
            reason: ["月假", "买药", "采购服装"][i % 3],
            status: "已通过",
            start: `2025-03-${String(i+1).padStart(2,'0')} 12:00:00`,
            end: `2025-03-${String(i+2).padStart(2,'0')} 18:00:00`,
            applyTime: `2025-02-${String(28 - i).padStart(2,'0')} 18:16:52`
        }));

        // 分页配置
        const PAGE_SIZE = 10;
        let currentPage = 1;

        // 初始化系统
        function initSystem() {
            renderTable();
            renderPagination();
        }

        // 渲染表格
        function renderTable() {
            const start = (currentPage - 1) * PAGE_SIZE;
            const end = start + PAGE_SIZE;
            const pageData = allRecords.slice(start, end);
            
            const tbody = document.getElementById('tableBody');
            tbody.innerHTML = pageData.map(record => `
                <tr>
                    <td><span class="clickable" onclick="showDetail(${record.id})">${record.id}</span></td>
                    <td>${record.name}</td>
                    <td>${record.reason}</td>
                    <td><span class="status-approved">${record.status}</span></td>
                </tr>
            `).join('');
        }

        // 渲染分页
        function renderPagination() {
            const totalPages = Math.ceil(allRecords.length / PAGE_SIZE);
            const pagination = document.getElementById('pagination');
            
            pagination.innerHTML = Array.from({length: totalPages}, (_, i) => `
                <span class="page-item ${i + 1 === currentPage ? 'active' : ''}" 
                      onclick="changePage(${i + 1})">
                    ${i + 1}
                </span>
            `).join('');
        }

        // 切换页码
        function changePage(page) {
            currentPage = page;
            renderTable();
            renderPagination();
        }

        // 显示详情弹窗
        function showDetail(recordId) {
            const record = allRecords.find(r => r.id === recordId);
            const modalContent = document.getElementById('modalContent');
            
            modalContent.innerHTML = `
                <div class="detail-row">
                    <div class="detail-label">学生姓名</div>
                    <div class="detail-value">${record.name}</div>
                </div>
                <div class="detail-row">
                    <div class="detail-label">请假事由</div>
                    <div class="detail-value">${record.reason}</div>
                </div>
                <div class="detail-row">
                    <div class="detail-label">开始时间</div>
                    <div class="detail-value">${record.start}</div>
                </div>
                <div class="detail-row">
                    <div class="detail-label">结束时间</div>
                    <div class="detail-value">${record.end}</div>
                </div>
                <div class="detail-row">
                    <div class="detail-label">申请时间</div>
                    <div class="detail-value">${record.applyTime}</div>
                </div>
                <div class="detail-row">
                    <div class="detail-label">审核状态</div>
                    <div class="detail-value status-approved">${record.status}</div>
                </div>
            `;

            document.getElementById('detailModal').style.display = 'flex';
        }

        // 关闭弹窗
        function closeModal() {
            document.getElementById('detailModal').style.display = 'none';
        }

        // 点击外部关闭
        window.onclick = function(e) {
            if (e.target.classList.contains('modal')) {
                closeModal();
            }
        }

        // 初始化
        window.onload = initSystem;
    </script>
</body>
</html>
