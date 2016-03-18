#复制文件内容

> 对于体积较大的二进制文件，比如音频、视频文件，动辄几个GB大小，如果使用这先读后存，很容易使内存“爆仓”。理想的方法应该是读一部分，写一部分，不管文件有大，只要时间允许，总会处理完成，这里就需要用到流的概念。

### Stream在nodejs中是EventEmitter的实现，并且有多种实现形式，例如：

* http responses request
* fs read write streams
* zlib streams
* tcp sockets
* child process stdout and stderr

    
    fs.createReadStream('/path/to/source').pipe(fs.createWriteStream('/path/to/dest'));

    // pipe自动调用了data,end等事件
    var fs = require('fs');
    var readStream = fs.createReadStream('/path/to/source');
    var writeStream = fs.createWriteStream('/path/to/dest');

    readStream.on('data', function(chunk) { // 当有数据流出时，写入数据
        if (writeStream.write(chunk) === false) { // 如果没有写完，暂停读取流
            readStream.pause();
        }
    });

    writeStream.on('drain', function() { // 写完后，继续读取
        readStream.resume();
    });
    
    readStream.on('end', function() { // 当没有数据时，关闭数据流
        writeStream.end();
    });
