# Claude Code中心の「開発・業務OS」拡張設計（prompt-review × cc-company 前提）

## A. 2つのリポジトリの比較

### 目的

**prompt-review**
- **事実（リポジトリより）**: Claude Codeのスキルとして動作し、過去のAIエージェント対話履歴（複数ツールのログ）を自動収集・分析して、技術理解度／プロンプティングパターン／AI依存度を推定し、日本語レポートを生成するツール。citeturn25view0turn24view0turn9view0  
- **事実（リポジトリより）**: `/prompt-review` を実行すると `reports/prompt-review-YYYY-MM-DD.md` を出力し、シークレット検出時は警告セクションも出す。citeturn24view0turn10view0turn7view1

**cc-company**
- **事実（リポジトリより）**: Claude Codeで仮想会社組織を構築・運営するプラグイン。`/company` を入口に「秘書が窓口→CEOが振り分け→各部署がファイル運用」という形で日常運営を回す。citeturn25view1turn24view1turn16view2  
- **事実（リポジトリより）**: 初回はオンボーディング（事業・目標・困りごと→部署提案→保存場所→言語）を対話で行い、その後 `.company/` 配下にルール・テンプレ・部署フォルダ等を自動生成する。citeturn24view1turn23view2turn16view1turn16view0

### 強み

**prompt-review**
- **事実（リポジトリより）**: 「成果物」ではなく「プロンプト（意図）」を見ることで、本人の理解/認識を推定して指導・改善に繋げる、という思想が明確。citeturn25view0  
- **事実（リポジトリより）**: Claude Codeのログ（`~/.claude/history.jsonl` と `~/.claude/projects/.../*.jsonl`）を含む複数ツールの保存場所・形式を整理し、収集スクリプトで自動検出する。citeturn9view0turn7view1turn24view0  
- **事実（リポジトリより）**: クレデンシャル・シークレット検出の仕組み（APIキー/トークン等のパターン検出とマスク）が入っている。citeturn7view1turn24view0

**cc-company**
- **事実（リポジトリより）**: 「秘書が常に窓口で、ユーザーは部署を意識しなくていい」というUX設計が強い。日常の“思いつき→相談→ファイル化”を一本道にできる。citeturn24view1turn25view1  
- **事実（リポジトリより）**: `.company/` 内に、TODO（日次）、Inbox（クイックキャプチャ）、Notes（壁打ち）、Decisions（意思決定ログ）、レビュー、各部署のテンプレを揃え、ファイル駆動で運用できる。citeturn24view1turn16view1turn16view0  
- **事実（リポジトリより）**: `.company/` の作成場所を「カレント／ホーム／カスタムパス」から選べる。citeturn24view1turn23view2  
- **事実（リポジトリより）**: カスタム部署追加もOK、という拡張性が明示されている。citeturn24view1turn23view2

### 弱み

**prompt-review**
- **事実（リポジトリより）**: 「運用を回すためのタスク/意思決定/ナレッジの流れ」を直接提供するのではなく、“分析レポート生成”が主。運用OSの実働部は別途必要。citeturn25view0turn24view0  
- **事実（リポジトリより）**: Python 3.10+ と、Copilot Chat解析にはSQLite3が必要（環境依存が増える）。citeturn25view0  
- **設計上の懸念（推測）**: ログ解析は「個人情報・機密」が混ざりやすい。OS統合するなら、解析対象・保存先・公開範囲（AIに読ませる/読ませない）を強制する仕組みが必要。

**cc-company**
- **事実（リポジトリより）**: デフォルト構造は `.company/` 中心。あなたが求める「Projects / Inbox / Knowledge / SOP / Decisions / Prompts…」の“情報分類中心ツリー”とは目的が少し違うため、そのままだと二重構造になりやすい。citeturn24view1turn16view1  
- **設計上の懸念（推測）**: すべてを秘書→CEO→部署で回すと、開発フロー（仕様→実装→検証→出荷）に必要な「品質ゲート」「記憶整理」「意思決定の型」が不足しがち。追加レイヤーが欲しくなる。

### 向いている用途

**prompt-review（向いていること）**
- **事実（リポジトリより）**: 期間指定やプロジェクト指定で、プロンプトの傾向と成長軌跡をレポート化して振り返る。citeturn25view0turn24view0  
- **設計提案（推測）**: 週次/月次の“プロンプト品質レビュー”＝運用OSの改善（SOPの更新、コマンドの追加/削除、CLAUDE.mdの整理）の根拠作りに最適。

**cc-company（向いていること）**
- **事実（リポジトリより）**: 日々の相談、TODO管理、壁打ち、メモ、部署別のファイル整理を「秘書窓口」で回し続ける。citeturn25view1turn24view1turn16view1  
- **設計提案（推測）**: 生活・業務の“司令塔”として、入力（思いつき）をInboxに入れ、CEOが各フォルダへ振り分ける運用に強い。

### 組み合わせたときのシナジー

- **事実（観測できる補完関係）**: cc-companyは「運用する器（組織とファイル）」を作り、prompt-reviewは「その運用をどう改善するか（あなたのプロンプト傾向・理解度・依存傾向）」を可視化する。citeturn25view1turn25view0turn24view1turn24view0  
- **設計提案（推測）**:  
  - cc-companyで毎日“意思決定ログ/タスク/設計メモ”がファイル化される → その結果、prompt-reviewの分析対象（プロンプト）が「より意図が明確」になり、改善点が具体化する。  
  - prompt-reviewが出す「曖昧指示の癖」「依存度が高い局面」「繰り返す課題」→ cc-company側の“部署ルール（CLAUDE.md）”や“スラッシュコマンド”にフィードバックして再発防止のSOP化ができる。citeturn24view0turn16view0turn16view1  

## B. 私向けの推奨アーキテクチャ案を3案

### 案1: 最小で早く動く

**構成要素（実装可能性優先）**
- 中央OSフォルダ（後述Fのルート構成）を `~/YourOS/` のように1つ作る（ローカルPC中心）。
- cc-companyをインストールし、中央OSフォルダに `.company/` を生成して“秘書窓口”を確保。citeturn25view1turn24view1turn23view2  
- prompt-reviewを **個人スキル**として `~/.claude/skills/prompt-review/` に配置し `/prompt-review` を使えるようにする（スキルは `~/.claude/skills/<skill>/SKILL.md` で全プロジェクト横断で使える）。citeturn24view0turn21view2  
- Claude Code標準の `/add-dir` で、普段の開発プロジェクトに「中央OSフォルダ」を追加して参照可能にする。citeturn21view0  
- 追加で作る自作コマンドは「まず10個だけ」（Eで提示）を **個人スキル**で作る。citeturn21view2turn18view0  

