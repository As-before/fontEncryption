# -*- encoding: utf-8 -*-
"""
@File    : run.py
@Time    : 2020/4/2 17:02
@Author  : pang.yudong
"""
from flask import Flask, render_template
from fontTools.ttLib import TTFont
import random
import json
import re
import time
import os
from threading import Thread
import requests
from flask_apscheduler import APScheduler

app = Flask(__name__)


# 定时任务
class APSchedulerJobConfig(object):
    SCHEDULER_API_ENABLED = True
    JOBS = [
        {
            'id': 'No1',  # 任务唯一ID
            'func': 'run:get_txt',
            # 执行任务的function名称，run.get_txt 就是 `run.py` 文件，`get_txt` 是方法名称。文件模块和方法之间用冒号":"，而不是用英文的"."
            'args': '',  # 如果function需要参数，就在这里添加
            'trigger': 'interval',
            'seconds': 300
        },
    ]


# 写入到本地
def get_txt():
    url = 'https://v1.hitokoto.cn?encode=json'
    with open(r'config/wz.txt', 'w', encoding='utf-8') as f:
        f.write(requests.get(url).json()['hitokoto'])


# 读取本地
def get_wz():
    with open(r'config/wz.txt', 'r', encoding='utf-8') as f:
        return f.read()


# 异步方法
def async_call(fn):
    def wrapper(*args, **kwargs):
        Thread(target=fn, args=args, kwargs=kwargs).start()

    return wrapper


# 异步删除方法，等待0.01s删除请求后的文件，
@async_call
def remove(times):
    time.sleep(0.1)
    font_xml = 'config/fontello_{}.xml'.format(times)
    font_txt = 'config/nmap_new_{}.txt'.format(times)
    font_woff = 'static/fontello_new_{}.woff'.format(times)
    try:
        os.remove(font_xml)
    except:
        pass
    try:
        os.remove(font_txt)
    except:
        pass
    try:
        os.remove(font_woff)
    except:
        pass


# 字体加密
def conver_font():
    # 获取当前访问变量
    today_time = int(time.time())
    # 映射表
    mmap = {}
    font = TTFont()
    # 读取原始字体文件
    with open(r'D:\pang\python\python\flasks\test\static\fontello.xml', 'r', encoding='utf-8') as f:
        data = f.read()
    # 读取原始映射文件
    with open(r'D:\pang\python\python\flasks\test\config\nmap.txt', 'r', encoding='utf-8') as fr:
        vmap = json.loads(fr.read())
    # 随机打乱原来的字体编码
    sjlit = list(vmap.values())
    random.shuffle(sjlit)
    bmap = {}
    for k, y in mmap.items():
        # 查询原始文件的位置并替换
        code = (re.findall('<map code="(.*?)" name="{}"/>'.format(k), data))[0]
        xc = '<map code="{}" name="{}"/>'.format(code, k)
        try:
            bn = sjlit.pop()
        except:
            continue
        xc_new = '<map code="{}" name="{}"/>'.format(bn, k)
        data = data.replace(xc, xc_new)
        bk = mmap[k]
        # 生成加密后的字典
        bmap[bk[0]] = bn
    # 写入到xml并生成字体文件
    with open(r'D:\pang\python\python\flasks\test\config\fontello_{}.xml'.format(today_time), 'w') as fn:
        fn.write(data)
    font.importXML(r'D:\pang\python\python\flasks\test\config\fontello_{}.xml'.format(today_time))
    font.save(r'D:\pang\python\python\flasks\test\static\fontello_new_{}.woff'.format(today_time))
    # 写入映射表到文件
    text = json.dumps(bmap, ensure_ascii=False, indent=4)
    with open(r'D:\pang\python\python\flasks\test\config\nmap_new_{}.txt'.format(today_time), 'w',
              encoding='utf-8') as fw:
        fw.write(text)
    # 读取并返回
    with open(r'D:\pang\python\python\flasks\test\config\nmap_new_{}.txt'.format(today_time), 'r',
              encoding='utf-8') as fb:
        df = fb.read().replace('0xe', r'\uE')
        CIPHER_BOOK = json.loads(df)
    return CIPHER_BOOK, today_time


def _encrypt_secret(secret):
    CIPHER_BOOK, times = conver_font()
    vb = []
    for c in secret:
        try:
            vb.append(CIPHER_BOOK[c])
        except:
            vb.append(c)
    return ''.join(vb), times


@app.route('/')
def index():
    data, times = _encrypt_secret(get_wz())
    df = '/static/fontello_new_{}.woff'.format(times)
    remove(times)
    return render_template('index.html', data=data, times=df)


app.debug = False
app.config.from_object(APSchedulerJobConfig)
if __name__ == "__main__":
    scheduler = APScheduler()
    scheduler.init_app(app)
    scheduler.start()
    app.run()
