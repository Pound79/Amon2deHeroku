# Amon2の導入
[Amon2入門](https://github.com/perl-entrance-org/Perl-Entrance-Textbook/blob/master/amon2/1.md)の手順でCartonのインストールと`$ carton install`による依存モジュールのインストールまで終わらせる.

# Herokuで公開
[Herokuで学ぶ、初めてのPerl](http://sssslide.com/speakerdeck.com/akiym/herokudexue-bu-chu-metefalseperl#105)のスライドの手順でHerokuで公開しようとするとうまくいかない・・・.
具体的には、Herokuのgitへpushするときにrejectされてしまう.

`$ heroku create appname --stack cedar --buildpack https://github.com/miyagawa/heroku-buildpack-per …`
`Creating ⬢ appname... done, stack is cedar-14`
`Setting buildpack to https://github.com/miyagawa/heroku-buildpack-per … done`
`https://appname.herokuapp.com/ | https://git.heroku.com/appname.git`

`$ git push heroku master`
`Counting objects: 118, done.`
`Delta compression using up to 4 threads.`
`Compressing objects: 100% (108/108), done.`
`Writing objects: 100% (118/118), 336.40 KiB | 0 bytes/s, done.`
`Total 118 (delta 22), reused 0 (delta 0)`
`remote: Compressing source files... done.`
`remote: Building source:`
`remote: `
`remote: -----> Failed to detect app matching https://github.com/miyagawa/heroku-buildpack-per … buildpack`
`remote: More info: https://devcenter.heroku.com/articles/buildpacks …`
`remote: `
`remote: ! Push failed`
`remote: Verifying deploy...`
`remote: `
`remote: ! Push rejected to appname.`
`remote: `
`To https://git.heroku.com/appname.git`
`! [remote rejected] master -> master (pre-receive hook declined)`
`error: failed to push some refs to 'https://git.heroku.com/appname.git'`

これはapp.psgiがルートディレクトリに存在しなかったために起きたエラー.
app.psgiにあたるファイルはscript/appname-serverであるため、これをルートディレクトリにコピーし、名前をapp.psgiに変更する.
しかしこのままだとモジュールが読み込めないので、パスを通す必要がある.
Perlではuseやrequireを使ってモジュールを読み込む場合、@INC配列に入ってるパスのどこかに所定の形式で配置されている必要があるそう.
パスの追加の方法はいろいろあるみたいだが、今回は以下のようにして必要なパスを追加した.

```html:app.psgi
BEGIN {
    push(@INC, '/app/lib');
    push(@INC, '/app/lib/appname');
    push(@INC, '/app/local/lib/perl5');
    push(@INC, '/app/local/lib/perl5/darwin-2level');
}
```

これで一つエラーは解消する.
この時点で`$ git push heroku master --force`でHerokuのgitにpushすることはできる.
しかし`$ heroku open`でブラウザ表示してもApplication Errorの表示になる.
調べると使っている[buildpack](https://github.com/miyagawa/heroku-buildpack-perl/tree/carton)がPerlのver5.24.1に対応していないようなので、対応している5.16.3にPerlのバージョンを落として再挑戦.
しかし同様のエラーが出てしまう.
しょうがないので[こちら](http://qiita.com/vzvu3k6k/items/6d893462c790742ed230)で紹介されていた[おすすめbuildpack](https://github.com/pnu/heroku-buildpack-perl)を使うことにしてみる.
すると、足りてないモジュールを大量に要求されるので、ひたすらcpanfileに追加する.

```html:cpanfile
requires 'TAP::Harness::Env';
requires 'Module::Build::Tiny';
requires 'ExtUtils::ParseXS', '3.18';
requires 'Getopt::Long', '2.39';
requires 'JSON::PP', '2.27300';
requires 'CPAN::Meta', '2.120921';
requires 'ExtUtils::MakeMaker', '6.64';
requires 'Devel::PPPort', '3.22';
requires 'IO::Socket::IP';
requires 'HTTP::Tiny', '0.034';
requires 'XSLoader', '== 0.24';
requires 'List::MoreUtils';
requires 'Parallel::Prefork';
requires 'Starlet';
```

だいたいこのあたりを追加することで、無事Herokuにpush&ブラウザ上で表示されることができた.
どこか抜けてるところもあるかも・・・.

# Amon2自分用メモ
`<p><a href="/test_data?value=10">テスト</a></p>`
`my $value = $c->req->parameters->{value};`

でURLに引数(value=10)を渡す処理ができた.
調べるとPerlではURLに引数を渡す場合QUERY_STRINGを参照する必要があるとあったが、Amon2では必要なし!?

`my $rs = $sth->fetchall_arrayref(+{});`
これでcsvのデータをテンプレート表示させられる.