**エージェントの役割**
- 人間（社長）: 方向性・最終判断（出荷/外部送信/重大な権限操作）。
- 秘書（cc-company）: 入力を受ける、TODO/Inbox/相談、必要ならCEOへ。citeturn24view1turn16view1  
- CEO（cc-company）: 部署に振り分け、意思決定ログを残す。citeturn24view1turn16view1  
- prompt-review（分析役）: 週次であなたのプロンプト運用の改善点を提示。citeturn24view0turn25view0  

**データの流れ**
1) 思いつき → `/company` で秘書に話す → `.company/secretary/inbox` または中央OSの `Inbox/` へ記録（後述Fの運用ルールに合わせて、生成後に秘書ルールを“保存先切替”するのがコツ）。citeturn24view1turn16view0turn16view1  
2) 具体化が必要 → CEO振り分け → `Projects/` / `Decisions/` / `SOP/` にファイル化。citeturn24view1turn16view1  
3) 開発プロジェクトで作業 → `/add-dir ~/YourOS` して、Claude CodeからOS参照しながら実装。citeturn21view0  
4) 週末 → `/prompt-review 7` 等でレポート生成 → ルール/コマンド改善。citeturn25view0turn24view0  

**メリット**
- 最短で「秘書窓口」と「自己分析」を手に入れられる（明日から動く）。
- 追加の仕組みはスキル（=スラッシュコマンド）に寄せられる（保守しやすい）。citeturn18view0turn17view0  

**デメリット**
- `.company/` とあなたの理想フォルダ（Projects等）の“二重構造”が最初は発生しやすい（のちに統合設計が必要）。
- 自動化（フック/ゲート/索引）がまだ弱い。

**導入難易度**
- 低（ファイル＋スキル＋既存プラグイン）。

### 案2: 実務でかなり使える

**構成要素（統合度を上げる）**
- 中央OSフォルダをGit管理（公開しない前提、ローカルバックアップも併用）。
- cc-companyの `.company/CLAUDE.md` と各部署 `CLAUDE.md` を **あなたのフォルダ設計（F）に合わせて再定義**（保存先・命名・タグ・追記ルール）。テンプレも差し替え。citeturn16view0turn16view1turn24view1  
- 自作スキル（10個＋α）を「中央OS側」にも置き、プロジェクトへは `/add-dir` で読み込ませる（追加ディレクトリ内の `.claude/skills` は自動ロードされる）。citeturn21view2turn21view0  
- Claude Codeの**フック**で「AI-blockedへの書き込みブロック」「編集後テスト自動実行」「タイムスタンプ自動付与」などを実装（“理想論”でなく運用負荷を下げるため）。citeturn19view3turn19view2turn22view2  
- prompt-reviewを「週次レビューSOP」に組み込み、結果を `Reviews/`（または `.company/reviews`）へ集約する。citeturn24view0turn25view0turn24view1  

**エージェントの役割**
- 社長: “何をやるか/やらないか”と最終承認。
- 秘書: 受付・整理・日次の管理（最も頻繁に使う）。citeturn24view1turn16view1  
- CEO: 振り分け・意思決定ログの整合性維持。citeturn16view1turn16view0  
- 新規レイヤー（Cで深掘りする上位3役）:
  - Spec Writer（仕様化）
  - Quality Gate（検証・レビュー）
  - Memory Curator（記憶/文脈の整形）

**データの流れ（“迷わない導線”を固定）**
- 入口は常に `/company` or `/inbox`（→ Inboxへ）。
- Inbox→（整理）→ Projects / Knowledge / SOP / Decisions いずれかへ“昇格”。
- 重要な判断は必ず Decisions に残し、Projects側からリンク。
- 参照の基点は「AI-readable」側（後述F）に寄せる（Claude Codeが読みやすい状態で“正”を置く）。

**メリット**
- 「思いつき→実行」導線が短いまま、データが育つ（蓄積が再利用可能）。
- フックにより“手で守るルール”が減る（運用負荷が下がる）。citeturn19view3turn22view2  

**デメリット**
- フック設計を雑にやると逆に詰む（誤検知・過剰自動化・不具合時の解除が面倒）。
- ルールの良し悪しが出る（最初の数週間はチューニングが必要）。

**導入難易度**
- 中（スキル＋テンプレ＋フックの最小実装）。

### 案3: 究極形に近いが重い

**構成要素（司令塔に寄せる）**
- 案2に加えて、ローカル **MCPサーバ（stdio）** を導入し、OSフォルダ（＋必要なら複数プロジェクト）を索引化して「検索・参照・集計」を道具として提供する。citeturn21view4turn17view4  
- 複数サブエージェント/チームを常用（探索・レビュー・テスト・文章化を並列化）。Claude Codeはサブエージェントで“コンテキスト隔離”できる。citeturn17view0turn18view3turn22view3  
- プラグイン化して、スキル/フック/MCP/サブエージェントを“全部入り”で配布可能な形に束ねる（あなたの環境の再現性が最大化）。citeturn17view0turn21view3turn21view4  
- スケジュール実行（Claude Codeの `/loop` など）で、定期集計（週次レビュー、Inboxゼロ、PR監視）を自動化。citeturn18view0turn20view0  

**エージェントの役割**
- 「秘書/CEO/部署」＋「Spec/Quality/Memory」＋「索引（MCP）」＋「監査（セキュリティ/権限）」が揃う。

**データの流れ**
- すべての作業が「コマンド（スキル）→ファイル（データ層）→索引（検索層）」で循環。
- 重要なアウトプットは必ずAI-readableに“要約版”が落ち、長期記憶になる。

**メリット**
- “司令塔”に一気に近づく（複数プロジェクト横断の文脈管理が現実的になる）。
- 検索・集計が高速化し、Claude Codeのコンテキスト節約にも効く。

**デメリット**
- 仕組みが重い＝故障点が増える（MCPの接続、索引の更新、権限、互換性）。
- 初期設計が甘いと「便利なはずが管理地獄」になりやすい。

**導入難易度**
- 高（実装・保守・デバッグのコストが乗る）。  

## C. 「社長 / 秘書 / CEO」の間に挟むべき役割候補

### 役割候補（少なくとも5案）

**候補1: Chief of Staff（参謀/幕僚）**
- **何を判断する役割か（提案）**: 社長の“目的”を、優先順位と実行可能な粒度に落とす（どれを今週やるか、やらないか）。
- **入力**: 社長の意図・制約（時間、締切、予算、気分）、現在のProjects/Inbox状況。
- **出力**: 今週のフォーカス（3つまで）、捨てるもの、秘書への運用指示（例: Inbox整理の基準）。
- **呼ぶタイミング**: 週次開始、方針転換時、タスク雪崩時。
- **効き方（提案）**: Claude Codeに投げる仕事が「今やる価値のあるもの」だけになり、迷走が減る。

