==========================================================
できる！Djangoでテスト!
==========================================================

| tell-k
| DjangoCongress JP 2018 (2018.05.19)

おまえ誰よ？
=====================================

.. image:: https://pbs.twimg.com/profile_images/1045138776224231425/3GD8eWeG_200x200.jpg

* tell-k
* BePROUD.inc
* 情弱プログラマー
* https://twitter.com/tell_k
* https://tell-k.github.io/djangocongressjp2018/

BePROUD - Pythonメインの受託開発
========================================

.. figure:: https://dl.dropboxusercontent.com/spa/ghyn87yb4ejn5yy/3ce742cb9bef4e4f925cb7385e494ff4.png
   :width: 70%

   https://www.beproud.jp/

connpass - エンジニアをつなぐIT勉強会支援プラットフォーム
===============================================================

.. figure:: https://dl.dropboxusercontent.com/spa/ghyn87yb4ejn5yy/c4f1511638f241daaaac9e29ed3151c1.png
   :width: 70%

   https://connpass.com/

PyQ - Pythonオンライン学習サービス
========================================

.. figure:: https://dl.dropboxusercontent.com/spa/ghyn87yb4ejn5yy/e9121e88c3b64179993a02198a7514f9.png
   :width: 70%

   https://pyq.jp/ ★ Djangoを使ったWeb開発も学習できます! ★

目的/動機
=====================================

* Djangoでユニットテスト書く時どうやってるの？と漠然と聞かれることがあった
* ユニットテストの書き方、利用してるライブラリ、気にしてる点。ハマりポイント。色々なトピックがあった
* それらを一回まとめてみたいなと思った次第です。

対象
=====================================

* Djangoをはじめようとしてる人
* ユニットテストとかをどうやってるのか知りたい人
* ある日、突然「 **いい感じにテスト書いて** 」と丸投げされて困惑してる人

今日の目標
=====================================

.. image:: https://dl.dropboxusercontent.com/spa/ghyn87yb4ejn5yy/40dbf595606e4879961ef4a13e5cea84.png
   :width: 60%

主な参考文献
=====================================

* `テスト駆動開発 <https://www.amazon.co.jp/dp/4274217884>`_
* `xUnit Test Patterns <http://xunitpatterns.com/>`_
* `エキスパートPythonプログラミング改訂2版 <https://www.amazon.co.jp/dp/4048686291>`_
* `Pylons 単体テストガイドライン <http://docs.pylonsproject.jp/en/latest/community/testing.html>`_

  * `効果的なunittest - または、callFUTの秘密 <http://pelican.aodag.jp/xiao-guo-de-naunittest-mataha-callfutnomi-mi.html>`_

* この辺から用語/トピックをピックアップします。

前提
=====================================

* サンプルコードは全て Python 3.6,  Django 2.0
* 非機能要件や受け入れテストの等の話はしません。
* テスト駆動開発そのものについては話しません
* おすすめパッケージも紹介はしますが、なるべく依存パッケージは増やさない方向です

テストの種類
=========================================

* ユニットテスト <- ``ほとんどこれの話``

  * 個々の関数やクラスをテストし、出力結果が予想通りであることを確認するテストです。

* 統合テスト

  * いくつかのモジュールを組み合わせて予想通りに動作するか確認するテスト。

* 機能テスト

  * ユーザーから見える範囲での機能を（例えばブラウザを使って）テストします。確実に想定した動作をするかといった内部構造は考慮しません。

ユニットテストに期待すること
===================================

* 実装が意図した通りに動くか素早く確認できること。
* 不安になくリファクタリングを始められるようになること。
* テストコード自体が簡単ドキュメントの役割を果たしてくれること。

Developer Testing
===================================

.. figure:: http://image.gihyo.co.jp/assets/images/dev/serial/01/tdd/0003/thumb/TH400_tdd03.png
   :width: 70%

   via. `第3回　「テスト」という言葉について <http://gihyo.jp/dev/serial/01/tdd/0003>`_

目次
==========================================

* テスト設置場所
* テストケースを書く
* テストを実行する
* フィクスチャー
* モック 
* コードカバレッジ
* 雑多なネタ
* まとめ

テスト設置場所
=============================

テスト設置場所
=============================

