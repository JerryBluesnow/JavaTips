

## webaudioapi

- [webaudioapi.com](https://webaudioapi.com/book/Web_Audio_API_Boris_Smus_html/toc.html)

- [译文-web audio api 入门](https://www.cnblogs.com/hanshuai/p/13620908.html)

- [Getting Started with Web Audio API](https://www.html5rocks.com/en/tutorials/webaudio/intro/)

- [HTML5 WebAudioAPI简介(一)](https://www.cnblogs.com/tianma3798/p/6033613.html)

- [BiquadFilterNode - 表示一个简单的低阶滤波器](https://www.mifengjc.com/api/BiquadFilterNode.html)

- [js实现pcm数据编码](https://github.com/2fps/demo/blob/master/view/2019/04/js实现pcm数据编码/index.html)
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>pcmtowav</title>
</head>
<body>
    <div>
        getUserMedia需要https，使用localhost或127.0.0.1时，可用http。
    </div>
    <button id="start">开始录音</button>
    <button id="end">结束录音</button>
    <button id="play">播放录音</button>
</body>
<script>
var context = null,
    inputData = [],
    size = 0,
    audioInput = null,
    recorder = null,
    dataArray;

document.getElementById('start').addEventListener('click', function() {
    context = new (window.AudioContext || window.webkitAudioContext)();
	// 清空数据
	inputData = [];
    // 录音节点
    recorder = context.createScriptProcessor(4096, 1, 1);

    recorder.onaudioprocess = function(e) {
        // getChannelData返回Float32Array类型的pcm数据
        var data = e.inputBuffer.getChannelData(0);

        inputData.push(new Float32Array(data));
        size += data.length;
    }

    navigator.mediaDevices.getUserMedia({
        audio: true
    }).then((stream) => {
        audioInput = context.createMediaStreamSource(stream);

    }).catch((err) => {
        console.log('error');
    }).then(function() {
        audioInput.connect(recorder);
        recorder.connect(context.destination);
    });
});
document.getElementById('end').addEventListener('click', function() {
    recorder.disconnect();
});
document.getElementById('play').addEventListener('click', function() {
    recorder.disconnect();
    if (0 !== size) {
        // 组合数据
        // var data = combine(inputData, size);
        // console.log(data.buffer);
        // 输出pcm二进制数据
		console.log(encodePCM());
    }
});
// ----------------------
// 以下是增加的内容
var oututSampleBits = 16;  // 输出采样数位

// 数据简单处理
function decompress() {
    // 合并
    var data = new Float32Array(size);
    var offset = 0; // 偏移量计算
    // 将二维数据，转成一维数据
    for (var i = 0; i < inputData.length; i++) {
        data.set(inputData[i], offset);
        offset += inputData[i].length;
    }
    return data;
};
// pcm编码
function encodePCM() {
    let bytes = decompress(),
        sampleBits = oututSampleBits,
        offset = 0,
        dataLength = bytes.length * (sampleBits / 8),
        buffer = new ArrayBuffer(dataLength),
        data = new DataView(buffer);

    // 写入采样数据 
    if (sampleBits === 8) {
        for (var i = 0; i < bytes.length; i++, offset++) {
            // 范围[-1, 1]
            var s = Math.max(-1, Math.min(1, bytes[i]));
            // 8位采样位划分成2^8=256份，它的范围是0-255; 16位的划分的是2^16=65536份，范围是-32768到32767
            // 因为我们收集的数据范围在[-1,1]，那么你想转换成16位的话，只需要对负数*32768,对正数*32767,即可得到范围在[-32768,32767]的数据。
            // 对于8位的话，负数*128，正数*127，然后整体向上平移128(+128)，即可得到[0,255]范围的数据。
            var val = s < 0 ? s * 128 : s * 127;
            val = parseInt(val + 128);
            data.setInt8(offset, val, true);
        }
    } else {
        for (var i = 0; i < bytes.length; i++, offset += 2) {
            var s = Math.max(-1, Math.min(1, bytes[i]));
            // 16位直接乘就行了
            data.setInt16(offset, s < 0 ? s * 0x8000 : s * 0x7FFF, true);
        }
    }

    return data;
}
</script>
</html>
```