**候補2: Dispatcher（案件トリアージ/配車係）**
- **何を判断する役割か（提案）**: 入力を「秘書で即応」「CEO判断」「要追加質問」「保留」に振り分ける。
- **入力**: 社長の一言、関連ファイル（Inbox/Projects/Decisions）。
- **出力**: ルーティング結果と、次に投げるコマンド（/spec、/task など）。
- **呼ぶタイミング**: 毎回の思いつき入力直後。
- **効き方（提案）**: “秘書が全部受ける”を維持しつつ、CEOが過負荷にならない。

**候補3: Spec Writer（仕様化担当/要件定義）**
- **何を判断する役割か（提案）**: 「何を作るか」を“検証可能な仕様（受け入れ条件付き）”に固定する。  
- **入力**: アイデア、既存コード/制約、過去の決定（Decisions）、SOP。
- **出力**: Specファイル、受け入れ条件、作業分解、テスト/検証手順。  
- **呼ぶタイミング**: 実装前（特に複数ファイル変更・新機能・不具合調査）。  
- **効き方（根拠＋提案）**: Claude Codeは「検証できる条件」があると劇的に安定する、という推奨があるため、ここを役割化すると成功率が上がる。citeturn22view1  

**候補4: Quality Gate（検証/レビュー責任者）**
- **何を判断する役割か（提案）**: “出荷して良いか”を基準化（テスト、差分、セキュリティ観点）。
- **入力**: git diff、テスト結果、仕様（Spec）。
- **出力**: GO/NOGO、修正指示、リスク一覧。
- **呼ぶタイミング**: コミット前、PR作成前、リリース前。
- **効き方（根拠＋提案）**: Claude Code運用では「テスト等で自己検証させる」のが最大レバレッジとされるため、最後に必ず通す“ゲート”が効く。citeturn22view1turn20view0  

**候補5: Memory Curator（記憶/文脈マネージャ）**
- **何を判断する役割か（提案）**: “残すべき情報”を決め、AIが読める形（短く、リンク付き、再利用可能）に整形して所定の場所へ保存する。  
- **入力**: 会話の結論、実装の要点、決定理由、未解決点。
- **出力**: Decisions更新、SOP更新、Knowledge要約、Handoff作成。
- **呼ぶタイミング**: タスク完了時、/compact 前後、週次レビュー前。
- **効き方（根拠＋提案）**: Claude Codeは各セッションが“新しいコンテキスト”から始まり、持ち越しはCLAUDE.mdやメモリに依存するため、文脈の外部化を担う役が長期運用で効く。citeturn21view1turn17view1  

**候補6: Risk & Privacy Officer（情報境界/秘密管理）**
- **何を判断する役割か（提案）**: どこまでAIに読ませるか、何をAI-blockedへ退避するか、秘密情報の扱い。
- **入力**: 取り込むファイル、ログ、貼り付けたテキスト。
- **出力**: マスク/移動指示、秘密ローテーション提案、ルール更新。
- **呼ぶタイミング**: 外部からコピペした時、鍵情報に触れた時、週次監査。
- **効き方（根拠＋提案）**: prompt-review側に“秘密検出→警告”があるので、OS側でも境界の自動化（フック）とセットで運用できる。citeturn7view1turn24view0turn19view2  

**候補7: Prompt Coach（プロンプト改善コーチ）**
- **何を判断する役割か（提案）**: あなたのプロンプト癖を見て、テンプレ化・短縮・制約の入れ方を指導。
- **入力**: prompt-reviewレポート、失敗事例（やりとり）。
- **出力**: 新しいプロンプト雛形、ルール修正、コマンド改善案。
- **呼ぶタイミング**: 週次/月次。
- **効き方（根拠＋提案）**: prompt-reviewの目的そのものを“運用改善に接続”する役。citeturn25view0turn24view0  

### 最も効果が高い役割を3つに絞る（深掘り）

ここでは「日常的にClaude Codeで開発し、速く・安定し・再利用しやすくする」観点で、**Spec Writer / Quality Gate / Memory Curator** を推奨します。根拠は、(1)検証可能性が成果を左右する、(2)セッションが揮発するため外部化が必要、(3)最後の品質ゲートが事故率を下げる、の3点です。citeturn22view1turn21view1turn17view1turn20view0  

**役割1: Spec Writer（仕様化担当）**
- **判断責任**: 「このタスクは何を満たせば成功か（受け入れ条件）」を確定する。未確定なら“質問して止める”責任を持つ。
- **受け取る情報**:  
  - 社長の要望（自然文でOK）  
  - 既存のDecisions（過去の前提）  
  - 対象コードの制約（技術/期限/安全）  
- **返す情報**:  
  - `Spec`（1ファイル）  
  - 受け入れ条件（チェックリスト）  
  - 実装タスク分解（5〜15個程度）  
  - 検証手順（テストやコマンド）  
- **他エージェントとの関係**:  
  - 秘書: 入力を受け、Spec Writer呼び出し。  
  - CEO: Specを見て部署へ割り振り。  
  - Quality Gate: Specの受け入れ条件を“検証軸”として使う。  
- **具体的な利用シーン**:  
  - 新機能追加、バグ修正、リファクタ、設計見直しなど「差分が一文で言えない」時。citeturn22view1turn22view0  
- **プロンプト雛形（そのままスキル/サブエージェントに転用可）**:
```text
あなたは Spec Writer です。目的は「実装前に、成功条件が検証可能なSpecを1枚に固定する」ことです。

入力:
- 要望: {{USER_REQUEST}}
- 対象プロジェクト: {{PROJECT}}
- 制約: {{CONSTRAINTS}}
- 参考: Decisions/ と SOP/ と Knowledge/ を必要に応じて参照

手順:
1) 不明点を最大5つに絞って質問する（致命的な不明点がなければ質問ゼロで進めてよい）
2) Specを作成する（フォーマットは後述）
3) 受け入れ条件を「チェック可能」な形で列挙する
4) タスクを分割し、順序と依存関係を明記する
5) 検証方法（テスト/コマンド/画面確認）を必ず書く

出力フォーマット:
- Specタイトル
- 背景
- 目的（1文）
- 非目的（やらないこと）
- 受け入れ条件（チェックリスト）
- タスク分解（番号付き）
- 検証手順（コマンド例 or 手順）
- リスク/未確定点（あれば）
```

