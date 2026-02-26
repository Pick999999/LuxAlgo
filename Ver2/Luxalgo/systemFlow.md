# System Flow: Deriv SMC Live Analysis

เอกสารนี้สรุปขั้นตอนการทำงานของระบบ ตั้งแต่ผู้ใช้กดปุ่ม **Live** ไปจนถึงการวาดกราฟและแสดงผลลัพธ์ SMC (Smart Money Concepts) เช่น BOS, HH, HL, LH

## 1. การเริ่มต้น (Initialization & Trigger)

*   **Action**: ผู้ใช้กดปุ่ม `▶️ Live` บนหน้าเว็บ (`smc_deriv.html`)
*   **File**: `js/smc/DerivSMC.js`
*   **Function**: `DerivSMC.prototype.toggleLive()`
    *   ทำการเชื่อมต่อ WebSocket ไปยัง Deriv (`this.connect()`)
    *   ส่ง request JSON เพื่อขอข้อมูลกราฟแท่งเทียน (`style: 'candles'`) พร้อม parameter `subscribe: 1` เพื่อขอข้อมูล real-time

## 2. การรับข้อมูล Real-time (Data Acquisition)

*   **File**: `js/smc/DerivSMC.js`
*   **Function**: `DerivSMC.prototype.setupMessageHandler()` (ภายใน `ws.onmessage`)
    *   ระบบได้รับข้อมูล JSON จาก WebSocket
    *   **กรณีข้อมูลเริ่มต้น (History)**: เก็บลงตัวแปร `this.ohlcData`
    *   **กรณีข้อมูล Real-time (Tick)**: ตรวจสอบ `data.ohlc` และเรียกฟังก์ชัน `this.updateCandle(candle)`

## 3. การอัปเดตแท่งเทียน (Candle Update Loop)

*   **File**: `js/smc/DerivSMC.js`
*   **Function**: `DerivSMC.prototype.updateCandle(candle)`
    *   **Logic**:
        *   ถ้ายลเวลา (Time) ตรงกับแท่งล่าสุด: อัปเดตราคา High, Low, Close ของแท่งเดิมใน `this.ohlcData`
        *   ถ้าเวลาใหม่กว่า: เพิ่มแท่งใหม่ (push) เข้าไปใน `this.ohlcData`
    *   **Trigger Analysis**: เรียก `this.analyze()` **ทันที** (ทุก Tick ที่มีการเปลี่ยนแปลง) เพื่อคำนวณ Indicator ใหม่

## 4. การคำนวณ SMC (SMC Calculation Core)

นี่คือขั้นตอนหลักที่ Boss, CHoCH, HH, LH ถูกสร้างขึ้น

*   **File**: `js/smc/DerivSMC.js` -> `js/smc/SMCIndicator.standalone.js` (Source: `SMCIndicator.js`)
*   **Caller**: `DerivSMC.prototype.analyze()`
    *   สร้าง instance ของ `SMCIndicator` พร้อม config
    *   เรียก `smcIndicator.calculate(this.ohlcData)`

### 4.1 ภายใน SMCIndicator.calculate()
*   **File**: `js/smc/SMCIndicator.js`
*   วน Loop ข้อมูลแท่งเทียนทั้งหมด (หรือ partial update) และเรียกฟังก์ชันย่อย:

#### A. การหาจุด Swing (HH, HL, LH, LL)
*   **Function**: `_processSwingPoints(index, ...)`
*   **Logic**: ตรวจสอบว่าเป็นจุดสูงสุด/ต่ำสุดในช่วง `swingLength` (default: 50) หรือไม่
*   **Storage (ตัวแปรที่เก็บ)**:
    *   เก็บใน Class Property: `this.swingPoints`
    *   Format: `{ time, price, type: 'HH'|'HL'|'LH'|'LL', swing: 'high'|'low' }`

#### B. การหาโครงสร้างราคา (BOS, CHoCH)
*   **Function**: `_processStructure(index, ...)`
*   **Logic**:
    *   ตรวจสอบราคาปิด (`close`) ว่าทะลุจุด Swing High/Low ล่าสุดหรือไม่
    *   ถ้าทะลุ Swing High เดิม (ในขาขึ้น) = **BOS** (Trend Continuation)
    *   ถ้าทะลุ Swing Low เดิม (ในขาขึ้น) = **CHoCH** (Trend Reversal)
*   **Storage (ตัวแปรที่เก็บ)**:
    *   เก็บใน Class Property: `this.structures`
    *   Format: `{ time, price, type: 'BOS'|'CHoCH', direction: 'bullish'|'bearish' }`

#### C. การหา Order Blocks (OB)
*   **Function**: `_storeOrderBlock(...)` และ `_checkOrderBlockMitigation(...)`
*   **Storage**: `this.orderBlocks`

#### D. การหา Fair Value Gaps (FVG)
*   **Function**: `_detectFVG(...)`
*   **Storage**: `this.fairValueGaps`

## 5. การรวบรวมผลลัพธ์ (Aggregating Results)

