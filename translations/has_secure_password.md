原文：[has_secure_password with Rails 4.1](https://robert-reiz.com/2014/04/12/has_secure_password-with-rails-4-1/)

我刚刚用Rails 4.1创建了一个新项目，并且试用了`has_secure_password `，很酷的功能呢。
但愿你没有在数据库里直接存储明文密码！为了防治密码被窃取，数据库中存储的始终应该是某种形式的哈希值，而非明文密码。
有几个很棒的教程讲述如何以安全的方式哈希和存储密码。我自己用Ruby实现过几次。
更复杂一点的解决方案是 [devise](https://www.versioneye.com/ruby/devise/3.2.4)，一个关于安全和验证的强大gem。
但是，[Rails](https://www.versioneye.com/ruby/rails/4.1.0) 3.1 之后你也可以利用[has_secure_password](http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html#method-i-has_secure_password)来实现。这个Rails内部的机制充分考虑了密码验证和加密。它需要model中有一个`password_digest `字段，用来存储加密后的密码。让我们来生成一个简单的model。
```ruby
rails g model user username:string password_digest:string
```
这user model中加入这段代码
```ruby
has_secure_password
```
这会为model加上`password `和`password_confirmation `两个属性。这两个属性是model的一部分，但是并不存在于数据库中！因为我们并不需要存储明文密码。
来加一些测试：
```ruby
require 'spec_helper'
 
describe User do
 
  it "fails because no password" do
    User.new({:username => "hans"}).save.should be_false
  end
 
  it "fails because passwrod to short" do
    User.new({:username => "hans", :password => 'han'}).save.should be_false
  end
 
  it "succeeds because password is long enough" do
    User.new({:username => "hans", :password => 'hansohanso'}).save.should be_true
  end
 
end
```
3个简单的测试用例：

- 不提供密码，用户创建失败
- 密码过短，用户创建失败
- 密码合法，用户创建成功
用 [RSpec](https://www.versioneye.com/ruby/rspec/2.14.1) 跑这些用例时，第二个用例会失败。因为默认Rails不会加入密码长度的验证。现在我们在model中自己加上。

```ruby
class User < ActiveRecord::Base
 
  has_secure_password
  validates :password, :length => { :minimum => 5 }
 
end
```
再跑一次测试，这次用例都通过了。检查一下数据库会发现users表的password_digest列存储着经过加密的数值，这正是我们想要的！
现在来做验证的部分。假设一个新的注册用户现在想要登录。你可以这样进行验证：
```ruby
user = User.find_by_username("USERNAME").authenticate("CLEAR_TEXT_PASSWORD")

```
如果从HTTP请求中传来的用户名和明文密码都正确，将从数据库中返回一个有效用户。否则返回`nil`！
`has_secure_password `只在对象创建时对密码进行验证。之后的操作便不再验证（例如update）。这个可以接受，因为这意味着你可以从数据库加载用户，在不知道密码的情况下修改和保存用户。
`has_secure_password `还为model提供了`password_confirmation `属性。只有当该属性为非`nil`值时才触发验证。如果该属性非`nil`，则其值必须与`password `属性相等。让我们针对该属性再加两个测试用例。
```ruby
it "fails because password confirmation doesnt match" do
  User.new({:username => "hans",
    :password => 'hansohanso',
    :password_confirmation => 'aa'}).save.should be_false
end
 
it "succeeds because password & confirmation match" do
  User.new({:username => "hans",
    :password => 'hansohanso',
    :password_confirmation => 'hansohanso'}).save.should be_true
end
```
为了让测试通过，在model中再加入一行代码。
```ruby
class User < ActiveRecord::Base   
  has_secure_password   
  validates :password, :length => { :minimum => 5 }
  validates_confirmation_of :password
end
```
`validates_confirmation_of :password`将会检查密码的一致性。
Rails不强制使用`password_confirmation `，但是你需要的话可以像这么来用。
我确实很喜欢这个功能，因为它节省了很多代码和开发时间。而且对大部分应用来说够用了。

##译注
`has_secure_password `会自动加入以下验证规则：
- 在创建时密码必须存在
- 密码长度小于等于72字符
- 确认密码（使用`password_confirmation `属性）
在实际项目中，通过第三方应用登录的用户是不需要密码的，这时可以通过传递`validations: false`参数移除验证规则。
```ruby
has_secure_password :validations => false
```

##另
- 模块验证规则查看
```ruby
User.validators
 ```
- 模块中某属性验证规则查看
```ruby
User.validators_on(:password)
```
