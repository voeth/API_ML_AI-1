# 随记APP产品文档
| 产品名称   | 随口记APP      |
| ---------- | ----------- |
| 产品负责人 | 钟靖欣      |
| 文档进度 | 未完成 |

---


## **1_价值宣言**
随记APP是一款智能生活记录APP,它面向有日常生活中有记录习惯的用户，其使用高效的语音识别和智能处理，方便记录生活里每个细节并进行梳理，优化用户生活。


### 1.1核心价值
生活记账一直是许多人的一种常规需求，尤其是现在处于手机支付越来越便利的时代，很多人记账的需求也越来越高，传统手输记账的方式操作多且慢，所以高效效率记账成为人们迫切需求。用户只需要一键打开功能键就可以进行语音记录，将其语音内容转换为文字反馈于用户，做到短时间内快速完成记账过程。并将记录的内容通过智能文本分类进行梳理呈现

### 1.2API加值宣言
APP通过使用百度语音识别技术，可以自定义上传和识别词库内容。用户只需正常说话，轻松动动嘴巴，就可以按照语义提取费用金额和将需要记录的内容自动转换为文字，然后清晰展示用户想要记录的内容。语音的交互功能和智能分类能让用户真正解放双手，在满足记账业务的基本需求外，也让用户能够节省时间，极大地提升便利性同时过程保持乐趣。

### 1.3用户痛点
- 记账前需要花时间去构思消费类型以及子分类，甚至各类账本
- 面对繁多的键盘操作键，输入过程繁琐至无法下手
- 繁琐的记录过程花费大量时间


### 1.4用户需求列表与人工智能API加值

用户需求 | 人工智能API
---|---
想要快速记录，不想手动输入信息 | 语音识别转文字
想要快速分类不同种类信息 | 文本分类

**用户使用场景**

- 小林发现刚发的工资哦“一下子”就花完了，想要知道钱花去哪了，决定从今天开始养成记账的习惯，但是发现传统记账方式过程繁琐，提不起兴趣去记录，不易养成习惯，于是打开口记APP，点击语音记账按钮，快速向手机说出需要记录的内容，内容迅速转换为文字记录在手机中
- 小林在App内触发语音识别功能，录入想让系统识别的语音内容