* Djagnoアプリの直下に ``tests`` パッケージを用意
* アプリ内のモジュールに対応する、モジュールを作成する

.. code-block:: bash

 sample
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── forms.py
    ├── models.py
    ├── utils.py
    ├── urls.py
    ├── views.py
    └── tests
       ├── __init__.py
       ├── test_admin.py  
       ├── test_forms.py
       ├── test_models.py
       ├── test_utils.py
       └── test_views.py


テストケースを書く
=============================

単純な関数をテストしたい
=============================

* 例えば以下のような関数をテストしたい

.. code-block:: python

  # sample/utils.py ----

  from datetime import date
  
  def diff_days(from_date, to_date):
      """ 
      - from_date から to_date までの日数を返す。
      - from_date が to_date 以降であれば None を返す。
      """
      if from_date >= to_date:
          return None
      return (to_date - from_date).days
  
  # Usage --
  date1 = date(2018, 1, 1)
  date2 = date(2018, 1, 6)
  
  print(diff_days(date1, date2)) # => 5
  print(diff_days(date2, date1)) # => None

テストケースを書く
=============================

.. code-block:: python

 # sample/tests/test_utils.py ----

 import unittest
 from datetime import date
 
 class TestDiffDays(unittest.TestCase): 
 
     def _callFUT(self, from_date, to_date):
         from spam import diff_days 
         return diff_days(from_date, to_date)
 
     def test_from_date_is_before(self): 
         """ from_date が to_date より古い日付 """
         actual = self._callFUT(date(2018, 1, 1), date(2018, 1, 6))
         self.assertEqual(5, actual)
 
     def test_from_date_is_after(self):
         """ from_date が to_date と同じか新しい日付 """
         actual = self._callFUT(date(2018, 1, 6), date(2018, 1, 1))
         self.assertIsNone(actual)


Pylons 単体テストガイドライン
===================================

* ここでは以下のルールに沿っています。
* `ルール: テスト対象のモジュールをテストモジュールのスコープでインポートしない <http://docs.pylonsproject.jp/en/latest/community/testing.html#id3>`_
* `ルール: 各テストケースメソッドは、 1つのことだけをテストする <http://docs.pylonsproject.jp/en/latest/community/testing.html#id3>`_

テスト対象のモジュールをテストモジュールのスコープでインポートしない
=============================================================================

* モジュールスコープでインポートエラーになると、他の関係ないテストも実行できなくなる
* できる限り他のテストケースに影響を与えないような配慮をする

.. code-block:: python

 # FUT = Function Under the Test = テスト対象の関数
 def _callFUT(self, from_date, to_date):
     from spam import diff_days 
     return diff_days(from_date, to_date)

.. code-block:: python

 # Bad --

 from spam import diff_days  # ImportErrorが発生する

 class TestDiffDays(unittest.TestCase): 
 
     def test_from_date_is_before(self): 
         # 〜 省略 〜 

 class TestOther(unittest.TestCase):  # <- X 関係ないテストも実行されない


各テストケースメソッドは、 1つのことだけをテストする
=========================================================

* 全部のテストパターンをごちゃまぜにしない。
* テストが落ちた時は片方しかテストされない。

.. code-block:: python

  # Bad --

  def test_all_test_cases(self): 
      
      # from_date < to_date 
      actual = self._callFUT(date(2018, 1, 1), date(2018, 1, 6))
      self.assertEqual(5, actual)

      # from_date >= to_date 
      actual = self._callFUT(date(2018, 1, 6), date(2018, 1, 1))
      self.assertIsNone(actual)
  

同値分割/境界値分析
=======================================

* **何を気にしてテストを書くのか？**
* 同値分割 ... テスト結果をグループ化し代表的な条件をピックアップしてテスト
* 境界値分析 ... テスト結果が変わる境目となる条件をテスト

  * 例えば 日付の範囲、数値の範囲
  * テストケースが成立するエッジケースをテストする

同値分割/境界値分析
=======================================

