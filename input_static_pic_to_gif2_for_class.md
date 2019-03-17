# ʶ�����������������Ƭ����̬���ɴ�ī���͵��̾�Ķ�ͼ
![demo.gif](../images/input_static_pic_to_gif2_for_class.gif)

## ����
* Ϊʲô��д����������ӣ�
> ������͵Ĺ��̣����ⷢ����һƪ��Ϊ[deal-with-it-generator-face-recognition](https://www.makeartwithpython.com/blog/deal-with-it-generator-face-recognition/)������,
ͨ����ƪ���£�ʹ������д������ӵ��뷨.���������ںܶ�����Ƶ�о���������������Ϣ���о������е�����.

* ��л!
> д������ӳ�������������[burningion](https://www.makeartwithpython.com/)�ķ���. 

* �仯��
> [deal-with-it-generator-face-recognition](https://www.makeartwithpython.com/blog/deal-with-it-generator-face-recognition/) ��ƪ������һ�����۾��ļ����ӣ�
������ԭ�����ϣ�������̾�Ĳ��֣����ҰѴ���ṹ�ع���һ�£�ʹ�������չ��ά����Ҳ�����Ķ�.


## ʵ������

> ����������в�����ȡͼƬ��Ϣ��Ȼ������ʹ��Dlib�е���������㷨���鿴�Ƿ����������ڡ�����У�����Ϊÿ����������һ������λ�ã��۾����̾���ƶ������������

> Ȼ��������Ҫ���ź���ת���ǵ��۾����ʺ�ÿ���˵��������ǽ�ʹ�ô�Dlib��68��ģ�ͷ��صĵ㼯���ҵ��۾������ģ���Ϊ����֮��Ŀռ���ת��

> �������ҵ��۾�������λ�ú���ת�����ǿ���Ϊgif�����������۾�����Ļ�������롣���ǽ�ʹ��MoviePy��һ��make_frame������������

> ͬ���̾�Ҳ��������

> Ӧ�ó������ϵ�ṹ�ǳ��򵥡��������Ƚ���ͼƬ��Ȼ����ת��Ϊ�Ҷ�NumPy���顣����û��������������Լ��˳���������ڣ����ǾͿ��Խ���⵽��������Ϣ���ݵ���������Ԥ��ģ���С�

> ͨ�����ص������������ǿ���ѡ���۾������ź���ת���ǵ��۾�������ʺ��˵��沿��С��

> ��Ȼ������򲻽���ֻ�����һ�����������Լ����������Ϣ��

> ���ͨ����ȡ�������б����ǿ���ʹ��MoviePy����һ����ͼ��Ȼ���������ǵĶ���gif��

* 1.�����Ӧ�Ĺ��߰�
```python
import moviepy.editor as mpy
import numpy as np
from PIL import Image
from imutils import face_utils

try:
    from dlib import get_frontal_face_detector, shape_predictor
except ImportError:
    raise
```

* ��������ʶ��Ĺ�����`FaceDetect`�����Ӧ�ķ���
```python
class FaceDetect(object):
	pass

```

* ����`detector`��`predictor`�������ԣ���������dlib�⺯��
```python
@property
def detector(self):
	"""
	����Ƿ�������
	:return:
	"""
	return get_frontal_face_detector()

@property
def predictor(self):
	"""
	Ԥ����������
	:return:
	"""
	return shape_predictor('shape_predictor_68_face_landmarks.dat')

```

* ����`init_mask`�������������������Ϣ(ī�����̾����Ϣ)
```python
@classmethod
def load(cls, img_src):
	"""
	����ͼƬתΪImage����
	:param img_src:
	:return:
	"""
	return Image.open(img_src)
	
	
def init_mask(self):
	"""
	�������
	:return:
	"""
	self.deal, self.text, self.cigarette = (
		self.load(x) for x in ["../images/deals.png", "../images/text.png", "../images/cigarette.png"]
	)
```

* �����ռ�������Ϣ�Ķ�Ӧ����
> ����get_glasses_info��������ݵ�ǰ����������ֵ��ͼƬ�������ã���ͼƬ�����������沿��λ�������۽���б�ȣ����ı��۾�����λ�ü��Ƕ�,��������Ϣ���ظ��沿��λ����
> get_cigarette_info ��������ݵ�ǰ����������ֵ��ͼƬ�������ã�������������͵�λ�ã������䷵�ظ��沿��λ����
> orientation �����Ὣ������������Ϣͨ��"get_cigarette_info"��"get_glasses_info"�����������һ�����ظ���ͼ���������仭ͼ

```python
def get_glasses_info(self, face_shape, face_width):
	"""
	��ȡ��ǰ�沿���۾���Ϣ
	:param face_shape:
	:param face_width:
	:return:
	"""
	left_eye = face_shape[36:42]
	right_eye = face_shape[42:48]

	left_eye_center = left_eye.mean(axis=0).astype("int")
	right_eye_center = right_eye.mean(axis=0).astype("int")

	y = left_eye_center[1] - right_eye_center[1]
	x = left_eye_center[0] - right_eye_center[0]
	eye_angle = np.rad2deg(np.arctan2(y, x))

	deal = self.deal.resize(
		(face_width, int(face_width * self.deal.size[1] / self.deal.size[0])),
		resample=Image.LANCZOS)

	deal = deal.rotate(eye_angle, expand=True)
	deal = deal.transpose(Image.FLIP_TOP_BOTTOM)

	left_eye_x = left_eye[0, 0] - face_width // 4
	left_eye_y = left_eye[0, 1] - face_width // 6

	return {"image": deal, "pos": (left_eye_x, left_eye_y)}

def get_cigarette_info(self, face_shape, face_width):
	"""
	��ȡ��ǰ�沿���۾���Ϣ
	:param face_shape:
	:param face_width:
	:return:
	"""
	mouth = face_shape[49:68]
	mouth_center = mouth.mean(axis=0).astype("int")

	cigarette = self.cigarette.resize(
		(face_width, int(face_width * self.cigarette.size[1] / self.cigarette.size[0])),
		resample=Image.LANCZOS)

	x = mouth[0, 0] - face_width + int(16 * face_width / self.cigarette.size[0])
	y = mouth_center[1]
	return {"image": cigarette, "pos": (x, y)}

def orientation(self):
	"""
	������λ
	:return:
	"""
	faces = []
	for rect in self.rects:
		face = {}
		face_shades_width = rect.right() - rect.left()
		predictor_shape = self.predictor(self.img_gray, rect)
		face_shape = face_utils.shape_to_np(predictor_shape)

		face['cigarette'] = self.get_cigarette_info(face_shape, face_shades_width)
		face['glasses'] = self.get_glasses_info(face_shape, face_shades_width)

		faces.append(face)

	return faces
```

* �����ǿ�ʼ��ʵ�ֻ�ͼ����`drawing`
> ���ݴ���Ĳ���t����������GIF�Ľ���,�������û�ͼ����ǰ2�룬���ƶ���ߣ����۾����̾���������ǰ�����ƶ���Ȼ���ٻ������壬���������������.
> ����ƶ���ʵ�־�������̬������ߵ�������.

```python
def drawing(self, t):
	"""
	��̬��ͼ
	:param t:
	:return:
	"""
	draw_img = self.image.convert('RGBA')
	if t == 0:
		return np.asarray(draw_img)

	for face in self.orientation():
		if t <= self.duration - 2:
			current_x = int(face["glasses"]["pos"][0])
			current_y = int(face["glasses"]["pos"][1] * t / (self.duration - 2))
			draw_img.paste(face["glasses"]["image"], (current_x, current_y), face["glasses"]["image"])

			cigarette_x = int(face["cigarette"]["pos"][0])
			cigarette_y = int(face["cigarette"]["pos"][1] * t / (self.duration - 2))
			draw_img.paste(face["cigarette"]["image"], (cigarette_x, cigarette_y), face["cigarette"]["image"])
		else:
			draw_img.paste(face["glasses"]["image"], face["glasses"]["pos"], face["glasses"]["image"])
			draw_img.paste(face["cigarette"]["image"], face["cigarette"]["pos"], face["cigarette"]["image"])
			draw_img.paste(self.text, (75, draw_img.height // 2 + 128), self.text)

	return np.asarray(draw_img)
```

* ���ϻ����ĺ�����ʵ���Ѿ���ɣ�������������һ�³�ʼ������
```python
class FaceDetect(object):
    def __init__(self, img_src, gif_path=None):
        self.gif_max_width = 500
        self.duration = 4
        self.image = self.load(img_src).convert('RGBA')
        self.img_gray = None
        self.rects = None
        self.deal = None
        self.text = None
        self.cigarette = None
        if not self.validate:
            print("û�м�⵽�����������˳�.")
            exit(1)
        self.init_mask()
        self.make_gif(gif_path=gif_path)

    @property
    def validate(self):
        """
        ��֤�Ƿ������������������ڷ���False
        :return:
        """
        if self.image.size[0] > self.gif_max_width:
            scaled_height = int(self.gif_max_width * self.image.size[1] / self.image.size[0])
            self.image.thumbnail((self.gif_max_width, scaled_height))
        self.img_gray = np.array(self.image.convert('L'))
        self.rects = self.detector(self.img_gray, 0)
        return len(self.rects) > 0

    def make_gif(self, gif_path=None):
        """
        :param gif_path: ����·��
        :return:
        """
        gif_path = gif_path or "deal.gif"
        animation = mpy.VideoClip(self.drawing, duration=self.duration)
        animation.write_gif(gif_path, fps=self.duration)
```

* �������ʵ��һ��main����
```python
if __name__ == '__main__':
    # ���� python input_static_pic_to_gif2_for_class.py -image ../images/1.jpg
    import argparse

    parser = argparse.ArgumentParser()
    parser.add_argument("-image", required=True, help="path to input image")
    parser.add_argument("-save", required=False, default="deal.gif", help="path to output image")
    args = parser.parse_args()
    FaceDetect(args.image, args.save)
```

* д��������С���ܾ��Ѿ�ʵ���ˣ���Ҳ�������ʹ��һ��.[��ȡ��������](https://github.com/tomoncle/face-detection-induction-course/blob/master/input_static_pic_to_gif2_for_class.py)

## д�����
* ��ע[face-detection-induction-course](https://github.com/tomoncle/face-detection-induction-course)����ֿ⣬����ʱ���¸��������ʶ��Сʾ����
* �������ṩ��һ����ʱ���Եĵ�ַ: https://liyuanjun.cn/gifmask��ע��:**��2019-03-20��ֹͣ����.**
* [ģ������](https://github.com/davisking/dlib-models/raw/master/shape_predictor_68_face_landmarks.dat.bz2)