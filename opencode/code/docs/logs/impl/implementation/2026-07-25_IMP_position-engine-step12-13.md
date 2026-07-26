---
agent: IMP
task_id: TASK-position-engine-step12-13
date: 2026-07-25
status: draft
category: log
destination: docs/logs/impl/implementation/
related:
  - "[TASK-position-engine-step12-13](../shared/tasks/active/TASK-position-engine-step12-13.md)"
  - "[Step 12 CollectionEventEngine]"
  - "[Step 13 CollectionSessionManager]"
tags:
  - IMP
  - implementation
  - position-engine
  - step12
  - step13
---

# Implementation Log: Step 12 + Step 13 — CollectionEventEngine + CollectionSessionManager

## Summary

Step 12（CollectionEventEngine）と Step 13（CollectionSessionManager）を実装した。
また、CollectionSessionManager からの呼び出しに必要な `completeSession` メソッドを SessionRepository に追加した。

## Changes

### 新規作成ファイル（3ファイル）

| # | ファイル | パッケージ | 説明 |
|---|----------|-----------|------|
| 1 | `event/CollectionEventEngine.kt` | `com.htlost5.frontieratlas.position.event` | 収集イベント管理エンジン + CollectionEvent sealed class |
| 2 | `collection/SessionStateMachine.kt` | `com.htlost5.frontieratlas.position.collection` | 収集セッション状態遷移マシン |
| 3 | `collection/CollectionSessionManager.kt` | `com.htlost5.frontieratlas.position.collection` | 収集セッションライフサイクル管理 |

### 変更ファイル（1ファイル）

| ファイル | 変更内容 |
|----------|----------|
| `storage/repository/SessionRepository.kt` | `completeSession()` メソッドを追加（endTime, status=COMPLETED, counts の一括更新） |

## 実装詳細

### 1. CollectionEventEngine

- **sealed class `CollectionEvent`**: 11種のイベント種別（SessionStarted/Paused/Resumed/Completed, CheckpointReached, SampleRecorded, OffGraphWarning, ExportProgress/Completed, Error）
- **`CollectionEventEngine`**: CopyOnWriteArrayList によるスレッドセーフなリスナー管理
- **抑制ルール**:
  - `SampleRecorded`: `enableSampleRecordedEvent` フラグで制御（デフォルト false）
  - `OffGraphWarning`: 前回発火から 3 秒以内は抑制
- 各 fire メソッドで android.util.Log 出力

### 2. SessionStateMachine

- **State**: NOT_INITIALIZED → INITIALIZED → RUNNING ⇄ PAUSED → STOPPED
- **遷移ルール**: タスク仕様の全遷移 + 禁止遷移を実装
  - `transitionTo()`: 不正遷移時は `IllegalStateException` をスロー
  - `canTransitionTo()`: 遷移可否判定

### 3. CollectionSessionManager

- **startCollection**: StateMachine 遷移 → CollectionSession 作成 → Repository.insert → Event 発火
  - DeviceInfo: `Build.MODEL`, `Build.MANUFACTURER`, `Build.VERSION.RELEASE`
  - NOT_INITIALIZED からの場合は INITIALIZED を経由
- **pauseCollection**: RUNNING → PAUSED → Repository.updateStatus → Event 発火
- **resumeCollection**: PAUSED → RUNNING → Repository.updateStatus → Event 発火
- **stopCollection**: RUNNING/PAUSED → STOPPED → Repository.completeSession → Event 発火
- 全メソッドは `Result<T>` を返し、StateMachine の不正遷移や DB エラーを捕捉
- StateMachine の遷移が先、DB 保存が後（StateMachine 失敗時は DB 操作を行わない）

## Build Result

```
./gradlew :position-engine:compileDebugKotlin → BUILD SUCCESSFUL in 5s
./gradlew ktlintCheck → BUILD SUCCESSFUL in 3s
```

コンパイルエラー・lint エラーなし。

## Handoff

**status**: 成功
**confidence**: high

実装完了。REV にレビューを依頼する。

**変更ファイル一覧（全4ファイル）**:

| ファイル | 変更種別 |
|----------|----------|
| `.../position/event/CollectionEventEngine.kt` | NEW |
| `.../position/collection/SessionStateMachine.kt` | NEW |
| `.../position/collection/CollectionSessionManager.kt` | NEW |
| `.../position/storage/repository/SessionRepository.kt` | MODIFIED（completeSession追加） |