.. code-block:: python 

  def test_from_date_is_before(self): 
      """ from_date が to_date より古い日付 """
      actual = self._callFUT(date(2018, 1, 1), date(2018, 1, 6))
      self.assertEqual(5, actual)

      # 1日前だったらという境界値
      actual = self._callFUT(date(2018, 1, 1), date(2018, 1, 2))
      self.assertEqual(1, actual)

  def test_from_date_is_after(self):
      """ from_date が to_date と同じか新しい日付 """

      actual = self._callFUT(date(2018, 1, 6), date(2018, 1, 1))
      self.assertIsNone(actual)

      # 同日だったらという境界値
      actual = self._callFUT(date(2018, 1, 1), date(2018, 1, 1))
      self.assertIsNone(actual)


Assertion Roulette
=======================

* xUnit Patterns の `テストの不吉な臭い <http://xunitpatterns.com/Test%20Smells.html>`_ の一つ
* 1つのテストケースで複数の入力パターンをテストしている

このような場合

* どのデータが原因でテストが失敗したかわかりにくい
* テスト失敗以後のアサーションが行われない

.. code-block:: python

  # Bad --

  def test_say_hello(self): 
      self.asserEqual(say_hello('tell-k'),   'hello tell-k')   # 1. 失敗
      self.asserEqual(say_hello('hirokiky'), 'hello hirokiky') # 2. 以後のアサーションは無視
      self.asserEqual(say_hello('django'),   'hello django')
      self.asserEqual(say_hello('bucho'),    'hello bucho')
      self.asserEqual(say_hello('james'),    'hello james')
      self.asserEqual(say_hello('nakagami'), 'hello nakagami')
      self.asserEqual(say_hello('crohaco'),  'hello crohaco')

Parameterized Test がおすすめ
===============================

* TestCase.subTest ... Python3.4 で追加。テストケースにテストケースを作れる
* pytest だと `pytest.mark.parametrize <https://docs.pytest.org/en/latest/parametrize.html>`_

.. code-block:: python

 # Good --

 def test_say_hello(self): 
     names = ['tell-k', 'james', 'django', 'bucho', ...]
     for name in names:
        with self.subTest(name=name, expected='hello %s' % name):
            self.assertEqual(say_hello(name), expected)  # <- テストが失敗しても次のサブテストは実行される

* `これであなたもテスト駆動開発マスター！？和田卓人さんがテスト駆動開発問題を解答コード使いながら解説します～現在時刻が関わるテストから、テスト容易性設計を学ぶ #tdd <https://web.archive.org/web/20160819210805/https://codeiq.jp/magazine/2013/11/1475/>`_

Django に 依存するテストケース
==================================================

* Djagnoに依存するテストは ``django.test.TestCase`` を利用します
* 1テストケース実行する度にDBをクリアしてくれます

.. figure:: https://docs.djangoproject.com/en/2.0/_images/django_unittest_classes_hierarchy.svg
   :width: 70%

django.test.TestCase
========================

* モデルを使うようなところでは必須
 
.. code-block:: python

 from django.test import TestCase
 
 from sample.models import Item
 
 class TestSample(TestCase):
 
     def test_one(self):
         Item.objects.create(name='name1')
         self.assertEual(1, len(Item.objects.all()))
 
     # テストケースが終わるとDBの中身はクリア(rollbackされる)
     def test_two(self):
         Item.objects.create(name='name1')
         self.assertEual(1, len(Item.objects.all())) 

テストを実行する
=======================================

* テストを探して実行する = テストランナー
* 最近は `pytest <https://docs.pytest.org/en/latest/>`_ を使う人も多いと思いますが、ここではDjangoの標準のものを使います。

.. code-block:: bash

 $ python manage.py test

 # テストt用に設定ファイルを用意して
 $ python manage.py test --settings sample.settings_test

.. code-block:: bash

 $ python  manage.py test 
 Creating test database for alias 'default'...  # <- テスト用DB生成
 System check identified no issues (0 silenced).
 .....................................................................
 .....................................................................
 ----------------------------------------------------------------------
 Ran 279 tests in 15.139s

 OK (skipped=0)
 Destroying test database for alias 'default'... # <- テスト用DB破棄


pytest を 使いたい人に注意点
=======================================

* ``XXXTest`` というクラス名はデフォルトでは探してくれない 
* ``TestXXX`` という風に ``Test`` プレフィックスが必要

  * https://stackoverflow.com/a/20277099
  * http://pytest.readthedocs.io/en/latest/goodpractices.html#conventions-for-python-test-discovery

