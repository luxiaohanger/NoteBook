## Stub 与 Driver

集成测试中，如果被测模块依赖的其他模块还没完成，需要用测试替身。

| 名称 | 作用 | 常见场景 |
|---|---|---|
| Stub | 被调用模块的替身 | 自顶向下集成时，下层未完成 |
| Driver | 调用被测模块的替身 | 自底向上集成时，上层未完成 |

例子：

- 测 Controller 时 Service 未完成，可以写 Service Stub。
- 测 Dao 时没有 UI 或 Controller，可以写 Driver 调用 Dao。

记忆：

> Stub 装作“下面的人”，Driver 装作“上面的人”。


