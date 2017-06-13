[Word Count Exercise in Ruby](http://www.mccartie.com/2013/08/19/word-count-exercise.html)

题目
>单词计数
编写程序来计算给定短语中每个单词出现的次数。
例如输入"olly olly in come free"
计数结果应为：
olly: 2
in: 1
come: 1
free: 1

简单吧，让我们来动手实现：

    class Phrase
      attr_accessor :words
      def initialize(words)
        @words = words
      end

      def word_count
        word_list = {}
        words.split(" ").each do |word|
        word_list = process_word_in_list(word, word_list)
        end 
        word_list
      end

      private
      def process_word_in_list(word, word_list)
        return word_list if word.empty?
        word_list[word] ||= 0
        word_list[word] += 1
        word_list
      end
    end
最初的这一版实现很直接。创建一个hash，当单词出现时，对其相关hash键的值进行计数。够简单咯。
但是这里有三个问题：
- word_count方法不易读
- process_word_in_list方法的实现相当丑陋
- 还对特殊字符和标点符号进行了计数
（接下来这个版本看起来更丑，但这只是一切变好之前的过渡）


    class Phrase
      attr_accessor :words
      def initialize(words)
        @words = words
      end
  
      def word_count
        prep_words_for_counting()
        word_list = {}
        words.split(" ").each do |word|
          word_list = process_word_in_list(word, word_list)
        end
        word_list
      end
    
     private
     def process_word_in_list(word, word_list)
       word = parse_word(word)
       return word_list if word.empty?
       word_list[word] ||= 0
       word_list[word] += 1
       word_list
     end

     def prep_words_for_counting
       words.gsub!(",", " ")
     end

     def parse_word(word)
       word.gsub!(/[^a-zA-Z0-9]/, "")
       word.downcase!
       word
     end
    end

哦天，这个实现太恶心了。找大神帮我review之后作出了如下改进：
1. 创建有默认值的hash。` Hash.new(0)`创建了一个hash并且将0作为各个键的默认值。
这就可以砍掉`word_list[word] ||= 0`这一行了。
2. 我不需要在遍历中将每个单词使用downcase转为小写——直接对整个字符串转换更好。
3. 使用魔幻的ruby字符串方法：`scan`
> Scan遍历字符串来匹配模式（可能是正则或者字符串）。对于每一个匹配，生成的结果可能返回到数组或者被传递到块中。
如下示例：

```ruby
a = "cruel world"
a.scan(/\w+/)        #=> ["cruel", "world"]
a.scan(/.../)        #=> ["cru", "el ", "wor"]
```
 基于新的ruby知识，接下来是我下一版的实现
```ruby
class Phrase
  attr_accessor :words
  def initialize(words)
    @words = words
  end

  def word_count
    word_list = Hash.new(0)
    words.downcase.scan(/\w+/) do |word|
      word_list[word] += 1
    end
    word_list
  end
end
```
现在清晰多咯！使用`scan`可以省去大量之前进行检查和模式匹配的代码。
为了离完美更近一步，我们可以再作出以下优化：
- 分离`word_count`方法
这一点确实有些苛求，但是为了让`word_count`方法中的`words.downcase.scan(/\w+/)`可以更有表现力。让我们试着将细节提取到块中怎么样？
最终实现：

```ruby
class Phrase
  attr_accessor :words
  def initialize(words)
    @words = words
  end

  def word_count
    list = Hash.new(0)
    each_word { |word| list[word] += 1 }
    list
  end

private
  def each_word
    words.downcase.scan(/\w+/) { |word| yield word }
  end
end
```
新的`each_word`方法更具有表现力：它清楚的告知了输出内容，同时将实现的细节小心封装为私有方法。
就这样，这个练习一开始困扰了我。但是在实现它的过程中，我见识到了`scan`的威力，另外，这也是一个很好的将复杂的方法移到Ruby块的例子。

### 更新
Twitter上有些大神建议我通过使用`inject`或者`each_with_object `来简化hash的创建。
他们给了我一个简单的原则：“如果只需要累加值（例如创建hash），使用`each_with_object `，如果需要改变对象，使用`inject`”。基于此原则，进一步简化实现如下：
```ruby
class Phrase
  attr_accessor :words
  def initialize(words)
    @words = words
  end

  def word_count
    each_word.each_with_object(Hash.new(0)) { |word, hash| hash[word] += 1 }
  end

private
  def each_word
    words.downcase.scan(/\w+/)
  end
end
```
