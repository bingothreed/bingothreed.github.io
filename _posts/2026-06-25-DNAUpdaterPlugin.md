---
layout: post
title: DNAUpdater — UE5.8 MetaHuman DNA 更新插件
date: 2026-06-25 19:00:00 +0800
categories: [Coding]
tags: [UE5, MetaHuman, DNA]
comments: true
math: true
image:
    path: ../assets/img/photos/DNAUpdaterTitle.png
---

## 1. 背景

Epic 在 UE5.6+ 中引入了新的 DNA 库 ([MetaHuman-DNA-Calibration](https://github.com/EpicGames/MetaHuman-DNA-Calibration) → `dna` 9.4.7)，并重构了 RigLogic 模块对 DNA 文件的加载方式。旧版 `ReadDNAFromFile` 被标记为 deprecated，新版使用 `UDNA` + `FDNAConfig::Legacy(Transform)` 来确保 RigLogic 的运行时行为一致性。

然而，UE5.8 内置的 DNA 导入流程只负责创建 `UDNA` 资产并附加到 SkeletalMesh — **它不会更新骨骼的 Reference Pose，也不会同时处理头部和身体的网格数据**。

DNAUpdater 插件填补了这一空白，提供一键式 DNA → SkeletalMesh 的完整更新：顶点位置、蒙皮权重、Morph Target、骨骼变换，全部自动完成。

## 2. 安装

插件位于 `E:\Code\UnrealEngine\Plugins\SourceCode\DNAUpdater`。

```
# 在你的 UE 项目中（或引擎目录）
mkdir -p Plugins/DNAUpdater
# 将 Source 目录复制过去
# 重新生成项目文件，编译
```

**依赖：**
- UE5.8 引擎（需要 RigLogic 模块）
- 运行时依赖 `RigLogicLib`、`dna`、`tdm` 等库，默认在 UE5.8 中已包含

插件模块类型为 `Editor`（仅在编辑器中可用，不打包到运行时）。

## 3. 已弃用的功能

以下菜单项已被 DNA 统一更新路径取代（代码保留在 `#if 0` 块中）：

- **Update height with FBX file** — 原用于从 FBX 更新身体骨骼
- **Update body base mesh** — 原用于从 OBJ 更新身体网格

## 4. 核心技术细节

### 4.1 双 Reader 策略

| Reader | 用途 | 配置 |
|---|---|---|
| `FLegacyDNAReader`（Geometry Reader） | 读取所有几何数据：顶点、法线、UV、权重、Morph Target | `FDNAConfig::Source()` (Ignore) |
| `FDNAReader`（UDNA 内部） | RigLogic 运行时评估 | `FDNAConfig::Legacy(Transform)` |

**为什么用两个 Reader？** `FLegacyDNAReader` 在 `GetVertexPosition` 中已经做了硬编码的 UE 坐标系变换，返回的顶点位置可以直接写入 `FSoftSkinVertex::Position`。而 `FDNAReader` 配合 `Source()` 配置不做变换，返回原始的 Maya 坐标系数据。

### 4.2 坐标系问题

DNA 文件使用 Maya 坐标系（Y-up, Z-forward），UE 使用 Z-up, Y-forward。UE5.8 的 `FDNAConfig::Legacy` 声明源坐标系 Y=down，导致 `Y → -Z` 的变换。

关键不对称性：`FLegacyDNAReader::GetVertexPosition` 内建了 UE 空间变换，但 `GetNeutralJointTranslation` **没有**。因此：

- **顶点**：直接用 Geometry Reader 的坐标，无需手动转换
- **关节旋转**：使用欧拉角顺序 `ZYX`，旋转符号 `Neg/Pos/Pos`（MetaHuman DNA 的实际编码格式），然后通过 `FQuat::MakeFromEuler({Z, X, Y})` 构建四元数
- **根关节**：DNA 中根关节的欧拉角为 `(0, 0, -90°)`，这是 Maya→UE 的坐标系伪影，必须跳过（`JtIdx = 1` 开始）

### 4.3 UpdateJoints 不调用

UE5.8 中，附加 `UDNA` 资产后 RigLogic 会**自动**在运行时驱动关节变换。Epic 官方的 `AssetDefinition_DNAAsset` 也**不**调用 `UpdateJoints`。

如果手动调用 `UpdateJoints`，会将 Maya 空间的关节变换写入 UE 空间的骨骼 Reference Pose，导致网格扭曲。

### 4.4 欧拉角 Swizzle

经过大量实验验证的正确转换：

```cpp
// DNA 存储的欧拉角顺序: ZYX
// DNA 存储的旋转符号: Neg/Pos/Pos (X Negative, Y Positive, Z Positive)
const FQuat Rot = FQuat::MakeFromEuler({
    ToDeg(Euler.Z),   // Z → Pitch（UE 的 Y）
    ToDeg(Euler.X),   // X → Roll  （UE 的 X）
    ToDeg(Euler.Y)    // Y → Yaw   （UE 的 Z）
});
```

**原则：原封不动使用 DNA 中的角度值，仅做顺序重排列（Swizzle），不做数值转换。**

### 4.5 CommitMeshDescription 的副作用

在 UE5.8 源码中，`FSkeletalMeshSourceModel::CommitMeshDescription` 只负责将 `MeshDescription` 序列化到 Bulk Data — **它不会重建 LODModel**。

LODModel 的重建发生在 `PostEditChange` → `FSkeletalMeshBuilder::Build`（异步）。因此：

- **DNA 更新路径**：使用 `CommitMeshDescription(false)` + `RebuildRenderData`，避免 `PostEditChange` 触发的 Build 覆盖手动写入的 `SoftVertices`
- **Blendshape 添加路径**：使用 `FScopedSkeletalMeshPostEditChange` 触发 Build Pipeline，自动将 `MeshDescription` 的 Morph 属性填充到 `UMorphTarget::LODModel`

## 5. 架构

```
DNAUpdater.cpp (~2900 lines, single file)
├── GetDNAMenu()              — 右键菜单构建
├── ExecuteDNAImport()        — 核心：DNA → Mesh 完整更新
├── ExecuteAddNewBlendShape() — OBJ → Morph Target
├── ApplyMorphTarget()        — Blendshape 实现
│
├── #if 0 (已弃用)
│   ├── ExecuteFBXImport()            — FBX 身体骨骼更新
│   ├── ExecuteBodyMeshImport()       — OBJ 身体网格更新
│   ├── ExecuteRefreshHeight()        — 高度刷洗
│   └── ApplyJointTransformsToSkeleton() — FBX 关节变换
│
├── #if 0 (诊断工具)
│   ├── ExecuteCompareDNAWithMesh()   — DNA vs 当前网格对比
│   ├── ExecuteExportDNAToOBJ()       — DNA/Mesh 导出 OBJ
│   └── ExecuteRuntimeOnlyDNAUpdate() — 仅运行时更新（二分法调试用）
│
└── 辅助函数
    ├── UpdateMeshDescriptionFromLODModel()
    ├── UpdateLOD0MeshDescriptionVertexPositions()
    ├── UpdateMorphTargetsInMeshDescription()
    ├── ReadDNAFromFile()            — 创建 FLegacyDNAReader
    ├── ValidateVertexCountForLOD0()
    ├── GetBaseVertices()
    ├── BuildMorphTargetDeltas()
    ├── BuildOBJToMeshVertexMap()
    └── ReadVertexFromFile()
```

## 6. 已知限制与注意事项

1. **仅支持 LOD0**：身体网格和 Blendshape 的更新目前只处理 LOD0
2. **MetaHuman 专用**：顶点映射和关节映射逻辑假定输入是 MetaHuman 管线生成的 DNA
3. **编辑器专用**：插件运行时不可用（`ModuleType = Editor`）
4. **DNA 文件格式**：仅支持 2.x 版本的 DNA 文件（与 UE5.8 兼容）

## 7. 调试

如需调试 DNA 解析是否正确，可以在 `DNAUpdater.cpp` 中取消 `#if 0` 诊断菜单的注释（将 `#if 0` 改为 `#if 1`），会显示三个额外菜单项：

- **Compare DNA with current mesh**：对比 DNA 顶点和当前网格顶点，输出匹配度统计
- **Export DNA + current mesh as OBJ**：导出 DNA 和当前网格的 OBJ 文件，可在 Blender 中对比
- **Test: runtime-only DNA update**：仅更新 LODModel，不持久化，用于二分法定位问题

## 8. 总结

DNAUpdater 是 UE5.8 中处理 MetaHuman DNA 文件的完整解决方案。它解决了官方导入流程中缺失的关键步骤（骨骼更新、身体网格更新），同时保留了与 RigLogic 运行时评估的完全兼容性。

核心设计原则：
- 几何数据使用 `FLegacyDNAReader`（已验证的 UE 空间坐标）
- 运行时行为使用 `FDNAReader::Legacy(Transform)`（RigLogic 兼容）
- 欧拉角不做数值变换，只做顺序重排列
- 骨骼更新跳过根关节，避免 Maya→UE 坐标系伪影
