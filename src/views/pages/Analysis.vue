<template>
    <div class="analysis-container">
        <!-- 顶部工具栏 -->
        <div class="toolbar">
            <h2 class="title">数据分析</h2>
            <div style="display: flex; gap: 20px;">
                <button class="edit-btn" @click="showEditDialog = true">
                    编辑数据
                </button>
                <button class="edit-btn" @click="() => chartData.push(Math.floor(Math.random() * 250))">
                    插入数据
                </button>
                <button class="edit-btn" @click="() => chartData.pop()">
                    删除数据
                </button>
            </div>
        </div>

        <!-- 统计信息卡片 -->
        <div class="stats-container">
            <div class="stat-card">
                <div class="stat-label">平均值</div>
                <div class="stat-value">{{ statistics.mean }}</div>
            </div>
            <div class="stat-card">
                <div class="stat-label">最大值</div>
                <div class="stat-value">{{ statistics.max }}</div>
            </div>
            <div class="stat-card">
                <div class="stat-label">最小值</div>
                <div class="stat-value">{{ statistics.min }}</div>
            </div>
            <div class="stat-card">
                <div class="stat-label">总数</div>
                <div class="stat-value">{{ statistics.sum }}</div>
            </div>
            <div class="stat-card">
                <div class="stat-label">标准差</div>
                <div class="stat-value">{{ statistics.stdDev }}</div>
            </div>
        </div>

        <!-- 图表容器 -->
        <div class="chart-container">
            <h2>当前数据</h2>
            <v-chart class="map-chart" :option="mapOptions" />
        </div>
        <br><br><br>
        <div class="predict-chart-container">
            <div style="display: flex; gap: 20px; align-items: center;">
                <h2>未来趋势</h2>
                <el-radio-group v-model="ModelMethod" style="border: 1px solid black; padding: 5px; border-radius:15px;">
                    <el-radio value="1" size="large">线性回归</el-radio>
                    <el-radio value="2" size="large">指数回归</el-radio>
                    <el-radio value="3" size="large">多项式回归</el-radio>
                    <el-radio value="4" size="large">ARIMA回归</el-radio>
                </el-radio-group>
            </div>
            <v-chart class="map-chart" :option="predictOptions" />
        </div>

        <!-- 编辑数据对话框 -->
        <div v-if="showEditDialog" class="dialog-overlay" @click="closeDialog">
            <div class="dialog" @click.stop>
                <div class="dialog-header">
                    <h3>编辑数据</h3>
                    <button class="close-btn" @click="closeDialog">×</button>
                </div>
                <div class="dialog-content">
                    <div class="input-group" v-for="(value, index) in tempChartData" :key="index">
                        <label>数据点 {{ index + 1 }}:</label>
                        <input type="number" v-model.number="tempChartData[index]" class="data-input" />
                    </div>
                </div>
                <div class="dialog-footer">
                    <button class="cancel-btn" @click="closeDialog">取消</button>
                    <button class="save-btn" @click="saveData">保存</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';
import VChart from 'vue-echarts';
import { use } from 'echarts/core';
import { CanvasRenderer } from 'echarts/renderers';
import { LineChart } from 'echarts/charts';
import {
    TitleComponent,
    TooltipComponent,
    LegendComponent,
    GridComponent
} from 'echarts/components';
import * as echarts from 'echarts/core';

const ModelMethod = ref('1')

// 注册必要的组件
use([
    CanvasRenderer,
    LineChart,
    TitleComponent,
    TooltipComponent,
    LegendComponent,
    GridComponent
]);

const { graphic } = echarts;

// 响应式数据
const chartData = ref([120, 132, 301, 134, 90, 230, 210, 100, 400]);
const showEditDialog = ref(false);
const tempChartData = ref([...chartData.value]);

// 计算统计信息
const statistics = computed(() => {
    const data = chartData.value;
    const sum = data.reduce((acc, val) => acc + val, 0);
    const mean = Math.round((sum / data.length) * 100) / 100;
    const max = Math.max(...data);
    const min = Math.min(...data);

    // 计算方差
    const variance = Math.round((data.reduce((acc, val) => acc + Math.pow(val - mean, 2), 0) / data.length) * 100) / 100;

    // 计算标准差
    const stdDev = Math.round(Math.sqrt(variance) * 100) / 100;

    return {
        mean,
        max,
        min,
        sum,
        stdDev
    };
});

