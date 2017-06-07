---
layout: post
title:  "[译]深入研究GCD在Swift中的用法"
date:   2017-06-06 17:46:00 +0800
categories: Swift
---


原作者: John Sundell  
原文地址: [https://www.swiftbysundell.com/posts/a-deep-dive-into-grand-central-dispatch-in-swift](https://www.swiftbysundell.com/posts/a-deep-dive-into-grand-central-dispatch-in-swift)    

[Grand Central Dispatch](https://developer.apple.com/reference/dispatch)(或者简称GCD)是Swift开发者经常使用的基本技术之一.主要因为它能够在不同的并发队列上分发任务,并且大多数人可能已经用它来编写如下代码:

    DispatchQueue.main.async {
        // Run async code on the main queue
    }

但是如果你再深入研究一下,会发现GCD还有一套你并不知道并且非常强大的API和功能.这周,让我们越过`async {}`看一下GCD在哪些情况下非常实用,以及如何为许多其他Foundation API提供更简单的（更多的“Swifty”）选项。

### 使用DispatchWorkItem延迟可取消的任务  

一个对GCD共同的误解是"一旦你安排了一个任务就无法取消,需要操作Operation的API来取消".虽然这以前是对的,但在iOS 8和macOS 10.10中DispatchWorkItem被引入,它提供了精准的功能,使用起来也非常简单.  

假设我们的界面有个搜索框,当用户输入一个字符,我们通过调用后台接口来进行搜索.但是用户可以输入的很快,我们又不想立马开始请求网络(这样会浪费大量的数据和服务器容量),反而我们想要"防反跳"这些事件,用户如果0.25秒还没有输入就执行一次请求.  

这就是`DispatchWorkItem`使用的场景.通过封装我们的请求代码在work item中,每当一个新的请求进入我们可以非常容易的取消它,就像这样:

    class SearchViewController: UIViewController, UISearchBarDelegate {
    // We keep track of the pending work item as a property
    private var pendingRequestWorkItem: DispatchWorkItem?

    func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {
        // Cancel the currently pending item
        pendingRequestWorkItem?.cancel()

        // Wrap our request in a work item
        let requestWorkItem = DispatchWorkItem { [weak self] in
            self?.resultsLoader.loadResults(forQuery: searchText)
        }

        // Save the new work item and execute it after 250 ms
        pendingRequestWorkItem = requestWorkItem
        DispatchQueue.main.asyncAfter(deadline: .now() + .milliseconds(250),
                                      execute: requestWorkItem)
      }
    }

如上所述,在Swift中使用`DispatchWorkItem`实际上要比`Timer`或者`Operation`更简单友好,这要归功于尾随闭包和GCD的在Swift中的导入程度.你不需要使用`@objc`这种标记语法,或者`#selector`,它们都可以使用闭包完成.

### 使用DispatchGroup进行分组或者链接任务  

有时你需要执行一个组任务才能继续把逻辑走通.假如你在创建模型之前需要先从一组数据源中加载数据.而不必一直注意数据源的动向,你可以轻松地使用`DispatchGroup`进行同步工作.  

使用`dispatch groups`还有一个很大的优势,就是你的任务可以并发运行在单独的队列中.这样你就可以开始轻松的添加并发任务如果你需要的话,而且无需重写任何任务.你所需要做的就是平衡调用`enter()`和`leave()`在`dispatch group`中来同步你的任务.

> 译者:enter()和leave()函数是成对调用.  

让我看一下例子,从本地数据库,iCloud,后台加载笔记数据,然后把获取的全部数据合并到一个`NoteCollection`中:  

    // First, we create a group to synchronize our tasks
    let group = DispatchGroup()
    
    // NoteCollection is a thread-safe collection class for storing notes
    let collection = NoteCollection()
    
    // The 'enter' method increments the group's task count…
    group.enter()
    localDataSource.load { notes in
        collection.add(notes)
        // …while the 'leave' methods decrements it
        group.leave()
    }
    
    group.enter()
    iCloudDataSource.load { notes in
        collection.add(notes)
        group.leave()
    }
    
    group.enter()
    backendDataSource.load { notes in
        collection.add(notes)
        group.leave()
    }
    
    // This closure will be called when the group's task count reaches 0
    group.notify(queue: .main) { [weak self] in
        self?.render(collection)
    }

上面的代码用起来没问题,但是重复率太高.我们给`Array`写个`extension`来重构下代码,当Element遵守数据源:   

    extension Array where Element: DataSource {
        func load(completionHandler: @escaping (NoteCollection) -> Void) {
            let group = DispatchGroup()
            let collection = NoteCollection()
    
            // De-duplicate the synchronization code by using a loop
            for dataSource in self {
                group.enter()
                dataSource.load { notes in
                    collection.add(notes)
                    group.leave()
                }
            }
    
            group.notify(queue: .main) {
                completionHandler(collection)
            }
        }
    }


通过上面的扩展,我们可以把代码精简成这样:  

    let dataSources = [localDataSource, iCloudDataSource, backendDataSource]
    
    dataSources.load { [weak self] collection in
        self?.render(collection)
    }
  
### 使用DispatchSemaphore等待异步任务   

`DispatchGroup`既提供了一个非常简单友好的方法来同步一组异步操作,同时又能**持续保持异步**,而`DispatchSemaphore`提供了一种**同步等待一组异步任务**的方法.这对于一个没有应用运行循环的命令行工具或者脚本非常有用,它只是在全局上下文中同步地执行,一直等到任务完成.

像`DispatchGroup`一样,信号量的API是非常简单的,你只需要调用`wait()`或者`signal()`来增加或者减少内部计数器.在调用`signal()`之前调用`wait()`将会阻塞当前线程,直到接收到信号.  

让我们在之前的`Array`上扩展一个`overload`?,它会同步地返回一个`NoteCollection`,否则会抛出一个错误.我们可以重用之前基于`DispatchGroup`的代码,但是只需要用信号量来简单地协调该任务.  

    extension Array where Element: DataSource {
        func load() throws -> NoteCollection {
            let semaphore = DispatchSemaphore(value: 0)
            var loadedCollection: NoteCollection?
    
            // We create a new queue to do our work on, since calling wait() on
            // the semaphore will cause it to block the current queue
            let loadingQueue = DispatchQueue.global()
    
            loadingQueue.async {
                // We extend 'load' to perform its work on a specific queue
                self.load(onQueue: loadingQueue) { collection in
                    loadedCollection = collection
    
                    // Once we're done, we signal the semaphore to unblock its queue
                    semaphore.signal()
                }
            }
    
            // Wait with a timeout of 5 seconds
            semaphore.wait(timeout: .now() + 5)
    
            guard let collection = loadedCollection else {
                throw NoteLoadingError.timedOut
            }
    
            return collection
        }
      }
  

在数组中使用上面的新方法,现在我们就可以通过脚本或者命令行工具同步地加载笔记数据,就像下面这样:  

    let dataSources = [localDataSource, iCloudDataSource, backendDataSource]
    
    do {
        let collection = try dataSources.load()
        output(collection)
    } catch {
        output(error)
    }

### 使用DispatchSource监控一个文件的变化   

我想提出的最后一个"不太了解"的GCD特性,是如何让你在文件系统中监控一个文件的改变.像`DispatchSemaphore`一样,这在脚本或者命令行工具中是非常有用的,如果你想自动对用户正在编辑的文件作出反应.这会使你轻松构建具有"live editing"特性的开发者工具.  

调度来源有一点不同,这取决于你正在观察的对象.这种情况下,我们将使用`DispatchSourceFileSystemObject`,它允许我们监控文件系统中的一些事件.  

你可以使用`fileDescriptor`和`DispatchQueue`来创建一个调度源来执行监控.下面有一个简单的`FileObserver`类实现例子,它允许你每次修改给定文件时运行闭包(可以使用[File](https://github.com/johnsundell/files)获取文件索引)  


    class FileObserver {
        private let file: File
        private let queue: DispatchQueue
        private var source: DispatchSourceFileSystemObject?
    
        init(file: File) {
            self.file = file
            self.queue = DispatchQueue(label: "com.myapp.fileObserving")
        }
    
        func start(closure: @escaping () -> Void) {
            // We can only convert an NSString into a file system representation
            let path = (file.path as NSString)
            let fileSystemRepresentation = path.fileSystemRepresentation
    
            // Obtain a descriptor from the file system
            let fileDescriptor = open(fileSystemRepresentation, O_EVTONLY)
    
            // Create our dispatch source
            let source = DispatchSource.makeFileSystemObjectSource(fileDescriptor: fileDescriptor,
                                                                   eventMask: .write,
                                                                   queue: queue)
    
            // Assign the closure to it, and resume it to start observing
            source.setEventHandler(handler: closure)
            source.resume()
            self.source = source
        }
    }


现在我们可以像这样使用`FileObserver`:  

    let observer = try FileObserver(file: file)
    
    observer.start {
        print("File was changed")
    }
  
### 结尾  

Grand Central Dispatch是一个非常强大的框架,它远比你看起来厉害多了.希望这篇文章起到一个抛砖引玉的作用.我建议你下次在需要完成文章中提到的任务之一时可以试一下这些方法.在我看来,很多基于`Timer`或者`OperationQueue`的代码,还有一些第三方的框架,都没有直接使用GCD来的简单些.   

你还知道其他关于GCD的非常实用的特性吗?请告诉我,如果有什么疑问,评论或者反馈你可以在下面评论,或者在Twitter上找到我[@johnsundell](https://twitter.com/johnsundell).

谢谢阅读! 🚀
