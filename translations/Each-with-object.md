[Ruby Bits: Each with object](https://subvisual.co/blog/posts/74-ruby-bits-each-with-object)

`Enumerable`是Ruby世界的核心模块。如果熟悉了它，可以说距离深谙Ruby之道也就不远了。
在所有`enumerable`模块的酷炫方法中，我最喜欢的，甚至公开承认过它是我在整个Ruby语言中的最爱，那就是`each_with_object`。
使用方法如下：
```ruby
numbers = [1, 2, 3, 4]

numbers.each_with_object([]) do |number, acc|
  acc << number * 3
end

## => [3, 6, 9, 12]
```
它通过遍历数组的每个元素，并向其做相关操作。和`each`方法相似，不同之处在于`each_with_object`提供一个累加器，也就是上面例子中的`acc`
这个对象可以在遍历元素之间传递。这和 `inject`或`reduce`的用法类似，不同之处在于块尾不用再返回累加器，方法会自动将其返回。
使用这个方法需要注意的是，累加器不能用`Fixnum`或者`String`类型的值。
在实际使用场景中可以被用来创建一个对象的数组，如下例：
```ruby
  (1..10).each_with_object([]) do |num, questions|
    questions << Question(name: num.to_s)
  end
```
