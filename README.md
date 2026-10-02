<div align="center">

# AI 照片智能修复 · Photo Restore Agent

**以「AI 照片智能修复」为切入点的家庭影像数字资产平台。**

老照片修复 · 黑白上色 · AI 背景替换 · 照片生成动态视频 —— 让泛黄的记忆重新清晰、有了颜色，还能对着你微笑。

![Status](https://img.shields.io/badge/status-active-brightgreen)
![Vue](https://img.shields.io/badge/Vue-3-42b883?logo=vuedotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178c6?logo=typescript&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.2-6DB33F?logo=springboot&logoColor=white)
![Java](https://img.shields.io/badge/Java-17-orange?logo=openjdk&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.10-3776AB?logo=python&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-blueviolet)

[功能特性](#功能特性) · [系统架构](#系统架构) · [快速开始](#快速开始) · [技术栈](#技术栈) · [路线图](#路线图) · [文档](#文档) · [许可与合规](#许可与合规)

</div>

---

## 简介

**AI 照片智能修复（Photo Restore Agent）** 是一套可落地、可部署的家庭影像处理平台：先把老照片修好、上色、换背景、动起来，再把家庭影像沉淀为可传承的数字资产。

平台提供四大核心能力 + 家庭影像库、AI 故事生成等增值能力，支持批量任务、进度回调与失败重试。

## 功能特性

| 能力 | 标识 | 输入 | 输出 | 核心技术 |
| --- | --- | --- | --- | --- |
| 老照片修复 | `restore` | 模糊 / 破损 / 低清老照片 | 高清清晰的人像照片 | GFPGAN + CodeFormer + Real-ESRGAN |
| 黑白照片彩色化 | `colorize` | 黑白 / 褪色照片 | 自然彩色照片（自然 / 复古 / 鲜艳） | DDColor + DeOldify |
| AI 背景替换 | `bg_replace` | 原图 + 背景图或提示词 | 保留主体、替换背景的成片 | SAM2 + BiRefNet + 融合补光 |
| 照片生成动态视频 | `animate` | 单张人像 + 驱动模板 | 3~15 秒动态视频 | LivePortrait + RIFE + FFmpeg |
| 家庭影像库 | `album` | 相册分组 / 时间轴 | 家庭影像归档 | 后端 + 前端 |
| AI 故事生成 | `story` | 影像 + 主题 | 家庭故事文案 / 旁白 | 大模型 API（可选增强） |

- **人脸专门修，不"塑料"**：人脸与背景分开修复再回贴，保留原始纹理，保真度三档可选；
- **换背景不留破绽**：边缘羽化 + 发丝级抠图 + 色温 / 亮度匹配 + 接地阴影；
- **让照片"活"过来**：无需驱动视频，选择模板即可生成微笑 / 点头 / 眨眼 / 挥手 / 说话；
- **桩模式可跑通全链路**：无 GPU 时以占位结果跑通流程，便于开发、联调与 CI。

## 系统架构

```text
Web（Vue3 + TS + Vite + Pinia + Element Plus）
        │ HTTPS / REST
        ▼
后端 Spring Boot 3.2（8080，context-path /api）── 鉴权 / 上传 / 任务 / 记录 / 影像库
        │ AMQP 投递              ▲ HTTP 回调（X-Internal-Token）
        ▼                        │
RabbitMQ  ai.photo.task.queue  ─┤  AI 服务 FastAPI（8000，/internal）
（死信 ai.photo.task.dlq）      │  预处理 → 修复/超分 → 上色 → 抠图替换 → 视频生成 → FFmpeg
                                 │
        MySQL 8（8 张表） · Redis 7 · MinIO（原图 / 成品）
```

## 快速开始

### 一键启动

```bash
cp .env.example .env          # Windows: Copy-Item .env.example .env
docker compose --env-file .env up -d --build
docker compose ps
```

| 服务 | 地址 |
| --- | --- |
| 前端平台 | http://localhost |
| 后端 API | http://localhost:8080/api |
| AI 服务内部文档 | http://localhost:8000/docs（仅内网） |
| RabbitMQ 管理台 | http://localhost:15672 |
| MinIO 控制台 | http://localhost:9001 |

### 端到端自检（无 GPU 亦可）

```bash
python -B tests/e2e/e2e_full_flow.py --base-url http://localhost:8080/api --stub
python -B tests/load/queue_backlog_test.py --dry-run
```

### 桩模式与真实 GPU 模式

| `AI_STUB_MODE` | 行为 | 适用场景 |
| --- | --- | --- |
| `true`（默认） | 不加载 torch / 真实模型，按管线跑通并产出占位结果 | 无 GPU 的开发联调、CI |
| `false` | 加载真实模型推理 | 有 GPU 的开发 / 生产环境 |

## 技术栈

| 层级 | 技术 |
| --- | --- |
| 前端 | Vue 3 · TypeScript · Vite · Pinia · Element Plus |
| 后端 | Spring Boot 3.2 · Spring Security(JWT) · MyBatis-Plus |
| AI 服务 | Python · FastAPI · RabbitMQ Worker · FFmpeg |
| 模型 | GFPGAN / CodeFormer / Real-ESRGAN / DDColor / DeOldify / SAM2 / BiRefNet / LivePortrait |
| 基础设施 | MySQL 8 · Redis 7 · MinIO · RabbitMQ · Docker Compose · Nginx |

## 验收基线

- 单张修复 ≤ 15 秒、上色 ≤ 10 秒、背景替换 ≤ 20 秒（RTX 4090，输入 ≤ 2000×2000）
- 10 秒 1080P 动态视频 ≤ 5 分钟
- 人脸一致性：全程无闪烁、无变形，相似度 ≥ 0.85
- 批量任务：单次 100 张稳定消费；业务接口 P95 < 500ms（100 并发）

## 路线图

- [x] 阶段 0~2：工程骨架、编排、测试素材库与四条图像能力管线
- [x] 阶段 3~4：动态视频引擎、FastAPI 推理服务与任务队列（重试 / DLQ / 并发控制）
- [x] 阶段 5~6：后端业务系统（8 张表）与前端平台（14 个页面）
- [x] 阶段 7~8：优化与压测脚本、Docker 编排、Nginx / 监控 / 备份
- [ ] GPU 实测：模型权重下载、单图验证、性能基准回填
- [ ] 批量队列压测与 24h 并发稳定性测试
- [ ] 灰度上线（内测 → 公测 → 全量）

## 文档

| 文档 | 说明 |
| --- | --- |
| [docs/01-系统架构设计.md](01-系统架构设计.md) | 分层架构、模块职责、能力标识与状态机 |
| [docs/02-对外API文档.md](02-对外API文档.md) | 对外 HTTP 接口全量说明（入参 / 返回 / 示例 / 错误码） |
| [docs/03-环境搭建说明.md](03-环境搭建说明.md) | 开发机与生产两套环境搭建 |
| [docs/04-部署上线手册.md](04-部署上线手册.md) | 生产部署、HTTPS、灰度上线、回滚 |
| [docs/05-模型部署与性能基准.md](05-模型部署与性能基准.md) | 模型部署方式、显存参考、性能基准表 |
| [docs/06-用户手册.md](06-用户手册.md) | 面向最终用户的逐能力操作说明 |
| [docs/07-内部接口与队列协议.md](07-内部接口与队列协议.md) | 后端 ↔ AI 服务内部接口与队列协议 |
| [docs/08-实施进度跟踪.md](08-实施进度跟踪.md) | 阶段逐任务明细与完成度 |
| [docs/09-优化与压测报告.md](09-优化与压测报告.md) | 优化手段、压测方法与结果 |
| [docs/宣传文案.md](宣传文案.md) | 面向公众号 / 社媒的中文宣传稿 |

## 许可与合规

- 本仓库**暂未添加开源许可证（LICENSE）**；对外开源 / 商用前请先确定本项目自身许可。
- 项目依赖 GFPGAN、CodeFormer、Real-ESRGAN、DDColor、SAM 2、BiRefNet、LivePortrait 等第三方模型，许可证各自独立，**上线前请逐一核查商用可行性**。
- 涉及真人肖像与家庭隐私影像，建议启用私有存储、限时访问与一键删除，并在用户协议中明确授权范围。

## 贡献

欢迎提交 Issue 与 Pull Request。分支规范 `main` / `dev` / `feat-xxx`，提交前缀 `feat: / fix: / docs: / chore:`。

<div align="center">

**如果这个项目对你有帮助，欢迎点一个 Star 支持我们。**

</div>

---

## 🏢 关于我们

<p align="center">
  <a href="http://www.net188.net">
    <img src="http://www.net188.net/images/logo1.png" alt="Net188 Logo" width="200" />
  </a>
</p>

<p align="center">
  <strong>Net188 · 互联网技术服务</strong>
</p>

<p align="center">
  专注于跨平台应用开发、AI Agent 集成与大模型应用落地。<br/>
  提供从产品设计、开发实施到部署运维的全栈技术解决方案。
</p>

<p align="center">
  🌐 <a href="http://www.net188.net"><strong>www.net188.net</strong></a>
</p>

---