**役割2: Quality Gate（検証・レビュー責任者）**
- **判断責任**: 「GO/NOGO」を出す。妥協する場合は“リスクと理由”を明文化してDecisionsに残す。
- **受け取る情報**: git diff、Specの受け入れ条件、テスト結果（または実行手順）。
- **返す情報**:  
  - GO/NOGO  
  - 修正指示（優先度付き）  
  - 既知リスク（軽減策付き）  
- **他エージェントとの関係**:  
  - Spec Writer: 受け入れ条件の質がゲートの精度を決める。  
  - CEO: GOならコミット/出荷へ、NOGOなら戻す。  
- **具体的な利用シーン**:  
  - コミット直前、PR直前、リリース直前。  
  - “Claudeに任せた変更”ほど必須（人間の脳内レビューは限界があるため）。citeturn22view1turn22view2turn20view0  
- **プロンプト雛形**:
```text
あなたは Quality Gate です。目的は「出荷事故を減らすため、差分と検証をもとにGO/NOGOを出す」ことです。

入力:
- Spec(受け入れ条件): {{SPEC_PATH}}
- 変更差分: git diff / 関連ファイル
- 実行したテスト/検証: {{VERIFICATION}}

手順:
1) 受け入れ条件を満たしているかチェック
2) 重大リスク（セキュリティ/データ破壊/互換性）を洗い出す
3) テストが不足なら「最低限ここは実行」と指示
4) GO/NOGO を結論として最初に宣言し、その根拠を短く列挙
5) NOGOの場合は「最短でGOにする修正順」を提示

出力:
- 結論: GO または NOGO
- 根拠（箇条書き3〜7）
- 修正指示（優先度: P0/P1/P2）
- 残リスク（あれば）
```

**役割3: Memory Curator（記憶/文脈マネージャ）**
- **判断責任**: 「この会話/変更のうち、未来の自分が再利用する情報は何か」を決める。残さないなら捨てる。
- **受け取る情報**: 実装の結論、決定理由、やり残し、学び、更新したファイル群。
- **返す情報**:  
  - Decisionsへの追記/新規作成  
  - SOP更新（再現手順/チェックリスト）  
  - Knowledge要約（短い“正”）  
  - Handoff（次回再開用メモ）  
- **他エージェントとの関係**:  
  - 秘書: 会話の最後に「記録しますか？」のトリガー役。  
  - CEO: 意思決定ログに責任。  
- **具体的な利用シーン**:  
  - タスク完了時、コンテキストが膨れた時（/compactの前後）、週次レビュー前。citeturn21view1turn20view0turn19view1  
- **プロンプト雛形**:
```text
あなたは Memory Curator です。目的は「未来の自分とClaudeが迷わない最小の記憶を、所定の場所に残す」ことです。

入力:
- この作業で起きたこと: {{SUMMARY}}
- 重要な決定: {{DECISIONS}}
- 変更点: {{CHANGES}}
- 未完了/宿題: {{OPEN_ITEMS}}

ルール:
- 長文ログは残さない。要点+リンク中心。
- 重要判断は Decisions/ に必ず残す。
- 再発しそうなものは SOP/ 化する。
- Claudeが読む“正”は AI-readable/ 側に短く置く。

出力:
1) 残すべきもの一覧（Decision/SOP/Knowledge/Handoff）
2) それぞれの保存先パス案
3) ファイルに書く内容（短い本文）
```

## D. /スラッシュコマンドの自作戦略

### Claude Codeでネイティブにできること（事実）

- **スキル（Skills）を作ると `/skill-name` が生える**。スキルは `SKILL.md`（YAMLフロントマター＋本文）で定義し、個人スコープなら `~/.claude/skills/<skill>/SKILL.md` に置く。citeturn18view0turn21view2  
- **自作コマンド（`.claude/commands/`）はスキルに統合された**。旧コマンドも動くが、今後はスキルが推奨。citeturn18view0  
- **スキルには“いつ/誰が呼べるか”の制御がある**（例: `disable-model-invocation: true` で自動起動させない）。副作用のあるコマンド向け。citeturn18view0  
- **`allowed-tools` でスキル中に使えるツールを制限できる**（安全性・安定性に効く）。citeturn18view0  
- **`context: fork` と `agent:` によりサブエージェントで隔離実行できる**（大量探索/レビューを主会話から切り離す）。citeturn18view0turn17view0  
- **動的コンテキスト注入**（`!`バッククオートコマンド）が使える：シェル実行結果をスキル本文へ埋め込める。citeturn18view0  
- **CLAUDE.md と自動メモリ（auto memory）**で、セッションを跨ぐ“持ち越し”ができる（ただしコンテキストであり強制ではない）。citeturn21view1turn17view1  
- **フック（Hooks）**で、特定イベント時に決定論的スクリプトを実行し、場合によってはブロック/フィードバックもできる（例: 編集後にテスト実行、特定ディレクトリへの書き込みブロック）。citeturn19view3turn19view2turn19view1  
- **プラグイン（Plugins）**で、スキル/フック/サブエージェント/MCP等を束ねられる。テストは `--plugin-dir` で可能。citeturn21view3turn19view4turn17view0  
- **MCP**で外部・ローカルツール連携ができ、ローカルstdioサーバも追加可能。citeturn21view4turn17view4  

### できない場合の代替案（現実路線の整理）

- **「強制的に守らせるルール」**は、CLAUDE.mdだけだと“従う努力”に留まる（コンテキスト扱い）。強制が必要ならフックでブロック/自動化に寄せるのが筋。citeturn21view1turn19view2turn22view4  
- **「巨大な知識ベースを毎回読み込ませる」**はコンテキストを圧迫し精度が落ちる。CLAUDE.mdは短く、必要時だけスキル/参照ファイルを読み込ませる（分割）ほうが安定。citeturn21view1turn17view0turn22view4  

### テンプレート化 / シェルスクリプト化 / MCP化 / ローカルCLI化 / タスクランナー化（比較）

- **テンプレート化（推奨: 早い）**: スキル配下に `templates/` を置いて参照させる。cc-companyの departmentsテンプレの思想を踏襲できる。citeturn16view1turn18view0  
- **シェルスクリプト化（中）**: 反復処理（フォルダ作成、日付ファイル生成、バックアップ、索引更新）を“決定論的”にする。フックから呼ぶと運用負荷が下がる。citeturn19view3turn19view1  
- **MCP化（重いが強い）**: 検索/索引/集計/外部APIを“ツール”にしてClaudeに渡す。ローカルstdioはローカル中心運用と相性が良い。citeturn21view4turn17view4  
- **ローカルCLI化（中）**: Claudeを開かずに“瞬間キャプチャ”するための小コマンド（例: `os inbox "..."`）を作る。あなたの「思いつき→即実行」需要に刺さるが、二重UIになりがちなので最小限に。
- **タスクランナー化（中〜重）**: 週次レビュー、Inboxゼロ、アーカイブなどを自動化。Claude Code側にも `/loop` 等があり、軽い定期処理ならまずそこからが現実的。citeturn18view0turn20view0  

