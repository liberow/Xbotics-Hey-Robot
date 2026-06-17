# Hey Robot

Hey Robot 是一个面向真实机器人部署的 embodied agent runtime。当前主线目标是
XLeRobot：由 SO101 arm、LeKiwi base、camera observation 和 battery/status
monitoring 组合成一个可被 Agent 调度的机器人系统。

这个项目不是简单的 LLM tool-calling demo。它的核心是把 Agent、Skill、Robot、
Camera、VLA 这些边界拆清楚：Agent 负责理解任务和选择 skill；Skill Runtime
负责合约、资源、readiness gate 和超时；Robot Driver 只负责真实硬件执行；VLA
这类长时间运行的模型能力通过独立 capability service 暴露给 skill 调用。

## Runtime Shape

```text
User Channel
  -> GatewayService
  -> RobotAgentService
      -> RobotAgentLoop
      -> TaskRunManager
      -> SceneRuntime
      -> RobotAgentCore
          -> AgentRuntime
          -> request_capability / request_perception
          -> SkillGateway
          -> SkillIntent
  -> SkillControllerService
      -> SkillContractRuntime
      -> CapabilityRuntime
  -> RobotService / RobotRuntime
      -> XLeRobotDriver / SO101Driver / LeKiwiDriver / MockRobotDriver
  -> CameraPublisherService
      -> camera.observation consumers
```

## 核心原则

- `Robot` 只表示身体和硬件执行边界。
- `Skill` 是 Agent 调用机器人能力的统一入口。
- `CameraPublisherService` 只发布 observation，多个 consumer 可以按需读取。
- `VLA` 不属于 robot driver，而是独立的 `capability_service`。
- Agent 通过 `request_capability` 调用机器人 skill；当前 XLeRobot 生产配置启用 11 个非 VLA skill。
- `vla_manipulation` 已注册但暂不启用，避免在 VLA 未稳定前进入 Agent 可调用面。
- Task Cockpit 在 `/cockpit`，展示 task state、timeline、scene evidence、recovery。
- 当前运行环境固定为 Python 3.12。

## 安装

```bash
uv sync --dev
```

## 常用验证

```bash
uv run ruff check src tests
uv run pytest -q --no-cov
```

XLeRobot Windows 硬件检查：

```powershell
uv run python scripts\ops\platform_doctor.py --config configs\xlerobot.real.windows.yaml
uv run hey-robot inspect --config configs\xlerobot.real.windows.yaml
uv run python scripts\robots\xlerobot\diagnose.py --config configs\xlerobot.real.windows.yaml
```

启动完整 runtime：

```powershell
uv run hey-robot run --config configs\xlerobot.real.windows.yaml
```
