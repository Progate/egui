# BrowserOS のための変更（Progate/egui の browseros-0.36）

上流 emilk/egui の 0.36.2 の上に、BrowserOS の Rust の target（`wasm32-browseros-linux-musl`）で組むための変更を足したブランチ。
使う側（Progate/packages の browser-deck など）は、`[patch.crates-io]` でこのブランチのコミットを指す。

## 変更

### `DroppedFileHandle` の `Send + Sync` を外すのは、ブラウザーの JS の上（`target_os = "unknown"`）だけにする

`crates/egui/src/data/input/dropped_file.rs`。上流は `all(target_arch = "wasm32", target_feature = "atomics")` のとき、
`web_sys::File` がスレッドをまたげないので `DroppedFileHandle` から `Send + Sync` を外す。BrowserOS の target は
wasm32 で、スレッドのために `target_feature = "atomics"` を cfg に出すが、`web_sys` は使わない Linux の環境である。
外れたままだと `egui::Context` が `Send` でなくなり、`Context` を持つ値をスレッドへ渡すアプリ（DeckCraft の
スペルチェックのスレッドなど）が組めない。条件に `target_os = "unknown"` を足し、ブラウザーの JS の上だけに限る。