// 图表配置
const mapOptions = computed(() => ({
    xAxis: {
        type: 'category',
        boundaryGap: false,
        data: chartData.value.map((_, index) => `${index + 1}`),
    },
    yAxis: {
        type: 'value',
    },
    grid: {
        top: '5%',
        left: '3%',
        right: '3%',
        bottom: '5%',
        containLabel: true,
    },
    color: ['#1976d2'],
    tooltip: {
        trigger: 'axis',
        backgroundColor: 'rgba(0,0,0,0.8)',
        borderColor: '#1976d2',
        textStyle: {
            color: '#fff'
        }
    },
    series: [
        {
            type: 'line',
            areaStyle: {
                color: new graphic.LinearGradient(0, 0, 0, 1, [
                    {
                        offset: 0,
                        color: 'rgba(25, 118, 210, 0.8)',
                    },
                    {
                        offset: 1,
                        color: 'rgba(25, 118, 210, 0)',
                    },
                ]),
            },
            lineStyle: {
                width: 4,
            },
            symbolSize: 8,
            data: chartData.value,
        }
    ],
}));

// 线性回归预测
function linearRegression(data) {
    const n = data.length;
    const x = Array.from({ length: n }, (_, i) => i);
    const y = data;

    const sumX = x.reduce((sum, val) => sum + val, 0);
    const sumY = y.reduce((sum, val) => sum + val, 0);
    const sumXY = x.reduce((sum, val, i) => sum + val * y[i], 0);
    const sumXX = x.reduce((sum, val) => sum + val * val, 0);

    const slope = (n * sumXY - sumX * sumY) / (n * sumXX - sumX * sumX);
    const intercept = (sumY - slope * sumX) / n;

    return slope * n + intercept;
}

// 指数平滑预测
function exponentialSmoothing(data, alpha) {
    if (data.length === 0) return 0;

    let smoothed = data[0];
    for (let i = 1; i < data.length; i++) {
        smoothed = alpha * data[i] + (1 - alpha) * smoothed;
    }

    // 计算趋势
    const recent = data.slice(-Math.min(5, data.length));
    const trend = recent.length > 1 ?
        (recent[recent.length - 1] - recent[0]) / (recent.length - 1) : 0;

    return smoothed + trend;
}

// 多项式回归预测
function polynomialRegression(data, degree) {
    const n = data.length;
    if (n < degree + 1) return data[n - 1];

    // 使用最小二乘法拟合多项式
    const x = Array.from({ length: n }, (_, i) => i);
    const y = data;

    // 构建范德蒙德矩阵
    const matrix = [];
    const target = [];

    for (let i = 0; i < n; i++) {
        const row = [];
        for (let j = 0; j <= degree; j++) {
            row.push(Math.pow(x[i], j));
        }
        matrix.push(row);
        target.push(y[i]);
    }

    // 简化的多项式拟合 (仅处理二次)
    if (degree === 2 && n >= 3) {
        const x1 = x[n - 3], y1 = y[n - 3];
        const x2 = x[n - 2], y2 = y[n - 2];
        const x3 = x[n - 1], y3 = y[n - 1];

        // 二次函数拟合: y = ax² + bx + c
        const denom = (x1 - x2) * (x1 - x3) * (x2 - x3);
        if (Math.abs(denom) < 1e-10) return y3 + (y3 - y2);

        const a = (x3 * (y2 - y1) + x2 * (y1 - y3) + x1 * (y3 - y2)) / denom;
        const b = (x3 * x3 * (y1 - y2) + x2 * x2 * (y3 - y1) + x1 * x1 * (y2 - y3)) / denom;
        const c = (x2 * x3 * (x2 - x3) * y1 + x3 * x1 * (x3 - x1) * y2 + x1 * x2 * (x1 - x2) * y3) / denom;

        const nextX = n;
        return a * nextX * nextX + b * nextX + c;
    }

    return data[n - 1];
}

