# 📘 คู่มือการ Integrate SMCIndicator.js

## สำหรับ Project: choppyMeerLab4/indexV3.html

---

## 📋 สารบัญ

1. [ภาพรวม](#1-ภาพรวม)
2. [ไฟล์ที่ต้องใช้](#2-ไฟล์ที่ต้องใช้)
3. [ขั้นตอนการ Integrate](#3-ขั้นตอนการ-integrate)
4. [การใช้งาน SMCIndicator](#4-การใช้งาน-smcindicator)
5. [การแสดงผลบน Chart](#5-การแสดงผลบน-chart)
6. [ตัวอย่าง Code](#6-ตัวอย่าง-code)
7. [ข้อควรระวัง](#7-ข้อควรระวัง)

---

## 1. ภาพรวม

**SMCIndicator.js** เป็น Standalone JavaScript Class ที่วิเคราะห์ Smart Money Concepts (SMC) ประกอบด้วย:

| Feature | คำอธิบาย |
|---------|----------|
| **Swing Points** | จุด Swing High/Low (HH, HL, LH, LL) |
| **Market Structure** | CHoCH (เปลี่ยนแนวโน้ม) และ BOS (ทะลุโครงสร้าง) |
| **Order Blocks** | แท่งที่สถาบันเข้าซื้อ/ขาย (โซนแนวรับ-แนวต้าน) |
| **Fair Value Gaps (FVG)** | ช่องว่างราคาที่มักถูก fill |
| **Equal Highs/Lows** | จุดที่ราคาเท่ากัน (liquidity zones) |

---

## 2. ไฟล์ที่ต้องใช้

### ไฟล์หลัก (เลือกใช้อันใดอันหนึ่ง):

```
📁 js/smc/
├── SMCIndicator.js           # Version ปกติ (แนะนำ)
└── SMCIndicator.standalone.js # Version รวมทุกอย่าง
```

### Copy ไปยัง Project:

```
📁 choppyMeerLab4/
└── js/
    └── SMCIndicator.js    👈 Copy มาไว้ที่นี่
```

---

## 3. ขั้นตอนการ Integrate

### ขั้นตอนที่ 1: เพิ่ม Script Tag

เปิด `indexV3.html` แล้วเพิ่ม script tag ก่อน `mainV3.js`:

```html
<!-- SMC Indicator -->
<script src="js/SMCIndicator.js"></script>

<!-- Main Script (ต้องมาหลัง SMCIndicator) -->
<script src="js/mainV3.js"></script>
```

### ขั้นตอนที่ 2: สร้าง Instance ใน mainV3.js

```javascript
// ประกาศตัวแปรเก็บ SMC Indicator
let smcIndicator = null;

// สร้าง Instance (ทำหลังจากสร้าง chart แล้ว)
function initSMC() {
    smcIndicator = new SMCIndicator({
        swingLength: 50,        // ความยาว Swing (10-100)
        internalLength: 5,      // ความยาว Internal Structure
        showOrderBlocks: true,  // แสดง Order Blocks
        showFVG: true,          // แสดง Fair Value Gaps
        showEqualHL: true       // แสดง Equal Highs/Lows
    });
}
```

### ขั้นตอนที่ 3: เรียกใช้เมื่อได้ข้อมูล OHLC

```javascript
// หลังจากได้ข้อมูล OHLC จาก Deriv API
function onDataReceived(ohlcData) {
    // OHLC data ต้องเป็น array ของ objects:
    // [{ time: 1234567890, open: 1.2, high: 1.3, low: 1.1, close: 1.25 }, ...]
    
    // วิเคราะห์ SMC
    smcIndicator.calculate(ohlcData);
    
    // ดึงผลลัพธ์
    const results = smcIndicator.getAllResults();
    
    // นำไปใช้งาน
    console.log('Swing Trend:', results.swingTrend);
    console.log('Internal Trend:', results.internalTrend);
    console.log('Order Blocks:', results.orderBlocks);
    console.log('FVGs:', results.fairValueGaps);
}
```

---

## 4. การใช้งาน SMCIndicator

### 4.1 Methods ที่ใช้บ่อย

```javascript
// วิเคราะห์ข้อมูล
smcIndicator.calculate(ohlcData);

// ดึงผลทั้งหมด
const results = smcIndicator.getAllResults();

// ดึงเฉพาะส่วน
const swingPoints = smcIndicator.getSwingPoints();
const structures = smcIndicator.getStructures();
const orderBlocks = smcIndicator.getOrderBlocks();
const fvgs = smcIndicator.getFVGs();
const eqHL = smcIndicator.getEqualHighsLows();
```

### 4.2 รูปแบบ Results

```javascript
results = {
    swingTrend: 'bullish' | 'bearish' | 'neutral',
    internalTrend: 'bullish' | 'bearish' | 'neutral',
    
    swingPoints: [
        { time: 1234567890, price: 1.25, swing: 'high', type: 'HH' },
        ...
    ],
    
    structures: [
        { time: 1234567890, type: 'CHoCH', direction: 'bullish', price: 1.25 },
        { time: 1234567890, type: 'BOS', direction: 'bearish', price: 1.20 },
        ...
    ],
    
    orderBlocks: [
        { time: 1234567890, high: 1.30, low: 1.25, bias: 'bullish', mitigated: false },
        ...
    ],
    
    fairValueGaps: [
        { time: 1234567890, top: 1.28, bottom: 1.25, bias: 'bullish', filled: false },
        ...
    ],
    
    equalHighsLows: [
        { type: 'EQH', time1: ..., time2: ..., price: 1.30 },
        ...
    ]
}
```

---

## 5. การแสดงผลบน Chart

### 5.1 แสดง Markers (Swing Points, CHoCH/BOS)

```javascript
function renderSMCMarkers(results) {
    const markers = [];
    
    // Swing Points
    for (const sp of results.swingPoints) {
        markers.push({
            time: sp.time,
            position: sp.swing === 'high' ? 'aboveBar' : 'belowBar',
            color: sp.swing === 'high' ? '#F23645' : '#089981',
            shape: sp.swing === 'high' ? 'arrowDown' : 'arrowUp',
            text: sp.type  // HH, HL, LH, LL
        });
    }
    
    // Structures (CHoCH/BOS)
    for (const s of results.structures) {
        markers.push({
            time: s.time,
            position: s.direction === 'bullish' ? 'aboveBar' : 'belowBar',
            color: s.direction === 'bullish' ? '#089981' : '#F23645',
            shape: 'circle',
            text: s.type  // CHoCH หรือ BOS
        });
    }
    
    // Set markers บน candleSeries
    candleSeries.setMarkers(markers.sort((a, b) => a.time - b.time));
}
```

### 5.2 แสดง Order Blocks เป็นกล่องสี

⚠️ **ต้องใช้ LightweightCharts 4.2+** สำหรับ primitives

```javascript
// ใช้ createPriceLine สำหรับ version 3.x
function renderOrderBlockLines(results) {
    const activeOBs = results.orderBlocks.filter(ob => !ob.mitigated);
    
    for (const ob of activeOBs) {
        // วาดเส้นที่ High ของ OB
        candleSeries.createPriceLine({
            price: ob.high,
            color: ob.bias === 'bullish' ? '#089981' : '#F23645',
            lineWidth: 1,
            lineStyle: 2, // Dashed
            title: ob.bias === 'bullish' ? 'OB↑' : 'OB↓'
        });
        
        // วาดเส้นที่ Low ของ OB
        candleSeries.createPriceLine({
            price: ob.low,
            color: ob.bias === 'bullish' ? '#089981' : '#F23645',
            lineWidth: 1,
            lineStyle: 2
        });
    }
}
```

### 5.3 แสดง Trend ใน UI

```html
<!-- HTML -->
<div id="smcTrend"></div>
```

```javascript
// JavaScript
function updateSMCTrendDisplay(results) {
    const el = document.getElementById('smcTrend');
    const trend = results.swingTrend;
    
    el.innerHTML = `
        <span style="color: ${trend === 'bullish' ? '#089981' : '#F23645'}">
            ${trend === 'bullish' ? '📈 ขาขึ้น' : '📉 ขาลง'}
        </span>
    `;
}
```

---

## 6. ตัวอย่าง Code เต็ม

### ตัวอย่างการ Integrate กับ Deriv WebSocket

```javascript
// mainV3.js

let smcIndicator = null;
let ohlcData = [];

// เริ่มต้น SMC
function initSMC() {
    smcIndicator = new SMCIndicator({
        swingLength: 50,
        showOrderBlocks: true,
        showFVG: true
    });
}

// เมื่อได้ข้อมูลจาก Deriv
ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    
    // Historical candles
    if (data.candles) {
        ohlcData = data.candles.map(c => ({
            time: c.epoch,
            open: parseFloat(c.open),
            high: parseFloat(c.high),
            low: parseFloat(c.low),
            close: parseFloat(c.close)
        }));
        
        // วิเคราะห์และแสดงผล
        analyzeSMC();
    }
    
    // Real-time update
    if (data.ohlc) {
        updateCandle(data.ohlc);
        analyzeSMC(); // วิเคราะห์ใหม่
    }
};

function analyzeSMC() {
    if (ohlcData.length < 50) return;
    
    smcIndicator.calculate(ohlcData);
    const results = smcIndicator.getAllResults();
    
    // แสดงผล
    renderSMCMarkers(results);
    updateSMCTrendDisplay(results);
}

// เรียกเมื่อ page load
document.addEventListener('DOMContentLoaded', () => {
    initChart();
    initSMC();
    connect();
});
```

---

## 7. ข้อควรระวัง

### ⚠️ สิ่งที่ต้องเช็ค:

| หัวข้อ | รายละเอียด |
|--------|-----------|
| **LightweightCharts Version** | ถ้าใช้ boxes ต้องเป็น 4.2+ |
| **OHLC Format** | ต้องมี `time`, `open`, `high`, `low`, `close` |
| **Time Format** | ใช้ Unix timestamp (seconds) |
| **Data Length** | ต้องมีอย่างน้อย 50 แท่งขึ้นไป |

### ⚡ Performance Tips:

```javascript
// ❌ อย่าวิเคราะห์ทุก tick
ws.onmessage = (event) => {
    analyzeSMC(); // ช้ามาก!
};

// ✅ วิเคราะห์เฉพาะเมื่อแท่งใหม่เกิด
let lastCandleTime = 0;

ws.onmessage = (event) => {
    const currentTime = getCurrentCandleTime();
    
    if (currentTime > lastCandleTime) {
        lastCandleTime = currentTime;
        analyzeSMC(); // วิเคราะห์เฉพาะแท่งใหม่
    }
};
```

---

## 📞 สรุป

การ Integrate ใช้เวลาประมาณ **30 นาที - 1 ชั่วโมง**

1. ✅ Copy ไฟล์
2. ✅ เพิ่ม script tag
3. ✅ สร้าง instance
4. ✅ เรียกใช้เมื่อได้ data
5. ✅ แสดงผลตามต้องการ

---
1. Copy SMCIndicator.js → js/
2. เพิ่ม <script> ใน HTML
3. new SMCIndicator(config)
4. smcIndicator.calculate(ohlcData)
5. เอา results ไปแสดง
*สร้างเมื่อ: 30 มกราคม 2569*
