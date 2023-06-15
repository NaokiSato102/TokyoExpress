# マンガOCR

日本語テキストのための光学文字認識で、主に日本のマンガに焦点を当てています。  
カスタムのエンドツーエンドモデルを使用し、  
Transformersの[ビジョンエンコーダデコーダ](https://huggingface.co/docs/transformers/model_doc/vision-encoder-decoder)フレームワークで構築されています。

マンガOCRは、一般的な印刷日本語OCRとして使用することもできますが、  
その主な目的は、マンガ特有のさまざまなシナリオに対してロバストな、高品質なテキスト認識を提供することでした：
- 縦書きと横書きのテキスト
- フリガナ付きのテキスト
- 画像上のテキスト
- 様々なフォントとフォントスタイル
- 低品質の画像

多くのOCRモデルとは異なり、  
マンガOCRは一回のフォワードパスで複数行のテキストを認識することが可能で、  
これにより、マンガに見つけられるテキストバブルを一度に処理することができます。  
行に分割することなく。

参照：
- [Poricom](https://github.com/bluaxees/Poricom) - マンガOCRを使用するGUIリーダー
- [mokuro](https://github.com/kha-white/mokuro) - マンガOCRを使用してマンガのHTMLオーバーレイを生成するツール
- [Xelieuのガイド](https://rentry.co/lazyXel) - マンガOCR/mokuro（およびその他の便利なヒント）を使った読解と  
マイニングのワークフローを設定する包括的なガイド
- 開発コード（訓練と合成データ生成のコードを含む）：[リンク](manga_ocr_dev)
- 合成データ生成パイプラインの説明+生成された画像の例：[リンク](manga_ocr_dev/synthetic_data_generator)

# インストール

Python 3.6, 3.7, 3.8, または 3.9が必要です。  
残念ながら、まだPyTorchはPython 3.10をサポートしていません。

GPUで実行したい場合は、[こちら](https://pytorch.org/get-started/locally/#start-locally)に記述されているようにPyTorchをインストールします。  
そうでない場合は、この手順はスキップできます。

コマンドラインで実行：

```commandline
pip3 install manga-ocr
```

## トラブルシューティング

- `ImportError: DLL load failed while importing fugashi: The specified module could not be found.` - Microsoft StoreからPythonをインストールしたためかもしれません。  
[公式サイト](https://www.python.org/downloads)からPythonをインストールしてみてください。
- ARMアーキテクチャで`mecab-python3`のインストールに問題がある場合  
[このワークアラウンド](https://github.com/kha-white/manga-ocr/issues/16)を試してみてください。

# 使用法

## Python API

```python
from manga_ocr import MangaOcr

mocr = MangaOcr()
text = mocr('/path/to/img')
```

または

```python
import PIL.Image

from manga_ocr import MangaOcr

mocr = MangaOcr()
img = PIL.Image.open('/path/to/img')
text = mocr(img)
```

## バックグラウンドでの実行

マンガOCRはバックグラウンドで実行し、新たに現れる画像を処理することができます。

[ShareX](https://getsharex.com/)のようなツールを使用して画面の領域を手動でキャプチャし、  
OCRにシステムのクリップボードから、または指定したディレクトリから読み取らせることができます。  
デフォルトでは、マンガOCRは認識したテキストをクリップボードに書き込みます。  
これから、[Yomichan](https://github.com/FooSoft/yomichan)のような辞書がテキストを読み取ることができます。  
画像をクリップボードから読み取るのはWindowsとmacOSだけで、Linuxでは代わりにディレクトリから読み取るべきです。

日本語のマンガを辞書で読むためのフルセットアップは次のようになるかもしれません：

ShareXで領域をキャプチャ -> クリップボードに画像を書き込み -> (続)  
(続)-> マンガOCR -> クリップボードにテキストを書き込み -> Yomichan

https://user-images.githubusercontent.com/22717958/150238361-052b95d1-0152-485f-a441-48a957536239.mp4

- クリップボードから画像を読み取り、認識したテキストをクリップボードに書き込むには、  
コマンドラインで以下を実行します：
    ```commandline
    manga_ocr
    ```
- ShareXのスクリーンショットフォルダから画像を読み取るには、  
コマンドラインで以下を実行します：
    ```commandline
    manga_ocr "/path/to/sharex/screenshot/folder"
    ```
クリップボードスキャンモードで実行すると、  
クリップボードにコピーした任意の画像がOCRによって処理され、  
認識されたテキストに置き換えられます。  
もし普通のように画像をコピー＆ペーストしたい場合は、  
代わりにフォルダスキャンモードを使用し、  
OCR専用のタスクをShareXで定義し、  
スクリーンショットをクリップボードにコピーせずに何らかのフォルダに保存するべきです。

初回の実行では、モデルのダウンロード（約400MB）に数分かかることがあります。  
`OCR ready`メッセージがログに表示されたら、OCRは使用する準備が整っています。

- 他のオプションを見るには、コマンドラインで以下を実行します：
    ```commandline
    manga_ocr --help
    ```
もし`manga_ocr`が動作しない場合は、  
それを`python -m manga_ocr`で置き換えて試してみてもいいかもしれません。

## 使用のヒント

- OCRは複数行のテキストをサポートしていますが、  
テキストが長ければ長いほど、エラーが発生する可能性が高まります。  
もし長いテキストの一部の認識が失敗した場合、  
画像の小さな部分でそれを実行してみることを試してみてください。
- モデルはマンガをうまく扱えるように特別に訓練されていますが、  
他の種類の印刷テキスト、たとえば小説やビデオゲームに対してもまずまずの仕事をするはずです。  
しかし、手書きのテキストを処理することはできないでしょう。
- モデルは常に画像上のテキストを認識しようと試みます、たとえそれが存在しなくても。  
トランスフォーマーデコーダを使用しているため、（したがって日本語の理解がある程度あります）  
それは実際に見える文章を"夢見る"ことさえあるかもしれません！  
これはほとんどの使用ケースでは問題にならないはずですが、  
次のバージョンで改善されるかもしれません。

# 例

以下はモデルの能力を示す一部のピックアップされた例です。  
全体的な性能を評価するためには、公開されている[合成データセット](manga_ocr_dev/synthetic_data_generator)を参照してください。

- [ビジョンエンコーダデコーダのデモ](https://huggingface.co/spaces/nateraw/vision-encoder-decoder) - Manga109を使用した初期のデモ。  
最新版はもっと良いはずです。
- [合成データの例](manga_ocr_dev/synthetic_data_generator) - モデルが見たトレーニングデータの一部です。  
モデルはこれらの状況に対して非常に優れたパフォーマンスを発揮します。
- リアルなマンガの例：[リンク](manga_ocr_examples)

# フィードバックと貢献

- このモデルはまだ改善の余地がたくさんあります。  
何か問題が発生した場合や、特に挑戦的なシナリオに遭遇した場合は、  
その画像（著作権を尊重してください）を共有して、[GitHubのissues](https://github.com/kha-white/manga-ocr/issues)で問題を報告してください。  
それはモデルの改善に役立ちます。
- また、[mokuro](https://github.com/kha-white/mokuro)、[poricom](https://github.com/bluaxees/Poricom)、またはその他のプロジェクトに貢献したい方は、どんな形でも歓迎します。

# ライセンスと免責事項

マンガOCRはMITライセンスで提供されています。  
ただし、これを使用して何らかのコンテンツを違法に配布することは許可されていません。  
詳細については、[ライセンス](LICENSE)を参照してください。

また、このプロジェクトはOpenAIではなく、個人が行っているものであり、OpenAIとは関連していません。