### 提案: 「最も現実的な方法」と「将来的に最強な方法」

**最も現実的（明日から）**
- 中央OSフォルダ + cc-company + prompt-review + “個人スキル10個”で開始。  
- 強制が必要なルール（AI-blockedへの書き込み禁止、編集後テスト）は“フックを2〜3本だけ”に絞る。citeturn19view3turn22view2  

**将来的に最強（ただし重い）**
- 自作プラグインに統合（スキル/フック/サブエージェント/テンプレ集）。citeturn17view0turn21view3turn19view4  
- ローカルMCPで索引・検索・集計を道具化し「司令塔」の実体を作る。citeturn21view4turn17view4  

## E. 実際に作るべきコマンド案

### コマンド案（最低20個）

以下は**すべて「スキル（Skills）」で実装可能**な粒度を基準にしています（必要に応じてフック/スクリプト/MCPへ拡張）。citeturn18view0turn19view3turn21view4  

1) **/company**  
- 目的: 会社OS（秘書→CEO→部署）起動・運営  
- 入力: 相談内容 / 初回はヒアリング  
- 出力: `.company/` 生成、日次TODO、Inbox、部署フォルダ  
- 実装方法候補: 既存プラグイン（cc-company）  
- 具体例: `/company` で「今日やること教えて」citeturn25view1turn24view1  

2) **/prompt-review**  
- 目的: 自分のプロンプト運用を分析し、改善点を出す  
- 入力: `[project] [days]`  
- 出力: `reports/prompt-review-YYYY-MM-DD.md`  
- 実装方法候補: 既存スキル（prompt-review）  
- 具体例: `/prompt-review 30`citeturn25view0turn24view0  

3) **/inbox**  
- 目的: 思いつきを“即・記録”  
- 入力: 1行〜数行メモ  
- 出力: `Inbox/YYYY/YYYY-MM-DD.md` に追記  
- 実装: スキル（Write）＋タイムスタンプ付与  
- 例: `/inbox 新しい案: スラッシュコマンドを10個から始める`  

4) **/triage**  
- 目的: Inboxの未整理を整理（Projects/Knowledge/SOP/Decisionsへ昇格）  
- 入力: 日付または “today”  
- 出力: 仕分け結果＋移動/コピー/リンク作成  
- 実装: スキル（Read/Grep/Write）  
- 例: `/triage today`  

5) **/spec**  
- 目的: 仕様化（Spec Writer）  
- 入力: やりたいこと  
- 出力: `Projects/<project>/Specs/YYYY-MM-DD--slug.md`  
- 実装: スキル（質問→テンプレ生成）  
- 例: `/spec 認証周りにrate limitを追加したい`  

6) **/task**  
- 目的: タスク化（1件追加/更新）  
- 入力: タスク内容 + 期限 + 優先度  
- 出力: `Projects/<project>/Tasks/` へ追記  
- 実装: スキル（Write）  
- 例: `/task p:alpha "ログイン失敗時の表示修正" due:2026-03-20 prio:high`  

7) **/next**  
- 目的: 次にやること提示（今日の最優先3つ）  
- 入力: optional（対象プロジェクト）  
- 出力: 今日のNext Actions + 参照リンク  
- 実装: スキル（Read→要約）  
- 例: `/next p:alpha`  

8) **/decide**  
- 目的: 意思決定ログ作成（Decision Record）  
- 入力: タイトル + 判断 + 理由（短く）  
- 出力: `Decisions/YYYY/YYYY-MM-DD--slug.md`  
- 実装: スキル（テンプレ埋め）  
- 例: `/decide "DB移行は今期やらない" reason:"リスク高/工数不足"`  

9) **/review-diff**  
- 目的: diffレビュー（Quality Gateの入口）  
- 入力: optional（焦点）  
- 出力: 指摘＋修正案＋NOGO/GO（暫定）  
- 実装: スキル（Bash git diff）  
- 例: `/review-diff セキュリティと例外処理中心で`  

10) **/verify**  
- 目的: 検証（テスト/コマンド/手順）を実行計画として固定  
- 入力: “何を通すか”  
- 出力: `SOP/dev/verification.md` への追記 or 実行ログ保存  
- 実装: スキル（Bash許可コマンドのallowlist推奨）citeturn22view2  
- 例: `/verify unit tests + lint`  

11) **/ship**  
- 目的: 出荷手順（テスト→コミット→PR）  
- 入力: メッセージ、対象範囲  
- 出力: コミット/PR/リリースメモ（環境に応じて）  
- 実装: スキル（disable-model-invocation推奨）citeturn18view0turn22view2  
- 例: `/ship "fix: login error message"`  

12) **/handoff**  
- 目的: 中断/引き継ぎメモ（次回すぐ再開）  
- 入力: “今どこで止めるか”  
- 出力: `Scratch/Handoffs/YYYY-MM-DD--slug.md`  
- 実装: スキル  

13) **/sop**  
- 目的: SOP作成/更新（再発防止の手順化）  
- 入力: 事象 + 手順  
- 出力: `SOP/<area>/...md`  
- 実装: スキル（テンプレ）  

14) **/knowledge**  
- 目的: Knowledge化（短い“正”）  
- 入力: 学び/結論  
- 出力: `Knowledge/<domain>/...md`  
- 実装: スキル（要約）  

15) **/search**  
- 目的: OS内検索（導線固定）  
- 入力: キーワード  
- 出力: ヒット一覧＋次のアクション  
- 実装: スキル（Bash rg）  

16) **/daily**  
- 目的: 今日のダッシュボード生成（Inbox数、Tasks、ブロッカー）  
- 入力: optional  
- 出力: `Inbox/YYYY/YYYY-MM-DD.md` 冒頭にサマリー追記 or `Scratch/Daily/`  
- 実装: スキル  

17) **/weekly**  
- 目的: 週次レビュー（完了/未完/学び/次週）  
- 入力: week（省略時は今週）  
- 出力: `Archive/Reviews/YYYY-WXX.md`  
- 実装: スキル（＋prompt-review呼び出しは将来統合）  

18) **/archive**  
- 目的: 終了プロジェクト/古いメモをArchiveへ移動  
- 入力: 対象  
- 出力: 移動＋リンク維持  
- 実装: スキル（Bash mv だと危険なので段階化）  

