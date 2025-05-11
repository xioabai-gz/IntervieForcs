# python基础语法

## 错误与异常

异常有的不会被程序捕捉

异常类型做为错误信息的一部分显示出来：示例中的异常分别为 零除错误（ ZeroDivisionError ） ，命名错误（ NameError） 和 类型错误（ TypeError ）。打印错误信息时，异常的类型作为异常的内置名显示。

·	try … except 语句可以带有一个 ***else子句***，该子句只能出现在所有 except 子句**之后**。当 try 语句**没有抛出异常时，需要执行一些代码**，可以使用这个子句。例如:



### 用户自定义异常

在程序中可以创建新的异常类型来命名自己的异常，异常类应该直接或间接从Exception类派生

```python
class MyError(Exception):
    def __init__(self,value):
        self.value = value
       	
```

如果一个新创建的模块中**需要抛出几种不同的错误时**，一个**通常的作法是为该模块定义一个异常基类**，然后**针对不同的错误类型派生出对应的异常子类:**

#### step1:定义异常基类和子类

```python
# image_processing.py

class ImageProcessingError(Exception):
    """图像处理模块的基类异常"""
    pass

class ImageNotFoundError(ImageProcessingError):
    """图像文件未找到"""
    pass

class UnsupportedFormatError(ImageProcessingError):
    """图像格式不受支持"""
    pass

class CorruptedImageError(ImageProcessingError):
    """图像内容为空或损坏"""
    pass

class InvalidImageSizeError(ImageProcessingError):
    """图像尺寸不符合要求"""
    pass

```

#### step2:模块中抛出异常

```python
import cv2
import os

def load_image(path):
    if not os.path.exists(path):
        raise ImageNotFoundError(f"找不到图像文件：{path}")
    
    image = cv2.imread(path)
    if image is None:
        raise CorruptedImageError(f"图像可能已损坏或格式不受支持：{path}")
    
    h, w = image.shape[:2]
    if h < 100 or w < 100:
        raise InvalidImageSizeError(f"图像尺寸太小：{w}x{h}")

    return image

```

#### Step 3：调用方使用异常处理

```python
from image_processing import load_image, ImageProcessingError, InvalidImageSizeError

try:
    img = load_image("test.jpg")
except InvalidImageSizeError as e:
    print("图像太小，不能用于处理。")
except ImageProcessingError as e:
    print(f"图像处理错误：{e}")

```

#### 好处总结：

- 所有错误统一继承自 `ImageProcessingError`，便于调用方捕获和统一处理；
- 调用者可以**根据具体错误子类做更细致处理**；
- 日后**新增错误类型（比如：图像过曝）只需添加新的子类即可**，不影响现有结构





### finally子句

 finally放在最后执行，在任何情况下不论是否有异常抛出都会被执行。

通常用于释放外部资源（文件或者网络链接之类）





## 类与函数



























