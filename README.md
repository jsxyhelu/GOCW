现在将Csharp调用基于Opencv编写的类库文件(Dll)的方法定下来，我取名叫做GOCW。
https://github.com/jsxyhelu/GOCW/
 
一、CLR编写的DLL部分
1、按照正常方法引入Opencv;
2、提供接口函数，进行图像处理（这里只是实现了cvtColor，实际过程中可以用自己编写的复杂函数)
String^  Class1::Method(cli::array<unsigned char>^ pCBuf1)
{
     pin_ptr<System::Byte> p1 = &pCBuf1[0];
     unsigned char* pby1 = p1;
     cv::Mat img_data1(pCBuf1->Length,1,CV_8U,pby1);
     cv::Mat img_object = cv::imdecode(img_data1,IMREAD_UNCHANGED);
     //////////////////////////////////处理过程/////////
     cvtColor(img_object,img_object,40);
 /////////////////////////////////////////////////////////////////////////////////
     if (!img_object.data)
        return nullptr;
     //获得目录，保存文件
     cv::imwrite("c:/Method.jpg",img_object);
     return "c:/Method.jpg";
}
 
String^  Class1::Method2(cli::array<unsigned char>^ pCBuf1)
{
    pin_ptr<System::Byte> p1 = &pCBuf1[0];
    unsigned char* pby1 = p1;
    cv::Mat img_data1(pCBuf1->Length,1,CV_8U,pby1);
    cv::Mat img_object = cv::imdecode(img_data1,IMREAD_UNCHANGED);
    //////////////////////////////////处理过程///////////////////////
    cvtColor(img_object,img_object,6);
 /////////////////////////////////////////////////////////////////////////////////
    if (!img_object.data)
        return nullptr;
    //获得目录，保存文件
    cv::imwrite("c:/Method2.jpg",img_object);
    return "c:/Method2.jpg";
}
二、Winform调用接口部分（TIP:不仅可以用Winform调用，asp.net/webservice都是可以调用的）
1、直接引用clr dll
2、编写helper文件（应该也可以叫做 warpper），通过外部IO的方法获取clr dll的文件
 class GOCsharpHelper
    {
        Class1 client = new Class1();
        string strResult1 = null;
        string strResult2 = null;
        //输入参数是string或bitmap
        public Bitmap ImageProcess(string ImagePath){
            Image  ImageTemp = Bitmap.FromFile(ImagePath);
            return ImageProcess(ImageTemp);
        }
        //输出结果是bitmap
        public Bitmap ImageProcess(Image image)
        {
            MemoryStream ms = new MemoryStream();
            image.Save(ms, System.Drawing.Imaging.ImageFormat.Jpeg);
            byte[] bytes = ms.GetBuffer();
            strResult1 = client.Method(bytes);
            Image ImageResult = Bitmap.FromFile(strResult1);
            return (Bitmap)ImageResult;
        }
        public Bitmap ImageProcess2(string ImagePath)
        {
            Image ImageTemp = Bitmap.FromFile(ImagePath);
            return ImageProcess2(ImageTemp);
        }
        //输出结果是bitmap
        public Bitmap ImageProcess2(Image image)
        {
            MemoryStream ms = new MemoryStream();
            image.Save(ms, System.Drawing.Imaging.ImageFormat.Jpeg);
            byte[] bytes = ms.GetBuffer();
            strResult2 = client.Method2(bytes);
            Image ImageResult = Bitmap.FromFile(strResult2);
            return (Bitmap)ImageResult;
        }
        public void Clear()
        {
            if (File.Exists(strResult1))
                File.Delete(strResult1);
            if (File.Exists(strResult2))
                File.Delete(strResult2);
        }
    }
3、使用例子(注意控件的dispose):
 
   private void button2_Click(object sender, EventArgs e)
        {
            if (pictureBox1.Image != null)
                pictureBox1.Image.Dispose();
            if (pictureBox2.Image != null)
                pictureBox2.Image.Dispose();
           Image image1 = gocsharphelper.ImageProcess(" E:/sandbox/logo.jpg");
           pictureBox1.Image = image1;
           Image image2 = gocsharphelper.ImageProcess2("E:/sandbox/lena.jpg");
           pictureBox2.Image = image2;
         
        }
 
三、解释说明 
使用外部I/O不仅仅是权宜之计，实际上Opencv的Decode使用的就是外部I/O。就目前研究的水平来说，这是最稳定的。
目前搭建成功的框架已经能够完成“csharp调用opencv的”目标，并且在调试、参数传递方面都很强。
如果是处理静态图片，已经够用。
四、杀手程序
GOImageResearch:
使用这种方法编写的图像处理预分析程序。

 
