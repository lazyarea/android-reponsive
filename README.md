# android-reponsive

はじめに

テストランナーとして「Karma」、テストフレームワーク・アサーションライブラリとして「Jasmine」を使う。
http://qiita.com/opengl-8080/items/cf3acafda9756f4b04c9

    SuiteとSpec
        Jasmine では、テストコードは Suite と Spec の２つで構成される。
        JUnit で言うと、 Suite はテストクラスで、 Spec はテストメソッド。
        Suite は describe 関数を使い、 Spec は it 関数で宣言する。
    値のチェックは expect(actualValue).toBe(expectedValue); で実施する（ toBe() 以外にも色々メソッドが用意されている ）。
    Spy
        特定のオブジェクトのメソッドが実行されたかどうかをチェックする
            ↑の引数をチェックする
            ↑で本来のメソッドを実行するかしないかを指定できる
                実行しない場合に別の処理を指定できる
            ↑で本来のメソッドの戻り値を捨てて別の値を返すよう指定できる
            ↑で戻り値を捨てて例外をthrowするよう指定できる
            h3. jasmine / karma の動作環境

前提として下記が使用できること

* node
* npm
* google-Chrome( + CentOS7)

上記の条件に該当するので、Windowsでも動作すると推測される。


h3. Angular.js

version 1.3.x (https://code.angularjs.org/1.3.17/angular-1.3.17.zip)

テストランナー

[Karma]
リモートテスト実行
CI(継続的インテグレーション)ツールとの連携機能がある

テストフレームワーク

[Jasmine]
コード化されたテストを実行
テスト結果の確認
モックライブラリの機能を持つ
Angular.js付属のモック機能を利用

・JasmineにはSpy, Mock機能が備わっている
http://nanananande.helpfulness.jp/post-62/

Jasmineはコマンドラインからテストの実行が出来る
Jasmineの構文やルールに従ってUT(UnitTest)を作成し、そのファイルが置かれているディレクトリを指定して(karma.conf.js作成で説明)Jasmineを実行という流れになる。Jasmineは指定したディレクトリ配下も参照をし見つかった全てのテストを実行する
デフォルトのテストファイル名規則はspecで終わる必要がある。(ex: foobar-spec.js 区切り方は調査してください)
テスト結果の確認
モックライブラリの機能を持つ
JasmineのmockはAngular.js付属のモック機能を利用
参考サイト(http://nanananande.helpfulness.jp/post-62/)

モック

[ngMock]
angular-mock.jsをincludeすることで利用可能
WebServerを利用しなくても自前のビルトインhttpサーバにより通信が出来たり、
タイマーを使っているコードで、指定した時間を待たなくても結果を得ることが可能となる
AngularJSは標準でDI(Dependency Injection)コンテナの機能を持つため、
依存モジュールを用意にモックに切り替えが可能

(参考)Jasmineのspy

http://www.buildinsider.net/web/bookjslib111/109]]

セットアップ

[参考]   http://qiita.com/YUTARO/items/9561478c7e82ffe0b988

Node.js

version v0.12.6 (脆弱性対応版)
(参考)Node.jsとは:http://liginc.co.jp/web/programming/node-js/85318

$ git clone git://github.com/creationix/nvm.git ~/.nvm
$ source ~/.nvm/nvm.sh
$ nvm install v0.12.6
$ node -v
    v0.12.6
$  nvm alias default v0.12.6
    default -> v0.12.6


$ vi ~/.bash_profile
1 if [[ -s ~/.nvm/nvm.sh ]];
2   then source ~/.nvm/nvm.sh
3 fi


$ source ~/.nvm/nvm.sh
$ git clone ssh://git01/var/lib/git/public_git/sample.git samplerepo
$ samplerepo

$ vi sample.js
1 var http = require('http');
2 
3 http.createServer(function (request, response) {
4   response.writeHead(200, {'Content-Type': 'text/plain'});
5   response.end('Hello World\n');
6 }).listen(8124);
7 console.log('Server running at http://127.0.0.1:8124/');

karma/jasmineセットアップ

$ npm install –save-dev karma
$ npm install karma-chrome-launcher
$ npm install karma-jasmine
$ npm install karma-coverage --save-dev

対話形式でkarma.conf.jsを作る
$ karma init
("$ karma init の詳細":https://wc.kyagroup.com:17677/projects/tokai1-unio/wiki/Karma%E3%83%BBjasminekarmaconfjs)

$ vi package.json
 1 {
 2     "name": "JSTestSample",
 3     "version": "0.0.1",
 4     "description": "Sample for JS Test.",
 5     "main": "index.js",
 6     "scripts": {
 7       "test": "node_modules/karma/bin/karma start" 
 8     },
 9     "author": "KYAgr.",
10     "license": "",
11     "devDependencies": {
12       "karma": "~0.12.9",
13       "karma-jasmine": "~0.1.5",
14       "karma-chrome-launcher": "~0.1.3" 
15     }
16   }

karma.conf.jsの一部
1 // list of files / patterns to load in the browser
2 files: [
3   './js/angular/angular.js',
4   './js/angular/angular-mocks.js',
5   './src/*.js',
6   './specs/*_spec.js',
7 ],

.
┃━━ js
┃    ┣━━ angular -> angular-1.3.9/
┃    ┗━━ angular-1.3.9
┣━━ karma.conf.js
┣━━ node_modules
┃    ┣━━ jasmine-core/
┃    ┣━━ karma/
┃    ┣━━ karma-chrome-launcher/
┃    ┗━━ karma-jasmine/
┣━━ package.json
┣━━ specs
┃    ┗━━ foo_spec.js
┗━━ src
      ┗━━ foo.js

テスト動作の流れ

    Jasmine のファイル・テスト対象のコード・テストコードを読み込んだ HTML をブラウザで表示することで、テストが実行され、結果が表示される。

boot.js

    boot.js は jasmine.js より後、かつテスト対象コードより前に読み込む必要がある。

テストスクリプトを書く

$ vi simple_spec.js
1 describe('一番最初のテスト', function () {
2   it('わざと足し算を失敗させる', function () {
3     expect(1 + 1).toEqual(3);
4   });
5 });

テストの実行コマンド

$ npm test 

コンソール出力例

[成功]
16 07 2015 19:50:30.296:WARN [karma]: No captured browser, open http://localhost:9876/
16 07 2015 19:50:30.307:INFO [karma]: Karma v0.13.0 server started at http://localhost:9876/
16 07 2015 19:50:30.326:INFO [launcher]: Starting browser Chrome
16 07 2015 19:50:32.983:INFO [Chrome 43.0.2357 (Linux 0.0.0)]: Connected on socket olVbxCh5f8hyG68oAAAA with id 47906405
Chrome 43.0.2357 (Linux 0.0.0): Executed 1 of 1 SUCCESS (0.02 secs / 0.002 secs)

[失敗]
INFO [karma]: Karma v0.12.37 server started at http://localhost:9876/
INFO [launcher]: Starting browser Chrome
INFO [Chrome 43.0.2357 (Linux 0.0.0)]: Connected on socket MHlwmInOFxtwrg0sGdOl with id 57309855
Chrome 43.0.2357 (Linux 0.0.0) 一番最初のテスト わざと足し算を失敗させる FAILED
      Expected 2 to equal 3.
          at Object.<anonymous> (/home/ksto/GITWORKS/simple_spec.js:3:19)
Chrome 43.0.2357 (Linux 0.0.0): Executed 2 of 2 (1 FAILED) (0.017 secs / 0.01 secs)

サンプルコード

$ vi src/foo.js
 1 function bar() {
 2 
 3 }
 4 // squaredNumber、sumefuncnameは任意の名前の関数となる
 5 // 引数を2乗する
 6 bar.prototype.squaredNumber = function(num){
 7     return num * num;
 8 };
 9 //引数を2倍にする
10 bar.prototype.sumefuncname = function(num){
11     return num + num;
12 };


$ vi specs/foo_spec.js
 1 describe("TestSample>", function(){
 2     describe("Logic>", function() {
 3         it("multiNumber", function() {
 4             var num = 3;
 5             var expected = 9;
 6 
 7             var target = new bar();
 8             var result = target.squaredNumber(num);
 9             expect(expected).toEqual(result);
10         })
11     });
12 
13     describe("Logic1>", function() {
14         it("multiNumber1", function() {
15             var num = 3;
16             var expected = 9;
17             var target = new bar();
18             var result = target.sumefuncname(num);
19             // 3 + 3 = 9であることを確認させる
20             expect(expected).toEqual(result);
21         })
22     });
23 });

	

このように書くことも可能
 1 describe("TestSample>", function(){
 2     var num = 3;
 3     var expected = 9;
 4     describe("Logic>", function() {
 5         it("multiNumber", function() {
 6             var target = new bar();
 7             var result = target.squaredNumber(num);
 8             expect(expected).toEqual(result);
 9         })
10     });
11 
12     describe("Logic1>", function() {
13         it("multiNumber1", function() {
14             var target = new bar();
15             var result = target.sumefuncname(num);
16             expect(expected).toEqual(result);
17         })
18     });
19 });

$ npm test

> JSTestSample@0.0.1 test /home/ksto/KARMA
> node_modules/karma/bin/karma start

17 07 2015 15:36:39.026:WARN [karma]: No captured browser, open http://localhost:9876/
17 07 2015 15:36:39.037:INFO [karma]: Karma v0.13.0 server started at http://localhost:9876/
17 07 2015 15:36:39.053:INFO [launcher]: Starting browser Chrome
17 07 2015 15:36:40.914:INFO [Chrome 43.0.2357 (Linux 0.0.0)]: Connected on socket jIPW8O7TCO_7_ri-AAAA with id 76902299
Chrome 43.0.2357 (Linux 0.0.0) TestSample> Logic1> multiNumber1 FAILED
    Expected 9 to equal 6.
        at Object.<anonymous> (/home/ksto/KARMA/specs/foo_spec.js:16:30)
Chrome 43.0.2357 (Linux 0.0.0) TestSample> Logic2> exception1 FAILED
    Error1 thrown
Chrome 43.0.2357 (Linux 0.0.0): Executed 4 of 4 (2 FAILED) (0.024 secs / 0.013 secs)

"TestSample> Logic1> multiNumber1"でFAILED。
"TestSample> Logic2> exception1"でFAILED。例外が投げられている

AngularJSのモジュールテスト

$ vi upperFilter_spec.js
1 angular.module('app', [])
2   .filter('upperFilter', function () {
3   return function (input) {
4     return angular.uppercase(input);
5   };
6 });

timeoutモックを利用したテスト(数値は秒)

1 describe("timerServiceのテスト", function () {
2   beforeEach(module('app'));
3   it("時間経過後にメッセージが変化する", inject(function ($timeout, timerService) {
4     expect(timerService.message).toEqual('まだだよ');
5     $timeout.flush();
6     expect(timerService.message).toEqual('またせたな');
7   }));

$httpBackendのモック

1 angular.module('app', ['ngResource'])
2     .factory('usersService', function ($resource) {
3       return function () {
4         return $resource('/api/users').query();
5       }
6     });
7   });

sample/chapter11/04_httpbackend_mock/users_service_spec.js&br;(本のサンプルコード:p306)

例外を投げる

$ vi src/exception.js
1 function bar() {}
2 // 例外を投げる
3 bar.prototype.exception1 = function(){
4     throw "Error1";
5 };


$ vi exception_spec.js
 1 describe("TestSample>", function(){
 2 
 3     describe("Logic2>", function() {
 4         it("exception1", function() {
 5             var target = new bar();
 6             var result = target.exception1();
 7         })
 8     });
 9 });

	

Chrome 43.0.2357 (Linux 0.0.0) TestSample> Logic2> exception1 FAILED
    Error1 thrown

例外発生させるフィルタ

$ vi sample/chapter11/05_exception_handler/simple_filter.js(本のサンプルコード:p307)
 1 angular.module('app.filter', [])
 2   .filter('upperFilter', function () {
 3     return function (input) {
 4       if (angular.isDefined(input) && !angular.isString(input)) {
 5         throw Error('input type is not String.')
 6       }
 7       return angular.uppercase(input);
 8     };
 9   });

$ vi sample/chapter11/05_exception_handler/unhandled_spec.js(本のサンプルコード:p309)
 1 describe("filter test", function () {
 2   beforeEach(module('app.service'));
 3   beforeEach(module(function ($exceptionHandlerProvider) {
 4     // (1) 補足されない例外が発生した場合は、例外情報をログに出力する
 5     $exceptionHandlerProvider.mode('log');
 6   }));
 7   it("補足されない例外の発生確認", inject(function (timerService, $timeout, $exceptionHandler) {
 8     // (2) 例外が発生していないことを確認
 9     expect($exceptionHandler.errors).toEqual([]);
10     // (3) 例外を発生させる
11     var cancel = timerService(10000);
12     $timeout.flush();
13     // (4) 例外が発生したことを確認
14     expect($exceptionHandler.errors[0].message).toEqual('timeout');
15   }));
16 });

Controller/ModelのUT

(参考サイト)http://qiita.com/uggds/items/5d0c643c187c14a1591b

複数テストを実行したい

前述の1 + 1 = 2の計算を間違えるテストを書くとする。
.
┣━━ specs
┃    ┗━━ sample0_spec.js
┃    ┗━━ sample1_spec.js
┗━━ src
      ┗━━ sample0.js
      ┗━━ sample1.js

$ npm test

[結果]
INFO [karma]: Karma v0.12.37 server started at http://localhost:9876/
INFO [launcher]: Starting browser Chrome
INFO [Chrome 43.0.2357 (Linux 0.0.0)]: Connected on socket sdbM-TASrlqHGZJ5kqmN with id 77485289
Chrome 43.0.2357 (Linux 0.0.0) ???????? ???????????? FAILED
        Expected 2 to equal 3.
            at Object.<anonymous> (/home/ksto/UT-example/specs/simple0_spec.js:3:21)
Chrome 43.0.2357 (Linux 0.0.0) ???????? ???????????? FAILED
        Expected 2 to equal 4.
            at Object.<anonymous> (/home/ksto/UT-example/specs/simple1_spec.js:3:21)
Chrome 43.0.2357 (Linux 0.0.0): Executed 2 of 2 (2 FAILED) ERROR (0.016 secs / 0.007 secs)

Coverage
karmaのプラグインを利用¶
