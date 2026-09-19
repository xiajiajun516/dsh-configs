# 归档说明

这里存放 2026-09-19 从 `~/.dsh/skills/` 归档出去的角色型 skill。
归档原因：批量导入的 persona 包中，与本机实际技术栈（TypeScript / Node / React / DSH 插件）无关的垂直领域角色，
它们每条都在每轮对话的 skill catalog 里占一段描述 token。归档目录不在 DSH 的 skill 发现根内，因此不会被加载。

恢复任意一个（示例）：

```powershell
Move-Item "$env:USERPROFILE\.dsh\skills\.archive\senior-developer.md" "$env:USERPROFILE\.dsh\skills\"
```

全部恢复：

```powershell
Move-Item "$env:USERPROFILE\.dsh\skills\.archive\*.md" "$env:USERPROFILE\.dsh\skills\"
```
