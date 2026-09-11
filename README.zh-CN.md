# TAP Artifact Contracts

[English](README.md) | [简体中文](README.zh-CN.md)

## 用途

TAP 相机应用、网页验证器和服务端共同遵循的规范中心。这里定义产品行为、媒体编码与解码、manifest 字段、签名覆盖的字节和 HTTP 接口。实现遵循契约；实现代码或其文档不能补充契约中缺失的规则。

## 用法

根据要解决的问题选择阅读入口：

| 文档 | 说明 |
| --- | --- |
| [契约索引](CONTRACTS.md) | manifest 字段、编码与解码布局、哈希和签名 |
| [产品契约](ProductContract.md) | 应用行为、生命周期与声明边界 |
| [服务端契约](BackendContract.md) | App Attest 注册、HTTP、信任与重放策略 |
| [平面设计](PlanesTechnicalDesign.md) | 照片几何算法及其限制 |
| [验收流程](Acceptance.md) | 分能力验证步骤及结果所需的证据 |
| [示例与测试向量](examples/README.md) | 合成字段示例、固定字节与变换结果 |
| [版本规则](VERSIONING.md) | 独立格式族与已审核的契约提交 |

这里没有需要安装或构建的应用。编辑规范后检查本地 Markdown 链接、锚点和 JSON 示例。使用 Python 3 检查 JSON 语法：

```sh
python3 - <<'PY'
import json
from pathlib import Path

for path in Path("examples").rglob("*.json"):
    json.loads(path.read_text(encoding="utf-8"))
print("JSON examples parsed")
PY
```

JSON 解析成功只代表语法正确，不代表符合契约。固定测试向量应保持原样，不能为了匹配实现而重新生成预期字节。修改行为或字节定义时遵循[版本规则](VERSIONING.md)。

## 原理

编码与解码规定媒体 blob 如何写入、如何读出，用于播放或分析。Manifest 描述拍摄产物；绑定与证明规则规定哪些已存储字节参与哈希，以及如何重建被签名的消息。解码是否成功、深度质量和采样时间的性质，不决定这些字节是否与哈希和签名对应。

产品契约和服务端契约定义外围行为与服务接口。验收流程说明如何验证这些要求，不代表某个实现已经通过验收。

## 目录结构

```text
.
├── CONTRACTS.md          # 产物格式与验签规则入口
├── ProductContract.md    # 产品行为与可见交互要求
├── BackendContract.md    # 共享 HTTP、凭证与信任要求
├── PlanesTechnicalDesign.md # 几何算法与限制
├── Acceptance.md         # 分能力验收流程
├── VERSIONING.md         # 格式版本与已审核的契约提交
├── manifests/           # 普通照片、Live Photo、TAP Video 元数据
├── containers/          # 照片/视频字节布局、深度与遥测编码
├── bindings/            # 哈希输入、签名消息、证明与验证
├── transport/           # 未签名的 .tapnap 包路由
└── examples/            # 合成示例与固定测试向量
    ├── manifests/       # 产物元数据示例
    ├── transport/       # 包路由示例
    └── vectors/         # 固定字节及预期的编码/变换结果
```

## 仓库依赖关系

文档依赖只有一个方向：

```text
相机应用 ───┐
网页验证器 ─┼──> TAPArtifactContracts
服务端 ─────┘
```

本仓库不依赖这些消费者的代码或文档。消费者记录自己采用的契约提交，各自维护构建、运行依赖和实现验证结果。已审核的 Git 提交与线上格式版本是两回事；仅调整文档表达无需更新消费者的固定引用。通用组件库维护自己的 API 文档，只有实现 TAP 专属规则时才需要依赖 TAP 契约。