* ``unittest.TestCase`` (≒ ``django.test.TestCase`` ) と組み合わせると一部使えない機能がある

  * https://docs.pytest.org/en/latest/unittest.html#pytest-features-in-unittest-testcase-subclasses

どこまでユニットテストの対象にすべきか? 
==========================================

* 自分たちが書いたコードに対してテストを書く。
* Djangoやサードパーティのライブラリのテストしない。
* テスト対象が依存してる処理/コンポーネントは対象としない

  * 個別にユニットテストする。
  * 依存部分はモック(後述)などで置き換える。

* デバッグ目的のコードは意図的にテストしないこともある

  * ``__repr__`` とかね

フィクスチャー
=============================

フィクスチャー
=============================

* テストに必要な状態や条件を用意した環境やデータのこと
* TestCase では ``setUp``, ``tearDown`` で用意できる
* メソッド群

  * ``setUp``         ... メソッド単位の前処理
  * ``tearDown``      ... メソッド単位の後処理
  * ``setUpClass``    ... クラス単位の前処理
  * ``tearDownClass`` ... クラス単位の後処理

* `djangoでDBを使ったテストを書く時、setUpTestData()を使うと早く出来る場合がある <https://qiita.com/podhmo/items/e941371cbe2b11f5951f>`_

フィクスチャー
=============================

.. code-block:: python
 
 from django.test import TestCase

 class TestDoSomething(TestCase):

     def _callFUT(self, data):
         from sample.api import compose_data
         return do_something(data)

     def setUp(self):
         # フィクスチャーの生成
         self.good_data = make_fixture_data(good=True)
         self.bad_data = make_fixture_data(bad=True)

     def tearDown(self):
         # フィクスチャーの破棄
         del self.good_data
         del self.bad_data
         
     def test_do_something_ok(self):
         self.assertTrue(self._callFUT(self.good_data))

     def test_do_something_ng(self):
         self.assertFalse(self._callFUT(self.bad_data))


なるべくセットアップを共有しない
===================================

* `ガイドライン: self の属性によってではなく、ヘルパーメソッドによってセットアップを共有する <http://docs.pylonsproject.jp/en/latest/community/testing.html#self>`_
* あるテストケースでは必要でも、他のテストケースでは必要ない
  
 * 無駄に生成している

* テストケース毎にカスタマイズしづらい
* 無駄なフィクスチャー生成が省ければ、テストの実行も早くなる

なるべくセットアップを共有しない
====================================

.. code-block:: python

 # Good --
 
 from django.test import TestCase

 class TestDoSomething(TestCase):

     def _callFUT(self, data):
         from sample.api import compose_data
         return do_something(data)

     def test_do_something_ok(self):
         good_data = make_fixture_data(good=True)
         self.assertTrue(self._callFUT(good_data))

     def test_do_something_ng(self):
         bad_data = make_fixture_data(bad=True)
         self.assertFalse(self._callFUT(bad_dataa))


Djangoモデルのフィクスチャー
==========================================

* TestCase に json/yaml からモデルオブジェクトを生成する機能がある
* `Providing data with fixtures <https://docs.djangoproject.com/en/2.0/howto/initial-data/#providing-data-with-fixtures>`_

だけど..

* 1レコード単位のデータをファイルで管理するの大変
* クラス単位でしか設定できない -> テストケースごとに変更が難しい
* あまりおすすめできない。

.. code-block:: python

 from django.test import TestCase
 from myapp.models import Animal

 class AnimalTestCase(TestCase):
    fixtures = ['animals.json']


factory_boy
==========================================

* http://factoryboy.readthedocs.io/en/latest/
* Djangoモデルのをいい感じに用意してくれる 
* Django以外にもSQLAlchemy、MongoEngineなど対応してくれる
* 同種のものに  `model_mommy <http://model-mommy.readthedocs.io/en/latest/index.html>`_ がある

factory_boy
==========================================

.. code-block:: python

 # sample/tests/factories.py  
 import factory
 
 from account.tests.factories import UserFactory
 
 class ItemFactory(factory.django.DjangoModelFactory):
     name = factory.Sequence(lambda n: 'name{}'.format(n))
     email = factory.Sequence(lambda n: 'hoge{}@example.com'.format(n))
     price = 100 
     owner = factory.SubFactory(UserFactory)
 
     class Meta:
         model = "sample.Item"

