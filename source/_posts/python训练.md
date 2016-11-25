title: python训练
date: 2016-10-21 17:10:59
categories:
tags:
  - python
  - simple training
---
#引言
这两年零零碎碎学了下python，除了造轮子爬1024就没干过别的事情orz，现在找些训练做做，训练参照[Python 练习册，每天一个小程序][link1]

#0000题
将你的 QQ 头像（或者微博头像）右上角加上红色的数字，类似于微信未读信息数量那种提示效果。 类似于<!--more-->图中效果[][pic1]
要在一张图中绘图，可以使用python的图像处理库*PIL：Python Imaging Library*,由于PIL仅支持到Python 2.7，加上年久失修，于是
一群志愿者在PIL的基础上创建了兼容的版本，名字叫[Pillow][link2]，支持最新Python 3.x。

绘图可以使用其中的<code>ImageDraw</code>来实现，以下是实现
```python
# -*- coding: utf-8 -*-

from PIL import Image, ImageDraw, ImageFont


def add_img(img):
    draw = ImageDraw.Draw(img)
    myfont = ImageFont.truetype('C:/windows/fonts/Arial.ttf', size=20) #设置绘制的style和size
    fillcolor = "#ff0000" #设置填充颜色
    width, height = img.size #为图片右边与底部
    draw.text((width-200, 0), '100', font=myfont, fill = fillcolor)
    img.save('out.jpg','jpeg')  #保存新图片

if __name__ == '__main__':
    image = Image.open('in.jpg') #你要编辑的图片
    add_img(image)

```
![原图][pic2]
![处理后][pic3]
然后脑洞了一下，把数字换成中文，结果无法显示，想了想，会不会是因为字体不支持，换了一下字体后，bingo！

```python
# -*- coding: utf-8 -*-

from PIL import Image, ImageDraw, ImageFont


def add_img(img):
    draw = ImageDraw.Draw(img)
    myfont = ImageFont.truetype('C:/windows/fonts/simsun.ttc', size=150) #选择正确字体
    fillcolor = "#ff0000"
    width, height = img.size
    draw.text((width-1000, 150), '抱头蹲防QAQ', font=myfont, fill = fillcolor)
    img.save('out2.jpg','jpeg')

if __name__ == '__main__':
    image = Image.open('in2.jpg')
    add_img(image)

```
![输出][pic4]

#0001题
做为 Apple Store App 独立开发者，你要搞限时促销，为你的应用生成激活码（或者优惠券），使用 Python 如何生成 200 个激活码（或者优惠券）？

生成激活码，就是把需要的字符放到一个list里，然后每次random取出其中一个，另外考虑到激活码的可读性，可以把容易混淆的字符去掉（如'I','l','1','O','0'）
```python
import random

lower_case = "abcdefghjkmnopqrstuvwxyz"
upper_case = lower_case.upper()
num_case = ''.join(map(str,range(2,11)))
init_case = ''.join((lower_case,upper_case,num_case))


if __name__ == '__main__':
    f = open('Activition_code.txt','a')
    for i in range(200):
        s = random.sample(init_case,6)
        f.write(''.join(s) + '\n')
    f.close()
```
激活码生成了，接下来我们可以结合<code>PIL</code>的<code>draw</code>来试着生成随机验证码,借鉴了[残阳似血（@秦续业)][link3]的[轮子][link4]

```python
import random
from PIL import Image, ImageDraw, ImageFont, ImageFilter

lower_case = "abcdefghjkmnopqrstuvwxyz"
upper_case = lower_case.upper()
num_case = ''.join(map(str,range(2,11)))
init_case = ''.join((lower_case,upper_case,num_case))

def create_verification_code(size=(100,20),
                            case = init_case,
                            length = 4,
                            img_type = "JPEG",
                            mode = "RGB",
                            bg_color = (255,255,255),
                            fg_color=(255,0,0),
                            font_size=16,
                            font_type="C:/windows/fonts/Arial.ttf"):
                            #图片格式
                            #字符集
                            #验证码长度
                            #默认保存格式
                            #图片模式
                            #背景默认白色
                            #验证码颜色
                            #验证码字体大小
                            #字体
    width, height = size
    img = Image.new(mode, size, bg_color)
    draw = ImageDraw.Draw(img)
    def get_num():          #生成验证码字符
        return random.sample(case,length)
    def create_ver_char():
        num_chars = get_num()
        ver_chars = ' %s ' % ' '.join(num_chars)

        font = ImageFont.truetype(font_type, font_size)
        font_width, font_height = font.getsize(ver_chars)

        draw.text(((width - font_width) / 3, (height - font_height) / 3),
                    ver_chars, font=font, fill=fg_color)
        return ''.join(num_chars)

    strs = create_ver_char()
   # 图形扭曲参数
    params = [1 - float(random.randint(1, 2)) / 100,
              0,
              0,
              0,
              1 - float(random.randint(1, 10)) / 100,
              float(random.randint(1, 2)) / 500,
              0.001,
              float(random.randint(1, 2)) / 500
              ]
    img = img.transform(size, Image.PERSPECTIVE, params) # 创建扭曲

    img = img.filter(ImageFilter.EDGE_ENHANCE_MORE) # 滤镜，边界加强（阈值更大）

    return img,strs


if __name__ == '__main__':
    code_img = create_verification_code()
    code_img[0].save("verification.jpeg", "JPEG")
```
生成验证码如下
![][pic5]

[link1]:https://github.com/Yixiaohan/show-me-the-code
[link2]:https://github.com/python-pillow/Pillow
[link3]:http://qinxuye.me/
[link4]:http://qinxuye.me/article/create-validate-code-image-with-pil/
[pic1]:/img/python_training/Sample0000.png
[pic2]:/img/python_training/in.jpg
[pic3]:/img/python_training/out.jpg
[pic4]:/img/python_training/out2.jpg
[pic5]:/img/python_training/verification.jpeg
