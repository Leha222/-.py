# -.py
import os
from PyQt5.QtWidgets import (
   QApplication, QWidget,
   QFileDialog,
   QLabel, QPushButton, QListWidget,
   QHBoxLayout, QVBoxLayout
)
from PyQt5.QtCore import Qt # нужна константа Qt.KeepAspectRatio для изменения размеров с сохранением пропорций
from PyQt5.QtGui import QPixmap # оптимизированная для показа на экране картинка

from PIL import Image
from PIL.ImageQt import ImageQt # для перевода графики из Pillow в Qt 
from PIL import ImageFilter
from PIL.ImageFilter import (
   BLUR, CONTOUR, DETAIL, EDGE_ENHANCE, EDGE_ENHANCE_MORE,
   EMBOSS, FIND_EDGES, SMOOTH, SMOOTH_MORE, SHARPEN,
   GaussianBlur, UnsharpMask
)


app = QApplication([])
win = QWidget()
win.resize(700,500)
win.setWindowTitle('Easy Editor')

btn_left = QPushButton('Лево')
btn_right = QPushButton('Право')
btn_flip = QPushButton('Зеркало')
btn_sharp = QPushButton('Резкость')
btn_bw = QPushButton('Ч/б')
btn_save = QPushButton('Сохранить')
btn_reset = QPushButton('Сбросить фильтры')
btn_dir = QPushButton('Папка')

lw_files = QListWidget()
lb_image = QLabel("Картинка")

row = QHBoxLayout()
col1 = QVBoxLayout()
col2 = QVBoxLayout()
row_tools = QHBoxLayout()

col1.addWidget(btn_dir)
col1.addWidget(lw_files)
row_tools.addWidget(btn_left)
row_tools.addWidget(btn_right)
row_tools.addWidget(btn_flip)
row_tools.addWidget(btn_sharp)
row_tools.addWidget(btn_bw)
row_tools.addWidget(btn_save)
row_tools.addWidget(btn_reset)
col2.addWidget(lb_image)
col2.addLayout(row_tools)

row.addLayout(col1)
row.addLayout(col2)

workdir = ""
def chooseWorkdir():
    global workdir
    workdir = QFileDialog.getExistingDirectory()

def filter(files, extension):
    result = []
    for filename in files:
        for extension in extension:
            if filename.endswith(extension):
                result.append(filename)
    return result

def showFilenamesList():
    extensions = ['.jpeg','.jpg','.png','.bmp']
    chooseWorkdir()
    filename = filter(os.listdir(workdir), extensions)
    lw_files.clear()
    for filename in filename:
        lw_files.addItem(filename)

btn_dir.clicked.connect(showFilenamesList)

class ImageProcessor():
    def __init__(self):
        self.image = None
        self.filename = None
        self.save_dir = 'Modified/'
        self.original_image = None

    def loadImage(self, filename):
        self.filename = filename
        image_path = os.path.join(workdir, filename)
        self.image = Image.open(image_path)
        self.original_image = self.image.copy()

    def showImage(self, image):
        qimage = ImageQt(image)
        pixmapimage = QPixmap.fromImage(qimage) 
        label_width, label_height = lb_image.width(), lb_image.height()      
        scaled_pixmap = pixmapimage.scaled(label_width, label_height, Qt.KeepAspectRatio)
        lb_image.setPixmap(scaled_pixmap)
        lb_image.setVisible(True)

    def do_bw(self):
        self.image = self.image.convert('L')
        self.showImage(self.image)

    def saveImage(self):
        path = os.path.join(workdir, self.save_dir)
        if not(os.path.exists(path) or  os.path.isdir(path)):
            os.mkdir(path)
        image_path = os.path.join(path, self.filename)
        self.image.save(image_path)

    def do_bw(self):
        self.image = self.image.convert('L')
        self.showImage(self.image)

    def do_left(self):
        self.image = self.image.transpose(Image.ROTATE_90)
        self.showImage(self.image)

    def do_right(self):
        self.image = self.image.transpose(Image.ROTATE_270)
        self.showImage(self.image)

    def do_sharpen(self):
        self.image = self.image.filter(SHARPEN)
        self.showImage(self.image)
    
    def do_flip(self):
        self.image = self.image.transpose(Image.FLIP_LEFT_RIGHT)
        self.showImage(self.image)
        
    def resetImage(self):
        self.image = self.original_image.copy()
        self.showImage(self.original_image)

workimage = ImageProcessor()

def showChosenImage():
    if lw_files.currentRow() >= 0 :
        filename = lw_files.currentItem().text()
        workimage.loadImage(filename)
        image_path = os.path.join(workdir, workimage.filename)
        workimage.showImage(image_path)

lw_files.currentRowChanged.connect(showChosenImage)

btn_bw.clicked.connect(workimage.do_bw)
btn_left.clicked.connect(workimage.do_left)
btn_right.clicked.connect(workimage.do_right)
btn_sharp.clicked.connect(workimage.do_sharpen)
btn_flip.clicked.connect(workimage.do_flip)
btn_reset.clicked.connect(workimage.resetImage)
btn_save.clicked.connect(workimage.saveImage)
win.setLayout(row)
win.show()
app.exec()