19) **/secrets-scan**  
- 目的: 生成物/ログ/差分の秘密検出（事故防止）  
- 入力: パス or diff  
- 出力: 警告＋マスク案  
- 実装: スキル（prompt-reviewの検出パターン流用）citeturn7view1turn24view0  

20) **/memory-pack**  
- 目的: Memory Curatorとして“残すべきもの”を一括作成  
- 入力: 今回の作業まとめ  
- 出力: Decisions/SOP/Knowledge/Handoff をまとめて生成  
- 実装: スキル（複数Write）  

21) **/context-pack**  
- 目的: 開発プロジェクトに必要なOS文脈を1枚にまとめ、CLAUDE.mdや参照ファイルを更新  
- 入力: project名  
- 出力: `Projects/<project>/Context/` 更新  
- 実装: スキル（CLAUDE.mdは短く保つ指針に従う）citeturn22view4turn21view1  

22) **/decision-link**  
- 目的: Spec/TaskからDecisionへのリンクを自動整備  
- 入力: 対象ファイル  
- 出力: 相互リンク追記  
- 実装: スキル  

23) **/retro**  
- 目的: 失敗/詰まりの振り返りをテンプレで残す  
- 入力: 事象  
- 出力: `Archive/Retros/`  
- 実装: スキル  

24) **/setup-os**  
- 目的: OSフォルダ初期化（ツリー生成、CLAUDE.md雛形、README）  
- 入力: ルートパス  
- 出力: ルート構成一式  
- 実装: スキル（Bash mkdir + テンプレWrite）  

25) **/rules**  
- 目的: AIに読ませる/読ませない境界・命名規則を表示  
- 入力: none  
- 出力: ポリシー表示＋リンク  
- 実装: スキル（Read）  

### 最初に作るべき10個（※前回案がこのスレッド内に無いので、上の案から選定）

ここでは「明日からローカルで着手でき、運用負荷を増やさず、成長する仕組みに直結する」ものを選びます。

対象10個: **/setup-os, /company, /inbox, /triage, /spec, /task, /next, /decide, /review-diff, /handoff**（+ 早期に /prompt-review を“導入”）  
※ /prompt-review は“作る”というより“導入＋統合”ですが、初期に入れる価値が高いので7日計画に組み込みます。citeturn25view0turn24view0  

以下、各コマンドの設計（仕様/例/I-O/失敗/保存先/接続/MCP拡張）。

**1) /setup-os**
- **仕様**: 指定ルートにFのフォルダツリーを生成し、最低限の `README.md` と `CLAUDE.md`（短い運用ルール）を作る。
- **実行例**: `/setup-os ~/YourOS`
- **入出力形式**:  
  - 入力: ルートパス（1つ）  
  - 出力: 作成したパス一覧＋次にやること（/companyを実行、など）
- **失敗時の扱い**:  
  - 既存フォルダがある場合は“上書きしない”。不足分のみ作成し、差分を表示。
- **保存先**: 指定ルート直下。
- **Claude Code接続方法**: 個人スキル（`~/.claude/skills/setup-os/SKILL.md`）。citeturn21view2turn18view0  
- **将来MCP化の拡張余地**: “OS状態（ツリー/統計）取得”をMCPツール化し、/dailyや/weeklyが高速に参照できる。

**2) /company**
- **仕様**: cc-companyによる会社運用の入口。初回オンボーディング→ `.company/` 自動生成→以降運営モード。citeturn24view1turn23view2turn16view1  
- **実行例**: `/company`
- **入出力形式**:  
  - 入力: 対話（AskUserQuestion）＋日常相談  
  - 出力: ファイル生成/更新（TODO/Inbox/Decisions/Departments）
- **失敗時の扱い**: `.company/` が壊れた場合は `.company/CLAUDE.md` の読み込みから復旧導線を作る（運用ルールに追記）。
- **保存先**: `.company/`（ただし“統合期”は後述Fへ寄せるようルール追記）。
- **Claude Code接続方法**: プラグイン導入（/plugin marketplace add / install）。citeturn25view1turn21view0  
- **将来MCP化の拡張余地**: 会社内ダッシュボード集計をMCPで行い、部署横断の状態集計を高速化。

**3) /inbox**
- **仕様**: どんな思いつきも“1発で”日次Inboxへ追記。タグとタイムスタンプを必須化。
- **実行例**: `/inbox [idea] Claude CodeでPR作成を半自動化したい`
- **入出力形式**:  
  - 入力: 自由テキスト  
  - 出力: `Inbox/YYYY/YYYY-MM-DD.md` に `HH:MM` 付き追記
- **失敗時の扱い**: 書き込み権限が無い場合は、本文を返して「手動で貼る」フォールバック。
- **保存先**: `Inbox/`（Fで定義）。
- **Claude Code接続方法**: 個人スキル。`allowed-tools: Write`。citeturn18view0  
- **将来MCP化の拡張余地**: 音声入力→Inbox、全文検索用インデックス更新をMCPで。

**4) /triage**
- **仕様**: Inboxを“処理済み”にする。各項目を Projects/Knowledge/SOP/Decisions/Scratch に昇格し、元にはリンクだけ残す。
- **実行例**: `/triage today`
- **入出力形式**:  
  - 入力: `today` or `YYYY-MM-DD`  
  - 出力: 仕分け結果（何をどこへ）＋作成ファイル一覧
- **失敗時の扱い**: 判断不能は `Scratch/Triage/` に“保留”として移す（捨てない）。
- **保存先**: `Inbox/` → 各カテゴリへ。
- **Claude Code接続方法**: 個人スキル（Read/Write）  
- **将来MCP化の拡張余地**: “類似項目提案”“重複検出”“関連Decision提示”を索引ツールで。

**5) /spec**
- **仕様**: Spec Writerロールで仕様化し、受け入れ条件と検証手順を必須にする（安定化の核）。citeturn22view1  
- **実行例**: `/spec p:alpha ログイン失敗時のメッセージ改善`
- **入出力形式**:  
  - 入力: `p:<project>` + 要望  
  - 出力: `Projects/<project>/Specs/YYYY-MM-DD--slug.md`（テンプレ埋め）
- **失敗時の扱い**: 不明点が致命的なら最大5問まで質問し、回答が無ければ“仮”として保存（status: draft）。
- **保存先**: `Projects/<project>/Specs/`
- **Claude Code接続方法**: スキル（推奨: `disable-model-invocation: true` で手動起動）。citeturn18view0turn21view2  
- **将来MCP化の拡張余地**: 既存コード解析（AST/依存）や既存仕様検索をMCPで高速化。

