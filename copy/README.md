#复制文件内容

> 对于体积较大的二进制文件，比如音频、视频文件，动辄几个GB大小，如果使用先读后存，很容易使内存“爆仓”。理想的方法应该是读一部分，写一部分，不管文件有大，只要时间允许，总会处理完成，这里就需要用到流的概念。

### Stream在nodejs中是EventEmitter的实现，并且有多种实现形式，例如：
* http responses request
* fs read write streams
* zlib streams
* tcp sockets
* child process stdout and stderr


### 实例：
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
    
### fs.createReadStream(path, [options])返回一个新的 ReadStream 对象,用于基于流的文件处理

    ptions 是一个包含下列缺省值的对象：
    { flags: 'r',       //文件打开的行为。
      encoding: null,   //encoding 可选为 'utf8', 'ascii' 或者 'base64'。
      fd: null,         //文件描述符。如果不为空，将会忽略path参数并从文件描述创建可读流。但种创建方法不会触发任何'open'事件。
      mode: 0666,        //设置文件模式(权限)，文件创建默认权限为 0666(可读，可写)。   
      autoClose: true,   //如果 autoClose 为 false 则即使在发生错误时也不会关闭文件描述符 (file descriptor)。                                                  //此时你需要负责关闭文件，避免文件描述符泄露 (leak)。 
                         //如果 autoClose 为 true （缺省值）， 当发生 error 或者 end 事件时，文件描述符会被自动释放
      start:0,      //start 和 end 值用于读取文件内的特定范围而非整个文件,单位为字节
      end:1000      //start 和 end 都是包含在范围内的（inclusive, 可理解为闭区间）并且以 0 开始
    }