.. code-block:: python

  item = ItemFactory()
  print(item.name) # => name0
  print(item.user) # => User object

  # フィールドの値も指定できる
  ItemFactory(name='newitem')

  # 一気に複数オブジェクトを生成することもできる
  ItemFactory.create_batch(10)

factroy_boy のハマりポイント
=================================

* デフォルト値に依存したテストを書いてしまう
* **誰かが知らずにデフォルト値を変更するとテストが失敗する**
* Fragile Test(Fragile Fixture) ... `テストの不吉な臭い <http://xunitpatterns.com/Test%20Smells.html>`_

  * フィクスチャの準備をするコードを修正したら、無関係なテストが失敗する

.. code-block:: python

  # テスト対象

  def get_display_price(item):
      return "{}円".format(item.price)

.. code-block:: python

  # Bad --

  def test_display_price(self):
      item = ItemFactory()  # <- ItemFactory.price 100から変更されたらテスト失敗
      expected = '100円'
      self.assertEqual(expected, self._callFUT(item)) 


factroy_boy のハマりポイント
=================================

* テストケース内で必要なフィクスチャは、テストケース内で生成する

.. code-block:: python
  
 # Good --
 
 def test_display_price(self):
     item = ItemFactory(price=100) # <- 100固定
     expected = '100円'
     self.assertEqual(expected, self._callFUT(item)) 

* デフォルト値に依存しないテストにする

.. code-block:: python

 # Good --
 
 def test_display_price(self):
     item = ItemFactory()
     expected = '{}円'.format(item.price)  # <- item.price を使って期待値を生成
     self.assertEqual(expected, self._callFUT(item)) 

モック
=================================

モック
=================================

* テスト対象が依存してる処理/コンポーネントを置き換える
* 例えば、以下のようなものを置き換える
   
  * 構築の準備に手間がかかるオブジェクト
  * 実際にネットワーク通信が必要になる処理 => 外部APIとの通信
  * テスト実行時に変化する値、日付

* xUnit Test Patterns では **Test Double** として分類/整理されている
* `xUnit Test PatternsのTest Doubleパターン(Mock、Stub、Fake、Dummy等の定義) <http://goyoki.hatenablog.com/entry/20120301/1330608789>`_

Test Double
=================================

.. figure:: http://xunitpatterns.com/Types%20Of%20Test%20Doubles.gif
   :width: 80%

Test Double
=================================

* **間接入力** ... テストコードから見えないテスト対象への入力
* **間接出力** ... テストコードから見えないテスト対象の出力

----

* **Dummy Object** ... テストに影響を与えない代替オブジェクトです。
* **Test Stub**    ... 間接入力値をテスト対象に返す
* **Test Spy**     ... 間接出力値を記録/参照可能にする
* **Mock Object**  ... 間接出力を記録/検証可能にする
* **Fake Object**  ... 実際のオブジェクトに近い処理をするが、簡易な実装となっている

----

* モック(=Test Dobule) の意として話ます。

unittest.mock
==================================

* Python3.3 から `unittest.mock <https://docs.python.org/ja/3.6/library/unittest.mock.html>`_ が追加

.. code-block:: python

  # sample/api.py ---
  from item.api import calc_tax_included_price

  # テスト対象
  def get_display_price(item):
      price = calc_tax_included_price(item)  # <- これをモック(Test Stub)に置き換える
      return "{}円".format(price)

.. code-block:: python

  def test_display_price(self):
      item = ItemFactory()

      # patch を通して 108 という間接入力値 をテスト対象(get_display_price) に渡してる
      with mock.patch('sample.api.calc_tax_included_price', return_value=108) as m:
           expected = '108円'
           
           self.assertEqual(expected, self._callFUT(item))  # => OK 

           # calc_tax_included_price に item引数が渡ったかチェック
           m.assert_called_with(item)

 
mock.patch の ハマりポイント
============================

* mock.patchがうまく当たらないケースがある

.. code-block:: python
 
  # egg.py  ---
  import spam

  def say_egg():
      return spam.say_spam() # <- patch対象

.. code-block:: python

  from unittest import mock
  from egg import say_egg

  with mock.patch('spam.say_spam', return_value="Patched!"):
      print(say_egg()) # => Patched! 

* 何の問題もなくパッチできている

