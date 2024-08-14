## python 启动WinSCP，进行文件上传下载

### 背景

原本是在服务器内写windows的bat脚本，进行启动winscp软件，进行上传下载。如果某次上传失败，需要进入服务器，才能进行操作。

为了更方便地主动上传文件，管理上传的服务，采用搭建python微服务的形式，进行调用。

### 实现

1. 先自主生成winscp需要的脚本文件
2. 在对应服务器上搭建python的flask框架
3. 请求python接口，返回脚本文件，winscp日志文件内容

### script脚本内容

```
open sftp://u89786096:QqX05******@160.43.94.20/ -hostkey="ssh-rsa 2048 24:95:*********"
lcd D:\poms\upload_bot\data
cd /
put EDU_BENC_SHARE_20240725.xlsx
exit
```



### app.py

```
from flask imort Flask,request
import sftp

app = Flask(__name__)


@app.route('/sftp/upload', methods=['POST'])
def sftpUpload():
    scriptpath = request.json.get('scriptpath')
    logpath = request.json.get('logpath')
    print(scriptpath)
    print(logpath)
    return sftp.sftupload(scriptpath, logpath)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port='5600')
```



### sftp.py

```
#-*-coding:UTF-8-*-
import subprocess
import os
def sftupload(scriptpath, logpath):
    log_content = ''
    script_content = ''
    try:
        # cmdFile = r'D:\\poms\\TSIP_results\\WinSCP\\script4.txt'
        # logfile = 'D:\\poms\\TSIP_results\\WinSCP\\WinSCP11.log' # 日志只能本地路径 // '\\10.21.22.183\temp\huihe48\log\script_113.log'会报错: '写日志时出错。日志已经被关闭'
        ret = subprocess.run(["D:\\Software\\WinSCP\\WinSCP.com", "/script=" + scriptpath, "/log=" + logpath],
                             shell=True, encoding='utf-8')
        print('normal: ret.returncode: ', ret.returncode) #0 成功， 1 失败 :
        print('os.path.yyye')
        if os.path.exists(scriptpath):
            with open(scriptpath, 'r', encoding='UTF-8') as f:
                script_content = f.read()

        if os.path.exists(logpath):
            with open(logpath, 'r', encoding='UTF-8') as f:
                log_content = f.read()

        print('logpathlogpathlogpath')
        print(logpath)
        if log_content.find('没有找到')!=-1:
            ret.returncode = 1
              
        return ({'returncode': ret.returncode, 'scriptcontent': script_content, 'logcontent': log_content})
    except Exception as e:
       print('exception: ret.returncode')  # 0 成功， 1 失败
       if os.path.exists(scriptpath):
           with open(scriptpath, 'r', encoding='UTF-8') as f:
               script_content = f.read()

       if os.path.exists(logpath):
           with open(logpath, 'r', encoding='UTF-8') as f:
               log_content = f.read()
       return ({'returncode': 1, 'scriptcontent': script_content, 'logcontent': log_content})
```