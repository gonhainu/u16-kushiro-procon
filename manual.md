# U-16釧路プログラミングコンテスト競技部門Rubyマニュアル（新ver.）
<p><img src="http://i.imgur.com/9yYwzxu.png" alt="ロゴ" width="30%" height="30%"></p>

## 1. 使用言語
U-16釧路プログラミングコンテスト競技部門ではRubyというプログラミング言語を使用します．

### 1.1 Rubyとは
Rubyとはまつもとゆきひろ(matz)氏によって開発されたオブジェクト指向スクリプト言語と呼ばれるプログラミング言語です．

## 2. ファイル構成
クライアントプログラムのファイル構成は次の通りです．
~~~
rubychaser
├── README.md
├── chasers
│   ├── active.rb
│   ├── base.rb
│   ├── my_chaser.rb
│   ├── silent.rb
│   └── template.rb
├── clientstart.sh
├── connection.rb
├── const.rb
├── main.rb
├── maps.rb
└── utils.rb
~~~

* connection.rb  -- サーバと通信するためのプログラム．
* const.rb       -- 定数が定義されている．
* main.rb        -- これを実行するとCHaserが動く．
* maps.rb        -- マップを作成するプログラム．
* utils.rb       -- いろんな処理が書かれているプログラム．
* base.rb        -- CHaserの元となるプログラム．
* silent.rb      -- ひたすらsearchなどを繰り返して移動しないCHaserプログラム．
* active.rb      -- 壁がない方向にランダムに移動するCHaserプログラム．
* my_chaser.rb   -- デモ用のCHaserプログラム．
* template.rb    -- CHaserプログラムを実際に作成する際のテンプレート．
* clientstart.sh -- これを実行するとポート番号とホスト名と使用するCHaserプログラムを訊かれ入力すると動くようになる．

## 3. 実際のプログラム
active.rbは壁のない方向へランダムに移動するというプログラム．
```ruby
require_relative 'base'
require_relative '../const'

WALK_UP    = 1
WALK_LEFT  = 2
WALK_RIGHT = 3
WALK_DOWN  = 4

class ActiveCHaser < CHaser
  # ---ユーザ定義関数
  def safe_walk(info, direction)
    # 壁がなければ指定方向へ進む
    case direction
    when WALK_UP
      if info[1] == TYPE_BLOCK
        return false
      end
      return self.walkUp
    when WALK_LEFT
      if info[3] == TYPE_BLOCK
        return false
      end
      return self.walkLeft
    when WALK_RIGHT
      if info[5] == TYPE_BLOCK
        return false
      end
      return self.walkRight
    when WALK_DOWN
      if info[7] == TYPE_BLOCK
        return false
      end
      return self.walkDown
    end
  end
  #----------------

  def run(info)
    # 敵が上下左右のどこかにいればブロックを置く
    if info[1] == TYPE_ENEMY
      self.putUp
    elsif info[3] == TYPE_ENEMY
      self.putLeft
    elsif info[5] == TYPE_ENEMY
      self.putRight
    elsif info[7] == TYPE_ENEMY
      self.putDown
    else
      # ランダムで壁のない方向へ
      # 1: 上, 2: 左, 3: 右, 4: 下
      loop do
        val = rand(4) + 1 # 1~4の乱数
        result = self.safe_walk(info, val)
        if result
          break
        end
      end
    end

    print @map.display_text(@position) # マップ表示
  end
end
```

## 4. CHaserのコマンド
4つの系列のコマンドがそれぞれ上下左右ある．

* walk系（進む）
  * walkRight 右に1マス動く
  * walkLeft 左に1マス動く
  * walkUp 上に1マス動く
  * walkDown 下に1マス動く
* search系（1列分の周囲情報を得る）
  * searchRight
  * searchLeft
  * searchUp
  * searchDown
* look系（3x3の周囲情報を得る）
  * lookRight
  * lookLeft
  * lookUp
  * lookDown
* put系（ブロックを置く）
  * putRight
  * putLeft
  * putUp
  * putDown

## 5. 実際にプログラムを書く
chasers/template.rbをchasers/以下に別ファイル名でコピーしてください

```ruby
require_relative 'active'

class HogeCHaser < CHaser
  #--- ユーザ定義関数 ---

  #----------------------
  def run(info)
    # この中に1ターン分の処理を書く
  end
end
```

### 5.1 周囲情報

`info`という配列に周囲情報が入っている．
周囲情報は

* 床（何もなし）：0
* キャラクタ（C or H）：1
* ブロック：2
* アイテム（ハート）：3

の4つのいずれかが入ります．

それぞれ，

* TYPE_FLOOR
* TYPE_ENEMY
* TYPE_BLOCK
* TYPE_ITEM

という定数で置き換えることができます．

#### 配列
配列は複数の値を入れておく入れもの．周囲情報はこの入れものに入っている．  
添字と呼ばれる番号でそれぞれの要素を取得できる．  
例えば`info[0]`とすると0番目の要素を取得できる．  
**※注** プログラミング言語のほとんどは0番目からスタートする．