mock.patch の ハマりポイント
============================

* 下記のように書き換えると **patchが失敗する**

.. code-block:: diff
 
 # egg.py  ---
 - import spam
 + from spam import say_spam

 def echo():
 -   return spam.say_spam()
 +   return say_spam()

* ``from import`` で importされたものは、元のモジュールから切り離される
* テスト対象( ``say_egg`` ) が利用してるものに ``patch`` をあてる。

.. code-block:: diff

 # Good
 
 - with mock.patch('spam.say_spam', return_value="Patched!"):
 + with mock.patch('egg.say_spam', return_value="Patched!"):
      print(say_egg()) # => Patched! 

* ``patch`` の影響下を局所化する意味でも import されてるところで patchする方が良いです。

モック その他
==================================

* 詳しいmockの使いかた

 * `まだmockで消耗してるの？mockを理解するための3つのポイント <http://note.crohaco.net/2015/python-mock/>`_

* ``datetime.now`` はpatchできない

 * `Python: freezegun で時刻のテストを楽に書く <http://blog.amedama.jp/entry/2016/12/06/220000>`_
 * `testfixtures - Mocking dates and times <https://testfixtures.readthedocs.io/en/latest/datetime.html>`_

* HTTPリクエスト(``requests`` )をモックしたい

  * `Responses <https://pypi.org/project/responses/>`_

モック その他
==================================

* Redisをモックしたい

  * `FakeRedis <https://pypi.org/project/fakeredis/>`_

* mock.patch を同時に複数あてたい

  * `contextlib.ExitStack <https://docs.python.jp/3/library/contextlib.html#contextlib.ExitStack>`_
  * `Python 3.3 からの with 文 <https://atsuoishimoto.hatenablog.com/entry/2013/01/25/000000>`_


可能な限り簡潔に
==================================

* `ガイドライン: fixture を可能な限り単純にしてください <http://docs.pylonsproject.jp/en/latest/community/testing.html#fixture>`_
* フィクスチャーやモックは可能限り簡潔にするのが良い
* モックを使いすぎて逆に混乱する
* モック使いすぎ = **設計見直し/リファクタリングのチャンス**

コードカバレッジ
===============================

コードカバレッジ
===============================

* テストコードがテスト対象を、どれくらいパスしているか計測したもの
* これを計測しながらユニットテストを書いていくのが個人的におすすめです。
* 見るべきポイント

 * テストが意図した通りにパスしてるか
 * テストが書かれてない場所がないか確認
 * 不要なコードをないか(使われてない, 到達不能なコード)

コードカバレッジの分類
===============================

* C0: 命令網羅(statement coverage)

  * 全ての命令を一度は実行する

* C1: 分岐網羅(branch coverage)

  * 全ての分岐条件をパスする

* C2: 条件網羅(condition coverage)

  * 全ての条件をパスする

* C0/C1 くらいまでならカバレッジツールが計測できる

カバレッジの計測ツール
===============================

* `coverage <https://pypi.org/project/coverage/>`_ パッケージを利用
* pytest であれば `pytest-cov <https://pypi.org/project/pytest-cov/>`_ とかで使える

カバレッジ
============================

* %もみるが、何行目がパスしてないかを良く見る。

.. code-block:: bash

 $ coverage run --omit 'manage.py','*/migrations/*' python manage.py test
 $ coverage report -m

* 分岐網羅まで計測したければ ``--branch`` オプションをつけて実行する

.. code-block:: text

  Name                          Stmts   Miss  Cover   Missing
  -----------------------------------------------------------
  account/__init__.py               1      0   100%
  account/admin.py                145     22    85%   19, 24-26, 59, 63-64, 100, 104-105, 137, 141-142, 173, 177-178, 209, 213-214, 247, 251-252
  account/apps.py                   9      0   100%
  account/forms.py                 29      0   100%
  account/models.py               239      0   100%
  account/views.py                150      5    97%   59, 101, 199

  〜 省略 〜
  -----------------------------------------------------------
  TOTAL                         28606   1240    96%


設定ファイル
==================================

* .coveragerc
* http://coverage.readthedocs.io/en/coverage-4.2/config.html#configuration-files

