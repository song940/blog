---
layout: post
title: "Install Ruby2.0 on Mac OS X without RVM"
date: 2014-01-13 14:41
comments: true
categories: blog
---
## Prepare

Download and upgrade `openssl`

from [openssl][1] download [latest version][2] package .

<!--more-->

    wget http://www.openssl.org/source/openssl-1.0.1e.tar.gz
    

untar package

    tar xzvf openssl-*.tar.gz -C openssl-source-code
    cd openssl-source-code
    
    ./configure darwin64-x86_64-cc
    make
    make test
    sudo make install
    

then

    cd /usr/bin
    
    #backup openssl to openssl.old
    mv openssl{,.old}
    

Create a link to the new openssl file

    sudo ln -s /usr/local/ssl/bin/openssl openssl
    

test

    openssl version
    

## Install Ruby

Download `Ruby` [Source Code][3] from [ruby website][4]

    wget ftp://ftp.ruby-lang.org/pub/ruby/2.0/ruby-2.0.0-p247.tar.gz
    

configure

    tar xvf ruby-2.0.0-p*.tar.gz -C ruby-source-code
    cd ruby-source-code
    
    #The configure script will say that --with-openssl-dir is an invalid option. It's lying.
    #bug see http://apple.stackexchange.com/a/83888
    
    ./configure \
        --prefix=/System/Library/Frameworks/Ruby.framework/Versions/2.0 \
        --with-openssl-dir=/usr/local/ssl   
    
    make 
    sudo make install
    

link to system

    cd /System/Library/Frameworks/Ruby.framework/Versions/2.0/
    
    mkdir usr
    
    cd usr
    
    #link ./bin to ./usr/bin
    sudo ln -s ../bin bin
    
    #fix system link
    

switch to `Ruby` 2.0.0 now .

    cd /System/Library/Frameworks/Ruby.framework/Versions
    
    #remove ruby 1.8 link
    sudo rm -f  ./Current
    
    #point to 2.0
    sudo ln -s 2.0 Current
    

fix `gem` point to 1.8 bug

    cd /usr/bin
    
    rm -f ./gem 
    
    #point to `Current` link
    sudo ln -s /System/Library/Frameworks/Ruby.framework/Versions/Current/usr/bin/gem gem
    

test

    $ ruby -v
    ruby 2.0.0p247 (2013-06-27 revision 41674) [x86_64-darwin12.4.0]
    

All done .

* * *

Note:

添加到环境变量 , 修复 gem install `gem_name` 全局命令找不到 `gem_name` 的问题

    sudo vim /etc/paths.d/ruby 
    
    /System/Library/Frameworks/Ruby.framework/Versions/Current/usr/bin
    

update:

<http://stackoverflow.com/questions/17382714/installing-rails-on-mac-os-10-9-mavericks-beta>

 [1]: http://openssl.org
 [2]: http://www.openssl.org/source/openssl-1.0.1e.tar.gz
 [3]: ftp://ftp.ruby-lang.org/pub/ruby/2.0/ruby-2.0.0-p247.tar.gz
 [4]: http://www.ruby-lang.org
