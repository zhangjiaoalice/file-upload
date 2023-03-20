
# 一、客户端文件上传背景
**Web 应用程序的一个主要痛点就是无法操作用户计算机系统上的文件**
1. 可以通过 `<input type="file">`标签或者拖拽的方式来选择本地的文件
2. 随后HTML5在DOM对象中加入的`File API`和`Blob API`让用户可以读取计算机文件的内容
```!javascript
<input type="file" onchange="onFileChange(this.files)">
<script>
	function onFileChange(files) {
		const fileObj = files[0]
		console.log('file 对象:', fileObj)
	}
</script>
```
------------

# 二、Blob Api
### <一>、概念
`**Blob 构造函数生成的对象，表示一个不可变的、原始数据的类文件对象, 是js对不可修改二进制数据的封装类型**`
1. blob 数据可以按照字符串或文本格式进行读取
2. 还可以转换成`ReadableStream`来用于数据操作
3. Blob表示的不一定是js原生格式的数据

### <二>、 属性
1. `Blob.prototype.size`:  只读属性，blob 对象中所包含数据的大小
2. `Blob.prototype.type`:  只读户型, 表明改blob对象所包含的`MIME`(Multipurpose Internet Mail Extensions 多用途互联网邮件扩展类型),字符串类型

### <三>、实例方法
1. `Blob.prototype.arrayBuffer()`:  返回一个promise实例，会返回一个包含blob中所有内容的二进制格式`ArrayBuffer`
2. `Blob.prototype.slice(start, end, contentType)`: 返回一个新的blob对象, 包含了源blob对象中指定范围的数据
	* start: 代表Blob 里的下标，表示第一个会被拷贝进新的Blob的字节的起始位置，如果为负数，则这个偏移量将会从数据的末尾从后到前开始计算
	* end:  Blob的下标，这个下标对应的字节将会是被拷贝进Blob的最后一个字节
	* contentType: 给新的Blob赋予一个新的文档类型
4. `Blob.prototype.text()`:  返回一个promise,会返回一个包含blob中所有内容的UTF-8格式的字符串



# 一、大文件上传基本概念
1. 分片上传: 将大文件按照一定的大小分割（file.slice()）为多个数据块进行上传,在服务端对这些数据块进行整合
2. 断点续传： 在数据块上传的过程中，遇到网络故障，可以从未上传完成的部分开始上传，不必将所有数据再重新上传一次
3. 秒传：将需要上传的文件上传，服务端会对上传的文件进行MD5校验，如果服务器上有一样的东西，就会直接客户端返回新的地址

# 二、 上传流程

