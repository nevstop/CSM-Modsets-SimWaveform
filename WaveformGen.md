# `WaveformGen` — CSM 模块接口文档

> ⚠️ **AI 自动生成，还未经过人工审阅**
>
> 本文档由 AI 基于代码仓库结构和脚本上下文自动生成，可能包含不准确或遗漏的信息。请在合并前进行人工核查。

---

## 功能简述

`WaveformGen` 是一个 CSM 模块，用于**模拟生成波形数据**，替代真实硬件采集设备为下游模块提供测试波形信号。

该模块可以配置波形参数（采样率、波形类型、幅值、频率等），并以广播方式将生成的波形数据推送给订阅者。主要用于开发、测试环境中没有实际采集硬件时的替代方案。

---

## 模块信息

| 属性           | 值                                              |
| -------------- | ----------------------------------------------- |
| LabVIEW 版本   | ≥ 2020                                          |
| 支持的操作系统 | Windows                                         |
| 支持 RT        | ❌ 不支持                                       |
| 支持 64-bit    | ✅ 支持                                         |
| 所属模块组     | `CSM-SimWaveform.lvlib`                         |

---

## 依赖项

| 依赖                                                                                                | 类型 |
| --------------------------------------------------------------------------------------------------- | ---- |
| [Communicable-State-Machine](https://github.com/NEVSTOP-LAB/Communicable-State-Machine)             | 必须 |
| [CSM-MassData-Parameter-Support](https://github.com/NEVSTOP-LAB/CSM-MassData-Parameter-Support)     | 可选 |
| [CSM-API-String-Arguments-Support](https://github.com/NEVSTOP-LAB/CSM-API-String-Arguments-Support) | 可选 |
| [CSM-INI-Static-Variable-Support](https://github.com/NEVSTOP-LAB/CSM-INI-Static-Variable-Support)   | 可选 |

---

## API 接口（消息接口）

以下是外部调用者可以发送给本模块的消息。

### `API: Initialize`

初始化内部资源和波形生成参数。

- **参数**：`APIString` — `String`：配置文件路径
- **响应**：N/A

### `API: Start`

启动波形生成。

- **参数**：N/A
- **响应**：N/A

### `API: Stop`

停止波形生成。

- **参数**：N/A
- **响应**：N/A

### `API: Get Status`

查询当前波形生成状态。

- **参数**：N/A
- **响应**：`APIString` — `String`：状态描述（如 `Running`、`Stopped`）

### 参数类型说明

| 类型        | 说明                                                                                              |
| ----------- | ------------------------------------------------------------------------------------------------- |
| `HexStr`    | 将 LabVIEW Variant 序列化为十六进制字符串，内置支持                                               |
| `ErrStr`    | 将错误信息编码为字符串，内置支持                                                                  |
| `APIString` | 支持嵌套键值对的纯文本字符串，需要 CSM API String Arguments Support 插件                          |
| `MassData`  | 内存映射缓冲区，传递 `Start:N,Size:M`，需要 CSM MassData Parameter Support 插件                   |

> **注意**：接口文档中对 `String` 类型数据统一使用 `APIString` 标注。

---

## 状态广播接口

以下是本模块**发出**的消息，用于通知订阅者内部状态变化。

### `Acquired Waveform`

**默认广播类型**：`Status`

波形数据已生成并就绪，通知订阅者获取。

- **参数**：`MassData` — `Waveform[]`：生成的一维波形数据数组

---

## 属性接口

本模块暂无对外暴露的属性。

---

## 配置说明

### 前面板参数

| 控件名称       | 类型   | 默认值 | 说明               |
| -------------- | ------ | ------ | ------------------ |
| `WaveformType` | Enum   | Sine   | 波形类型（正弦/方波/三角波等） |
| `Amplitude`    | DBL    | 1.0    | 波形幅值           |
| `Frequency`    | DBL    | 10.0   | 波形频率（Hz）     |
| `SampleRate`   | DBL    | 1000.0 | 采样率（Hz）       |
| `NumSamples`   | I32    | 1000   | 每次生成的采样点数 |

> 以上参数通过 `WfGenConf--Cluster.ctl` 配置文件管理。

---

## 调用限制与注意事项

> [!IMPORTANT]
>
> - `API: Initialize` **必须**在其他任何 API 之前调用。
> - 本模块为**单例**——同一时间不可运行多个实例。
> - 波形数据通过 `MassData` 机制传递，订阅方需安装 CSM MassData Parameter Support 插件。

---

## 使用示例

> 将 `WaveformGen` 替换为启动模块 VI 时实际使用的名称。

### 基本生命周期

```csm
// 初始化并启动模块
API: Initialize >> c:\config\wfgen.ini -@ WaveformGen
API: Start -@ WaveformGen

// 订阅波形数据
Acquired Waveform@WaveformGen >> API: Update Waveforms@tagRouter -><register>

// 停止模块
API: Stop -@ WaveformGen
```

---

## 备注

- 本模块使用模拟数据替代实际硬件采集，方便开发测试。
- 波形参数可通过 INI 配置文件或前面板设置。
- 生成的波形数据通过 `MassData` 内存映射缓冲区高效传递，避免大数组拷贝开销。

---

- _完整 CSM 语法参考：<https://github.com/NEVSTOP-LAB/Communicable-State-Machine/blob/main/.doc/Syntax.md>_
- _CSM Wiki：<https://nevstop-lab.github.io/CSM-Wiki/>_
