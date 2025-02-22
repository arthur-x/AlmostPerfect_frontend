# AlmostPerfect_frontend

This is a WebApp for the AlmostPerfect project, mainly used to collect human rating data.

这是一个适用于AlmostPerfect项目的网页应用，主要用于收集人类打分数据。

**Data Security:** All operations are done locally in the browser, no data will be transmitted through the internet.

**数据安全:** 所有操作仅在浏览器本地进行，不上传任何数据。

## Features / 功能特点

### Core Functions / 核心功能
- **📁 Folder Upload** - Directly upload entire image folders with directory selection  
  **文件夹上传** - 直接通过目录选择上传整个图片文件夹
- **⭐ 5-Level Rating** - Score images with ±1 increments (-5 to +5)  
  **五级评分** - 以±1为步长进行评分（-5 至 +5）
- **🔍 Fullscreen Preview** - High-resolution image viewing with keyboard navigation  
  **全屏预览** - 支持键盘操作的高清图片查看
- **📊 Real-time Filter** - Show only unrated images with one click  
  **实时过滤** - 一键切换仅显示未评分图片
- **💾 Data Export** - Download ratings as JSON file  
  **数据导出** - 将评分数据导出为JSON文件
- **🔄 Auto Loading** - Automatically load existing score files  
  **自动加载** - 自动识别并加载已有评分文件

## Quick Start / 快速开始

### Online Demo / 在线演示
[Live Demo](https://arthur-x.github.io/AlmostPerfect)

### Local Run / 本地运行
```bash
# 1. Clone repository / 克隆仓库
git clone https://github.com/arthur-x/AlmostPerfect_frontend.git

# 2. Install dependencies / 安装依赖
npm install

# 3. Start dev server / 启动开发服务器
npm run dev

# 4. Open in browser / 浏览器访问
http://localhost:5173
```

## How To Use / 如何使用

### Basic Workflow / 基本流程
1. **Upload Images**  
   **上传图片**
   - Click the folder icon → Select image folder  
     点击文件夹图标 → 选择图片文件夹

2. **Rating Operations**  
   **评分操作**
   - `↑/↓` Arrow keys to adjust score in fullscreen mode  
     全屏模式下使用`↑/↓`方向键调整分数
   - Click 👍/👎 buttons for quick rating  
     点击👍/👎按钮快速评分
   - Score range: -5 (Worst) to +5 (Best)  
     评分范围：-5（最差）到 +5（最佳）

3. **Download Scores**  
   **下载评分**
   - Export: Click download button → Get scores.json  
     导出：点击下载按钮 → 获取scores.json

### Hotkeys / 快捷键
| Key         | Function       | 功能          |
|-------------|----------------|---------------|
| ← →         | Navigate       | 图片切换      |
| ↑ ↓         | Adjust Score   | 调整评分      |
| ESC         | Exit Fullscreen| 退出全屏      |

## Notes / 注意事项

### Best Practices / 最佳实践
- Keep original file names unchanged  
   保持原始文件名不变
- Regular export during long sessions  
   长时间标注时定期导出数据
- Store images and JSON in same folder  
   将图片与JSON保存在同一目录

> [!Important]
>
> To save the rating for an image, enter fullscreen mode of that image and navigate past it (either hit ← or →). A default rating of 0 will be saved if you did't hit ↑ or ↓ (👍/👎). This is an intentional design.
> 
> 保存一张图片评分的唯一方法是在这张图片的全屏模式下切换图片（按下 ← 或 →）。如果没有主动按下过 ↑ 或 ↓ （👍/👎），那么将默认保存0的评分。设计如此。

## Credit / 致谢

DeepSeek-R1