# 基于ObjectARX的AutoCAD楼板绘制插件（Draw_FloorSlab）
This plugin allows you to quickly draw floor slabs in AutoCAD.  

# 摘要
为了满足3D3S中生成楼板的需求，基于ObjectARX开发了一款AutoCAD的插件，用户可以根据参数一键生成楼板实体。本程序运用到了MFC的Dialog对话框，使用户更好地进行交互式操作，从而能够方便程序的使用。本程序还通过派生自定义梁实体和自定义板实体，来更方便地绘制实体，并进行相关信息的查询。  

# 使用简介
首先在AtuoCAD的命令行中输入“APPLOAD”进行加载ARX的动态链接库，然后输入“DrawPlate”命令显示出对话框，主界面如下图所示。  

![image1](https://github.com/taoziganbei/Draw_FloorSlab/blob/main/FloorSlabImage/FloorSlabImage1.jpg)  

用户设置好参数之后，可以根据自己需要添加轴线，设置轴线间距、梁的参数，以及显示样式等属性。点击“绘制楼板”，即可绘制出如下图所示的楼板。具有轴网、楼板实体、梁实体、XY平面的标注和XZ平面及YZ平面的标注。  
并可以选择显示样式是三维线框还是着色实体。并且可以查询实体的信息。  

![image2](https://github.com/taoziganbei/Draw_FloorSlab/blob/main/FloorSlabImage/FloorSlabImage2.jpg)  
![image3](https://github.com/taoziganbei/Draw_FloorSlab/blob/main/FloorSlabImage/FloorSlabImage3.jpg)  

# 代码说明

![image4](https://github.com/taoziganbei/Draw_FloorSlab/blob/main/FloorSlabImage/FloorSlabImage4.jpg)  

本程序具有三个与绘制相关的类：MyDlgPara、Plate、Beam。  

对话框MyDlgPara继承自CDialog，是对话框IDD_DIALOG_Plate关联的类。主要作用是操作窗体的控件、传递数据以及一些方便绘图的辅助函数。  

![image5](https://github.com/taoziganbei/Draw_FloorSlab/blob/main/FloorSlabImage/FloorSlabImage5.jpg)  

OnInitDialog()	对话框进行初始化填充数据  
OnBnClickedButtonAddAxial()	点击“加轴线”对ListControl添加一列  
OnNMDblclkListAxialInterval()、OnEnKillfocusEditGrid()	双击ListControl某一子项对其修改数据，并在鼠标移开之后隐藏EditControl  
OnBnClickedButtonDarw()	点击“绘制楼板”即进行绘制，并能在鼠标点选位置绘制  
OnBnClickedButtonShowType()	选择实体显示的样式，通过鼠标点选选择实体  
OnBnClickedButtonQuery()	选择实体查询属性信息  

自定义楼板实体CPlate类：  
为了方便绘制楼板，从AcDbEntity派生了自定义楼板实体，通过板四周的点位、板的厚度等参数进行楼板的绘制。这里板的定位点为板顶面。  
m_aPtNode	楼板的定位点，这里选择用楼板顶部的点作为定位点  
m_dThick	楼板厚度  
m_dTopElevation	楼板顶部标高  
m_Material	板的材料  
m_iShowOption	楼板的显示样式  

自定义梁实体CBeam类：  
同样，为了方便地绘制梁，从AcDbLine派生了自定义的梁实体CBeam类，通过梁的点位、梁截面高度、梁截面宽度等信息来绘制梁。  
m_aPt	梁的定位点，这里选截面顶部的中点为定位点  
m_dWidth和m_dHeight	梁截面的宽度和高度  
m_dTopElevation	梁截面顶部标高  
m_Material	梁的材料  
m_iShowOption	梁的显示样式  

添加轴网：  
点击“加轴线”即可增加一列，来继续添加轴网。通过OnBnClickedButtonAddAxial函数进行实现，通过获取当前行列数，并增加一列  

双击ListControl修改子项：  
通过双击ListControl某一子项，来修改当前格的参数。通过OnNMDblclkListAxialInterval函数和OnEnKillfocusEditGrid函数进行实现。  
其中，OnNMDblclkListAxialInterval函数实现：双击ListControl某一子项，然后界面上原本就有的EditControl显示在该位置（一开始隐藏了显示），并可以对EditControl内容进行编辑，看起来就像在编辑ListControl一样（因为实际上ListControl并不能直接编辑子项内容）。  
OnEnKillfocusEditGrid函数实现：当鼠标焦点从当前EditControl移开时，EditControl自动隐藏，并将当前得到的内容返回给ListControl对应位置的子项。  

绘制轴网：  
首先通过获取ListControl里的轴网间距，计算得到横纵轴线交点的坐标，为之后绘制楼板、梁、其他标注的基本点。重点编写AddAxial函数方便绘制  

绘制楼板和梁：  
通过前面获得的横纵轴网交点坐标，以及派生的自定义实体CPlate和CBeam，通过计算得到梁、板实体的定位点来绘制梁、板。  
同时绘制的参数还有板的厚度，梁的高宽，顶部标高，构件材料。  
绘制的时候m_iShowOption默认为0，即为三维线框的显示方式。  
为了方便绘制，用到了AddPlate和AddBeam函数，进行输入参数即可方便地绘制。  

绘制轴网标注、梁板平面标注、梁板高度：  
标注根据视图的方向不同，有：  
XY平面的标注（包含轴网间距、轴网总尺寸、楼板的长度和宽度、梁的宽度）  
XZ平面的标注（包含楼板厚度、梁的高度）  
YZ平面的标注（包含梁的高度）  
根据不同方向的标注实现是由AddDimension函数进行实现，其中的参数dimeMode表示标注方向的不同  
dimeMode=1，代表XY平面的标注；  
dimeMode=2，代表XZ平面的标注（竖直尺寸的标注）  
dimeMode=3，代表YZ平面的标注（竖直尺寸的标注）  

通过鼠标点选确定绘制位置：  
为了更方便地绘制楼板，本程序设置左下角的轴网交点为鼠标点选的基点，通过鼠标点选空间上一点的坐标，来设置轴网的基点，以此来计算其他坐标点。  
通过acedGetPoint()函数来实现

设置不同的显示模式：  
为了让自定义梁板实体类更好地进行显示，这里有两个显示模式可供选择，一是三维线框模式，二是着色实体模式。  
程序的逻辑顺序如下：  
首先通过acedSSGet函数，通过鼠标点选取当前图像中某个实体；  
然后通过一系列逻辑判断语句，来判断选中的是否为自定义的梁板实体，只有自定义实体才能改变显示形态。  
最后通过设置自定义梁板实体的m_iShowOption来改变其绘制形态。  

查询自定义实体的属性信息：  
首先通过acedSSGet函数，通过鼠标点选取当前图像中某个实体；  
然后通过一系列逻辑判断语句，来判断选中的是否为自定义的梁板实体，只有自定义实体才有属性信息可以查询；  
最后将属性信息acutPrintf()到控制台上  