// ARIMA简化版本 (自回归移动平均)
function arimaSimplified(data) {
    const n = data.length;
    if (n < 3) return data[n - 1];

    // AR(2) 模型: y(t) = φ1*y(t-1) + φ2*y(t-2) + ε
    const y1 = data[n - 1];
    const y2 = data[n - 2];

    // 简单的AR系数估计
    const phi1 = 0.7; // 一阶自回归系数
    const phi2 = 0.2; // 二阶自回归系数

    // 计算残差的移动平均
    const errors = [];
    for (let i = 2; i < n; i++) {
        const predicted = phi1 * data[i - 1] + phi2 * data[i - 2];
        errors.push(data[i] - predicted);
    }

    const avgError = errors.length > 0 ?
        errors.reduce((sum, err) => sum + err, 0) / errors.length : 0;

    // MA(1) 组件
    const theta = 0.3; // 移动平均系数
    const prediction = phi1 * y1 + phi2 * y2 + theta * avgError;

    return prediction;
}

const predictData = computed(() => {
    const data = chartData.value;
    const predictions = [];

    // 确保有足够的数据进行预测
    if (data.length < 3) return Array(5).fill(data[data.length - 1] || 0);

    for (let i = 0; i < 5; i++) {
        let predictedValue;
        const currentData = [...data, ...predictions];
        const len = currentData.length;

        switch (ModelMethod.value) {
            case '1': // 线性回归
                predictedValue = linearRegression(currentData);
                break;
            case '2': // 指数平滑
                predictedValue = exponentialSmoothing(currentData, 0.3);
                break;
            case '3': // 多项式回归 (二次)
                predictedValue = polynomialRegression(currentData, 2);
                break;
            case '4': // ARIMA简化版本
                predictedValue = arimaSimplified(currentData);
                break;
        }

        // 季节性调整 (模拟周期性波动)
        const seasonalFactor = Math.sin((len + i) * Math.PI / 6) * 0.1 + 1;
        predictedValue *= seasonalFactor;

        // 趋势衰减 (距离越远，趋势影响越小)
        const trendDecay = Math.exp(-i * 0.1);
        const baseline = currentData.slice(-Math.min(5, currentData.length))
            .reduce((sum, val) => sum + val, 0) / Math.min(5, currentData.length);

        predictedValue = baseline + (predictedValue - baseline) * trendDecay;

        // 添加渐减的随机噪声
        const noiseIntensity = 15 * Math.exp(-i * 0.3);
        const noise = (Math.random() - 0.5) * noiseIntensity;

        // 确保预测值合理
        predictedValue = Math.max(0, Math.round(predictedValue + noise));

        // 异常值检测和修正
        if (predictions.length > 0) {
            const lastPrediction = predictions[predictions.length - 1];
            const maxChange = Math.abs(lastPrediction * 0.3); // 最大变化30%

            if (Math.abs(predictedValue - lastPrediction) > maxChange) {
                const direction = predictedValue > lastPrediction ? 1 : -1;
                predictedValue = lastPrediction + direction * maxChange;
            }
        }

        predictions.push(predictedValue);
    }

    return predictions;
});

const predictOptions = computed(() => ({
    xAxis: {
        type: 'category',
        boundaryGap: false,
        data: predictData.value.map((_, index) => `${index + chartData.value.length + 1}`),
    },
    yAxis: {
        type: 'value',
    },
    grid: {
        top: '5%',
        left: '3%',
        right: '3%',
        bottom: '5%',
        containLabel: true,
    },
    color: ['#dccc13'],
    tooltip: {
        trigger: 'axis',
        backgroundColor: 'rgba(0,0,0,0.8)',
        borderColor: '#dccc13',
        textStyle: {
            color: '#fff'
        }
    },
    series: [
        {
            type: 'line',
            areaStyle: {
                color: new graphic.LinearGradient(0, 0, 0, 1, [
                    {
                        offset: 0,
                        color: 'rgba(255, 246, 143, 0.8)',
                    },
                    {
                        offset: 1,
                        color: 'rgba(255, 246, 143, 0)',
                    },
                ]),
            },
            smooth: true,
            lineStyle: {
                width: 4,
            },
            symbolSize: 8,
            data: predictData.value,
        }
    ],
}));

// 方法
const closeDialog = () => {
    showEditDialog.value = false;
    tempChartData.value = [...chartData.value]; // 重置临时数据
};

const saveData = () => {
    chartData.value = [...tempChartData.value];
    showEditDialog.value = false;
};

// 监听对话框开启，更新临时数据
watch(showEditDialog, (newVal) => {
    if (newVal) {
        tempChartData.value = [...chartData.value];
    }
});
</script>