**6) /task**
- **仕様**: タスク1件を追加/更新。Specと相互リンクを張る。
- **実行例**: `/task p:alpha "ログイン失敗文言のA/B案作成" due:2026-03-18 prio:normal`
- **入出力形式**:  
  - 入力: プロジェクト + 本文 + 属性  
  - 出力: `Projects/<project>/Tasks/YYYY-MM-DD--slug.md`（または日次タスクへ追記）
- **失敗時の扱い**: パースできない引数は“そのまま本文”として保存し、後で整形。
- **保存先**: `Projects/<project>/Tasks/`
- **Claude Code接続方法**: スキル（Write）
- **将来MCP化の拡張余地**: タスク状態集計、依存関係グラフ化。

**7) /next**
- **仕様**: 今日やることトップ3（最優先/通常/余裕）を出し、ファイルリンクを返す。
- **実行例**: `/next p:alpha`
- **入出力形式**:  
  - 入力: optional（プロジェクト）  
  - 出力: “次の行動”＋参照パス
- **失敗時の扱い**: タスクが無い場合は Inbox から候補を提案し、/triageを促す。
- **保存先**: 生成物は保存不要（必要なら `Scratch/Daily/` に保存するオプション）。
- **Claude Code接続方法**: スキル（Read）
- **将来MCP化の拡張余地**: カレンダー/外部締切連携（MCP）。

**8) /decide**
- **仕様**: 意思決定を1件1ファイルで残す（理由・フォローアップ必須）。cc-companyのDecisionテンプレ思想を踏襲。citeturn16view1turn16view0  
- **実行例**: `/decide "DBマイグレーションは来月" why:"今週は機能優先" follow:"来週再評価"`
- **入出力形式**:  
  - 入力: タイトル＋理由  
  - 出力: `Decisions/YYYY/YYYY-MM-DD--slug.md`
- **失敗時の扱い**: タイトルが無い場合は自動生成し、あとでrename可能にする。
- **保存先**: `Decisions/`
- **Claude Code接続方法**: スキル（Write）
- **将来MCP化の拡張余地**: 既存Decisionの類似検索、決定木の可視化。

**9) /review-diff**
- **仕様**: git diffを読み、Specの受け入れ条件に対してレビューする。必要ならNOGO。  
  - なお“スキルはビルトインコマンドを直接実行できない”ため、`git diff` 等をBashで取る設計にする。citeturn18view0turn20view0  
- **実行例**: `/review-diff focus:"例外処理とセキュリティ"`
- **入出力形式**:  
  - 入力: optional focus  
  - 出力: 指摘/改善/GO-NOGO
- **失敗時の扱い**: git repoでない場合は対象ファイル一覧を要求し、読み取りレビューに切替。
- **保存先**: 重要レビューは `Scratch/Reviews/` に保存（任意）。
- **Claude Code接続方法**: スキル（Bash(git *), Read）
- **将来MCP化の拡張余地**: PR情報（コメント/CI結果）をMCPで取得してレビュー精度を上げる。

**10) /handoff**
- **仕様**: “次に何をすれば再開できるか”を1枚に固定（未来の自分向け）。
- **実行例**: `/handoff p:alpha 今日は認証周りの調査まで。次はテスト追加。`
- **入出力形式**:  
  - 入力: 現状メモ  
  - 出力: `Scratch/Handoffs/YYYY-MM-DD--project--slug.md`
- **失敗時の扱い**: 保存失敗時は本文を返し、手動貼り付けで復旧。
- **保存先**: `Scratch/Handoffs/`
- **Claude Code接続方法**: スキル（Write）
- **将来MCP化の拡張余地**: “最後に触ったファイル/未コミット差分”を自動取得して添付。

## F. PC内全データ管理の設計（実践的フォルダ構成）

### どのデータをどこに置くべきか（提案）

- **Inbox**: すべての入口（未整理）。最短で書けることを最優先。
- **Projects**: “進行中の作業単位”。Spec/Tasks/Notes/Contextをプロジェクト単位で束ねる。
- **Decisions**: 横断で参照できる“意思決定の台帳”。Projectsから必ずリンク。
- **SOP**: 再現可能な手順＝運用資産。詰まりを減らす。
- **Knowledge**: 短く要約された“正”。外部リンク・根拠・結論を最小で。
- **Prompts**: プロンプト雛形、スキル設計、定型指示。育つ場所。
- **Scratch**: 作業中の一時置き場（散らかってOK。ただし“週次で整理”）。
- **Archive**: 完了・陳腐化したものの退避（参照はできるが日常の視界から外す）。
- **Private**: 個人情報・機密・日記・金銭など（原則AIに読ませない）。
- **AI-readable**: AIが読める“整形済みの正本/要約”を置く（検索の起点にする）。
- **AI-blocked**: AIに読ませない領域（Private以外の機密、鍵、契約書、顧客生データなど）。

この「AI-readableを正・AI-blockedへ隔離」は、cc-companyが重視する“迷ったらInbox、既存は上書きしない、1トピック1ファイル”の思想と相性が良いです。citeturn16view0turn24view1  

### フォルダ構成案（ルート構成を必須要件どおりに）

```text
~/YourOS/
  README.md
  CLAUDE.md

  Projects/
    _index.md
    alpha/
      Context/
      Specs/
      Tasks/
      Notes/
      Links/

  Inbox/
    2026/
      2026-03-15.md

  Knowledge/
    dev/
    biz/
    tools/

  SOP/
    dev/
    ops/
    security/

  Decisions/
    2026/
      2026-03-15--slug.md

  Prompts/
    commands/
    roles/
    snippets/

  Scratch/
    Daily/
    Handoffs/
    Reviews/
    Experiments/

  Archive/
    Projects/
    Reviews/
    Retros/
    Snapshots/

  Private/
    personal/
    finance/
    identity/

  AI-readable/
    README.md
    Pinned/
    Summaries/
    Index/

  AI-blocked/
    README.md
    secrets/
    clients-raw/
    legal/
```

※ cc-companyを使う場合は、同じルートに `.company/` を置いて “窓口組織” として動かしつつ、保存先ルールを上記へ寄せるのが現実的です（最初から作り替えない）。citeturn23view2turn24view1  

### AIに読ませる領域 / 読ませない領域（運用ルール）

- **AIに読ませる（原則）**: `AI-readable/`, `Projects/*/Context`, `Projects/*/Specs`, `SOP/`, `Decisions/`, `Knowledge/`, `Prompts/`
- **AIに読ませない（原則）**: `AI-blocked/`, `Private/`
- **強制（提案）**:  
  - CLAUDE.mdで「AI-blockedとPrivateは読まない/書かない」を明記（ただしコンテキストなので100%強制ではない）。citeturn21view1turn22view4  
  - 本当に守りたいならフックで `PreToolUse` を使い、Write/Edit（場合によりReadも）をブロックする。citeturn19view2turn19view3  

