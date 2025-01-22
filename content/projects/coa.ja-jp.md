+++
title = "Coa プログラミング言語"
tags = ["プログラミング言語", "情報科学の達人"]
+++

著者: 柴田謙, [高前田 伸也](https://sites.google.com/site/shinyaty/home-japanese)

主流のプログラミング言語では、並列実行によって実行速度を向上させる為に特別な構文等が必要になることが多く、並列実行を実現するのは容易ではありません。
並列実行をデフォルトに並列指向プログラミング言語「Coa」を開発しました。
競合状態を回避する為に、CPU のアウトオブオーダ実行と同様に、変数の依存関係を自動的に検出し、プログラムを並列実行します。
したがって、外から見た振る舞いは逐次実行と同じであり、プログラムの複雑さを増やさずに実行速度を向上させることが期待できます。

[Githubレポジトリ](https://github.com/nyiyui/go-coa)
&bull; [発表資料（英語のみ）](/projects/coa/coa_swopp-2022.pdf)
&bull; [発表スライド](https://docs.google.com/presentation/d/1Ooln_pojSlW_T7ZXb0aWG5PyNMTQwvMJoWqgsoFvu0U/preview)
&bull; [情報の達人プログラムでの発表動画](https://youtube.com/watch?v=fqGx2pAEzeg)
