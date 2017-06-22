# Ruby中的nil?，empty?与Rails中的blank?，present?

[A concise explanation of nil v. empty v. blank in Ruby on Rails](https://stackoverflow.com/questions/885414/a-concise-explanation-of-nil-v-empty-v-blank-in-ruby-on-rails)
`.nil?`可以用在一切对象上，当对象为nil时，返回true
`.empty?`可以用于字符串，数组或者哈希，当满足下列条件时，返回true

- String length == 0
- Array length == 0
- Hash length == 0

在nil对象上调用 `.empty?`会抛出`NoMethodError `异常。
`.blank?`就是为此而生的。这个方法是[Rails](https://apidock.com/rails/Object/blank%3f)实现的，类似于`.empty?`可以作用于字符串，数组或者哈希，`.blank?`可以作用于认为对象。
```ruby
nil.blank? == true
false.blank? == true
[].blank? == true
{}.blank? == true
"".blank? == true
5.blank? == false
0.blank? == false
```
对于只包含空格的字符串，`.blank?`的返回同样为true
```ruby
"  ".blank? == true
"  ".empty? == false
```
Rails同样提供了[`.present?`](http://apidock.com/rails/Object/presence) 方法，其返回值与`.blank?`相反。

注意，就算数组中的元素都为blank时，对数组调用`.blank?`仍然会返回false。对于这种情况时，使用`.all?`配合`.blank?`，如下例：
```ruby
[ nil, '' ].blank? == false
[ nil, '' ].all? &:blank? == true 
```
