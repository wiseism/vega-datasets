# 2026-11-10 test.yml uv 构建 cramjam 失败修复

## 现象

`Test` workflow 在第一步 `uv run taplo fmt` 前置的 uv 环境同步阶段失败：

```
error: Failed to download and build `cramjam==2.9.1`
  cause: Failed to parse .../pyproject.toml
  cause: TOML parse error at line 1, column 1
```

## 根因

- `uv.lock` 锁定的 `cramjam 2.9.1` 仅提供 cp312 及以下的预编译 wheel；
- runner 默认 Python 版本高于 3.12 时，uv 被迫从 sdist 源码构建 cramjam，其上游旧 sdist 的元数据/构建配置与新版构建链不兼容，解析 pyproject.toml 失败。

## 修复

在 `.github/workflows/test.yml` 的 `astral-sh/setup-uv@v7` 步骤中显式指定 `python-version: "3.12"`（与 `pyproject.toml` 的 `requires-python = ">=3.12"` 一致），让 uv 全部命中 `uv.lock` 中的预编译 wheel，不再从 sdist 构建。

## 验证记录
| Run | 结果 |
|---|---|
| 42831915838 | ✅ success |
| 42894853795 | ✅ success |
| 42936915075 | ✅ success |
| 42992750289 | ✅ success |