## 2，产品架构与产品原型设计
### 2.1产品架构图
![输入图片说明](https://images.gitee.com/uploads/images/2019/1223/171302_c36983c6_1648163.png "架构图.png")
### 2.2产品功能架构图
![输入图片说明](https://images.gitee.com/uploads/images/2019/1223/171343_138cfa20_1648163.png "功能架构.png")
### 2.3使用流程图
![输入图片说明](https://images.gitee.com/uploads/images/2019/1223/171449_f1cf7739_1648163.png "流程图.png")

### 2.4产品原型设计
 **首页** 

![输入图片说明](https://images.gitee.com/uploads/images/2019/1223/171542_74111788_1648163.jpeg "捕获.JPG")



## API的输出入展示

- 文本使用的产品：语音转写（Long Form ASR）基于深度全序列卷积神经网络，将长段音频（5小时以内）数据转换成文本数据，为信息处理和数据挖掘提供基础。

- 文本接口描述：接口服务对实时音频流进行识别，同步返回识别结果，达到“边说边出文字”的效果。

- 请求地址：http[s]: //raasr.xfyun.cn/api/xxx
- 输入


```
import websocket
import requests
import datetime
import hashlib
import base64
import hmac
import json
import os, sys
import re
from urllib.parse import urlencode
import logging
import time
import ssl
import wave
from wsgiref.handlers import format_date_time
from datetime import datetime
from time import mktime
from pyaudio import PyAudio,paInt16
from get_audio import get_audio # 导入录音.py文件
input_filename = "input.wav" # 麦克风采集的语音输入
input_filepath = "./audios/" # 输入文件的path
in_path = input_filepath + input_filename
type = sys.getfilesystemencoding()
path_pwd = os.path.split(os.path.realpath(__file__))[0]
os.chdir(path_pwd)
try:
 import thread
except ImportError:
 import _thread as thread
logging.basicConfig()
STATUS_FIRST_FRAME = 0 # 第一帧的标识
STATUS_CONTINUE_FRAME = 1 # 中间帧标识
STATUS_LAST_FRAME = 2 # 最后一帧的标识
framerate = 8000
NUM_SAMPLES = 2000
channels = 1
sampwidth = 2
TIME = 2
global wsParam
class Ws_Param(object):
 # 初始化
 def __init__(self, host):
 self.Host = host
 self.HttpProto = "HTTP/1.1"
 self.HttpMethod = "GET"
 self.RequestUri = "/v2/iat"
 self.APPID = "5d312675" # 在控制台-我的应用-语音听写（流式版）获取APPID
 self.Algorithm = "hmac-sha256"
 self.url = "wss://" + self.Host + self.RequestUri
 # 采集音频 录音
 get_audio("./audios/input.wav")
 # 设置测试音频文件，流式听写一次最多支持60s，超过60s会引起超时等错误。
 self.AudioFile = r"./audios/input.wav"
 self.CommonArgs = {"app_id": self.APPID}
 self.BusinessArgs = {"domain":"iat", "language": "zh_cn","accent":"mandarin"}
 def create_url(self):
 url = 'wss://ws-api.xfyun.cn/v2/iat'
 now = datetime.now()
 date = format_date_time(mktime(now.timetuple()))
 APIKey = 'a6aabfcca4ae28f9b6a448f705b7e432' # 在控制台-我的应用-语音听写（流式版）获取APIKey
 APISecret = 'e649956e14eeb085d1b0dce77a671131' # 在控制台-我的应用-语音听写（流式版）获取APISecret
 signature_origin = "host: " + "ws-api.xfyun.cn" + "\n"
 signature_origin += "date: " + date + "\n"
 signature_origin += "GET " + "/v2/iat " + "HTTP/1.1"
 signature_sha = hmac.new(APISecret.encode('utf-8'), signature_origin.encode('utf-8'),
 digestmod=hashlib.sha256).digest()
 signature_sha = base64.b64encode(signature_sha).decode(encoding='utf-8')
 authorization_origin = "api_key=\"%s\", algorithm=\"%s\", headers=\"%s\", signature=\"%s\"" % (
 APIKey, "hmac-sha256", "host date request-line", signature_sha)
 authorization = base64.b64encode(authorization_origin.encode('utf-8')).decode(encoding='utf-8')
 v = {
 "authorization": authorization,
 "date": date,
 "host": "ws-api.xfyun.cn"
 }
 url = url + '?' + urlencode(v)
 return url
# 收到websocket消息的处理 这里我对json解析进行了一些更改 打印简短的一些信息
def on_message(ws, message):
 msg = json.loads(message) # 将json对象转换为python对象 json格式转换为字典格式
 try:
 code = msg["code"]
 sid = msg["sid"]
 
 if code != 0:
 errMsg = msg["message"]
 print("sid:%s call error:%s code is:%s\n" % (sid, errMsg, code))
 else:
 result = msg["data"]["result"]["ws"]
 # 以json格式显示
 data_result = json.dumps(result, ensure_ascii=False, sort_keys=True, indent=4, separators=(',', ': '))
 print("sid:%s call success!" % (sid))
 print("result is:%s\n" % (data_result))
 except Exception as e:
 print("receive msg,but parse exception:", e)
# 收到websocket错误的处理
def on_error(ws, error):
 print("### error:", error)
# 收到websocket关闭的处理
def on_close(ws):
 print("### closed ###")
# 收到websocket连接建立的处理
def on_open(ws):
 def run(*args):
 frameSize = 1280 # 每一帧的音频大小
 intervel = 0.04 # 发送音频间隔(单位:s)
 status = STATUS_FIRST_FRAME # 音频的状态信息，标识音频是第一帧，还是中间帧、最后一帧
 with open(wsParam.AudioFile, "rb") as fp:
 while True:
 buf = fp.read(frameSize)
 # 文件结束
 if not buf:
 status = STATUS_LAST_FRAME
 # 第一帧处理
 # 发送第一帧音频，带business 参数
 # appid 必须带上，只需第一帧发送
 if status == STATUS_FIRST_FRAME:
 d = {"common": wsParam.CommonArgs,
 "business": wsParam.BusinessArgs,
 "data": {"status": 0, "format": "audio/L16;rate=16000",
 "audio": str(base64.b64encode(buf),'utf-8'),
 "encoding": "raw"}}
 d = json.dumps(d)
 ws.send(d)
 status = STATUS_CONTINUE_FRAME
 # 中间帧处理
 elif status == STATUS_CONTINUE_FRAME:
 d = {"data": {"status": 1, "format": "audio/L16;rate=16000",
 "audio": str(base64.b64encode(buf),'utf-8'),
 "encoding": "raw"}}
 ws.send(json.dumps(d))
 # 最后一帧处理
 elif status == STATUS_LAST_FRAME:
 d = {"data": {"status": 2, "format": "audio/L16;rate=16000",
 "audio": str(base64.b64encode(buf),'utf-8'),
 "encoding": "raw"}}
 ws.send(json.dumps(d))
 time.sleep(1)
 break
 # 模拟音频采样间隔
 time.sleep(intervel)
 ws.close()
 thread.start_new_thread(run, ())
if __name__ == "__main__":
 wsParam = Ws_Param("ws-api.xfyun.cn") #流式听写 域名
 websocket.enableTrace(False)
 wsUrl = wsParam.create_url()
 ws = websocket.WebSocketApp(wsUrl, on_message=on_message, on_error=on_error, on_close=on_close)
 ws.on_open = on_open
 ws.run_forever(sslopt={"cert_reqs": ssl.CERT_NONE})

```
- 输出

```
{
    "ok":0,
    "err_no":0,
    "failed":null,
    "data":"[{\"bg\":\"0\",\"ed\":\"4950\",\"onebest\":\"此次消费15元。\",\"speaker\":\"0\"}]"
}
```


## API使用比较分析 

**市场产品成熟度：**

市面上比较成熟的商用语音交互平台有很多如百度和科大讯飞，这两家的语音识别技术成熟，准确率高达97%，其中科大讯飞作为中国最大的智能语音技术提供商，在智能语音技术领域有着长期的研究积累，并在中文语音合成、语音识别、口语评测等多项 技术上拥有国际领先的成果。

1. 科大讯飞提供语音识别、语音合成、声纹识别等全方位的语音交互平台。
2. 拥有自主知识产权的智能语音技术，科大讯飞已推出从大型电信级 应用到小型嵌入式应用，从电信、金融等行业到企业和家庭用户，从PC到手机到MP3/MP4/PMP和玩具，能够满足不同应用环境的多种产品。
3. 科大讯飞占有中文语音技术市场60%以上市场份额，语音合成产品市场份额达到70%以上。

**费用和性价比的比较：**

![讯飞语音识别](https://images.gitee.com/uploads/images/2019/1224/010650_965dd473_1648163.jpeg "讯飞语音识别.JPG")![百度语音识别](https://images.gitee.com/uploads/images/2019/1224/010720_d4e1f943_1648163.jpeg "百度语音识别.JPG")

从试验的结果来看，科大讯飞语音识别功能前5个小时可以免费使用，转换精准率非常高；百度语音识别功能可以免费使用但是准确率无付费的高.



## 使用后风险报告 
1，目前系统支持的语音时长上限为60秒，如果超过这个长度会有错误返回
2，讯飞语音输入的识别率相对较高，但是每个地区、每个人的发音又各有特点，不尽相同，识别率有待进一步提高


