[rubocop](http://batsov.com/rubocop/)是Ruby的代码分析器，在工作中使用可以帮助开发人员编写出符合Ruby规范的代码。
此文就结合我的实际工作，记录rubocop常用的命令，以便备忘查阅。
### 安装
```ruby
gem install rubocop
```
或将其添加到Gemfile中，使用bundle安装
```ruby
group :development do
  ...
  gem 'rubocop', require: false
end
```
### 使用
安装完成后，在项目目录下执行`rubocop `即可以对整个项目进行检查。
或者通过指定文件路径，对单独的文件进行检查。

例如：
```ruby
rubocop app/controllers/api/v4/XXX_controller.rb
```
输出

![rubocop](http://upload-images.jianshu.io/upload_images/4269060-3c1b9086acad6c67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到rubocop提示使用`skip_before_action`代替`skip_before_filter`。
（`skip_before_filter`是弃用方法）



