# Antigravity CLI実行契約

## 利用環境の確認

Antigravity CLI（`agy`）を別途導入し、利用者自身の環境で認証を済ませる。CLI、認証情報、アカウント設定はこの配布物に含めない。`command -v agy`、`agy --version`、`agy --help`、`agy models` で実体、版、対応する引数とモデルを確認する。

指定値は `--model gemini-3.8-flash-high --effort high`。モデルや引数が利用できない場合はその旨を報告し、別モデル・別経路へ黙って切り替えない。以下の呼出し例はCLI 1.2.2の形式に基づくため、利用環境のヘルプと応答形式を優先して確認する。

## 入力と実行

初回は[編集プロンプト](../prompts/gemini-editor.md)、レビュー後の修復は[修復プロンプト](../prompts/gemini-repair.md)を使う。二つを連結しない。初回jobには原則modeとblocksのみを入れ、Codexが作った保持事項・構成案・レビュー基準は添えない。ユーザーが今回明示した条件は必要な範囲で添える。

選択したプロンプトとjobオブジェクトを、JSONエンコーダーで `{"instructions": "...", "job": {...}}` にする。CLI引数へ原文を展開せず、標準入力へ渡す。シェル文字列へ埋め込まない。CLI 1.2.2では、空のprint引数とtext形式では標準入力を受け取れなかった。`--input-format stream-json --output-format stream-json` を使い、JSON文字列を `event: user` のメッセージに包んだNDJSONを1行送る。

Pythonでの呼出し例（パスとjobは呼出し側で指定）：

```python
payload = json.dumps({"instructions": prompt, "job": job}, ensure_ascii=False)
stream_input = json.dumps({"event": "user", "message": {"role": "user",
    "content": [{"type": "text", "text": payload}]}}, ensure_ascii=False) + "\n"
result = subprocess.run(
    [agy_path, "--print=", "--model", "gemini-3.8-flash-high",
     "--effort", "high", "--sandbox",
     "--disable-slash-commands", "--input-format", "stream-json",
     "--output-format", "stream-json", "--print-timeout", "120s"],
    input=stream_input, text=True, capture_output=True, timeout=150,
    cwd=isolated_job_directory,
)
```

必要に応じて `--json-schema` に候補のスキーマファイルを指定する。起動時には長い処理を非同期の実行セッションにして、利用者への進捗連絡を可能にする。

既存会話・プロジェクトを継続する引数を使わず、原稿以外のファイルを置かない一時ディレクトリで実行する。`--dangerously-skip-permissions`を使わない。CLI固有の認証情報を直接読み出さない。sandboxによるCLI起動制限があれば通常の承認機構で対応し、失敗を生成済みと扱わない。

## 外部操作の境界

CLI 1.2.2では `--disable-slash-commands` と併用した `--mode plan` は無効になる警告が出たため、例ではplanに依存しない。`--sandbox`も完全なツール無効化の保証ではない。プロンプトでもツール禁止を明記し、権限要求を自動承認しない。専用エージェントを作る場合は、その時点の公式定義方式と既存名を確認し、読み取り・検索も含むツール制限を実際に設定できることを確認する。未確認の `agent.md` 配置規約や設定キーを発明しない。グローバル設定を一括変更しない。

ツール実行要求が現れた場合は承認せず、そのジョブを止める。結果だけではツール非使用が証明できない場合は「非使用未確認」と記録する。強いツール遮断が必要な文書では、制御可能な専用エージェントの確認まで送信しない。

## 応答の検証

stdoutはNDJSONイベント列として保管する。各行をJSONとして読み、`event == "init"` の `init.model` を確認する。`event == "result"` の `result.status` と `result.error` を確認し、成功時の `result.response` を候補JSONとして解析する。resultの欠落・重複は採用しない。CLI更新で構造が違えば実際の形式を確認する。候補のJSONをログ内の最初の波括弧から推測しない。stderrと終了コード、エラー・中断・タイムアウトを別に確認する。終了コード0だけで成功にしない。

CLI 1.2.2形式の成功値は `result.status == "SUCCESS"`。小文字の `success` を仮定して成功応答を失敗扱いしない。形式が変わった場合は実際のメタデータを確認し、未知の値を一律に成功へ正規化しない。

候補は `{"blocks":[{"id":"p001","text":"..."}]}`。全IDが入力と同じ順序で一意に揃い、textが文字列であることを確認する。空でない原文に空の候補が来た場合も失敗扱い。IDは編集単位の対応を守るためのもので、内部の段落数や文数の固定には使わない。入力時に関連段落をまとめておき、統合のために空IDを作る必要がない形にする。モデルの実績情報が出る場合は指定値との一致を確認し、出ない場合は「指定モデル」と「応答が証明するモデル」を区別する。

壊れたJSON、欠落、拒否、想定外のツール利用、モデル不一致は採用しない。原因を報告し、原文を保持する。無制限の再試行や別経路への切替はしない。内容の修復はSKILL.mdに従い最大1回。
