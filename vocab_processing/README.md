# 語彙の整備
## 学習したトークナイザーから語彙を取得
- 例）語彙とスコアの一覧をデータフレームにする。
```python
import sentencepiece as spm
import pandas as pd

# 語彙をデータフレーム化
sp = spm.SentencePieceProcessor()
sp.Load(model_path) # SentencePieceのtrainから得られたxxxx.modelを読み込む

vocab_size = sp.get_piece_size()  # 語彙サイズの取得

# トークンとスコアのリストを作成
tokens = []
scores = []

for i in range(vocab_size):
    token = sp.id_to_piece(i)
    score = sp.get_score(i)
    tokens.append(token)
    scores.append(score)

# データフレームの作成
df = pd.DataFrame({
    'token': tokens,
    'score': scores
})

# データフレームの表示
display(df)
```

## 除外・結合・追加・重複削除・語彙数調整などの処理

### 除外
- 日本語の語彙は純粋な日本語のみを取得するため、アルファベットのみで構成される語彙を除外した。

### 結合
- それぞれの言語の語彙を結合。

### 追加
#### 日本語
- 常用漢字や一般語を手動で追加。
    - 参考
        - [常用漢字一覧表](https://www.bunka.go.jp/kokugo_nihongo/sisaku/joho/joho/kakuki/14/pdf/jyouyou_kanjihyou.pdf)
        - [Wikitinary 日本語](https://ja.wiktionary.org/wiki/%E3%82%AB%E3%83%86%E3%82%B4%E3%83%AA:%E6%97%A5%E6%9C%AC%E8%AA%9E_%E5%90%8D%E8%A9%9E)
        - 日本の地名・東京23区・山手線の駅名など
        - 定型表現・口語
#### 算数
- 数字や記号を追加

### 結合
- データフレームをマージする。

### 重複削除
- 重複したトークンを削除。
```python
df_dedup = df.drop_duplicates(subset="token", keep='first')
```
### 語彙数調整
- 56,320の語彙になるようにスコアの低い語彙を削除する。
- 特殊トークンとして以下のトークンが追加されることも考慮して最終的な語彙数を調整していく。（参考：[llm-jp-tokneizer](https://github.com/llm-jp/llm-jp-tokenizer/blob/main/scripts/howToCreateModel_ver2.md#%E8%AA%9E%E5%BD%99%E3%81%AE%E3%83%9E%E3%83%BC%E3%82%B8)
```shell
<unk>
<s>
</s>
<pad>
<CLS>
<SEP>
<EOD>
<MASK>
\n
▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁	
▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁	
▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁▁▁▁▁	
▁▁▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁▁
▁▁▁▁▁▁▁	
▁▁▁▁▁▁
▁▁▁▁▁
▁▁▁▁
▁▁▁
▁▁
▁
```
- ※事前学習ライブラリの処理の中で、語彙サイズが128の倍数でない場合にダミートークンを追加して128の倍数になるような処理が入っていたことから、初めから128の倍数として設定しておくこととした。（56,320）


