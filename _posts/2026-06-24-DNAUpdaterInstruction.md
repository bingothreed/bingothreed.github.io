---
layout: post
title: DNAUpdater — MetaHuman DNA Update Tool for UE5.8
date: 2026-06-24 19:30:00 +0800
categories: [Coding]
tags: [UE5, MetaHuman, DNA]
comments: true
math: false
image:
    path: ../assets/img/photos/DNAUpdaterTitle.png
---

<style>
.lang-switch { display: inline-flex; border: 1px solid #ccc; border-radius: 4px; overflow: hidden; margin-bottom: 1em; }
.lang-switch button { border: none; padding: 4px 14px; cursor: pointer; background: #f5f5f5; font-size: 13px; }
.lang-switch button.active { background: #007bff; color: #fff; }
.lang-zh { display: none; }
</style>

<div class="lang-switch">
  <button id="btn-en" class="active" onclick="document.getElementById('lang-en').style.display='block';document.getElementById('lang-zh').style.display='none';document.getElementById('btn-en').classList.add('active');document.getElementById('btn-zh').classList.remove('active');">English</button>
  <button id="btn-zh" onclick="document.getElementById('lang-zh').style.display='block';document.getElementById('lang-en').style.display='none';document.getElementById('btn-zh').classList.add('active');document.getElementById('btn-en').classList.remove('active');">中文</button>
</div>

<div id="lang-en">

<h2>1. Background</h2>

<p>UE5.8's built-in DNA import only creates a <code>UDNA</code> asset for RigLogic runtime evaluation — <strong>it does not update the skeletal mesh's Reference Pose, vertex positions, skin weights, or Morph Targets</strong>.</p>

<p>This means: if you modify a MetaHuman's body proportions or joint positions in Maya, export a new DNA file, and reimport it in UE, <strong>nothing changes on the mesh</strong>.</p>

<p><strong>DNAUpdater</strong> fixes this. It reads everything from the DNA file — vertices, skin weights, BlendShapes, joints — and applies it all to your selected SkeletalMesh in one click.</p>

<h2>2. Usage</h2>

<p>In the Content Browser, <strong>right-click any SkeletalMesh</strong> (MetaHuman body or head) and navigate to <strong>DNA Actions</strong>:</p>

<h3>2.1 Update mesh with DNA file</h3>

<p><strong>This is the main feature.</strong> Select a <code>.dna</code> file and the plugin automatically:</p>

<ul>
  <li>Updates neutral-pose vertex positions</li>
  <li>Updates all Morph Targets (BlendShapes)</li>
  <li>Updates eye-region skin weights</li>
  <li>Updates skeleton Reference Pose (root joint is skipped automatically)</li>
  <li>Creates/updates the <code>UDNA</code> asset for RigLogic compatibility</li>
</ul>

<p><strong>One click. Head and body, all at once.</strong></p>

<h3>2.2 Add a blendshape</h3>

<p>Add a new Morph Target from an OBJ file. OBJ is expected in Maya coordinate space (Y-up); the plugin converts to UE space automatically.</p>

<p>Use case: sculpted a new facial expression in Maya, export as OBJ, add it to the existing mesh.</p>

<h2>3. Notes</h2>

<ul>
  <li>LOD0 only</li>
  <li>MetaHuman DNA files only</li>
  <li>Editor-only (not packaged at runtime)</li>
</ul>

<h2>4. Typical Workflow</h2>

<pre><code>Modify MetaHuman in Maya (shape, skeleton, expressions)
       ↓
Export new .dna file
       ↓
UE5.8 Content Browser → Right-click SkeletalMesh → DNA Actions → Update mesh with DNA file
       ↓
Choose .dna file → Done
       ↓
Save asset. RigLogic animations adapt to the updated mesh automatically.
</code></pre>

</div>

<div id="lang-zh">

<h2>1. 背景</h2>

<p>UE5.8 中，MetaHuman 的 DNA 文件导入后，官方流程只负责创建 <code>UDNA</code> 资产供 RigLogic 运行时使用——<strong>它不会更新骨骼的 Reference Pose，也不会把 DNA 里的顶点数据、蒙皮权重、Morph Target 写入网格</strong>。</p>

<p>这意味着：如果你在 Maya 中修改了 MetaHuman 的身体比例或骨骼位置，导出 DNA 后在 UE 中重新导入，<strong>模型不会发生变化</strong>。</p>

<p><strong>DNAUpdater</strong> 就是解决这个问题的。它把 DNA 文件里的所有数据——顶点、蒙皮、BlendShape、骨骼——一键更新到你选中的 SkeletalMesh 上，让 Maya 中的修改真正反映到 UE 里。</p>

<h2>2. 使用</h2>

<p>在 Content Browser 中，<strong>右键点击任意 SkeletalMesh 资产</strong>（MetaHuman 身体或头部），找到 <strong>DNA Actions</strong> 子菜单：</p>

<h3>2.1 Update mesh with DNA file</h3>

<p><strong>这是核心功能。</strong> 选一个 <code>.dna</code> 文件，插件自动完成：</p>

<ul>
  <li>更新网格顶点位置（中性姿态）</li>
  <li>更新所有 Morph Target（融合变形）</li>
  <li>更新眼部蒙皮权重</li>
  <li>更新骨骼 Reference Pose（根关节自动跳过，避免坐标系伪影）</li>
  <li>创建/更新 <code>UDNA</code> 资产，确保 RigLogic 运行时正确</li>
</ul>

<p><strong>一句话：DNA 里有什么，就更新什么。头身通用，无需区分。</strong></p>

<h3>2.2 Add a blendshape</h3>

<p>从 OBJ 文件添加一个新的 Morph Target。OBJ 使用 Maya 坐标系（Y-up），插件自动转换为 UE 坐标系。</p>

<p>适用场景：在 Maya 中雕刻了一个新的面部表情，导出 OBJ，添加到已有网格上。</p>

<h2>3. 注意事项</h2>

<ul>
  <li>仅支持 LOD0</li>
  <li>仅适用于 MetaHuman 管线生成的 DNA 文件</li>
  <li>仅编辑器可用，不会打包到运行时</li>
</ul>

<h2>4. 典型工作流</h2>

<pre><code>Maya 中修改 MetaHuman（体型、骨骼、表情）
       ↓
导出新的 .dna 文件
       ↓
UE5.8 Content Browser → 右键 SkeletalMesh → DNA Actions → Update mesh with DNA file
       ↓
选择 .dna 文件 → 完成
       ↓
保存资产，RigLogic 动画自动适配新网格
</code></pre>

</div>

<div id="lang-en">

<h2>5. Additional Tool — DNAUpdater.py (PyQt5 Standalone)</h2>

<p>In addition to the UE5 Editor plugin, a <strong>standalone PyQt5 desktop tool</strong> is also provided for DNA file operations outside the engine — inspection, JSON round-trip, OBJ extraction, and mesh merge/restore for sculpting workflows.</p>

<h3>5.1 Launch</h3>

<pre><code>cd E:\Code\python\PyQT
python DNAUpdater.py
</code></pre>

<p><strong>Dependencies:</strong> Python 3.11+, PyQt5, OpenRigLogic v13.2.7, NumPy, SciPy.</p>

<h3>5.2 UI Layout</h3>

<pre><code>┌──────────────────────────────────────────────────┐
│ Head DNA: [path]                 [Choose...] ┌──┐│
│ Body DNA: [path]                 [Choose...] │EN││
│ Output path: [D:/]                           └──┘│
├──────────────────────────────────────────────────┤
│ PART1 : Modify DNA (BaseMesh / BlendShape)       │
│ PART2 : Full Body (Merge / Restore)              │
│ PART4 : DNA Inspector & Tools                    │
│   [Load Info] [Validate] [Export JSON] ...       │
└──────────────────────────────────────────────────┘
</code></pre>

<p>Click the <strong>EN / 中</strong> button at the top-right to toggle between English and Chinese.</p>

<h3>5.3 Part 1 — Modify DNA (BaseMesh / BlendShape)</h3>

<p>Write sculpted OBJ meshes back into a DNA file. Supports both BaseMesh and BlendShape modifications, batchable via a task list.</p>

<ol>
  <li>Select <strong>Head DNA</strong> at the top</li>
  <li>Set <strong>Output path</strong></li>
  <li>Choose <strong>BaseMesh</strong> or <strong>BlendShape</strong></li>
  <li>Click <strong>Choose OBJ</strong> and select the sculpted file</li>
  <li>Pick the target name from the dropdown</li>
  <li>Click <strong>Add to list</strong></li>
  <li>Click <strong>Write</strong> → outputs <code>modified_&lt;name&gt;.dna</code></li>
</ol>

<p><strong>Note:</strong> OBJ vertex count must match the target mesh in the DNA file.</p>

<h3>5.4 Part 2 — Full Body (Merge / Restore)</h3>

<p><strong>Merge:</strong> Combines head mesh (from Head DNA) and body mesh (from Body DNA) into a single full-body OBJ for sculpting in external 3D software.</p>

<p><strong>Restore:</strong> Takes the sculpted full-body OBJ, splits it back into head and body components, and writes both back into separate DNA files.</p>

<p>Restore outputs <strong>4 files</strong>:</p>
<ul>
  <li><code>Restored_Head_&lt;name&gt;.dna</code></li>
  <li><code>Restored_Body_&lt;name&gt;.dna</code></li>
  <li><code>Restored_Head.obj</code></li>
  <li><code>Restored_Body.obj</code></li>
</ul>

<h3>5.5 Part 4 — DNA Inspector &amp; Tools</h3>

<p><strong>DNA Info:</strong> Load a DNA file and inspect its full structure — descriptor, meshes (vertex/face counts), BlendShape channels (delta counts), joint hierarchy, metadata, and validation status — all in an expandable tree view.</p>

<p><strong>DNA ↔ JSON:</strong></p>
<ul>
  <li><strong>Export DNA to JSON</strong> — complete round-trip-safe JSON export including all geometry and behavior data. Uses compact flat arrays for manageable file size.</li>
  <li><strong>Import JSON to DNA</strong> — reconstruct a DNA file from JSON.</li>
</ul>

<p><strong>DNA → OBJ:</strong> Extract individual or all meshes from a DNA file as OBJ files (vertices, UVs, normals, faces) for use in external 3D software.</p>

<h3>5.6 File Structure</h3>

<pre><code>PyQT/
├── DNAUpdater.py           # Main GUI
├── ModifyDNA.py            # DNA I/O &amp; mesh merge/restore
├── DNAInspector.py         # DNA info, JSON, OBJ export/import
├── RigLogicPreview.py      # RigLogic evaluation (commented out)
└── libs/dnalib/            # Legacy DNA library (fallback)
</code></pre>

</div>

<div id="lang-zh">

<h2>5. 附加工具 — DNAUpdater.py（PyQt5 独立桌面版）</h2>

<p>除了 UE5 编辑器插件，本仓库还提供了一个 <strong>独立的 PyQt5 桌面工具</strong>，用于在引擎外部操作 DNA 文件——包括 DNA 信息查看、JSON 往返转换、OBJ 网格提取，以及全身雕刻工作流的合并与还原。</p>

<h3>5.1 启动</h3>

<pre><code>cd E:\Code\python\PyQT
python DNAUpdater.py
</code></pre>

<p><strong>依赖：</strong> Python 3.11+, PyQt5, OpenRigLogic v13.2.7, NumPy, SciPy。</p>

<h3>5.2 界面布局</h3>

<pre><code>┌──────────────────────────────────────────────────┐
│ Head DNA: [path]                 [Choose...] ┌──┐│
│ Body DNA: [path]                 [Choose...] │中││
│ Output path: [D:/]                           └──┘│
├──────────────────────────────────────────────────┤
│ PART1 : 修改 DNA (基础网格 / 变形目标)           │
│ PART2 : 全身处理 (合并 / 还原)                   │
│ PART4 : DNA 检查器 & 工具                        │
│   [加载信息] [验证] [导出JSON] ...               │
└──────────────────────────────────────────────────┘
</code></pre>

<p>右上角的 <strong>中 / EN</strong> 按钮可随时切换界面语言。</p>

<h3>5.3 Part 1 — 修改 DNA（BaseMesh / BlendShape）</h3>

<p>将雕刻后的 OBJ 网格数据写回 DNA 文件。支持基础网格和变形目标的批量修改。</p>

<ol>
  <li>顶部选择 <strong>Head DNA</strong> 文件</li>
  <li>设置 <strong>输出路径</strong></li>
  <li>选择 <strong>BaseMesh</strong> 或 <strong>BlendShape</strong></li>
  <li>点击 <strong>Choose OBJ</strong> 选择雕刻后的文件</li>
  <li>从下拉框中选择目标名称</li>
  <li>点击 <strong>Add to list</strong></li>
  <li>点击 <strong>Write</strong> → 输出 <code>modified_&lt;名称&gt;.dna</code></li>
</ol>

<p><strong>注意：</strong> OBJ 顶点数必须与 DNA 中目标 mesh 的顶点数一致。</p>

<h3>5.4 Part 2 — 全身处理（Merge / Restore）</h3>

<p><strong>合并（Merge）：</strong> 将头部 DNA 的头部网格与身体 DNA 的身体网格合并为单个全身 OBJ，方便在 Maya/Blender 中统一雕刻。</p>

<p><strong>还原（Restore）：</strong> 将雕刻后的全身 OBJ 拆分回头部和身体，分别写回独立的 DNA 文件和 OBJ 文件。</p>

<p>还原输出 <strong>4 个文件</strong>：</p>
<ul>
  <li><code>Restored_Head_&lt;名称&gt;.dna</code></li>
  <li><code>Restored_Body_&lt;名称&gt;.dna</code></li>
  <li><code>Restored_Head.obj</code></li>
  <li><code>Restored_Body.obj</code></li>
</ul>

<h3>5.5 Part 4 — DNA 检查器 &amp; 工具</h3>

<p><strong>DNA 信息查看：</strong> 加载 DNA 文件后以树形结构展示完整信息——描述符、Mesh（顶点/面数）、BlendShape 通道（delta 数量）、关节层级、元数据和验证状态。</p>

<p><strong>DNA ↔ JSON：</strong></p>
<ul>
  <li><strong>Export DNA to JSON</strong> — 完整导出 DNA 为 JSON（含 geometry + behavior），使用紧凑 flat array 格式控制文件大小，支持完整往返还原。</li>
  <li><strong>Import JSON to DNA</strong> — 从 JSON 文件重建 DNA。</li>
</ul>

<p><strong>DNA → OBJ：</strong> 从 DNA 中提取指定或全部 mesh 为 OBJ 文件（含顶点、UV、法线、面），供外部 3D 软件使用。</p>

<h3>5.6 文件结构</h3>

<pre><code>PyQT/
├── DNAUpdater.py           # 主界面
├── ModifyDNA.py            # DNA 读写 & 网格合并/还原
├── DNAInspector.py         # DNA 信息、JSON/OBJ 导出导入
├── RigLogicPreview.py      # RigLogic 评估（已注释）
└── libs/dnalib/            # 旧版 DNA 库（备用）
</code></pre>

</div>
