# データ前処理
トークナイザー学習のために、テキストは.txtファイルとして作成する。

## 日本語テキストの前処理
- 日本語以外のテキストは前処理をせずそのまま用いる。

## 形態素解析の実行
```shell
pip install fugashi[unidic]
python -m unidic download
```
