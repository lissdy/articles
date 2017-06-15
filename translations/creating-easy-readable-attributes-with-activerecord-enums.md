 [Creating Easy, Readable Attributes With ActiveRecord Enums](http://www.justinweiss.com/articles/creating-easy-readable-attributes-with-activerecord-enums/)

设想一个问题的状态可能为“暂停”，“通过”或“标注”。或者一个电话号码可能是“家庭号码”，“办公号码”，“手机号码”或者“传真号码”（1982年的话）
有些模块需要这种类型的数据：只对应少许值的属性，并且这些值几乎永远不会改变。
如果使用纯Ruby的话，可以通过使用`symbol`来解决这个问题。
可以创建`PhoneNumberType`或者`QuestionStatus`模块，并通过定义`belongs_to`关系来关联这些值，但是这么简单的需求似乎并不值得这么做，因为仅仅将这些值放在`yaml`文件中一样可以满足需求。
现在我们来看一个解决这类需求的真正利器：在Rails4.1中引入的 [ActiveRecord enums](http://api.rubyonrails.org/v4.1.0/classes/ActiveRecord/Enum.html)。

## Model中的少量值
ActiveRecord enums用法很简单，你可以为model创建一个整数类型的列：
```ruby
bin/rails g model phone number:string phone_number_type:integer
```
列出这个属性可能的值
```ruby
class Phone < ActiveRecord::Base
  enum phone_number_type: [:home, :office, :mobile, :fax]
end
 ```
现在你可以直接操作字符串而不是数字了。
从前：
```ruby
irb(main):001:0> Phone.first.phone_number_type
=> 3
```
采用ActiveRecord enums后：
```ruby
irb(main):002:0> Phone.first.phone_number_type
=> "fax"
```
通过字符串或者数字都可以改变属性的值
```ruby
irb(main):003:0> phone.phone_number_type = 1; phone.phone_number_type
=> "office"
irb(main):004:0> phone.phone_number_type = "mobile"; phone.phone_number_type
=> "mobile"
```
甚至是通过感叹方法
```ruby
irb(main):005:0> phone.office!
=> true
irb(main):006:0> phone.phone_number_type
=> "office"
```
查看属性是否支持某些值：
```ruby
irb(main):007:0> phone.office?
=> true
```

查询满足属性值的所有对象：
```ruby
irb(main):008:0> Phone.office
  Phone Load (0.3ms)  SELECT "phones".* FROM "phones" WHERE "phones"."phone_number_type" = ?  [["phone_number_type", 1]]
```
查看属性所有可用值：
```ruby
irb(main):009:0> Phone.phone_number_types
=> {"home"=>0, "office"=>1, "mobile"=>2, "fax"=>3}
```
在HTML表单中也可以方便使用：
```ruby
<div class="field">
  <%= f.label :phone_number_type %><br>
  <%= f.select :phone_number_type, Phone.phone_number_types.keys %>
</div>
```
![表单效果](http://upload-images.jianshu.io/upload_images/4269060-c3ff60b24fd90b6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 需要注意的是
`Enums`并不是没有缺陷，如果不想在日后陷入麻烦的话，这里有几个问题需要注意。
**定义enum时，注意顺序**。假设你突然决定使用字母顺序来排列enum值：
```ruby
class Phone < ActiveRecord::Base
   enum phone_number_type: [:fax, :home, :mobile, :office]
end
```
那么你电话号码的类型匹配就有问题咯。可以通过设定序号值来解决这个问题：
```ruby
class Phone < ActiveRecord::Base
   enum phone_number_type: {fax: 3, home: 0, mobile: 2, office: 1}
end
```
但是讲真，最好的选择是不要改变值的顺序。
另一个更大的问题存在于Rails之外。尽管Rails将这些enum值当作字符串处理，而在数据库中它们只是一些数字。其他人看原始数据时并不知道这些数字所代表的含义，这就意味着读取这个数据库在所有应用都要有一份enum值的映射。
碰到这样需求的时候可以选择将enum映射存入数据库或yaml文件中。但这不符合`DRY`原则，因为现在你在两个地方定义了enum。而且随着应用规模的扩展，可能我们一开始想要避免的做法反而更好：创建另一个模型和关联，即`Phone` `belong_to` `PhoneNumberType`
但是如果想保持简单，`enums`依然是不错的选择。
