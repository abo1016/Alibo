---
title: composer.lock文件的作用
tags: []
id: '409'
categories:
  - - all-blog
    - PHP
date: 2019-03-12 22:27:47
---

**使用composer管理器时除了composer.json文件外 还有一个composer.lock文件也非常重要** composer install 命令从当前目录读取 composer.json 文件，处理依赖关系，并把依赖安装到 vendor 目录下。 如果当前目录下存在 composer.lock 文件，它会从此文件读取依赖版本，而不是根据 composer.json 文件去获取依赖。这确保了该库的每个使用者都能得到相同的依赖版本。 如果没有 composer.lock 文件，composer 将在处理完依赖关系后创建它。 为了获取依赖的最新版本，并且升级 composer.lock 文件，你应该使用 composer update 命令。 在团队协同开发时，我们应该将composer.lock文件加入版本控制，这样可以保证所有团队人员php组件版本保持一致，避免在开发中遇到不必要的麻烦。