# VIGA リポジトリ構造 詳細ドキュメント

このドキュメントでは、VIGAリポジトリの構造、各プログラムの役割、および実行に必要な環境について詳細に説明します。

## 目次

1. [概要](#概要)
2. [リポジトリ構造](#リポジトリ構造)
3. [主要コンポーネント](#主要コンポーネント)
4. [必要な環境](#必要な環境)
5. [セットアップ手順](#セットアップ手順)
6. [使用方法](#使用方法)
7. [開発とカスタマイズ](#開発とカスタマイズ)

---

## 概要

### VIGAとは

VIGA（Vision-as-Inverse-Graphics Agent via Interleaved Multimodal Reasoning）は、視覚逆グラフィックスアプローチによるプログラマティック視覚再構築のための分析合成コードエージェントです。ターゲット画像に対して、シーンを生成・レンダリング・検証する反復的なループを通じて動作します。

### 主な特徴

- **デュアルエージェント設計**: GeneratorとVerifierの2つの役割を持つ自己反省的エージェント
- **ツールベースアーキテクチャ**: 計画、コード実行、アセット取得、シーンクエリ用のツール
- **複数のドメインサポート**: 3Dグラフィックス編集、スライド生成、シーン再構築
- **ファインチューニング不要**: 書く→実行→比較→修正のセルフコレクティングループ

### サポートされるモード

| モード | 説明 | 出力形式 |
|-------|------|---------|
| **BlenderBench** | マルチステップ3Dグラフィックス編集（レベル1-3） | Blender Python |
| **BlenderGym** | シングルステップ3Dグラフィックス編集 | Blender Python |
| **SlideBench** | 2Dスライド/ドキュメントレイアウト合成 | PowerPoint |
| **Static Scene** | 単一視点からの3D再構築 | Blenderシーン |
| **Dynamic Scene** | 物理演算を含む4D動的シーン | Blenderアニメーション |

---

## リポジトリ構造

```
VIGA/
├── main.py                    # メインエントリーポイント
├── README.md                  # プロジェクト概要
├── LICENSE                    # MITライセンス
│
├── agents/                    # エージェント実装
│   ├── generator.py           # コード生成エージェント
│   ├── verifier.py            # 検証エージェント
│   ├── tool_client.py         # ツールクライアント（MCP通信）
│   └── prompt_builder.py      # プロンプト構築ユーティリティ
│
├── tools/                     # MCPツールサーバー
│   ├── generator_base.py      # Generator基本ツール
│   ├── verifier_base.py       # Verifier基本ツール
│   ├── initialize_plan.py     # プラン作成ツール
│   ├── blender/               # Blender実行・調査ツール
│   ├── slides/                # スライド生成ツール
│   ├── assets/                # 3Dアセット生成ツール
│   └── sam3d/                 # SAMベースシーン再構築
│
├── prompts/                   # システムプロンプト
│   ├── prompt_manager.py      # プロンプト管理
│   ├── blendergym/            # BlenderGym用プロンプト
│   ├── blenderbench/          # BlenderBench用プロンプト
│   ├── static_scene/          # 静的シーン用プロンプト
│   ├── dynamic_scene/         # 動的シーン用プロンプト
│   └── slidebench/            # SlideBench用プロンプト
│
├── runners/                   # バッチ実行スクリプト
│   ├── blendergym/            # BlenderGym実行スクリプト
│   ├── blenderbench/          # BlenderBench実行スクリプト
│   ├── slidebench/            # SlideBench実行スクリプト
│   ├── static_scene.py        # 静的シーン実行
│   ├── dynamic_scene.py       # 動的シーン実行
│   └── shared/                # 共有ユーティリティ
│
├── data/                      # データセットとタスク
│   ├── blendergym/            # BlenderGymデータセット
│   ├── blenderbench/          # BlenderBenchデータセット
│   ├── slidebench/            # SlideBenchデータセット
│   ├── static_scene/          # 静的シーンデータ
│   └── dynamic_scene/         # 動的シーンデータ
│
├── evaluators/                # 評価メトリクス
│   ├── blendergym/            # BlenderGym評価
│   ├── blenderbench/          # BlenderBench評価
│   └── slidebench/            # SlideBench評価
│
├── models/                    # ローカルモデルサーバー
│   ├── server.py              # vLLMサーバー
│   ├── client_chat.py         # チャットクライアント例
│   └── client_vision.py       # ビジョンクライアント例
│
├── requirements/              # 環境依存関係
│   ├── requirement_agent.txt          # エージェント環境
│   ├── requirement_blender.txt        # Blender環境
│   ├── requirement_pptx.txt           # PowerPoint環境
│   ├── requirement_sam.txt            # SAM環境
│   ├── requirement_sam3d.txt          # SAM3D環境
│   ├── requirement_eval-blender.txt   # Blender評価環境
│   └── requirement_eval-pptx.txt      # PowerPoint評価環境
│
├── utils/                     # ユーティリティ
│   ├── common.py              # 共通ユーティリティ
│   ├── _api_keys.py.example   # APIキー設定例
│   ├── _path.py.example       # パス設定例
│   └── third_party/           # サードパーティライブラリ
│       ├── infinigen/         # Infinigen Blenderツール
│       ├── sam/               # Segment Anything Model
│       └── slides/            # スライド生成ライブラリ
│
└── docs/                      # ドキュメント
    ├── architecture.md        # アーキテクチャ説明
    ├── README.md              # ドキュメント一覧
    └── images/                # 画像・図表
```

---

## 主要コンポーネント

### 1. エージェントシステム (`agents/`)

VIGAの中核となるデュアルエージェントシステムの実装です。

#### `generator.py` - Generatorエージェント

**役割**:
- ターゲット画像の分析に基づいてシーンコードを生成
- Verifierからのフィードバックを受けてコードを改善
- ツールを使用してシーン情報の取得やアセット生成を実行

**主な機能**:
- `run()`: メインの実行ループ
- コード生成と実行の管理
- メモリ管理（スライディングウィンドウ方式）
- ツール呼び出しの制御

**使用するツール**:
- `make_plan`: 高レベルアクションプランの作成
- `execute_code`: シーン構築/更新プログラムの実行
- `get_scene_info`: オブジェクト属性のクエリ
- `get_better_assets`: 3Dアセットの取得・生成
- `end_process`: 完了シグナル

#### `verifier.py` - Verifierエージェント

**役割**:
- レンダリング出力を複数の視点から検証
- 視覚的不一致の特定
- 次の反復のためのフィードバック提供

**主な機能**:
- `analyze()`: レンダリング結果の分析
- 視点の推奨
- 差異の詳細な説明
- 改善提案の生成

**使用するツール**:
- `initialize_viewpoint`: 標準的な診断視点の計算
- `set_camera`: 特定のカメラポーズへの移動
- `investigate`: 自然言語によるカメラ調整
- `set_keyframe`: 4Dシーンでのタイムライン操作
- `set_visibility`: オブジェクトの可視性切り替え
- `get_scene_info`: シーン状態のテキスト要約

#### `tool_client.py` - ツールクライアント

**役割**:
- MCP（Model Context Protocol）サーバーとの通信管理
- 外部ツールサーバーへの接続
- ツール呼び出しの同期/非同期実行

**主な機能**:
- `connect_servers()`: ツールサーバーへの接続
- `call_tool()`: ツール呼び出しの実行
- エラーハンドリングとリトライ

#### `prompt_builder.py` - プロンプトビルダー

**役割**:
- エージェントプロンプトの動的構築
- メモリとコンテキストの管理
- Few-shot例の挿入

### 2. ツールシステム (`tools/`)

各ツールはMCPサーバーとして実装され、特定の環境で実行されます。

#### 基本ツール

**`generator_base.py`**
- 環境: agent
- ツール: `end_process` - Generatorの完了シグナル

**`verifier_base.py`**
- 環境: agent
- ツール: `end_process` - Verifierの完了シグナル

**`initialize_plan.py`**
- 環境: agent
- ツール: `make_plan` - 高レベルアクションプランの作成

#### Blenderツール (`tools/blender/`)

**`exec.py`**
- 環境: blender
- ツール:
  - `execute_code`: Blender Pythonコードの実行
  - `undo`: 最後の操作の取り消し
  - `render`: シーンのレンダリング

**`investigator.py`**
- 環境: blender
- ツール:
  - `set_camera`: カメラ位置の設定
  - `investigate`: 自然言語によるシーン調査
  - `get_scene_info`: シーン情報の取得

**`glb_import.py`**
- 環境: blender
- GLBファイルのインポート機能

**`investigator_core.py`**
- カメラ操作のコア機能
- 視点計算アルゴリズム

**`script_generators.py`**
- Blenderスクリプトの動的生成

#### スライドツール (`tools/slides/`)

**`exec.py`**
- 環境: pptx
- ツール:
  - `execute_code`: PowerPoint生成コードの実行

#### アセット生成ツール (`tools/assets/`)

**`meshy.py`**
- 環境: agent
- ツール:
  - `get_better_assets`: Meshy APIを使用した3Dアセット生成

**`meshy_api.py`**
- Meshy APIクライアントの実装

#### SAM3Dツール (`tools/sam3d/`)

**`init.py`**
- 環境: sam3d
- ツール:
  - `initialize_scene`: SAMベースのシーン初期化

**`bridge.py`**
- SAMモデルとのブリッジ

**`sam_worker.py`**
- SAMセグメンテーションワーカー

**`sam3_worker.py`**
- SAM3ワーカー

**`sam3d_worker.py`**
- SAM3Dワーカー

### 3. プロンプトシステム (`prompts/`)

各モードに特化したシステムプロンプトとFew-shot例を管理します。

#### `prompt_manager.py`

**役割**:
- プロンプトの読み込みと管理
- モード別プロンプトの取得
- 例の動的挿入

**使用方法**:
```python
from prompts.prompt_manager import PromptManager

pm = PromptManager(mode="blendergym")
generator_prompt = pm.get_generator_system()
verifier_prompt = pm.get_verifier_system()
```

#### モード別プロンプト

各モードディレクトリには以下が含まれます：
- `generator.py`: Generator用システムプロンプト
- `verifier.py`: Verifier用システムプロンプト
- `examples/`: Few-shot学習用の例

### 4. ランナーシステム (`runners/`)

バッチ処理と並列実行のためのスクリプト群です。

#### BlenderGym Runner (`runners/blendergym/`)

**`ours.py`**
- VIGAメソッドの実行
- 並列処理サポート（max-workers）
- タスクタイプ別実行（blendshape, geometry, lighting, material, placement）

**`baseline.py`**
- ベースライン手法の実行
- 比較評価用

**`run_all_code.py`**
- すべてのタスクの一括実行

**`alchemy.py`**
- Alchemyベースライン手法

#### BlenderBench Runner (`runners/blenderbench/`)

**`main.py`**
- メイン実行スクリプト

**`ours.py`**
- VIGAメソッドの実行
- レベル別実行（level1, level2, level3）

**`alchemy.py`**
- Alchemyベースライン手法

#### SlideBench Runner (`runners/slidebench/`)

**`ours.py`**
- VIGAメソッドによるスライド生成

**`baseline.py`**
- ベースライン手法の実行

**`create_slide.py`**
- スライド作成ユーティリティ

**`library/`**
- スライド生成ライブラリ関数

**`prompt/`**
- スライド生成用プロンプト例

#### 個別タスクランナー

**`static_scene.py`**
- 静的シーン再構築タスクの実行
- カスタムシーンデータのサポート

**`dynamic_scene.py`**
- 動的シーン生成タスクの実行
- アニメーションと物理演算

#### 共有ユーティリティ (`runners/shared/`)

**`blender_executor.py`**
- Blenderコマンドの実行ラッパー

**`code_generator.py`**
- コード生成ユーティリティ

**`image_utils.py`**
- 画像処理ユーティリティ

**`tournament.py`**
- トーナメント方式の評価

### 5. データセット (`data/`)

各ベンチマークのデータとタスク固有のスクリプトです。

#### BlenderGym (`data/blendergym/`)

- シングルステップ3D編集タスク
- カテゴリ: blendshape, geometry, lighting, material, placement
- スクリプト:
  - `generator_script.py`: Generator用Blenderスクリプト
  - `verifier_script.py`: Verifier用Blenderスクリプト
  - `pipeline_render_script.py`: レンダリングパイプライン
  - その他のユーティリティスクリプト

#### BlenderBench (`data/blenderbench/`)

- マルチステップ3D編集タスク
- レベル1: カメラ調整
- レベル2: マルチステップ推論
- レベル3: 複合的編集
- スクリプト: BlenderGymと同様の構造

#### SlideBench (`data/slidebench/`)

- プログラマティックスライド生成タスク
- スクリプト:
  - `create_dataset.py`: データセット作成
  - `library.py`: スライドライブラリ関数
  - `parse_media.py`: メディア解析
  - `reproduce_code.py`: コード再現
  - `seed_instruction.py`: シード命令

#### Static Scene (`data/static_scene/`)

- 単一視点からの3D再構築
- スクリプト:
  - `generator_init_script.py`: 初期化スクリプト
  - `generator_script.py`: Generator用スクリプト
  - `verifier_script.py`: Verifier用スクリプト

#### Dynamic Scene (`data/dynamic_scene/`)

- 4D動的シーン生成
- `artist/`: 例示データ
- スクリプト:
  - `generator_script.py`: Generator用スクリプト
  - `verifier_script.py`: Verifier用スクリプト

### 6. 評価システム (`evaluators/`)

各ベンチマークの評価メトリクス計算です。

#### 評価メトリクス

1. **PL Loss (Photometric Loss)**
   - ピクセルレベルの差分測定
   - 低いほど良い

2. **N-CLIP Score (Negative-CLIP Score)**
   - セマンティックアライメント測定
   - 低いほど良い

3. **VLM Score**
   - タスク完了度、視覚品質、空間精度、詳細度
   - 0-5スケール（高いほど良い）

#### ディレクトリ構造

各ベンチマーク（blendergym, blenderbench, slidebench）に対して：

**`evaluate.py`**
- VIGAメソッドの評価

**`evaluate_baseline.py`**
- ベースライン手法の評価

**`gather.py`**
- 結果の収集

**`gather_baseline.py`**
- ベースライン結果の収集

**SlideBench固有**:
- `match.py`: マッチング評価
- `page_eval.py`: ページ単位評価
- `reference_free_eval.py`: 参照なし評価
- `metrics/`: メトリクス実装（clip, color, position, text）

**BlenderBench固有**:
- `ref_based_eval.py`: 参照ベース評価
- `ref_free_eval.py`: 参照なし評価

### 7. モデルサーバー (`models/`)

ローカルVLMサーバーの実装です。

#### `server.py`

**役割**:
- vLLMベースのHTTPサーバー
- OpenAI互換APIの提供
- Qwen2-VL-7B-Instructのホスティング

**使用方法**:
```bash
python models/server.py --host 0.0.0.0 --port 8000 \
  --model Qwen/Qwen2-VL-7B-Instruct \
  --tensor-parallel-size 1
```

#### `client_chat.py`

チャットクライアントの例

#### `client_vision.py`

ビジョンクライアントの例

### 8. ユーティリティ (`utils/`)

#### `common.py`

共通のユーティリティ関数

#### `_api_keys.py.example`

APIキー設定のテンプレート：
```python
OPENAI_API_KEY = "your-key"
CLAUDE_API_KEY = "your-key"
GEMINI_API_KEY = "your-key"
MESHY_API_KEY = "your-key"
```

#### `_path.py.example`

環境パス設定のテンプレート：
```python
path_to_cmd = {
    "tools/blender/exec.py": "/path/to/conda/envs/blender/bin/python",
    "tools/blender/investigator.py": "/path/to/conda/envs/blender/bin/python",
    "tools/slides/exec.py": "/path/to/conda/envs/pptx/bin/python",
}
```

#### `third_party/`

サードパーティライブラリ：
- **infinigen**: Blenderツールチェーン
- **sam**: Segment Anything Model
- **slides**: スライド生成ライブラリ

---

## 必要な環境

### ハードウェア要件

#### 最小構成
- CPU: マルチコアプロセッサ（4コア以上推奨）
- RAM: 16GB以上
- ストレージ: 50GB以上の空き容量

#### 推奨構成（3Dモード用）
- GPU: NVIDIA GPU（CUDA対応）
  - VRAM: 8GB以上
  - 推奨: RTX 3080以上
- CPU: 8コア以上
- RAM: 32GB以上
- ストレージ: 100GB以上（SSD推奨）

### ソフトウェア要件

#### 必須
- **Linux OS**: Ubuntu 20.04以上推奨（macOS/Windowsも可能だが制限あり）
- **Conda**: Miniconda/Anaconda
- **Git**: バージョン管理
- **CUDA**: 11.8以上（GPU使用時）
- **cuDNN**: CUDA対応バージョン

#### オプション
- **LibreOffice**: SlideBenchモード用
- **Google Chrome**: HTML/スクリーンショット用

### Python環境

VIGAは複数のConda環境を使用します：

#### 1. Agent環境（Python 3.10）

**目的**: メインエージェントの実行

**主な依存パッケージ**:
- `openai==2.6.1`: OpenAI APIクライアント
- `mcp==1.20.0`: Model Context Protocol
- `pydantic==2.12.3`: データ検証
- `httpx==0.28.1`: HTTP通信
- `pillow==12.0.0`: 画像処理
- `python-pptx==1.0.2`: PowerPoint操作
- `tqdm==4.67.1`: プログレスバー
- `psutil==7.1.3`: システム情報
- `gpustat==1.1.1`: GPU監視

#### 2. Blender環境（Python 3.11）

**目的**: Blenderでの3Dシーン操作

**主な依存パッケージ**:
- `bpy==4.2.0`: Blender Python API
- `numpy==1.26.4`: 数値計算
- `opencv-python==4.8.1.78`: 画像処理
- `scipy==1.16.3`: 科学計算
- `matplotlib==3.10.7`: プロット作成
- `trimesh==3.22.5`: 3Dメッシュ処理
- `pyrender==0.1.45`: レンダリング
- `scikit-image==0.19.3`: 画像処理
- `infinigen`: カスタムBlenderツール（ローカルインストール）
- `mcp==1.20.0`: ツールサーバー通信

#### 3. SAM環境（Python 3.10）

**目的**: Segment Anything Modelの実行

**依存パッケージ**（requirement_sam.txtから）:
- PyTorch
- Segment Anything関連パッケージ
- 画像処理ライブラリ

#### 4. SAM3D環境（Python 3.11）

**目的**: 3Dセグメンテーションツールの実行

**依存パッケージ**（requirement_sam3d-objects.txtから）:
- SAM3D関連パッケージ

#### 5. PPTX環境（Python 3.10）

**目的**: PowerPointスライド生成

**主な依存パッケージ**（requirement_pptx.txtから）:
- `python-pptx`: PowerPoint生成
- スライド生成関連ライブラリ

#### 6. 評価環境

**Blender評価環境（Python 3.11）**:
- requirement_eval-blender.txt

**PPTX評価環境（Python 3.10）**:
- requirement_eval-pptx.txt

### 外部サービス・API

#### 必須APIキー

1. **OpenAI API**
   - 用途: GPT-4o, GPT-4-turboなどのVLMモデル
   - 取得先: https://platform.openai.com/

2. **Meshy API**
   - 用途: 3Dアセット生成
   - 取得先: https://www.meshy.ai/

#### オプションAPIキー

3. **Anthropic API**
   - 用途: Claude Sonnet/Opusモデル
   - 取得先: https://www.anthropic.com/

4. **Google Gemini API**
   - 用途: Gemini 2.5 Proモデル
   - 取得先: https://ai.google.dev/

5. **Qwen API**
   - 用途: Qwen-VLモデル
   - 取得先: https://dashscope.aliyun.com/

---

## セットアップ手順

### 1. リポジトリのクローン

```bash
# リポジトリをクローン
git clone https://github.com/benriworks/VIGA.git
cd VIGA

# サブモジュールを初期化
git submodule update --init --recursive
```

### 2. Conda環境の作成

#### Agent環境

```bash
# Agent環境を作成
conda create -n agent python=3.10 -y
conda activate agent
pip install -r requirements/requirement_agent.txt
```

#### Blender環境

```bash
# Blender環境を作成
conda create -n blender python=3.11 -y
conda activate blender
pip install -r requirements/requirement_blender.txt

# Infinigenのインストール
cd utils/third_party/infinigen
INFINIGEN_MINIMAL_INSTALL=True bash scripts/install/interactive_blender.sh
cd ../../..

# エラーが発生しても、utils/third_party/infinigen/blenderが存在すればOK
```

#### SAM環境（オプション）

```bash
# SAM環境を作成
conda create -n sam python=3.10 -y
conda activate sam
pip install -r requirements/requirement_sam.txt

# SAMモデルのダウンロード
wget -P utils/third_party/sam https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth
```

#### SAM3D環境（オプション）

```bash
# SAM3D環境を作成
conda create -n sam3d python=3.11 -y
conda activate sam3d
bash requirements/install_sam3d.sh
```

#### PPTX環境（SlideBenchモード用）

```bash
# PPTX環境を作成
conda create -n pptx python=3.10 -y
conda activate pptx
pip install -r requirements/requirement_pptx.txt

# LibreOfficeのインストール（Ubuntu/Debian）
sudo apt-get install -y libreoffice unoconv
```

#### 評価環境（オプション）

```bash
# Blender評価環境
conda create -n eval-blender python=3.11 -y
conda activate eval-blender
pip install -r requirements/requirement_eval-blender.txt

# PPTX評価環境
conda create -n eval-pptx python=3.10 -y
conda activate eval-pptx
pip install -r requirements/requirement_eval-pptx.txt
```

### 3. APIキーの設定

```bash
# APIキー設定ファイルを作成
cp utils/_api_keys.py.example utils/_api_keys.py

# エディタで編集
nano utils/_api_keys.py
# または
vim utils/_api_keys.py
```

`_api_keys.py`に以下を設定：

```python
# 必須
OPENAI_API_KEY = "sk-..."  # OpenAI APIキー
MESHY_API_KEY = "..."      # Meshy APIキー

# オプション
CLAUDE_API_KEY = "..."     # Anthropic APIキー
GEMINI_API_KEY = "..."     # Google Gemini APIキー
QWEN_API_KEY = "..."       # Qwen APIキー
```

### 4. 環境パスの設定

```bash
# パス設定ファイルを作成
cp utils/_path.py.example utils/_path.py

# エディタで編集
nano utils/_path.py
```

Condaインストールパスを確認：

```bash
conda env list
# 出力例:
# agent           /home/user/anaconda3/envs/agent
# blender         /home/user/anaconda3/envs/blender
```

`_path.py`を編集：

```python
import os

# Condaベースパスを設定（環境変数または直接指定）
CONDA_BASE = os.getenv("VIGA_CONDA_BASE", os.path.expanduser("~/anaconda3/envs"))

# 各ツールの実行環境を指定
path_to_cmd = {
    "tools/blender/exec.py": f"{CONDA_BASE}/blender/bin/python",
    "tools/blender/investigator.py": f"{CONDA_BASE}/blender/bin/python",
    "tools/slides/exec.py": f"{CONDA_BASE}/pptx/bin/python",
    "tools/assets/meshy.py": f"{CONDA_BASE}/agent/bin/python",
    "tools/sam3d/init.py": f"{CONDA_BASE}/sam3d/bin/python",
}
```

または環境変数を使用：

```bash
export VIGA_CONDA_BASE=~/anaconda3/envs
```

### 5. インストールの検証

```bash
# Agent環境のテスト
conda activate agent
python -c "import openai; import mcp; print('Agent environment OK')"

# Blender環境のテスト
conda activate blender
python -c "import bpy; print('Blender environment OK')"

# PPTX環境のテスト（作成した場合）
conda activate pptx
python -c "import pptx; print('PPTX environment OK')"
```

---

## 使用方法

### 基本的な実行

#### 1. Main.pyを使用した個別タスク実行

```bash
# Agent環境をアクティベート
conda activate agent

# Dynamic Sceneモードの実行例
python main.py \
    --mode dynamic_scene \
    --task-name artist \
    --model gpt-4o \
    --max-rounds 10 \
    --generator-tools tools/blender/exec.py,tools/generator_base.py,tools/initialize_plan.py \
    --verifier-tools tools/blender/investigator.py,tools/verifier_base.py
```

#### 2. Runner スクリプトを使用したバッチ実行

**BlenderGym:**

```bash
python runners/blendergym/ours.py \
    --dataset-path data/blendergym \
    --task all \
    --model gpt-4o \
    --max-rounds 10 \
    --max-workers 4
```

**BlenderBench:**

```bash
python runners/blenderbench/ours.py \
    --dataset-path data/blenderbench \
    --task level1 \
    --model gpt-4o \
    --max-rounds 10
```

**SlideBench:**

```bash
python runners/slidebench/ours.py \
    --dataset-path data/slidebench \
    --task all \
    --model gpt-4o \
    --max-rounds 10
```

**Static Scene:**

```bash
python runners/static_scene.py \
    --dataset-path data/static_scene \
    --task all \
    --model gpt-4o \
    --max-rounds 100 \
    --prompt-setting scene_graph
```

**Dynamic Scene:**

```bash
python runners/dynamic_scene.py \
    --dataset-path data/dynamic_scene \
    --task artist \
    --model gpt-4o \
    --max-rounds 100
```

### モード別詳細

#### BlenderGymモード

**特徴**:
- シングルステップ3D編集タスク
- 5つのカテゴリ（blendshape, geometry, lighting, material, placement）

**タスク指定**:
```bash
# すべてのタスク
--task all

# 特定カテゴリ
--task blendshape
--task geometry
--task lighting
--task material
--task placement

# 特定タスクID
--task-id 1
```

**並列実行**:
```bash
--max-workers 8  # 8並列で実行
```

#### BlenderBenchモード

**特徴**:
- マルチステップ3D編集タスク
- 3つのレベル（level1, level2, level3）

**レベル説明**:
- Level 1: カメラ調整タスク
- Level 2: マルチステップ推論タスク
- Level 3: 複合的編集タスク

**実行例**:
```bash
python runners/blenderbench/ours.py \
    --dataset-path data/blenderbench \
    --task level2 \
    --model gpt-4o \
    --memory-length 24
```

#### SlideBenchモード

**特徴**:
- プログラマティックなスライド生成
- PowerPoint形式の出力

**実行例**:
```bash
python runners/slidebench/ours.py \
    --dataset-path data/slidebench \
    --task business \
    --model gpt-4o \
    --generator-tools tools/slides/exec.py,tools/generator_base.py
```

#### Static Sceneモード

**特徴**:
- 単一視点からの3D再構築
- ゼロからのシーン構築

**プロンプト設定**:
```bash
--prompt-setting none          # デフォルト
--prompt-setting procedural    # プロシージャル生成ヒント
--prompt-setting scene_graph   # シーングラフ構造ヒント
--prompt-setting get_asset     # アセット取得ヒント
```

**初期化設定**:
```bash
--init-setting none        # 空のシーン
--init-setting minimal     # 最小限の初期化
--init-setting reasonable  # 合理的な初期化
```

**実行例**:
```bash
python runners/static_scene.py \
    --dataset-path data/static_scene \
    --task custom_scene \
    --model gpt-4o \
    --max-rounds 100 \
    --prompt-setting scene_graph \
    --init-setting minimal \
    --generator-tools tools/blender/exec.py,tools/generator_base.py,tools/assets/meshy.py,tools/initialize_plan.py
```

#### Dynamic Sceneモード

**特徴**:
- 4D動的シーン（アニメーション）
- 物理演算のサポート

**実行例**:
```bash
python runners/dynamic_scene.py \
    --dataset-path data/dynamic_scene \
    --task artist \
    --model gpt-4o \
    --max-rounds 100 \
    --prompt-setting init \
    --generator-tools tools/blender/exec.py,tools/generator_base.py,tools/initialize_plan.py,tools/sam3d/init.py
```

### カスタムデータの使用

#### Static Sceneのカスタムタスク

1. データディレクトリを作成：

```bash
mkdir -p data/static_scene/my_scene
```

2. ターゲット画像を配置：

```bash
# 画像をコピー
cp /path/to/target.png data/static_scene/my_scene/
```

3. 説明ファイルを作成（オプション）：

```bash
echo "A cozy living room with a sofa and a coffee table" > data/static_scene/my_scene/description.txt
```

4. 実行：

```bash
python runners/static_scene.py \
    --dataset-path data/static_scene \
    --task my_scene \
    --model gpt-4o
```

#### Dynamic Sceneのカスタムタスク

1. データディレクトリを作成：

```bash
mkdir -p data/dynamic_scene/my_animation
```

2. ターゲット動画/画像シーケンスを配置

3. 説明ファイルを作成：

```bash
nano data/dynamic_scene/my_animation/description.txt
```

4. 実行：

```bash
python runners/dynamic_scene.py \
    --dataset-path data/dynamic_scene \
    --task my_animation \
    --model gpt-4o
```

### 評価の実行

#### BlenderGym評価

```bash
# VIGAメソッドの評価
python evaluators/blendergym/evaluate.py \
    --output-dir output/blendergym/<timestamp> \
    --dataset-path data/blendergym

# ベースライン評価
python evaluators/blendergym/evaluate_baseline.py \
    --output-dir output/blendergym/baseline/<timestamp>
```

#### BlenderBench評価

```bash
# 参照ベース評価
python evaluators/blenderbench/ref_based_eval.py \
    --output-dir output/blenderbench/<timestamp>

# 参照なし評価
python evaluators/blenderbench/ref_free_eval.py \
    --output-dir output/blenderbench/<timestamp>
```

#### SlideBench評価

```bash
# VIGAメソッドの評価
python evaluators/slidebench/evaluate.py \
    --output-dir output/slidebench/<timestamp> \
    --dataset-path data/slidebench

# ページ単位評価
python evaluators/slidebench/page_eval.py \
    --output-dir output/slidebench/<timestamp>
```

### 出力の確認

実行結果は`output/`ディレクトリに保存されます：

```
output/<mode>/<timestamp>/<task>/
├── generator_thoughts/    # Generator の思考過程
│   ├── 1.json
│   ├── 2.json
│   └── ...
├── verifier_thoughts/     # Verifier の分析結果
│   ├── 1.json
│   ├── 2.json
│   └── ...
├── renders/               # レンダリング画像
│   ├── 1/
│   │   ├── view1.png
│   │   └── view2.png
│   ├── 2/
│   └── ...
├── codes/                 # 生成されたコード
│   ├── 1.py
│   ├── 2.py
│   └── ...
├── scores.json            # 最終評価スコア
└── blender_file.blend     # 最終Blenderファイル（3Dモード）
```

---

## 開発とカスタマイズ

### 新しいモードの追加

1. プロンプトを作成：

```bash
mkdir -p prompts/my_mode
touch prompts/my_mode/__init__.py
touch prompts/my_mode/generator.py
touch prompts/my_mode/verifier.py
mkdir -p prompts/my_mode/examples
```

2. ツールを追加（必要に応じて）：

```bash
touch tools/my_mode_exec.py
```

3. ランナーを作成：

```bash
touch runners/my_mode.py
```

4. データディレクトリを準備：

```bash
mkdir -p data/my_mode
```

### カスタムツールの追加

MCPツールサーバーを作成：

```python
# tools/my_custom_tool.py
import asyncio
from mcp.server import Server
from mcp.server.stdio import stdio_server

app = Server("my-custom-tool")

@app.call_tool()
async def my_tool(arg1: str, arg2: int) -> str:
    """ツールの説明"""
    # ツールの実装
    result = f"Processed {arg1} with {arg2}"
    return result

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(
            read_stream,
            write_stream,
            app.create_initialization_options()
        )

if __name__ == "__main__":
    asyncio.run(main())
```

### エージェントのカスタマイズ

#### メモリ長の調整

```bash
--memory-length 24  # より長いコンテキスト
--memory-length 6   # より短いコンテキスト（高速）
```

#### 最大ラウンド数の調整

```bash
--max-rounds 20   # より多くの反復
--max-rounds 5    # より少ない反復（高速）
```

#### ツールの選択

```bash
# 最小限のツールセット
--generator-tools tools/generator_base.py \
--verifier-tools tools/verifier_base.py

# フルツールセット
--generator-tools tools/blender/exec.py,tools/generator_base.py,tools/assets/meshy.py,tools/initialize_plan.py \
--verifier-tools tools/blender/investigator.py,tools/verifier_base.py
```

#### ツール無効化モード

```bash
--no-tools  # ツール呼び出しを無効化（テスト用）
```

### ローカルモデルの使用

#### vLLMサーバーの起動

```bash
# 専用の環境を作成（オプション）
conda create -n vllm python=3.10 -y
conda activate vllm
pip install -r models/requirements.txt

# サーバー起動
python models/server.py \
    --host 0.0.0.0 \
    --port 8000 \
    --model Qwen/Qwen2-VL-7B-Instruct \
    --tensor-parallel-size 1 \
    --gpu-memory-utilization 0.90
```

#### VIGAでローカルモデルを使用

```bash
conda activate agent

# ローカルAPIを指定
export OPENAI_BASE_URL="http://localhost:8000/v1"
export OPENAI_API_KEY="not-needed"

python main.py \
    --mode dynamic_scene \
    --model Qwen2-VL-7B-Instruct \
    --api-base-url http://localhost:8000/v1 \
    --api-key not-needed \
    --task artist
```

### デバッグとトラブルシューティング

#### ログレベルの調整

`main.py`または各ランナーの`logging`設定を変更：

```python
logging.basicConfig(
    level=logging.DEBUG,  # INFO から DEBUG に変更
    format="%(asctime)s - %(levelname)s - %(message)s",
)
```

#### ツールサーバーの直接テスト

```bash
# Blender execツールのテスト
conda activate blender
python tools/blender/exec.py
# MCPプロトコルでコマンド送信
```

#### 環境の確認

```bash
# Conda環境のリスト
conda env list

# アクティブな環境のパッケージ
conda list

# Python実行パスの確認
which python

# GPUの確認
nvidia-smi
```

### パフォーマンスチューニング

#### 並列実行の最適化

```bash
# CPUコア数に応じて調整
--max-workers $(nproc)

# GPUメモリに応じて調整
--max-workers 4  # 各タスクが2GB使用する場合、8GB GPUで4並列
```

#### GPUメモリの管理

```bash
# 使用するGPUを指定
export CUDA_VISIBLE_DEVICES=0,1

# GPU指定をスクリプトに渡す
python runners/blendergym/ours.py \
    --gpu-devices 0,1 \
    ...
```

#### Blenderレンダリングの高速化

`data/*/pipeline_render_script.py`のレンダリング設定を調整：

```python
# サンプル数を減らす（品質は下がるが高速）
bpy.context.scene.cycles.samples = 64  # デフォルト128

# 解像度を下げる
bpy.context.scene.render.resolution_x = 512  # デフォルト1024
bpy.context.scene.render.resolution_y = 512
```

---

## トラブルシューティング

### よくある問題と解決方法

#### 1. `ModuleNotFoundError`

**症状**: `ModuleNotFoundError: No module named 'xxx'`

**原因**: 間違った環境をアクティベートしている

**解決方法**:
```bash
# 正しい環境をアクティベート
conda activate agent  # または blender, pptx など

# 環境のパッケージを確認
conda list

# 必要に応じて再インストール
pip install -r requirements/requirement_agent.txt
```

#### 2. Infinigenインストール失敗

**症状**: `bash scripts/install/interactive_blender.sh`が失敗

**解決方法**:
```bash
# 依存パッケージをインストール
sudo apt-get install build-essential cmake

# Python 3.11を確認
python --version  # 3.11.x である必要がある

# 再試行
cd utils/third_party/infinigen
INFINIGEN_MINIMAL_INSTALL=True bash scripts/install/interactive_blender.sh

# エラーが出ても、utils/third_party/infinigen/blender が存在すればOK
ls -la blender/
```

#### 3. CUDA not found

**症状**: `RuntimeError: CUDA not available`

**解決方法**:
```bash
# GPUを確認
nvidia-smi

# CUDAバージョンを確認
nvcc --version

# PyTorchを正しいCUDAバージョンで再インストール
pip uninstall torch torchvision torchaudio
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

#### 4. PPTX変換失敗

**症状**: スライド生成は成功するがPDF変換が失敗

**解決方法**:
```bash
# LibreOfficeを再インストール
sudo apt-get remove --purge libreoffice-core
sudo apt-get install -y libreoffice unoconv

# LibreOfficeバージョンを確認
libreoffice --version
```

#### 5. 間違ったPythonパス

**症状**: `FileNotFoundError: [Errno 2] No such file or directory: '/path/to/python'`

**解決方法**:
```bash
# 正しいパスを確認
conda activate blender
which python  # /home/user/anaconda3/envs/blender/bin/python

# _path.pyを更新
nano utils/_path.py
# CONDA_BASEを正しいパスに設定
```

#### 6. APIキーエラー

**症状**: `AuthenticationError: Invalid API key`

**解決方法**:
```bash
# APIキーファイルを確認
cat utils/_api_keys.py

# APIキーが正しいか確認（OpenAIの場合）
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"

# _api_keys.pyを更新
nano utils/_api_keys.py
```

#### 7. メモリ不足

**症状**: `RuntimeError: CUDA out of memory`

**解決方法**:
```bash
# バッチサイズを減らす（該当する場合）
# モデルサーバーのGPUメモリ利用率を下げる
python models/server.py --gpu-memory-utilization 0.7

# 並列実行数を減らす
--max-workers 2

# 解像度を下げる（レンダリング時）
```

#### 8. ツールサーバー接続エラー

**症状**: `Failed to connect to tool server`

**解決方法**:
```bash
# _path.pyのパスを確認
cat utils/_path.py

# 手動でツールサーバーをテスト
conda activate blender
python tools/blender/exec.py
# Ctrl+Cで終了

# パーミッションを確認
ls -la tools/blender/exec.py
chmod +x tools/blender/exec.py
```

---

## 付録

### 環境変数リファレンス

| 変数名 | 説明 | デフォルト値 |
|--------|------|-------------|
| `OPENAI_API_KEY` | OpenAI APIキー | なし（必須） |
| `OPENAI_BASE_URL` | OpenAI互換API URL | https://api.openai.com/v1 |
| `MESHY_API_KEY` | Meshy APIキー | なし（必須） |
| `CLAUDE_API_KEY` | Anthropic APIキー | なし（オプション） |
| `GEMINI_API_KEY` | Google Gemini APIキー | なし（オプション） |
| `CUDA_VISIBLE_DEVICES` | 使用するGPU | すべて |
| `VIGA_CONDA_BASE` | Conda環境ベースパス | ~/anaconda3/envs |

### コマンドラインオプション完全リファレンス

#### main.py

```
usage: main.py [-h] --mode {blendergym,autopresent,blenderstudio,static_scene,dynamic_scene}
               [--model MODEL] [--api-key API_KEY] [--api-base-url API_BASE_URL]
               [--max-rounds MAX_ROUNDS] [--memory-length MEMORY_LENGTH]
               [--init-code-path INIT_CODE_PATH] [--init-image-path INIT_IMAGE_PATH]
               [--target-image-path TARGET_IMAGE_PATH] [--target-description TARGET_DESCRIPTION]
               [--output-dir OUTPUT_DIR] [--task-name TASK_NAME] [--assets-dir ASSETS_DIR]
               [--resource-dir RESOURCE_DIR] [--gpu-devices GPU_DEVICES] [--clear-memory]
               [--explicit-comp] [--no-tools]
               [--init-setting {none,minimal,reasonable}]
               [--prompt-setting {none,procedural,scene_graph,get_asset,init}]
               [--num-candidates NUM_CANDIDATES] [--blender-command BLENDER_COMMAND]
               [--blender-file BLENDER_FILE] [--blender-script BLENDER_SCRIPT]
               [--blender-save BLENDER_SAVE] [--meshy_api_key MESHY_API_KEY]
               [--va_api_key VA_API_KEY] [--browser-command BROWSER_COMMAND]
               [--generator-tools GENERATOR_TOOLS] [--verifier-tools VERIFIER_TOOLS]
```

主要なオプション：
- `--mode`: 実行モード（必須）
- `--model`: 使用するVLMモデル（デフォルト: gpt-4o）
- `--max-rounds`: 最大反復回数（デフォルト: 10）
- `--memory-length`: メモリ長（デフォルト: 12）
- `--generator-tools`: Generatorツールのカンマ区切りリスト
- `--verifier-tools`: Verifierツールのカンマ区切りリスト

### サポートされるモデル

#### OpenAI
- `gpt-4o`: 最新の汎用モデル（推奨）
- `gpt-4-turbo`: 高性能モデル
- `gpt-4o-mini`: 軽量モデル

#### Anthropic
- `claude-sonnet-4`: バランスの取れたモデル
- `claude-opus-4.5`: 最高性能モデル

#### Google
- `gemini-2.5-pro`: Gemini最新モデル
- `gemini-2.0-flash`: 高速モデル

#### Qwen
- `qwen-vl-max`: Qwen最高性能モデル
- `qwen-vl-plus`: Qwen標準モデル

#### ローカル（vLLM）
- `Qwen/Qwen2-VL-7B-Instruct`: ローカル実行可能

### ディレクトリ構造の凡例

```
📁 ディレクトリ
📄 ファイル
🐍 Pythonスクリプト
⚙️ 設定ファイル
📊 データファイル
📝 ドキュメント
```

### 参考リンク

#### 公式リソース
- [プロジェクトWebサイト](https://fugtemypt123.github.io/VIGA-website/)
- [arXiv論文](https://arxiv.org/abs/2601.11109)
- [BlenderBenchデータセット](https://huggingface.co/datasets/DietCoke4671/blenderbench)

#### 関連プロジェクト
- [BlenderGym](https://blendergym.github.io/)
- [AutoPresent](https://github.com/para-lost/AutoPresent)
- [Infinigen](https://github.com/princeton-vl/infinigen)
- [Segment Anything](https://segment-anything.com/)

#### ツールとライブラリ
- [Model Context Protocol (MCP)](https://github.com/modelcontextprotocol)
- [OpenAI API](https://platform.openai.com/)
- [vLLM](https://vllm.ai/)
- [Blender](https://www.blender.org/)

---

## ライセンス

このプロジェクトはMITライセンスの下で公開されています。詳細は[LICENSE](../LICENSE)ファイルを参照してください。

## 引用

このプロジェクトを研究に使用する場合は、以下のように引用してください：

```bibtex
@misc{yin2026visionasinversegraphicsagentinterleavedmultimodal,
      title={Vision-as-Inverse-Graphics Agent via Interleaved Multimodal Reasoning},
      author={Shaofeng Yin and Jiaxin Ge and Zora Zhiruo Wang and Xiuyu Li and Michael J. Black and Trevor Darrell and Angjoo Kanazawa and Haiwen Feng},
      year={2026},
      eprint={2601.11109},
      archivePrefix={arXiv},
      primaryClass={cs.CV},
      url={https://arxiv.org/abs/2601.11109},
}
```

---

**最終更新**: 2026年2月
**バージョン**: 1.0
**メンテナー**: VIGA開発チーム