前にも書いたとおり周囲情報の入っている`info`は配列で出来ている．  
添字と周囲の位置関係は次のようになっている．4がキャラクタの位置である．

|   | L | C | R |
|:-:|:-:|:-:|:-:|
| U | 0 | 1 | 2 |
| C | 3 | 4 | 5 |
| D | 6 | 7 | 8 |

また，添字は次のような定数で置き換えることができる．

|       | L         | C      | R          |
|:-----:|:---------:|:------:|:----------:|
| **U** | UP_LEFT   | UP     | UP_RIGHT   |
| **C** | LEFT      | CENTER | RIGHT      |
| **D** | DOWN_LEFT | DOWN   | DOWN_RIGHT |

なので`info[UP_LEFT]`とすれば`info[0]`の要素，つまり，左上の情報が得られる．

## 6. 制御構造
制御構造には条件によって処理を変えるif文やforなどの繰り返しなどがある．

### 6.1 条件分岐
このCHaserというゲームにおいて最も重要な命令．  
これによってキャラクタの周りに何があるか判定し行動を実行する．

#### 6.1.1 if文
「もし〜ならば…する」という命令

必ず`if ... end`で囲み，その部分がひとかたまりとなる．  
分けたい処理の個数によって`elsif`や`else`などを使用する．

* ある条件時のみ処理をする場合

```ruby
if 条件
  処理
end
```

 また，処理が一つしかない場合には

 ```ruby
 処理 if 条件
 ```

 と書ける．

* 条件1と条件2でそれぞれ処理をする場合

```ruby
if 条件1
  処理1
elsif 条件2
  処理2
end
```

**注意**

上のif文と下のif文では意味が違う．  
上のif文では条件1に当てはまった場合，
処理1が行われその後の条件については処理を行わない．
条件1に当てはまらなかった場合，条件2に当てはまるか判定する．  
しかし，下のif文では条件1に当てはまりかつ条件2に当てはまる場合，
両方の処理が行われてしまう．

```ruby
if 条件1
  処理1
end
if 条件2
  処理2
end
```

* 条件1と条件2とその他で処理を分けたい場合

```ruby
if 条件1
  処理1
elsif 条件2
  処理2
else
  処理3
end
```

* 例

```ruby
if info[1] == TYPE_ENEMY
  info = putUp
elsif info[1] == TYPE_BLOCK
  info = walkRight
else
  info = walkUp
end
```

上の例は「もしキャラクタの上が敵ならブロックを上に置く，
もしブロックなら右に進む，その他なら上に進む」という命令である．  

##### 比較演算子
if文の条件式で比較などを行う記号のことを比較演算子と言う．

~~~
式1 == 式2 #式1と式2が等しい（等価）
式1 != 式2 #式1と式2が等しくない（否定）
式1 > 式2  #式2より式1が大きい
式1 >= 式2 #式1が式2以上
式1 < 式2  #式2より式1が小さい
式1 <= 式2 #式1が式2以下
条件式1 && 条件式2 #条件式1が成り立つ かつ 条件式2が成り立つ（論理積）
条件式1 || 条件式2 #条件式1または条件式2が成り立つ（論理和）
~~~

#### 6.1.2 case文
ある変数1つについて複数のifがあるならcase文のほうが簡単

```ruby
case info[1]
when TYPE_ENEMY
  info = putUp
when TYPE_BLOCK
  info = walkRight
else
  info = walkUp
end
```
これは上のif文の例と同じ命令になる．

### 6.2 繰り返し
#### 6.2.1 loop

```ruby
loop do
  # 処理
end
```

`loop do ... end`に囲われた部分の処理を繰り返す．  
この処理から抜け出したい場合は`break`をする．

####6.2.2 for

```ruby
for i in [1, 2, 3]
  # 処理
end
```

`for ... end`のなかの処理を3回繰り返す．  
forあとの`i`は1~3に代わり，処理内で使用することもできる．  

#### 6.2.3 each

```ruby
(1..3).each do |i|
  # 処理
end
```

上のforと同じように3回繰り返す．  
`1..3`は1~3の範囲を意味する．  
doのあとの`|i|`は1~3の値が順番に入る．


## 7. ステップアップ
### 7.1 真っ直ぐ進む
ひたすら真っすぐ進むプログラムを作ってみよう．

### 7.2 壁を避ける
真っ直ぐ進むだけでは壁にぶつかってしまう．  
進みたい方向に壁があった場合，進まず別な方向へ進むようにしよう．

### 7.3 アイテムを取る
隣にアイテムがあったら取るようにしよう．

### 7.4 敵にブロックを置く
隣に敵がいるときブロックを置こう

## 参考図書
* たのしいRuby 第4版: (著)高橋征義, 後藤裕蔵 (監修)まつもとゆきひろ (出版社)ソフトバンククリエイティブ

## 更新履歴
2014/05/13 周囲情報に説明追加, 参考図書追加: Nobuyuki SAKAI  
2014/05/12 if文について説明を追加: Nobuyuki SAKAI  
2014/03/11 "Rubyとは"を追加: Nobuyuki SAKAI  
2014/03/04 ロゴを追加: Nobuyuki SAKAI