### メタデータ設計（提案：YAML frontmatterを統一）

cc-companyのテンプレ群がfrontmatter（date/type/tags等）を使っているので、それに寄せると一貫する。citeturn16view1turn24view1  

- 共通フィールド案:
  - `created:` `YYYY-MM-DD`
  - `updated:` `YYYY-MM-DD`
  - `type:` `inbox|spec|task|decision|sop|knowledge|handoff`
  - `project:` `alpha` 等
  - `status:` `draft|active|done|archived`
  - `tags:` `[]`（後述）

### 命名規則（提案）

cc-companyテンプレの命名思想（`YYYY-MM-DD.md`、`kebab-case-title.md`、追記中心）を採用。citeturn16view0turn24view1  

- 日次: `YYYY-MM-DD.md`
- Decision: `YYYY-MM-DD--short-title.md`
- Spec: `YYYY-MM-DD--feature-slug.md`
- Task: `YYYY-MM-DD--task-slug.md`
- プロジェクトフォルダ: `kebab-case`（例: `customer-portal`）

### タグ設計（提案：少数精鋭、接頭辞で衝突回避）

- `p:<project>`（例: `p:alpha`）
- `a:<area>`（例: `a:dev`, `a:biz`）
- `t:<type>`（例: `t:decision`, `t:sop`）
- `s:<status>`（例: `s:active`）
- `risk:<level>`（例: `risk:high`）

タグは増やしすぎると管理コストが跳ねるので、**最大でも5個まで**を推奨（提案）。

### 更新ルール（提案）

- **追記主義**（上書きしない）: いつ、何が変わったかが残る。cc-companyも同方針。citeturn16view0turn24view1  
- 追記時は必ず `HH:MM` を付ける（/inbox で自動化）。  
- 完了したら `Archive/` へ移す（/weekly or /archive で半自動）。

### 検索戦略（提案）

- 入口は `AI-readable/Index/`（短い要約・リンク集）に集約し、そこから辿れるようにする。  
- “全文検索”は `rg`（ripgrep）などのCLI検索に依存してよい（MCP化は後回し）。  
- `Decisions/` と `SOP/` は“タイトルに必ずキーワードを入れる”ことで検索を単純化。

### バックアップ戦略（提案）

- 中央OSフォルダは  
  1) ローカルスナップショット（例: OS標準バックアップ）  
  2) 任意で暗号化されたリモート（プライベート）  
  の二重化を推奨。  
- Private/AI-blocked は“さらに厳しめ”のバックアップポリシーに分離（提案）。

### Claude Codeから参照しやすい構造（事実＋提案）

- **事実（Claude Code機能）**: `/add-dir <path>` でセッションに追加ディレクトリを加えられる。citeturn21view0  
- **事実（Claude Code機能）**: CLAUDE.mdは `@path` で追加ファイルをインポートできる。citeturn21view1turn22view4  
- **提案**:  
  - ふだんの開発はコードプロジェクトで起動 → `/add-dir ~/YourOS` → 必要な文脈（Spec/Decision/SOP）を参照して実装、の型にする。  
  - OSルートの `CLAUDE.md` は短くし、頻繁に使う“入口リンク”だけ置く（詳細はPromptsやSOPへ）。citeturn22view4turn17view0  

## G. 最初の7日間でやるべきこと（1日ごとの作業計画）

前提: 今日が **2026-03-15（日）** なので、**Day1＝2026-03-16（月）** から開始します（JST）。

**Day1（2026-03-16）: OSルートを作って「入口」を固定**
- `~/YourOS/` を作り、Fのフォルダツリーを作成（まず手でもOK、可能なら /setup-os をこの日に作る）。
- `README.md` に “このOSで何を管理するか（3行）” を書く。
- `CLAUDE.md`（OSルート）に「AI-readable/AI-blocked/Private の境界」「追記主義」「命名規則」を短く書く（200行未満目標）。citeturn21view1turn22view4  

**Day2（2026-03-17）: cc-companyを入れて秘書窓口を立ち上げる**
- cc-companyをインストールし、`/company` を実行（保存先は `~/YourOS` を選んで `.company/` を作る）。citeturn25view1turn24view1turn23view2  
- 生成された `.company/CLAUDE.md` と `secretary/CLAUDE.md` を編集し、「記録は原則 `~/YourOS/Inbox` に書く」など、Fの設計へ寄せる（まず“書き込み先の統一”だけやる）。citeturn16view0turn16view1  

**Day3（2026-03-18）: prompt-reviewを導入して“改善ループ”を動かす**
- `~/.claude/skills/prompt-review/` に prompt-review を配置（レポート出力が `reports/` になるので、OS側に `Reports/` を作ってリンクするか、運用ルールで統一）。citeturn24view0turn25view0turn21view2  
- `/prompt-review 7` を一度実行し、レポートを読む（“改善点”を1つだけ決めてPromptsにメモ）。citeturn25view0turn24view0turn10view0  

**Day4（2026-03-19）: 最初の10コマンドのうち「入口3点セット」を作る**
- /inbox（最優先）: 1発で記録できるようにする。  
- /triage: 今日のInboxをProjects/Knowledgeへ仕分け。  
- /handoff: 作業を止められるようにする。  
（この3つが揃うと「思いつき→実行→中断→再開」が回り始める）

**Day5（2026-03-20）: 開発フローの核（/spec, /task, /next, /decide）を作る**
- /spec（受け入れ条件・検証手順を必須）  
- /task（Specとリンク）  
- /next（今日の最優先3）  
- /decide（Decision Recordの型を固定）  
この日から、開発タスクは必ずSpec→Task→実装、の順に試す（2回でいい）。

**Day6（2026-03-21）: 品質ゲート（/review-diff）と安全策（最小フック）**
- /review-diff を作り、コミット前に必ず通す。  
- 可能ならフックを1本だけ入れる: 「AI-blocked/PrivateへのWriteをブロック」（PreToolUse）。citeturn19view2turn19view3  
- `/permissions` で安全なコマンドだけallowlist（例: `git status`, `git diff`, `npm test` など）し、許可ダイアログ疲れを減らす。citeturn22view2turn20view0  

**Day7（2026-03-22）: 週次レビューを回して“育つ仕組み”にする**
- /weekly（未実装なら手動）で「完了/未完/学び/次週」を `Archive/Reviews/` に残す。  
- `/prompt-review 7` を再実行し、Day3と比較して“改善が効いたか”を見る。citeturn25view0turn24view0  
- 改善が効いていない場合は、Prompts/roles にある雛形を1つだけ直す（“全部直す”は禁物）。