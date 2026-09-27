# 計画

## 概要
- 対象タスク: pending リポジトリ `affaan-m/ECC`（AIエージェントのハーネス性能最適化システム・Skills/instincts/memory/security 統合・tag=`定着`・fetched=2026-09-21・★826/累計263,758）を ai-eval スキルの全 Phase（1〜4）に沿って1件評価し、評価md追記〜verdict書き戻し〜commit/push まで完走する。ai-trending spec v1.2 G2（毎日提示仕様・評価疲れ対策）の必須枠・1日1件キャップ内の処理。
- ECCは本環境（superpowers スキル群・CLAUDE.md層1ルール・file-based memory・multi-llm-review）と**同領域（エージェント性能最適化）が高度に重複する**可能性が高いため、評価の主軸は「既存機構との重複判定」。層1ルール「代替判定の表層化」に従い、代替済みと結論する場合は**比較対象の本文照合+根拠引用を必須**とする。verdict（keep/skip）は Phase 2-3 の実質評価と MLR-lite の結果で確定させ、計画段階で先決しない。

## 実装手順
1. **Phase 1（スコープチェック）**: `~/projects/obsidian-ssot/00_SYSTEM/stats/pending-ai-repos.json` を読み `affaan-m/ECC` が pending 内に実在することを確認（実測済み・本計画立案時に entry を確認済み）。pending内なので通常フロー（adhocではない）。
2. **Phase 2（評価ドラフト追記）**: `30_RESEARCH/MCPサーバー/2026-06-24_GitHub急上昇AIリポジトリ10選評価.md` のファイル末尾に固定形式セクションを追記（frontmatterは触らない）。URL・説明・観測欄（tag/fetched/stars）・CC評価ドラフト（環境適合度★と根拠）・**「取り入れ候補（設計の盗み）」欄は省略禁止**（具体性ゲート: リポ内の具体的機構名引用+自環境のどの機構に足すか、またはゼロ根拠ペア）・verdict は「保留（MLR後に確定）」で書く。
3. **Phase 3（multi-llm-review-lite）**: Skill ツールで発動（対象=評価ドラフト・観点=環境適合性/代替存在/保守状態/取り入れ候補の技術的有用性）。完了後、`~/.claude/state/multi-llm-review.jsonl` の最新ok行の `ts` を `MLR_ROUND_START` と機械比較し `within_round: True` を確認。ok行ゼロなら verdict を「unknown（mlr証跡ゼロ）」に修正して終了。
4. **Phase 4（verdict書き戻し）**: 手動編集禁止・`ai-eval-writeback.py` で実行。keep → `--remove --verdict keep`／skip → `--verdict skip --reason ... --re-eval-trigger ...`（re_eval_trigger 必須）。stdout の `RUN_ID`・diff行・VERIFY行と `--list-leftovers` 再実行（`LEFTOVERS=0` 期待）を生出力として貼付。評価mdの verdict行を固定形式に更新し、事前に `grep -c "取り入れ候補（設計の盗み"` を実行（0件なら書き戻し中止）。
5. **完了処理**: obsidian-ssot へ commit+push（pre-commit hook `ai-eval-pending-consistency` の通過を確認）・完了報告に「検証範囲宣言」セクションを含める。

## テスト方針
- **テスト区分: 対象外** — コードファイル変更が無いタスクのため（評価md追記+既存スクリプト `ai-eval-writeback.py` の実行のみ・新規コード・テストコードの追加は行わない）。起票時のテスト方針「3=評価md記録系」と整合。
- 検証は機械証跡で代替: MLR ok行の `within_round: True`、writeback スクリプトの生 exit code・RUN_ID・VERIFY行、`grep -c` の取り入れ候補欄存在確認、`LEFTOVERS=0`、pre-commit hook 通過。

## 受け入れ条件
- 証跡は **exit code / テスト緑件数 / coverage / 本番CLI実測ログ** のみ有効。
- **本番CLI実測ログ**: `ai-eval-writeback.py` の stdout（`RUN_ID=<suffix>`・diff行・VERIFY行を含む生ログ）+ `--list-leftovers` 再実行（`LEFTOVERS=0`）を完了報告にそのまま貼付。RUN_ID なし報告は捏造扱い。
- **exit code**: MLR 証跡確認コマンド（`LAST_OK_EXIT=0`・`within_round: True`）、`grep -c` 取り入れ候補欄チェック（≥1）、commit/push コマンドに生 exit code 添付（パイプ挟まず `; echo "EXIT=$?"`・`set -o pipefail`）。
- **coverage**: コード変更ゼロのため coverage 変動 0pt（既存値±5pt以内を満たす）。テスト緑件数は既存テストに変更を加えないため起票時点の実測値から減少なし。
- 評価mdに `## affaan-m/ECC (2026-09-26)` セクションが実在し、verdict が固定形式（keep/skip/unknown のいずれか・「保留」のまま完了しない）で確定していること。skipped移動時は `re_eval_trigger` 必須。

## 想定リスク
- **代替判定の表層化リスク（最重要）**: ECCは本環境の superpowers/CLAUDE.md/memory 機構と領域が重複するため、「既存で代替済み→skip」の楽な結論に流れる動機がある。13類型「代替判定の表層化」対策として、比較対象機構名の本文照合+根拠引用をドラフト内で必ず実施する。failケース: 引用なき代替済み判定を出すこと。
- **MLR-lite失敗リスク**: OpenRouter free枠が落ちるとok証跡ゼロ→verdict unknown で終了（手動再評価可能状態を維持する設計済み・仕様どおりの正常系扱い）。
- **daily キャップ・重複評価リスク**: 本日既に同日評価が実行済みの場合は1日1件キャップにより skip する（実行前に当日の評価md追記有無を確認）。lazy判定的な「既に済んだはず」の楽観は禁止・grep実測で確認する。
- **writeback省略リスク**: 過去12/13回省略された実績があるため、手動編集を排しスクリプト生出力貼付を機械的に強制（Phase 4 手順どおり）。

🟡[GLM]
