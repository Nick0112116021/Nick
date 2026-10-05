# STM32 有限狀態機 (FSM) 與中斷控制系統 README

本專案基於 STM32 HAL 庫實作一個**三階段有限狀態機（Finite State Machine, FSM）**系統，透過**輪詢（Polling）**與**外部中斷（EXTI）**混合觸發機制，控制三色 LED 燈的警示狀態與閃爍頻率。

---

## 📖 目錄 (Table of Contents)

- [1. 系統狀態與轉移 (FSM State Machine)](#1-系統狀態與轉移-fsm-state-machine)
- [2. 中斷與硬體腳位對照 (Hardware & Interrupt Structure)](#2-中斷與硬體腳位對照-hardware--interrupt-structure)
- [3. 軟體架構與事件處理機制 (Software Architecture)](#3-軟體架構與事件處理機制-software-architecture)
- [4. 程式腳本說明 (Code Summary)](#4-程式腳本說明-code-summary)

---

## 1. 系統狀態與轉移 (FSM State Machine)

系統定義了三個主要工作狀態：

| 狀態 (State) | 代表數值 | LED 輸出行為 | 說明 |
| :--- | :---: | :--- | :--- |
| **NORMAL** | `0` | 綠燈 (PC8) **1 Hz 閃爍** (500ms ON / 500ms OFF) | 系統正常運作模式 |
| **WARNING** | `1` | 黃燈 (PC5) **2 Hz 閃爍** (250ms ON / 250ms OFF) | 系統警告模式 |
| **EMERGENCY** | `2` | 紅燈 (PC6) **常亮 (Always ON)** | 緊急狀態，無法透過 MODE 按鍵切換回其他狀態 |

### 狀態轉移圖 (State Transitions)

```text
       [ 開機 Initial ]
              │
              ▼
    ┌──────────────────┐
    │   STATE_NORMAL   │◄───┐
    └────────┬─────────┘    │
             │              │ MODE 按鍵 (PC9 Polling)
MODE 按鍵    │              │
(PC9 Polling)│              │
             ▼              │
    ┌──────────────────┐    │
    │  STATE_WARNING   ├────┘
    └────────┬─────────┘
             │
             │ STOP 按鍵 (PC13 EXTI 中斷)
             │ 任意狀態皆可觸發
             ▼
    ┌──────────────────┐
    │ STATE_EMERGENCY  │ (MODE 按鍵在此狀態下無效)
    └──────────────────┘
