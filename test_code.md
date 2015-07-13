# テストコード  

## テストコード基礎  

1. テストファイルをつくる  
    * t/に---.tという名前でテストファイルを作る 
         use strict;  
         use warnings;  
         use Data::Dumper;  
         use Test::More;  

         BEGIN { use_ok('Point') } ;  
         my $point = new_ok('Point');   

         done_testing;  
         みたいな  

2. テスト実行  
    * prove -Ilib もしくはMakefileがあればmake test  

3. テストの中身  
    * use_ok('Package') useできるか  
    * new_ok('Package') newできるか  
    * can_ok($module,@methods) メソッドを実行できるか  
    とか、テストの終わりは done_testing;  

4. カバレッジを測る  
    * Devel::Coverを使って網羅率を測定  
    * cover -testで測定開始(Makefileが無いとだめ？)  
    * 各種パラメータ  
        - stmt ステートメント 行を通ったかどうか  
        - bran ブランチ if,elseなどの枝分かれ  
        - cond コンディション and,orとかの条件評価 100%はむずい  
        - sub サブルーチン が呼ばれているかどうか  

5. htmlでカバレッジを見る  
    * plackup -p 5015 -e 'use Plack::App::Directory; Plack::App::Directory->new({root => "./cover_db"})->to_app'  

6. 覚え書き  
    * エラーを拾う時(Test::More)は、メッセージも確かめたほうがいい(throws_okとか)  
        - 予期せぬエラーで死んでいたら困るから  
    * autoprove コマンド　モジュールは忘れた .tを保存するたびにテスト実行  
    * Jenkins アプリ?定期的に自動でテストを実行してくれる  


## Catalystのテスト  

1. Catalyst::Test 'Bookmark::Web';
    * requestとかresponseが取得できるようになる
    * my $res - request('URL');  
    * $res->content => レスポンスの中身（html）
    * $res->is_success/is_error/code => 成功か/エラーか/応答ステータス(200,404とか)
    * like($res->content, qr/Error: /); contentに"Error: "が含まれればsuccess

2. ctx_request関数
    * my ($res,$c) = ctx_request('URL')でHTTPレスポンスとコンテキストを取得できる
    * is($c->stash->{msg}, "Error!") stash->{msg}が"Error!"ならsuccess

3. HTTP:Request::Common
    * $req = POST( "URL", "Content" => [ name => teshima, ～ ] ); ($res,$c)=ctx_request($req);
    * POSTでURLにリクエストを送ったのと同じ　$c->req->param('name')でteshimaが得られる
