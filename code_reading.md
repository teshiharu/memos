* croak
    * エラーを出してdie

* use integer;
    * ブロック内で整数算術を使う
    * 計算前に切り捨て

* splice(@array,0,2)
    * @arrayの0番目から2個を削除
    * 切り抜いた配列を返す

* +{},+[]
    * +でハッシュリファレンス、配列リファレンスであることを明示

* do { ... }
    * ブロック内で最後に評価された値を返す
    * 無名関数を作って即実行的な

* caller
    * 呼び出し元の情報を返す
    * @_ = caller  呼び出し元の(パッケージ名, ファイル名, 行数)
    * $_ = caller  呼び出し元のパッケージ名
 
* wantarray
    * 呼び出し元で戻り値にリストを要求していたらtrueになる
    * スカラを要求していればfalse

* __DATA__
    * perl内で__DATA__以下にある部分はコードとして処理されない
    * DATAという特殊ファイルハンドルで参照可能
    * <DATA>

* Scalar::Util::bressed
    * ブレスされていればパッケージ名を返す
    * されていなければundef

* no strict "refs"
    * 当該ブロック内でuse strictを解除

* \Q ... \E
    * 正規表現内で、はさまれた文字をメタ文字ではなく通常の文字として処理

* fileno FILEHUNDLE
    * ファイルハンドルがオープンしていれば数字(ファイル記述子)を返す
    * オープンしていなければundef
    * ファイル記述子　何番目にファイルを開いているか、みたいな
    * 0:標準入力、1:標準出力、2:標準エラー出力であらかじめとられている