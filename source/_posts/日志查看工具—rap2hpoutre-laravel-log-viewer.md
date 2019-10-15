---
title: 日志查看工具—— rap2hpoutre/laravel-log-viewer
categories: [后端,php,Laravel]
tags: [php,laravel] 
date: 2019-10-15 14:43:59
---

## 日志查看工具—— rap2hpoutre/laravel-log-viewer

日常的开发工作中，查看日志应该是我们最常干的事情了，日志对于我们调试程序，修复 Bug，优化代码来说必不可少也非常关键。使用 Linux 的 `tail -f` 命令持续监控日志文件是我们比较常用的方法：`tail -f storage/logs/laravel.log`。

但是有时候我们需要去测试环境或者正式环境检查错误日志，有些非开发人员在进行测试的时候也想查看日志，最好能有一个 Web 的方式显示日志内容，方便所有人自行查看日志内容。[rap2hpoutre/laravel-log-viewer](https://github.com/rap2hpoutre/laravel-log-viewer)


## 安装


    composer require rap2hpoutre/laravel-log-viewer


## 使用

`rap2hpoutre/laravel-log-viewer` 使用起来非常简单，只需要定义一个路由，指向扩展包提供的控制器方法即可。

*routes/web.php*

    .
    .
    .
    Route::get('logs', '\Rap2hpoutre\LaravelLogViewer\LogViewerController@index');


添加路由 `logs`，访问 http://larabbs.test/logs 。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/cJm5WmJc56.png)

页面结构很简单，左侧是日志文件列表，显示所有 `storage/logs` 目录下的日志文件，可以点击切换，目前我们只有一个文件 `laravel.log`。顶部可以进行分页调整，搜索。右侧是格式化显示的文件内容，显示更加的友好，可以按时间日志 Level 排序。

## 优化

使用起来非常方便，`rap2hpoutre/laravel-log-viewer` 已经提供了控制器以及页面，我们只需要添加一个路由即可。但是现在整个页面是由扩展包提供的，我们如何自定义该页面，将页面嵌套在我们提供的 `layout` 中呢？

回忆一下 [重写扩展包视图](https://learnku.com/docs/laravel/5.6/packages/1394#096449) 提示 Laravel 已经为我们提供了方法。我们可以通过 `vendor:publish` 命令将扩展包中的视图文件发布出来，`--tag="views"` 可以指定只发布扩展包的视图。


    $ php artisan vendor:publish --provider="Rap2hpoutre\LaravelLogViewer\LaravelLogViewerServiceProvider" --tag="views"


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/HESC1uSPUA.png)


Laravel 会先加载 `resources/views/vendor` 对应扩展包的视图文件，如果没有视图才去扩展包中寻找对应的视图文件。所以可以定义 `resources/views/vendor/laravel-log-view/log.blade.php` 重写这个视图页面，增加一些 `header` 和 `footer` 信息。

适当的修改发布出来的视图文件：
*resources/views/vendor/laravel-log-view/log.blade.php*

    @extends('layouts.app')
    
    @section('title', '日志')
    
    @section('styles')
    <link rel="stylesheet" type="text/css" href="https://cdn.datatables.net/1.10.16/css/dataTables.bootstrap4.min.css">
    <style>
    
      h1 {
        font-size: 1.5em;
        margin-top: 0;
      }
    
      #table-log {
          font-size: 0.85rem;
      }
    
      .sidebar {
          font-size: 0.85rem;
          line-height: 1;
      }
    
      .btn {
          font-size: 0.7rem;
      }
    
    
      .stack {
        font-size: 0.85em;
      }
    
      .date {
        min-width: 75px;
      }
    
      .text {
        word-break: break-all;
      }
    
      a.llv-active {
        z-index: 2;
        background-color: #f5f5f5;
        border-color: #777;
      }
    
      .list-group-item {
        word-wrap: break-word;
      }
    </style>
    @endsection
    
    @section('content')
    <div class="container-fluid">
      <div class="row">
        <div class="col-sm-2 sidebar mb-3">
          <h1><i class="fa fa-calendar" aria-hidden="true"></i> Laravel Log Viewer</h1>
          <p class="text-muted"><i>by Rap2h</i></p>
          <div class="list-group">
            @foreach($files as $file)
              <a href="?l={{ \Illuminate\Support\Facades\Crypt::encrypt($file) }}"
                 class="list-group-item @if ($current_file == $file) llv-active @endif">
                {{$file}}
              </a>
            @endforeach
          </div>
        </div>
        <div class="col-sm-10 table-container">
          @if ($logs === null)
            <div>
              Log file >50M, please download it.
            </div>
          @else
            <table id="table-log" class="table table-striped" data-ordering-index="{{ $standardFormat ? 2 : 0 }}">
              <thead>
              <tr>
                @if ($standardFormat)
                  <th>Level</th>
                  <th>Context</th>
                  <th>Date</th>
                @else
                  <th>Line number</th>
                @endif
                <th>Content</th>
              </tr>
              </thead>
              <tbody>
    
              @foreach($logs as $key => $log)
                <tr data-display="stack{{{$key}}}">
                  @if ($standardFormat)
                    <td class="text-{{{$log['level_class']}}}"><span class="fa fa-{{{$log['level_img']}}}"
                                                                    aria-hidden="true"></span>  {{$log['level']}}</td>
                    <td class="text">{{$log['context']}}</td>
                  @endif
                  <td class="date">{{{$log['date']}}}</td>
                  <td class="text">
                    @if ($log['stack']) <button type="button" class="float-right expand btn btn-outline-dark btn-sm mb-2 ml-2"
                                           data-display="stack{{{$key}}}"><span
                          class="fa fa-search"></span></button>@endif
                    {{{$log['text']}}}
                    @if (isset($log['in_file'])) <br/>{{{$log['in_file']}}}@endif
                    @if ($log['stack'])
                      <div class="stack" id="stack{{{$key}}}"
                           style="display: none; white-space: pre-wrap;">{{{ trim($log['stack']) }}}
                      </div>@endif
                  </td>
                </tr>
              @endforeach
    
              </tbody>
            </table>
          @endif
          <div class="p-3">
            @if($current_file)
              <a href="?dl={{ \Illuminate\Support\Facades\Crypt::encrypt($current_file) }}"><span class="fa fa-download"></span>
                Download file</a>
              -
              <a id="delete-log" href="?del={{ \Illuminate\Support\Facades\Crypt::encrypt($current_file) }}"><span
                    class="fa fa-trash"></span> Delete file</a>
              @if(count($files) > 1)
                -
                <a id="delete-all-log" href="?delall=true"><span class="fa fa-trash"></span> Delete all files</a>
              @endif
            @endif
          </div>
        </div>
      </div>
    </div>
    @endsection
    
    @section('scripts')
    <!-- jQuery for Bootstrap -->
    <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
    <!-- FontAwesome -->
    <script defer src="https://use.fontawesome.com/releases/v5.0.6/js/all.js"></script>
    <!-- Datatables -->
    <script type="text/javascript" src="https://cdn.datatables.net/1.10.16/js/jquery.dataTables.min.js"></script>
    <script type="text/javascript" src="https://cdn.datatables.net/1.10.16/js/dataTables.bootstrap4.min.js"></script>
    <script>
      $(document).ready(function () {
        $('.table-container tr').on('click', function () {
          $('#' + $(this).data('display')).toggle();
        });
        $('#table-log').DataTable({
          "order": [$('#table-log').data('orderingIndex'), 'desc'],
          "stateSave": true,
          "stateSaveCallback": function (settings, data) {
            window.localStorage.setItem("datatable", JSON.stringify(data));
          },
          "stateLoadCallback": function (settings) {
            var data = JSON.parse(window.localStorage.getItem("datatable"));
            if (data) data.start = 0;
            return data;
          }
        });
        $('#delete-log, #delete-all-log').click(function () {
          return confirm('Are you sure?');
        });
      });
    </script>
    @endsection


再次访问 http://larabbs.test/logs ：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/e6FJFWA34y.png)


由于我们继承了 `layouts.app` 页面，所以会自动为页面添加 `header` 和 `footer` 信息，有了这些信息我们就可以在合适的点增加链接，方便其他人员跳转过来查看日志了。

当然这里只是个例子，告诉大家如何重载某个视图页面，我们当然应该将这个页面整合进后台的某个页面。


