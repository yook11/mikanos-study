# MikanOS 作業チートシート

## ビルドと起動

### 準備（ターミナルを開くたびに1回）

```bash
source ~/osbook/devenv/buildenv.sh
```

`<cstdint>` などのヘッダや newlib の場所を `make` に教えるための設定。
これを忘れると `'cstdint' file not found` になる。

### カーネルをビルドする

```bash
cd ~/workspace/mikanos/kernel && make
```

### カーネルを一からビルドし直す

```bash
cd ~/workspace/mikanos/kernel && make clean && make
```

`make clean` は `*.o` だけを消す。`hankaku.bin` や `.*.d` は残る。

### ローダーをビルドする（Main.c を変えたときだけ）

```bash
cd ~/edk2 && source edksetup.sh && build
```

できあがるファイル：`~/edk2/Build/MikanLoaderX64/DEBUG_CLANG38/X64/Loader.efi`

### QEMU で起動する

```bash
~/osbook/devenv/run_qemu.sh ~/edk2/Build/MikanLoaderX64/DEBUG_CLANG38/X64/Loader.efi ~/workspace/mikanos/kernel/kernel.elf
```

## Git

### ブランチの使い方

- `main`：確認が終わった日の分だけを入れる
- `my_dayXX`：その日の作業用。`main` から切る
- `master`：本家 MikanOS の完成版（自分の作業とは別物。切り替えない）

### その日の作業を始める

```bash
git switch main && git pull
```

```bash
git switch -c my_day06
```

```bash
git push -u origin my_day06
```

### コミットしてプッシュする

```bash
git status --short
```

```bash
git add 変更したファイル 新しく作ったファイル
```

```bash
git commit -m "day06a: やったことを一言で"
```

```bash
git push
```

`apps/` と `disk.img` はコミットしない。

### 作業ブランチを main に取り込む（その章が終わったら）

```bash
git switch main && git merge my_day06 && git push
```

PR で取り込みたいとき：

```bash
gh pr create --base main --head my_day06 --fill
```

## 本家のコードを見る

本家のタグ（`osbook_day05e` など）は手元にあるので、ダウンロードしなくても見られる。

### 本家のある日のファイルを表示する

```bash
git show osbook_day05e:kernel/console.cpp
```

### 本家のファイルを手元に持ってくる

```bash
git show osbook_day05c:kernel/hankaku.txt > kernel/hankaku.txt
```

### 本家でその日に何が変わったか見る

```bash
git diff osbook_day05d osbook_day05e --stat
```

```bash
git diff osbook_day05d osbook_day05e -- kernel/main.cpp
```

### 自分のファイルと本家を比べる（空白の違いは無視）

```bash
git diff --no-index -w <(git show osbook_day05e:kernel/main.cpp) kernel/main.cpp
```

## 中身を調べる

### .o や kernel.elf に入っている記号を見る

```bash
nm -C kernel/kernel.elf
```

`T`：関数の中身がある　`D`：データがある　`U`：別のファイルにある（未解決）　`A`：絶対値

### バイナリファイルの中身を見る

```bash
xxd -s 1040 -l 16 kernel/hankaku.bin
```

`-s` は何バイト目から、`-l` は何バイト分。

### ファイルを変えたら何が作り直されるか確かめる（実行はしない）

```bash
cd ~/workspace/mikanos/kernel && make -n -W graphics.hpp
```

## よくあるエラー

| エラー | 原因 |
|---|---|
| `'cstdint' file not found` | `buildenv.sh` を読み込んでいない |
| `ld.lld: error: undefined symbol: ○○` | ○○ の入った .o が Makefile の `OBJS` にない、または .hpp と .cpp で引数が違う |
| `implicit declaration of function '○○'`（ローダー） | ○○ を宣言しているヘッダの `#include` がない |