![截屏2023-03-17 14.44.09.png#607px #702px](/download/attachments/2470304806/%E6%88%AA%E5%B1%8F2023-03-17%2014.44.09.png?version=1&modificationDate=1679035453079&api=v2)

```!javascript
const obj = {name: 'dog_1'}
const blob = new Blob([JSON.stringify(obj, null, 2)], 'application/json')
const reader = new FileReader()
// 读取blob文件
reader.readerAsArrayBuffer(blob) // 读取文件数据
// reader.readerAsBinaryString(blob) // 读取文件原始二进制格式
// reader.readerAsDataURL(blob) // 以data:URL格式表示文件内容
// reader.readerAsText(blob) // 将blob或者File对象根据特殊的编码格式转化为字符串形式的内容
```

# 三、File API
File 接口提供有关文件的信息，并允许网页中的js访问文件内容
### <一>、File 对象来源
1. 用户在一个`<input>`元素上选择文件后返回的FileList对象
2. 来自于拖放操作生成的DataTransfer对象
3.来自HTMLCanvasElement上的mozGetAsFile API

### <二>、File对象的属性
1. `File.name`: 返回当前File对象所引用文件的名字
2. `File.size`: 文件大小
3. `File.webkitRelativePath`: 返回File相关的path 或者URL
4. `File.type`: 返回文件的MIME类型

### <三>、方法
没有任何方法，但是从Blob对象继承了`slice` 方法


# 四、Stream Api 流
### <一>、 概念
**1. Stream Api 解决了什么问题？**
	* 曾经，如果我们想要处理某种资源(视频、文本文件等)，我们必须要下载完整的文件，然后等待它反序列化成适当的格式，然后在完整地接受到所有的内容后再进行处理
	* 使用流，只要原始数据在客户端可用，就可以通过js按位处理数据，不再需要缓冲区、字符串或者blob
	* 流会将我们通过网络请求获取的资源分成一个个小的块，让后按位处理这些数据
	
**2. 主要的应用场景？**
	* `大块的数据可能不会一次性都使用`。网络请求响应是一个典型的例子。网络负载是以连续信息包的形式交付的，而流式处理可让数据一到达就能使用，而不必等待所有数据都加载完毕
	* `大块数据可能要分为小部分处理`。视频处理、数据压缩、图像编码、JSON解析都是可以分成小部分进行处理的，而不必等到素有数据都在内存中时再处理
	
**3. 理解流**
	* 流的基本单位是`块`。块可以是任意数据类型，通常是一个定型数组
	* 每个块都是一个离散的流片段，可以作为一个整体进行处理
	* 块不是以固定大小的流片段，也不会按照固定的间隔到达指定的端（理想流当中的块的大小近似相等，到达的间隔时间也近似相同）

4. **流平衡的三种情形**
	* 流出口处理数据的速度比流入口处理数据的快，流入口经常处于空闲状态,这样会浪费一点内存和计算资源，可接受
	* 流入和流出均衡，理想状态
	* 流入口数据处理速度比流出口数据快，流不平衡

**5. 解决流不平衡的问题**
`针对流不平衡的问题，所有的流都会为已入流但未离开流的块提供一个内部队列`
如果块入列速度大于块出列速度，内部队列就会不断的增大。流不能允许内部队列无限扩增大，会使用`反压`通知流入口停止发送数据，知道队列大小降到某个既定的阈值之下，这个值由排列策略决定，这个策略决定了内部队列可以占用的最大内存(`高水位线`)

### <二>、Stream API
**Stream API 定义了三种流：**
1. `可读流`：可以通过某个公共接口读取数据块的流。数据在内部从底层源进入流，然后由消费者（consumer）进行处理
	* `ReadableStream`: 表示数据的可读流。用于处理fetch API 返回的响应，或者开发者自定义的流
	* `ReadableStreamDefaultReader`: 表示默认reader,用于读取来自网络的数据流,读取器对象
	* `ReadableStreamDefaultController`: 表示一个controller， 用于控制ReadableStream 的状态及内部队列。默认的controller用于处理非字节流
```!javascript
// 配合Fetch api使用，处理从网络获取的资源
fetch("http://localhost:9999")
.then(response => response.body)
.then(rb => {
	// 创建一个读取器对象，并锁定流
	const reader = rb.getReader()
	// 读取并处理读取器对象中的流片段
	return new ReadableStream({
		start(controller){
			// controller 控制器对象，用于控制ReadableStream内部状态和队列
			// 读取读取器中锁定的流信息
			function push() {
				reader.read().then(({done, value}) => {
					if(done) {
						// 流处理完成
						console.log('process done:', done)
						// 关闭控制器
						controller.close()
						return
					}
					// 将流添加到内部队列中
					controller.enqueue(value)
					console.log('processing stream:', done, value)
					// 递归处理流
					push()
				})
			}
			push()
		}
	})
})
.then(stream => {
	console.log('获取处理好的流信息:', stream)
	return new Response(stream, {header: {"Content-Type": "application/json"}}).text()
})
.then(res => {
	console.log("获取转换后的结果:", res)
}) 
```

2. `可写流`：将流数据写入目的地(sink)提供的一个标准抽象，是一个可转移的对象。生产者(producer)将数据写入，数据在内部传入底层数据槽
	*  `WritableStream`: 提供将流写入目标整个过程的标准抽象表示(sink),内置被压和队列机制
	* `WritableStreamDefaultWriter`:  表示writer, 用于将数据写入可写流中
	* `WritableStreamDefaultController`: controller, 用于控制WritableStream的状态
```!javascript
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>可写流</title>
</head>

<body>
    <a href="#" onclick="handelWrite()">开始写入流</a>
    <div id="box"></div>
    <script>
        const box = document.querySelector('#box')
        function sendMessage(message, writableStream) {
            // 获取 WritableStreamDefaultWrite 实例，用于将数据写入可写流中
            const defaultWrite = writableStream.getWriter();
            const encoder = new TextEncoder();
            // 将message内容进行编码
            const encoded = encoder.encode(message, { stream: true });
            encoded.forEach((chunk) => {
                defaultWrite.ready
                    .then(() => {
                        // 写入流
                        if(chunk) {
                            console.log('开始写入流:', chunk);
                            return defaultWrite.write(chunk)
                        }
                    })
                    .catch((err) => {
                        throw (err)
                    })
            })
            defaultWrite.ready
                .then(() => {
                    defaultWrite.close()
                })
                .catch(err => {
                    console.log('Stream error:', err)
                })
        }


        const decoder = new TextDecoder("utf-8")
        const queuingStrategy = new CountQueuingStrategy({ highWaterMark: 1 })
        let result = ''
        const writableStream = new WritableStream({
            write(chunk) {
                return new Promise((resolve, reject) => {
                    let buffer = new ArrayBuffer(1)
                    let view = new Uint8Array(buffer)
                    view[0] = chunk
                    let decoded = decoder.decode(view, { stream: true })
                    const listItem = document.createElement('p')
                    listItem.textContent = "chunk decoded:" + decoded
                    box.appendChild(listItem)
                    result += decoded
                    resolve()
                })
            },
            close() {
                let listItem = document.createElement('p')
                listItem.textContent = "[MESSAGE RECIVED]" + result
                box.appendChild(listItem)
            },
            abort(error) {
                console.log("Sink error:", error);
            }
        }, queuingStrategy)


        function handelWrite() {
            sendMessage('Hello World', writableStream)
        }
    </script>
</body>

</html>
```
3. `转换流`： 表示链式传输管道，可写流用于接收数据(可写端)，可读流用于输出数据(可读端)，可读流和可写流之间的转换程序，可以根据需要检查和修改流内容， 可以用于解码/编码视频帧，解压数据或者将流从XML转换到JSON
	* `TransformStream`： 表示一组可转化的数据
	* `TransformStreamDefaultController`: 提供操作和转换流关联的ReadableStream 和 WritableStream 的方法
```!javascript
// 将任意对象转化为unit8数组
        const transformContent = {
            start() {}, // 必传项
            async transform(chunk, controller) {
                chunk = await chunk
                switch(typeof chunk) {
                    case 'object':
                        if(chunk === null) {
                            controller.terminate()
                        } else if (ArrayBuffer.isView(chunk)) {
                            controller.enqueue(new Uint8Array(chunk.buffer, chunk.byteOffset, chunk.byteLength))
                        } else if (Array.isArray(chunk) &amp;&amp; chunk.every(val => typeof val === 'number')) {
                            controller.enqueue(new Uint8Array(chunk))
                        } else if('function' === typeof chunk.valueOf &amp;&amp; chunk.valueOf() !== chunk) {
                            this.transform(chunk.valueOf(), controller)
                        } else if('toJSON' in chunk) {
                            this.transform(JSON.stringify(chunk), controller)
                        }
                        break
                    case 'symbol':
                        console.error(`cannot send a symbol as chunk part`)
                        break
                    case 'undefined':
                        console.error('cannot send a undefined as chunk part')
                    default: 
                        controller.enqueue(this.textencoder.encode(String(chunk)))

                }
            },
            flush() {},
        }
        class AnyTypeToU8Stream extends TransformStream {
            constructor() {
                super({...transformContent, textencoder: new TextDecoder()})
            }
        }
``` 

# 一、大文件基本概念
1. 分片上传: 将大文件按照一定的大小分割（file.slice()）为多个数据块进行上传,在服务端对这些数据块进行整合
2. 断点续传： 在数据块上传的过程中，遇到网络故障，可以从未上传完成的部分开始上传，不必将所有数据再重新上传一次
3. 秒传：将需要上传的文件上传，服务端会对上传的文件进行MD5校验，如果服务器上有一样的东西，就会直接客户端返回新的地址

# 二、 上传流程

![截屏2023-03-17 14.44.09.png#607px #702px](/download/attachments/2470304806/%E6%88%AA%E5%B1%8F2023-03-17%2014.44.09.png?version=1&modificationDate=1679035453079&api=v2)

