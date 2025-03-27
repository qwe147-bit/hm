<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>客户信息分类统计</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        textarea { width: 400px; height: 300px; margin-bottom: 10px; }
        table { border-collapse: collapse; margin: 20px 0; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
        .container { display: flex; flex-direction: column; gap: 20px; }
        button { padding: 5px 10px; }
    </style>
</head>
<body>
    <div class="container">
        <div>
            <textarea id="inputData" placeholder="请输入客户信息..."></textarea>
            <br>
            <button onclick="processData()">统计</button>
            <button onclick="clearInput()">清空</button>
        </div>

        <div id="personStats">
            <h3>人员统计</h3>
            <table id="personTable"></table>
        </div>

        <div id="categoryStats">
            <h3>分类统计</h3>
            <table id="categoryTable"></table>
        </div>

        <div id="summaryStats">
            <h3>汇总信息</h3>
            <table id="summaryTable"></table>
            <h4>所有号码列表</h4>
            <div id="phoneList"></div>
        </div>

        <div id="detailedStats">
            <h3>详细统计表</h3>
            <table id="detailedTable">
                <thead>
                    <tr>
                        <th>名字</th>
                        <th>当天接待客户数量</th>
                        <th>当天留存顾客数量</th>
                        <th>当月总留存客户</th>
                        <th>当前热聊客户</th>
                        <th>目前意向客户</th>
                        <th>当月首次存款人数</th>
                    </tr>
                </thead>
                <tbody></tbody>
            </table>
        </div>
    </div>

    <script>
        function processData() {
            const input = document.getElementById('inputData').value;
            const lines = input.split('\n').filter(line => line.trim() !== '');
            
            const persons = {};
            let uselessTotal = 0;
            let noResponseTotal = 0;
            const phones = new Set();

            lines.forEach(line => {
                line = line.trim();
                
                // 识别姓名（假设第一行是姓名）
                if (!line.match(/^\d/) && !line.includes(':') && !line.includes('：')) {
                    if (!persons[line]) {
                        persons[line] = {
                            newPeople: 0,
                            retainedToday: 0,
                            hotChat: 0,
                            investment: 0,
                            firstDeposit: 0,
                            total: 0
                        };
                    }
                    return;
                }

                // 识别电话号码
                const phoneMatch = line.match(/^\d{11}$/) || line.match(/^\d{7,8}-\d{4}$/);
                if (phoneMatch) {
                    phones.add(line.replace('-', ''));
                    return;
                }

                // 统计数据
                const cnMatch = line.match(/(.+?)：\s*(\d*)/);
                const enMatch = line.match(/(.+?):\s*(\d*)/);
                const match = cnMatch || enMatch;
                if (match) {
                    const key = match[1].trim();
                    const value = parseInt(match[2]) || 0;
                    const lastPerson = Object.keys(persons).pop();

                    if (key.includes('新增人数') || key.includes('new people added')) {
                        persons[lastPerson].newPeople = value;
                    } else if (key.includes('无用') || key.includes('Not useful')) {
                        uselessTotal += value;
                    } else if (key.includes('无回应') || key.includes('No response')) {
                        noResponseTotal += value;
                    } else if (key.includes('今日留存') || key.includes('Retained customers today')) {
                        persons[lastPerson].retainedToday = value;
                    } else if (key.includes('当前热聊') || key.includes('Current hot chat')) {
                        persons[lastPerson].hotChat = value;
                    } else if (key.includes('投资意向') || key.includes('investment intentions')) {
                        persons[lastPerson].investment = value;
                    } else if (key.includes('首笔入金') || key.includes('first deposit')) {
                        persons[lastPerson].firstDeposit = value;
                    } else if (key.includes('客户总数') || key.includes('Total Customers')) {
                        persons[lastPerson].total = value;
                    }
                }
            });

            // 人员统计
            let personTable = '<tr><th>姓名</th><th>新增人数</th></tr>';
            for (const [name, data] of Object.entries(persons)) {
                personTable += `<tr><td>${name}</td><td>${data.newPeople}</td></tr>`;
            }
            document.getElementById('personTable').innerHTML = personTable;

            // 分类统计
            document.getElementById('categoryTable').innerHTML = `
                <tr><th>分类</th><th>数量</th></tr>
                <tr><td>无用</td><td>${uselessTotal}</td></tr>
                <tr><td>无回应</td><td>${noResponseTotal}</td></tr>
            `;

            // 汇总信息
            document.getElementById('summaryTable').innerHTML = `
                <tr><th>项目</th><th>数量</th></tr>
                <tr><td>号码总数</td><td>${phones.size}</td></tr>
            `;
            document.getElementById('phoneList').innerHTML = Array.from(phones).join('<br>');

            // 详细统计表
            let detailedBody = '';
            for (const [name, data] of Object.entries(persons)) {
                detailedBody += `
                    <tr>
                        <td>${name}</td>
                        <td>${data.newPeople}</td>
                        <td>${data.retainedToday}</td>
                        <td>${data.total}</td>
                        <td>${data.hotChat}</td>
                        <td>${data.investment}</td>
                        <td>${data.firstDeposit}</td>
                    </tr>
                `;
            }
            document.getElementById('detailedTable').querySelector('tbody').innerHTML = detailedBody;
        }

        function clearInput() {
            document.getElementById('inputData').value = '';
            document.getElementById('personTable').innerHTML = '';
            document.getElementById('categoryTable').innerHTML = '';
            document.getElementById('summaryTable').innerHTML = '';
            document.getElementById('phoneList').innerHTML = '';
            document.getElementById('detailedTable').querySelector('tbody').innerHTML = '';
        }
    </script>
</body>
</html>
