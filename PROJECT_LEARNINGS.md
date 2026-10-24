# PROJECT LEARNINGS — AviUtl2プラグイン共通

> このRepositoryはプラグイン共通基盤。以下の初期事例は2026年10月のMontage開発会話から再構成した移植可能な教訓であり、Foundation自身で発生したBugとは主張しない。実際のログ・Commit・Windows実機の再現情報が未取得の項目はその旨を記す。

## 運用ルール

- 新しいAviUtl2プラグイン／スクリプトの実装・修正前に、関連するFailureとGuardを読む。新しいProjectでもPROJECT_LEARNINGS.mdを持ち、当該Projectで得た根拠付き事例をそこへ追記する。
- トラブル時は『症状 → 環境・ログ・再現手順 → 差分・切り分け → 原因の確度 → 最小修正 → 自動テスト＋AviUtl2実機テスト → 再発防止Guard』の順に進める。原因を推測してパッチを積み重ねない。
- 事実／有力仮説／未確認を明示する。エラーが消えたことと原因特定は別。Build成功・単体テスト成功・実機での表示・機能の品質評価は別の検証状態とする。
- 高コスト失敗は再現可能なRegression Testが最優先。実機でしか判定できない事象は、動作・表示・Undo・AviUtl2バージョン・隔離テストをチェックリスト化する。
- 通常のテストは専用AviUtl2環境で実施。既存環境を破壊せず、既知良好版への復帰手段を用意する。証拠ログには個人情報・秘密情報・動画本体を含めない。

## Montage開発から引き継ぐ事例

### PL-F-001 起動クラッシュ 0xC0000005 とプラグイン相互干渉

- Date: 2026-10-19
- Status: monitoring（根本原因未確定）
- Severity / Cost: high / high
- Symptom: 自作拡張を多数追加後、AviUtl2起動時に0xC0000005でクラッシュ。過去の診断メモにはaviutl2.exe+0x20EC9A、enum_effect_name_callback()が記録されている。
- Trigger / Reproduction: 自作20本を導入した環境で発生した。単独での完全再現手順は未確定。
- Root Cause: 未確定。旧VmtKillImpact / VmtRamPreviewの一方または組合せの関与が疑われたが、単独原因の証明なし。Anchor Punch単独原因も未証明。
- Observed mitigation: 自作20本を隔離すると起動成功。後に旧VMTの2本も隔離した環境で効果一覧を表示できた。
- Final Fix: 根本原因のコード修正は未確認。隔離による復旧のみ確認。
- Regression Guard: SDKと実機バージョンを固定して記録し、最小セット起動→1本ずつ追加→一覧表示→機能実行の順で検証する。クラッシュ時はdump/ログ、導入差分、順序、ファイルHashを保存して二分探索する。
- Prevention: 既知問題のある旧プラグインを一括復帰させない。Build成功を起動成功と混同しない。

### PL-F-002 Color Drainがメニューに出ない

- Date: 2026-10-19
- Status: resolved（表示経路の誤認）
- Severity / Cost: low / medium
- Symptom: 導入したColor Drainが見つからないように見えた。
- Root Cause: 誤ったメニューを確認していた。動画オブジェクト側の「フィルタ効果を追加」が正しい操作場所だった。
- Final Fix: 正しいメニューから追加し、AviUtl2実機で実際の色変化を確認。
- Regression Guard: 各スクリプト／プラグインの対象オブジェクト、表示されるメニュー、追加手順をREADMEとスモークテストで確認する。
- Prevention: 『読み込まれていない』と判断する前に、拡張の種類・対象・メニュー配置を調べる。

### PL-F-003 Kill Impactの演出が遅く弱く見える

- Date: 2026-10-19
- Status: resolved for user acceptance of v0.5 / implementation root cause not independently validated
- Severity / Cost: medium / medium
- Symptom: v0.3は動作したが、キル時刻に合わせても少し遅れる・弱い・見た目に満足できない。
- Root Cause: 過去の試作分析では弱い初期値、約260msの復帰、クリップ先頭基準の時間処理が要因として挙げられた。コード・実測による独立検証は未取得。
- Final Fix: v0.4でフレーム基準等を調整。後のv0.5がユーザーから高評価を受け基準版とされた。v0.3/0.4を最終基準版と混同しない。
- Regression Guard: ヒット時刻±前後フレーム、30/60fps、クリップ開始位置変更、シーク・逆再生、最大/最小値を検証。定量的な発火タイミングと実機の主観的な見た目を別々に確認する。
- Prevention: デフォルト値を決定する前に短い実映像でタイミングと強度を評価する。未評価のバージョンを完成扱いしない。

### PL-F-004 SmartSlowの編集ソフト内部速度制御が期待通り動かない

- Date: 2026-10-19〜2026-10-24
- Status: monitoring（元の不具合原因は未確定）
- Severity / Cost: medium / high
- Symptom: v0.2/v0.3で画面や設定は表示されても、AviUtl2内の速度変更が期待通り動作しない。
- Root Cause: SDK／時間制御のどの制約・実装箇所が原因かは未特定。
- Observed mitigation: 内部制御方式からFFmpegで指定区間を変換する方式へ切り替え。SmartSlow Render v0.2では、区間スロー、出力fps、上書き防止、元動画保護、3種類の補間について7件の自動テストに成功。
- Final Fix: 内部速度制御自体は未解決。Render方式も音声処理、AviUtl2直接連携、Windows実機読み込みは未完了。
- Regression Guard: 『設定画面が開く』『効果が出る』『正しく書き出せる』『音声同期』『AviUtl2で再読込できる』を別のテスト項目にする。入力・出力のfps/長さ/フレーム境界と元動画のHashを検査する。
- Prevention: SDK機能の実現性は最小実機プロトタイプで検証し、UI実装成功だけで処理方式を採用しない。

## 新機能着手前チェック

1. 今回触る機能と類似するPL-Fを確認し、Guardをテスト計画へ転記したか。
2. 変更前のAviUtl2/SDK/OS/依存ライブラリ/入力ファイル条件/既知良好版を控えたか。
3. 再現手順、ログ、変更差分を残し、仮説を観測した事実と分けたか。
4. 新規プラグインは単体導入→共存→効果一覧→適用→Undo→再起動まで試したか。
5. 設定画面の表示と実際の機能実行を両方確認したか。
6. テスト結果のうち自動／実機／主観評価がどこまで完了したかを明示したか。
7. 再発時に検知可能なテストか実機チェックを残したか。

## 今後の追加ルール

個別プラグインRepoに実際のIssue/PR/Commit/ログが見つかった場合は、各事例へ根拠リンクと特定済みRoot Causeを追記する。未確認を無理にresolvedへ昇格しない。特定済みの新しい失敗は新しいIDでこの知識を継続更新する。
