---
title: vim编辑Dockerfile时语法高亮
categories: [运维,容器]
tags: [docker,centos7] 
date: 2019-11-04 15:16:12
---

> 参考[Dockerfile构建容器---语法高亮](https://www.cnblogs.com/chenglee/p/10315498.html)

三个文件扔进相关的目录即可


#### 1. `/usr/share/vim/vimfiles/doc/dockerfile.txt`

    *dockerfile.txt*  Syntax highlighting for Dockerfiles
    
    Author: Honza Pokorny <https://honza.ca>
    License: BSD
    
    INSTALLATION                                                     *installation*
    
    Drop it on your Pathogen path and you're all set.
    
    FEATURES                                                             *features*
    
    The syntax highlighting includes:
    
    * The directives (e.g. FROM)
    * Strings
    * Comments
    
     vim:tw=78:et:ft=help:norl:

#### 2. `/usr/share/vim/vimfiles/ftdetect/dockerfile.vim`

    au BufNewFile,BufRead [Dd]ockerfile,Dockerfile.*,*.Dockerfile set filetype=dockerfile


#### 3. `/usr/share/vim/vimfiles/syntax/dockerfile.vim`

    " dockerfile.vim - Syntax highlighting for Dockerfiles
    " Maintainer:   Honza Pokorny <https://honza.ca>
    " Version:      0.5
    
    
    if exists("b:current_syntax")
        finish
    endif
    
    let b:current_syntax = "dockerfile"
    
    syntax case ignore
    
    syntax match dockerfileKeyword /\v^\s*(ONBUILD\s+)?(ADD|ARG|CMD|COPY|ENTRYPOINT|ENV|EXPOSE|FROM|HEALTHCHECK|LABEL|MAINTAINER|RUN|SHELL|STOPSIGNAL|USER|VOLUME|WORKDIR)\s/
    highlight link dockerfileKeyword Keyword
    
    syntax region dockerfileString start=/\v"/ skip=/\v\\./ end=/\v"/
    highlight link dockerfileString String
    
    syntax match dockerfileComment "\v^\s*#.*$"
    highlight link dockerfileComment Comment
    
    set commentstring=#\ %s
    
    " match "RUN", "CMD", and "ENTRYPOINT" lines, and parse them as shell
    let s:current_syntax = b:current_syntax
    unlet b:current_syntax
    syntax include @SH syntax/sh.vim
    let b:current_syntax = s:current_syntax
    syntax region shLine matchgroup=dockerfileKeyword start=/\v^\s*(RUN|CMD|ENTRYPOINT)\s/ end=/\v$/ contains=@SH
    " since @SH will handle "\" as part of the same line automatically, this "just works" for line continuation too, but with the caveat that it will highlight "RUN echo '" followed by a newline as if it were a block because the "'" is shell line continuation...  not sure how to fix that just yet (TODO)

#### 效果展示

![QQ截图20191104152050.png](https://i.loli.net/2019/11/04/x982NLrgtIqFZwy.png)

#### 附录

[上面三个文件的git地址](https://github.com/qnyt1993/Docker)