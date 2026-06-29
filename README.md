# 舟券アカデミア v64 Googleログイン・クラウド保存版

視聴者提供用V63をベースに、Googleログインとクラウド保存を追加した版です。
ログインしない人向けには、これまでの古いV63リンクをYouTube概要欄に残して運用してください。

## 変更内容 v6.4.0

- Googleログイン欄を追加
- ログイン済みユーザーごとにクラウド保存
- 保存対象はユーザー作成データのみ
  - 買い目リスト
  - 配当入力
  - 舟券収支履歴
- 展示、モーター、オッズ、枠別情報、ST、F持ち、風、進入など外部サイト由来データはクラウド保存しない
- 未ログイン時は従来どおり端末内保存で利用可能
- オールクリアは画面入力だけを消し、端末内保存/クラウド保存には影響しない
- 「今すぐ保存」ボタンを追加
- 舟券収支履歴の全削除ボタンは維持
- Androidコピペ強化、オッズ折り返し、モーター重複、展示F.ST対応は維持
- package version: 6.4.0

## Vercel設定

- Framework Preset: Vite
- Build Command: `npm run build`
- Output Directory: `dist`
- Install Command: `npm install`
- Node.js: 20.x

## 必要なVercel環境変数

Vercelの Project Settings → Environment Variables に追加します。

```text
VITE_SUPABASE_URL=https://xxxxx.supabase.co
VITE_SUPABASE_ANON_KEY=Supabaseのanon public key
VITE_SUPABASE_TABLE=hunaken_user_data
```

`VITE_SUPABASE_TABLE` は省略しても `hunaken_user_data` になります。
この3つのうちURLとANON KEYが未設定の場合、Googleログイン欄は表示されず、従来どおり端末内保存だけで動きます。

## Supabaseで作るテーブル

Supabase SQL Editorで以下を実行してください。

```sql
create table if not exists public.hunaken_user_data (
  user_id uuid primary key references auth.users(id) on delete cascade,
  cart jsonb not null default '[]'::jsonb,
  payout_odds_input text not null default '',
  bet_records jsonb not null default '[]'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.hunaken_user_data enable row level security;

drop policy if exists "hunaken select own" on public.hunaken_user_data;
drop policy if exists "hunaken insert own" on public.hunaken_user_data;
drop policy if exists "hunaken update own" on public.hunaken_user_data;
drop policy if exists "hunaken delete own" on public.hunaken_user_data;

create policy "hunaken select own"
on public.hunaken_user_data
for select
to authenticated
using (auth.uid() = user_id);

create policy "hunaken insert own"
on public.hunaken_user_data
for insert
to authenticated
with check (auth.uid() = user_id);

create policy "hunaken update own"
on public.hunaken_user_data
for update
to authenticated
using (auth.uid() = user_id)
with check (auth.uid() = user_id);

create policy "hunaken delete own"
on public.hunaken_user_data
for delete
to authenticated
using (auth.uid() = user_id);
```

## Supabase Googleログイン設定

Supabaseの Authentication → Providers → Google を有効化します。
Google Cloud側でOAuth Clientを作り、Supabaseに Client ID / Client Secret を登録してください。

Supabase側の Authentication → URL Configuration で、以下を登録してください。

```text
Site URL: https://あなたのVercelドメイン
Redirect URLs:
https://あなたのVercelドメイン
https://あなたのVercelドメイン/*
```

例：

```text
https://hunaken-academia-xxxx.vercel.app
https://hunaken-academia-xxxx.vercel.app/*
```

## GitHubへ上書きする時の注意

ZIPを解凍して、中身だけをGitHubの一番上に置いてください。

正しい配置：

```text
package.json
index.html
src
api  ※存在する場合
vercel.json
vite.config.js
.npmrc
```

フォルダごと1段深く入れないでください。
`package-lock.json` は入れていません。

## ローカル確認

```bash
npm install
npm run build
npm run dev
```