<style scoped>
.analysis-container {
    padding: 20px;
    background: linear-gradient(135deg, #e3f2fd 0%, #f1f8e9 100%);
    min-height: 100vh;
}

.toolbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    padding: 0 10px;
}

.title {
    color: #1565c0;
    margin: 0;
    font-weight: 600;
}

.edit-btn {
    background: linear-gradient(45deg, #1976d2, #42a5f5);
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 25px;
    cursor: pointer;
    font-weight: 500;
    transition: all 0.3s ease;
    box-shadow: 0 2px 8px rgba(25, 118, 210, 0.3);
}

.edit-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 15px rgba(25, 118, 210, 0.4);
    background: linear-gradient(45deg, #1565c0, #2196f3);
}

.stats-container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 15px;
    margin-bottom: 25px;
}

.stat-card {
    background: white;
    padding: 20px;
    border-radius: 12px;
    box-shadow: 0 2px 10px rgba(25, 118, 210, 0.1);
    text-align: center;
    transition: transform 0.3s ease;
    border-left: 4px solid #1976d2;
}

.stat-card:hover {
    transform: translateY(-3px);
    box-shadow: 0 4px 20px rgba(25, 118, 210, 0.15);
}

.stat-label {
    font-size: 14px;
    color: #546e7a;
    margin-bottom: 8px;
    font-weight: 500;
}

.stat-value {
    font-size: 24px;
    font-weight: bold;
    color: #1976d2;
}

.chart-container {
    background: white;
    padding: 20px;
    border-radius: 12px;
    box-shadow: 0 2px 10px rgba(25, 118, 210, 0.1);
    border: 3px solid #1976d2;
}

.predict-chart-container {
    background: white;
    padding: 20px;
    border-radius: 12px;
    box-shadow: 0 2px 10px rgba(25, 118, 210, 0.1);
    border: 3px solid #dccc13;
}

.map-chart {
    width: 100%;
    height: 400px;
}

.dialog-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(25, 118, 210, 0.1);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
    backdrop-filter: blur(2px);
}

.dialog {
    background: white;
    border-radius: 15px;
    width: 90%;
    max-width: 500px;
    max-height: 80vh;
    overflow: hidden;
    box-shadow: 0 10px 30px rgba(25, 118, 210, 0.3);
    border-top: 4px solid #1976d2;
}

.dialog-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 20px;
    border-bottom: 1px solid #e3f2fd;
    background: linear-gradient(135deg, #e3f2fd, #f8f9fa);
}

.dialog-header h3 {
    margin: 0;
    color: #1565c0;
}

.close-btn {
    background: none;
    border: none;
    font-size: 24px;
    cursor: pointer;
    color: #546e7a;
    width: 30px;
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    transition: all 0.3s ease;
}

.close-btn:hover {
    background: #e3f2fd;
    color: #1976d2;
}

.dialog-content {
    padding: 20px;
    max-height: 400px;
    overflow-y: auto;
}

.input-group {
    display: flex;
    align-items: center;
    margin-bottom: 15px;
}

.input-group label {
    flex: 0 0 120px;
    color: #546e7a;
    font-weight: 500;
}

.data-input {
    flex: 1;
    padding: 8px 12px;
    border: 2px solid #e3f2fd;
    border-radius: 8px;
    font-size: 14px;
    transition: border-color 0.3s ease;
}

.data-input:focus {
    outline: none;
    border-color: #1976d2;
    box-shadow: 0 0 0 3px rgba(25, 118, 210, 0.1);
}

.dialog-footer {
    display: flex;
    justify-content: flex-end;
    gap: 10px;
    padding: 20px;
    border-top: 1px solid #e3f2fd;
    background: linear-gradient(135deg, #e3f2fd, #f8f9fa);
}

.cancel-btn,
.save-btn {
    padding: 10px 20px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    font-weight: 500;
    transition: all 0.3s ease;
}

.cancel-btn {
    background: #eceff1;
    color: #546e7a;
    border: 1px solid #cfd8dc;
}

.cancel-btn:hover {
    background: #cfd8dc;
    color: #37474f;
}

.save-btn {
    background: linear-gradient(45deg, #1976d2, #42a5f5);
    color: white;
    box-shadow: 0 2px 8px rgba(25, 118, 210, 0.3);
}

.save-btn:hover {
    background: linear-gradient(45deg, #1565c0, #2196f3);
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(25, 118, 210, 0.4);
}
</style>