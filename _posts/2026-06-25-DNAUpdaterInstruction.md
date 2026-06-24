---
layout: post
title: DNAUpdater — MetaHuman DNA Update Tool for UE5.8
date: 2026-06-25 19:30:00 +0800
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