::

  [run]
  omit = */migrations/*,manage.py


対象を絞ってテストする
=============================

* どこかで使われていて、たまたまカバレッジが上がってるだけのケース
* 対象を絞ってテストすることで...
  
 * **テストがない** ところがわかる
 * 素早くテストも終わる

.. code-block:: bash

 # 特定のモジューブ
 $ python manage.py test spam.tests.test_models   

 # 特定のテストクラス
 $ python manage.py test spam.tests.test_models.TestSpamClass 

 # 特定のテストケース
 $ python manage.py test spam.tests.test_models.TestSpamClass.test_sham 

システム全体での カバレッジ 100% に固執しない
=====================================================

* 「カバレッジ100% = テストが書かれている」ことがわかるだけ
* コードベースが巨大なほど、カバレッジをあげるのは大変になります。
* コード・システムの質は設計の見直し/リファクタリングを繰り返し行うことであがります。

先人たちのお言葉
================================

* 100％のテストカバレッジを誇りに思うということは、新聞のあらゆる言葉を読むことを誇りに思うようなものです。いくつかは他よりも重要です。(Kent beck)
* カバレッジ解析の価値は何なんだろう？まずは、テストが不十分なコードを見つけるのに役立つ。カバレッジツールをしょっちゅう実行して、テストのないコードを見つけておくのは価値のあることだ。それらのコードがテストされていないことで不安になるだろうか？ (Martin Fowler)

雑多なネタ
============

viewのテストどうしてる？
==============================

* `django.test.TestCase.client <https://docs.djangoproject.com/en/2.0/topics/testing/tools/#the-test-client>`_ を利用している。
* ダミーのWebブラウザとして利用できる。
* HTMLの中身まではあまりテストしない -> HTMLが頻繁に変わると辛いから
* 複雑なロジックはviewの中にかかない -> 外出してユニットテストを書く
* ``request`` オブジェクトを view 以外に持ち出さない -> requsetから取り出したデータを渡す
* ``TemplateResponse.context`` はcontextの辞書をそのあまま持ってるのでテストしやすい -> ``render`` はもってない
* TestCase.clientを使う => **機能テスト** 

viewのテストどうしてる？
==============================

.. code-block:: python

 class TestItemDetailView(TestCase):
 
     def _getTarget(self, pk):
         return reverse('item:detail', kwargs={'pk': pk})
 
     def test_not_found(self):
         res = self.client.get(self._getTarget(1))
         self.assertEqual(404, res.status_code)
 
     def test_display_item(self):
         item = ItemFactory()

         res = self.client.get(self._getTarget(item.id))

         self.assertEqual(item.id, res.context['item'].id)
         self.assertEqual(200, res.status_code)
         self.assertTemplateUsed(res, 'item/detail.html')

Eメール送信のテストは
==============================

*  ``manage.py test`` では **実際にメールは送信されない**
* 送信内容 ``mail.outbox`` から取得可能 + それをテストする

.. code-block:: python

 from django.core import mail
 from django.test import TestCase
 
 class TestEmail(TestCase):
 
     def test_send_email(self):
 
         mail.send_mail('subject', 
                    'body.', 
                    'from@example.com', 
                    ['to@example.com'], fail_silently=False)
 
         self.assertEqual(len(mail.outbox), 1)
         self.assertEqual(mail.outbox[0].subject, 'subject')

Django コマンドのテスト
==============================

* `django.core.management.call_command <https://docs.djangoproject.com/en/2.0/ref/django-admin/#django.core.management.call_command>`_ でテストしている

.. code-block:: python

 class TestSpamCommand(TestCase):

     def _callCommand(self, *args, **kwargs):
         from django.core.management import call_command
         return call_command('spam', *args, **kwargs)

     def test_spam_command(self):
         ... 

ログ/標準出力の確認
==============================

* 明確にチェックしたいデータがあるわけではない場合
* ログや標準出力で書き出された内容を確認したい時がある
* testfixturesに `LogCapture <https://testfixtures.readthedocs.io/en/latest/logging.html>`_ と `OutputCapture <https://testfixtures.readthedocs.io/en/latest/streams.html>`_ というのがあります。便利。

.. code-block:: pycon

 >>> import logging
 >>> from testfixtures import LogCapture
 >>> with LogCapture() as l:
 ...     logger = logging.getLogger()
 ...     logger.info('a message')
 ...     logger.error('an error')
 ...     
 >>> l.check(
 ...     ('root', 'INFO', 'a message'),
 ...     ('root', 'ERROR', 'an error'),
 ... )



テスト用に一時的にsettingsの中身を変更したい
==============================================

* 専用のユーティリティがある
* `override_settings <https://docs.djangoproject.com/en/2.0/topics/testing/tools/#django.test.override_settings>`_
* `modify_settings <https://docs.djangoproject.com/en/2.0/topics/testing/tools/#django.test.modify_settings>`_

.. code-block:: python

  class LoginTestCase(TestCase):
  
      @override_settings(LOGIN_URL='/other/login/')
      def test_login(self):
          response = self.client.get('/sekrit/')
          self.assertRedirects(response, '/other/login/?next=/sekrit/')

.. code-block:: python 

  class MiddlewareTestCase(TestCase):

      @modify_settings(MIDDLEWARE={
          'append': 'django.middleware.cache.FetchFromCacheMiddleware',
          'prepend': 'django.middleware.cache.UpdateCacheMiddleware',
      })
      def test_cache_middleware(self):
          response = self.client.get('/')


ジョブキュー(Celery)のユニットテスト
=========================================

* 非同期でジョブが動く
* ちゃんと動かすには celeryデーモンが必要

どうすれば良いのか？

* 同期的に実行するオプションがある
* ``CELERY_ALWAYS_EAGER = True`` 
* 関数としてそのままジョブの中身が実行される
* celeryデーモンも不要
* http://docs.celeryproject.org/en/3.1/configuration.html#std:setting-CELERY_ALWAYS_EAGER

tox
=========================================

* https://tox.readthedocs.io/en/latest/
* Pythonライブラリを複数バージョンでテストするツールです。
* パッケージングしなくても魅力的。
* 専用のvirtualenvを作ってくれる。
* 静的解析ツール(flake8, mypy)と併せて利用できる。

tox の 設定
=========================================

:: 

  [tox]
  envlist = py36, flake8
  skipsdist = true
  
  [testenv]
  deps = -r{toxinidir}/dev-requires.txt
  
  setenv =
    DJANGO_SETTINGS_MODULE = settings.test
  
  changedir = {toxinidir}/src
  commands = coverage run --omit "manage.py","*/migrations/\*" python manage.py test {posargs}
   
  [testenv:flake8]
  deps =
    flake8
    mccabe
  
  commands = flake8 .
  
  [flake8]
  exclude = tests/\*, \*/migrations/\*, urls.py, manage.py
  max-line-length = 100
 

