# Doubao WLED Light Recipe Skill

豆包 WLED 光配方技能 - 通过自然语言对话为 WLED 灯带生成动态灯光效果。

## 功能概述

对豆包说"我需要茉莉花盛开的灯光效果"，技能会自动：
1. 搜索茉莉花相关图片
2. 解析图片颜色分布
3. 生成自定义调色板上传到 WLED
4. 应用最佳匹配的灯光效果
5. 保存为 WLED 预设（循环播放直到更换）
6. 导出可分享的光配方 JSON 文件

## 安装方法

### 方法一：从 GitHub 仓库导入（推荐，手机版/桌面版通用）

仓库地址：`https://github.com/symi-daguo/doubao-wled-skill`

1. 打开豆包 App（手机版或桌面版）
2. 进入技能管理页面（通常在"我的"→"技能"或设置中）
3. 选择"从 GitHub 导入"或"添加技能"
4. 输入仓库地址：`https://github.com/symi-daguo/doubao-wled-skill`
5. 确认导入，豆包会自动读取根目录的 `SKILL.md` 加载技能
6. 技能名称显示为 `wled-light-recipe`，安装完成

### 方法二：本地导入

1. `git clone https://github.com/symi-daguo/doubao-wled-skill.git`
2. 打开豆包桌面端，切换到"办公任务"模式
3. 输入 `/创建技能` 或 `/install-skill`
4. 选择本地导入，指定 clone 下来的目录
5. 按提示完成安装

### 方法三：直接引用 Raw 链接

如果豆包支持 Raw 链接导入，可直接引用：
`https://raw.githubusercontent.com/symi-daguo/doubao-wled-skill/main/SKILL.md`

## 配置说明

安装后，编辑 `config.json` 配置你的 WLED 设备：

```json
{
  "wled": {
    "ip": "192.168.1.100",           // 你的 WLED IP（留空则自动发现）
    "mdns_name": "wled-xxxxxx",       // WLED 的 mDNS 名称
    "auto_discover": true             // 启用自动发现
  },
  "layout": {
    "type": "tv_backlight",           // 布局类型: tv_backlight/linear/ring/matrix
    "total_leds": 56,                 // 总灯珠数
    "edges": {                        // TV 背光四边灯珠数
      "top": 16, "bottom": 16, "left": 12, "right": 12
    },
    "start_corner": "top_left",       // 起始角落
    "direction": "cw"                 // 方向: cw(顺时针)/ccw(逆时针)
  }
}
```

## 使用示例

### 创建灯光效果

| 说什么 | 效果 |
|---|---|
| "我需要茉莉花盛开的灯光效果" | 搜索茉莉花图片，生成绿色白色光配方 |
| "给我一个日落的灯光场景" | 搜索日落图片，生成橙红色暖光配方 |
| "樱花飘舞的灯光" | 搜索樱花图片，生成粉色光配方 |
| "海洋深处的灯光" | 搜索海洋图片，生成蓝色冷光配方 |

### 管理光配方

| 说什么 | 效果 |
|---|---|
| "列出所有光配方" | 显示已保存的光配方列表 |
| "应用茉莉花配方" | 重新应用已保存的配方 |
| "关闭灯光" | 关闭 WLED 灯带 |
| "切换到预设 100" | 调用指定 ID 的预设 |

## 技术要求

- WLED 固件 0.14+（推荐 0.15+ 或 Nightly）
- Python 3.8+ with Pillow（`pip install Pillow`）
- 豆包专业版（支持代码执行技能）
- WLED 设备与豆包在同一局域网

## 目录结构

```
doubao-wled-skill/
├── SKILL.md                    # 技能核心指令（豆包读取）
├── config.json                 # 设备与布局配置
├── scripts/                    # Python 脚本
│   ├── discover_wled.py        # mDNS 自动发现
│   ├── analyze_image.py        # 图片颜色解析
│   ├── apply_recipe.py         # 应用光配方
│   ├── save_preset.py          # 保存 WLED 预设
│   ├── upload_palette.py       # 上传自定义调色板
│   ├── export_recipe.py        # 导出光配方 JSON
│   └── import_recipe.py        # 导入并应用光配方
├── references/                 # 参考文档
│   ├── wled_api.md             # WLED JSON API 参考
│   └── effects_palettes.md     # 效果与调色板清单
├── recipes/                    # 导出的光配方文件
└── assets/                     # 测试资源
```

## 光配方文件格式

导出的光配方为便携式 JSON，可在任意 WLED 设备导入：

```json
{
  "name": "茉莉花盛开",
  "colors": [[82,107,79], [105,145,92], [53,80,46]],
  "fx": 10, "sx": 174, "ix": 159, "bri": 169,
  "theme": "nature",
  "rationale": "Green/yellow tones - nature/forest effect"
}
```

## 常见问题

**Q: 手机版豆包能用吗？**
A: 可以。技能使用 Python 标准库 + Pillow，兼容手机版豆包代码执行环境。如果手机版无法执行 Python，豆包会用视觉能力直接分析图片并输出光配方 JSON。

**Q: 多个 WLED 设备怎么办？**
A: config.json 只配置一个主设备。如需控制多个，复制技能目录并修改 config.json 中的 IP。

**Q: 灯光效果会一直播放吗？**
A: 是的，效果会循环播放直到你更换其他配方或关闭灯光。保存为预设后，WLED 重启也会自动恢复。

**Q: 如何分享光配方给朋友？**
A: 将 `recipes/` 目录下的 JSON 文件发送给朋友，朋友导入到自己的技能目录即可使用。

## 版本

- v1.0.0 (2026-07-29): 初始版本，支持图片搜索、颜色解析、调色板上传、预设保存、光配方导入导出
