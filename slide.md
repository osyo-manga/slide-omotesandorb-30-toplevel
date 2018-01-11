## 表参道.rb #30
- - -
### Ruby のトップレベルについておさらい

---

## 自己紹介
- - -

* なまえ  : おしょー
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* 言語    : C++ / Ruby
* エディタ: Vim
* Ruby 2.5 で Method#=== が取り込まれた！！！
* Rails はできません
 
---

## [自作 gem](https://rubygems.org/profiles/osyo-manga)
- - -

* [iolite](https://github.com/osyo-manga/gem-iolite)
  * メソッドを遅延呼び出しするための gem
* [use_arguments](https://github.com/osyo-manga/gem-use_arguments)
  * ブロック引数を省略出来る gem
* [unmixer](https://github.com/osyo-manga/gem-unmixer)
  * mixin したモジュールを削除する gem
* [proc-unbind](https://github.com/osyo-manga/gem-proc-unbind)
  * `Proc` から `UnboundMethod` を定義する

---

#### アドベントカレンダー
- - -

* [Ruby Advent Calendar 2017](https://qiita.com/advent-calendar/2017/ruby)
  * [Ruby で型チェックを実装してみよう](http://secret-garden.hatenablog.com/entry/2017/12/01/000154)
* [一人 Ruby Advent Calendar 2017](https://qiita.com/advent-calendar/2017/ruby_pink_bangbi)
  * Suby に関する小ネタみたいなのを書いてます
* [一人 vimrc Advent Calendar 2017](https://qiita.com/advent-calendar/2017/vimrc_pink_bangbi)
  * vimrc に関する小ネタみたいなのを書いてます

---

## 今日話すこと
- - -
## Ruby のトップレベルについて
## おさらい


---

#### トップレベルとは
- - -

そのファイルの一番上のスコープのこと。

```ruby
# ここのこと

def hoge
	# ここではない
end


# ここのこと


class X
	# ここではない
end


# ここのこと
```

---

#### トップレベルの self
- - -

* トップレベルの self は main という特別なオブジェクト     <!-- .element: class="fragment" -->
* main は Object クラスのインスタンス     <!-- .element: class="fragment" -->
  * main = Object.new している感じ
* main という名前では参照できない     <!-- .element: class="fragment" -->


>>>

```ruby
# self は main という名前の特別なオブジェクト
p self
# => main

# main は Object クラスのインスタンスオブジェクト
p self.class
# => Object

# Error: undefined local variable or method `main' for main:Object (NameError)
# main という名前では参照できない
main
```


---

#### トップレベルで定義したメソッド
- - -

* Object クラスのインスタンスメソッドとして定義される     <!-- .element: class="fragment" -->
* 定義されるメソッドは private メソッド     <!-- .element: class="fragment" -->
* Object クラスのインスタンスメソッドなのでどこからでも参照出来る     <!-- .element: class="fragment" -->
* main で定義されると勘違いしてる人もいるが、main とは関係ない     <!-- .element: class="fragment" -->
  * Object クラスのインスタンスメソッドとして定義されるので結果として main からも参照できる     <!-- .element: class="fragment" -->

>>>

#### トップレベルでメソッドを定義
- - -

Object クラスの private インスタンスメソッドとして定義される

```ruby
def hoge
end

p method(:hoge).owner
# => Object

p method(:hoge).receiver
# => main

# hoge は private メソッドなのでレシーバを付けて呼び出しは出来ない
# Error: private method `hoge' called for main:Object (NoMethodError)
p self.hoge
```

>>>

#### トップレベルで定義したメソッドを参照

```ruby
def hoge
	self
end

# hoge のレシーバは main なので hoge 内の self は main を返す
p hoge
# => self

class X
	# hoge のレシーバは X なので hoge 内の self は X クラスオブジェクトを返す
	p hoge
	# => X

	def foo
		# hoge のレシーバは self なので hoge 内の self は
		# X のインスタンスオブジェクトを返す
		hoge
	end
end

p X.new.foo
# => #<X:0x000055a458842db0>
```

>>>

#### 小ネタ
- - -

Ruby 2.5 と 2.4 ではトップレベルの method の出力結果が異なる。
（結果が異なるだけで挙動は同じ？

```ruby
def hoge; end

# 2.4
p method :hoge
# => #<Method: Object#hoge>

# 2.5
p method :hoge
# => #<Method: main.hoge>
```

---

#### トップレベルで定義した定数
- - -

* ここでいう定数はクラスやモジュールも含まれる     <!-- .element: class="fragment" -->
* Object クラスの定数として定義される     <!-- .element: class="fragment" -->
* Ruby 2.5 からこの定数を探査する仕様が変わった     <!-- .element: class="fragment" -->
  * private 定数ではないことに注意

>>>

```ruby
Hoge = 42

p Object.constants(false).include? :Hoge

class Foo
end

p Object.constants(false).include? :Foo

class X
	p Hoge
	# => 42
	
	p Foo
	# => Foo
end

# Ruby 2.5 からこういうアクセスはできなくなった
# Error: uninitialized constant X::Hoge (NameError)
p X::Hoge
```

>>>

#### 明示的に Object クラスに定数を定義した時の注意
- - -

```ruby
class X; end

class Object
	Hoge = 42
end

# Error: uninitialized constant X::Hoge (NameError)
# 明示的に定義した定数も `::` で参照できないので注意
X::Hoge

# Kernel や BasicObject で定義した定数なら参照できる
module Kernel
	Foo = 1
end
class BasicObject
	Bar = 2
end

p X::Foo # => 1
p X::Bar # => 2
```


---

#### トップレベルで定義したローカル変数
- - -

* そのファイルのトップレベルでのみ参照出来るローカル変数     <!-- .element: class="fragment" -->
* ローカル変数なのでトップレベルのメソッドやクラスからは呼べない     <!-- .element: class="fragment" -->
* メソッドから参照したい場合は define_method 等を使用してブロックからキャプチャする     <!-- .element: class="fragment" -->

>>>

```ruby
hoge = 42

# メソッド内からはトップレベルのローカル変数は参照できない
def foo
	# Error: undefined local variable or method `hoge' for main:Object (NameError)
	hoge
end

foo


# define_method などでブロックを使ってキャプチャする
define_method(:foo){
	hoge
}

p foo
# => 42
```

---

#### トップレベルで定義したインスタンス変数
- - -

* main オブジェクトのインスタンス変数として定義される

>>>

```ruby
@hoge = 42

# main に定義される
p self.instance_variables # => [:@hoge]

def hoge
	@hoge
end

# hoge のレシーバは main なので main のインスタンス変数 @hoge が参照される
p hoge # => 42

class X
	def foo
		# hoge のレシーバは X のインスタンスオブジェクトなので
		# hoge 内の @hoge は X のインスタンス変数を参照する
		hoge # => nil

		@hoge = -4
		hoge # => -4
	end
end

p X.new.foo
```

---

## まとめ
- - -

* トップレベルの self は main という特別なオブジェクト     <!-- .element: class="fragment" -->
* トップレベルで定義したメソッドは Object の private インスタンスメソッドとして定義される     <!-- .element: class="fragment" -->
* トップレベルの定数も Object の定数として定義されるが探査方法が特殊なので注意     <!-- .element: class="fragment" -->
* トップレベルで定義したメソッドや定数はどこからでも参照されるので定義する場合は影響が大きいことに注意する    <!-- .element: class="fragment" -->

>>>

```ruby
# トップレベルで定義したメソッドや定数は
def hoge
	
end
Hoge = 42

# 以下のように Object で定義されるのと等価
class Object
	private def hoge
		
	end

	Hoge = 42
end
```

---

#### おまけ：安全にトップレベルでメソッドを定義する
- - -

```ruby
# A.rb

# refinements を使い、その中で Object に対してメソッドをテイギスル
using Module.new {
	refine Object do
		def foo
			:foo
		end
	end
}

p foo # => :foo
```

```ruby
# B.rb
require_relative "./A.rb"

# Error: undefined local variable or method `foo' for main:Object (NameError)
p foo
```

---

## ご清聴
## ありがとうございました