tox の 実行
=========================================

.. code-block:: bash

 $ tox # 全ての testenv が実行される
 $ tox -e flake8 # flake8のtestenvだけ実行される

テストの高速化
=========================================

* (特にローカルでは) テストを頻繁に実行するので、可能な限り早くなってほしい。
* `Djangoでテストを速くするためにいろいろやってみた <http://y0m0r.hateblo.jp/entry/20130615/1371305730>`_

トピック

* django.test.TestCase -> unittest.TestCaseへの変更
* sqlite3 の in-meory DBを使う
* migrations を OFFにする
* PASSWORD_HASHERSの変更

----

* setupTestDataの利用検討
* python manage.py test の --parallel オプションを使う 

まとめ
=========================================

* 後から不安なく、設計を見直したり/リファクタリングできるようにテストを書こう
* まずはシンプルで必要十分なテストを書くところから始めよう
* フィクスチャーやモックのツールを使おう
* 先人達の知恵を有効活用しよう 

Pythonプロフェショナルプログラミング 第3版
============================================

来月くらいに第3版がでます！よろしくお願いします！(予定)

.. figure:: https://images-na.ssl-images-amazon.com/images/I/41jP7BdvluL._SX385_BO1,204,203,200_.jpg
   :width: 40%

Kent Beck のお言葉
=========================================

* Tests are the Programmer's Stone, transmuting fear into boredom
* テストはプログラマーの石(?)、恐れを退屈に変えてくれます。
* **不安が退屈に変わるまでテストしよう**

参考
===============================

* https://github.com/tell-k/djangocongressjp2018/blob/master/reference.rst
* Webページ や 書籍 の著者の皆さん 本当に ありがとうございます。m(_ _)m


ご静聴ありがとうございました
======================================