*   **File**: `js/smc/DerivSMC.js` (inside `analyze`)
*   **Function**: `smcIndicator.getAllResults()`
*   **Variable**: เก็บผลลัพธ์ทั้งหมดไว้ในตัวแปร local ชื่อ `results`
    *   `results.swingPoints` -> ข้อมูล HH, HL...
    *   `results.structures` -> ข้อมูล BOS, CHoCH
    *   `results.orderBlocks` -> ข้อมูล OB
    *   `results.fairValueGaps` -> ข้อมูล FVG

## 6. การวาดกราฟ (Rendering / Visualization)

*   **File**: `js/smc/DerivSMC.js`
*   **Function**: `DerivSMC.prototype.renderMarkers(results)`

### A. วาดจุด Swing และโครงสร้าง (Labels/Icons)
*   **Method**: `this.candleSeries.setMarkers(markers)`
*   แปลงข้อมูลจาก `results.swingPoints` และ `results.structures` เป็น format ของ Lightweight Charts:
    *   **HH/LH**: แสดงเป็นลูกศรลง สีแดง (หรือตาม theme) พร้อมข้อความ
    *   **HL/LL**: แสดงเป็นลูกศรขึ้น สีเขียว พร้อมข้อความ
    *   **BOS/CHoCH**: แสดงเป็นจุดกลมเล็กๆ หรือป้ายกำกับ

### B. วาดกล่อง (Order Blocks / FVG)
*   **Method**: `this.renderOrderBlocks()` และ `this.renderFVGs()`
*   **Logic**:
    *   สร้าง Custom Primitive (สี่เหลี่ยมระบายสี) ผ่าน `this.createBoxPrimitive(...)`
    *   สั่ง `this.candleSeries.attachPrimitive(primitive)` เพื่อแปะรูปลงบนกราฟ

## สรุปตัวแปรสำคัญ (Variable Mapping)

| ข้อมูล (Data) | ฟังก์ชันคำนวณ (Calculation) | ไฟล์ (File) | ตัวแปรที่เก็บ (Variable) |
| :--- | :--- | :--- | :--- |
| **OHLC Data** | `updateCandle` | `DerivSMC.js` | `this.ohlcData` (Array of objects) |
| **HH, HL, LH, LL** | `_processSwingPoints` | `SMCIndicator.js` | `this.swingPoints` (ใน Indicator instance) |
| **BOS, CHoCH** | `_processStructure` | `SMCIndicator.js` | `this.structures` (ใน Indicator instance) |
| **Order Blocks** | `_storeOrderBlock` | `SMCIndicator.js` | `this.orderBlocks` |
| **FVG** | `_detectFVG` | `SMCIndicator.js` | `this.fairValueGaps` |


## Additional: การเรียกใช้ SMC โดยไม่ต้องวาดกราฟ (Headless Usage)

หากต้องการคำนวณหาค่า Swing Points, BOS, CHoCH เพื่อนำไปใช้ใน logic อื่นในหน้า HTML หน้าอื่นๆ โดยไม่ต้องแสดงผลบนกราฟ สามารถทำได้ดังนี้:

### 1. Include Script
โหลดไฟล์ Standalone version เพียงไฟล์เดียว:
```html
<script src="js/smc/SMCIndicator.standalone.js"></script>
```

### 2. Usage Pattern
```javascript
// A. Prepare Data: ต้องมีข้อมูล OHLC เรียงจากอดีต -> ปัจจุบัน
const ohlcData = [
    { time: 1620000000, open: 1.2000, high: 1.2050, low: 1.1990, close: 1.2010 },
    // ...
];

// B. Initialize: สร้าง instance ของ Indicator
const smc = new SMCIndicator({
    swingLength: 50,    // ปรับความกว้างของ swing ตามต้องการ
    internalLength: 5
});

// C. Calculate: สั่งคำนวณ
smc.calculate(ohlcData);

// D. Get Results: ดึงค่าที่ต้องการ
// 1. ดึง Swing Points (HH, HL, LH, LL)
const swings = smc.getSwingPoints();
// ตัวอย่าง Output: [{ time: ..., price: 1.2050, type: 'HH', swing: 'high' }, ...]

// 2. ดึง Structure (BOS, CHoCH)
const structure = smc.getStructures();
// ตัวอย่าง Output: [{ time: ..., price: 1.2050, type: 'BOS', direction: 'bullish' }, ...]

// 3. ดึงผลลัพธ์ทั้งหมด
const allResults = smc.getAllResults();
```

### 3. Key Functions & Files

*   **File**: `js/smc/SMCIndicator.standalone.js`
*   **Class**: `SMCIndicator`
*   **Methods**:
    *   `.calculate(data)`: ประมวลผลข้อมูล
    *   `.getSwingPoints()`: ดึงจุด HH/HL/LH/LL
    *   `.getStructures()`: ดึงจุด BOS/CHoCH
    *   `.getOrderBlocks()`: ดึง Order Blocks
    *   `.getFairValueGaps()`: ดึง FVGs
