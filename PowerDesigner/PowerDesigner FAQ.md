### NAME和COMMENT联动问题

Tools -> General Options-[Dialog](https://so.csdn.net/so/search?q=Dialog&spm=1001.2101.3001.7020) 中的Name to Code mirroring（不选中） 可以解决这个问题 。

![180012_QM43_2686282.png](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/180012_QM43_2686282.png)

![180014_FWEJ_2686282.png](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/180014_FWEJ_2686282.png)



### Table视图同时显示Code和Name

PowerDesigner中Table视图同时显示Code和Name,像下图这样的效果：

![image-20220209101409721](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220209101409721.png)

实现方法：Tools-Display Preference

![image-20220209101425020](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220209101425020.png)

![image-20220209101435142](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220209101435142.png)





### **pdm视图看不到表注释comment问题**

A: 配置comment attribute

 

1:Model->Extensions->2:点击Insert a row->4:右击General->A:右击ProFile->add mateclass->B:选择column->C:确定->6:右击column->extended attrubute ->8重命名，9:Data_Type:String ,10:选择Computed,11:选择read only->12:点击Get Method Script ->13:复制：%Get% = Rtf2Ascii (obj.Comment)->14:确定

![img](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/965102-20200518111844062-1892855117.png)

B: 选择自己配置的comment attribute

1:Tools->Display Perferences->2:Table->3:Advanced

 

->4:Columns->5:list Columns->6:lin_column->ok

 

 ![img](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/965102-20200518111920916-192854426.png)



### **视图显示Comment注释**

双击表->1：Columns->2：Customize Columns and Filter->3：comment(我的是自己配置的comment)->4：确定

**![img](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/965102-20200518112635549-1901344187.png)**



### NAME和COMMENT的互相转换

在使用PowerDesigner对数据库进行概念模型和物理模型设计时，一般在NAME或Comment中写中文，在Code中写英文。Name用来显示，Code在代码中使用，但Comment中的文字会保存到数据库Table或Column的Comment中，当Name已经存在的时候，再写一次 Comment很麻烦。

代码一：将Name中的字符COPY至Comment中，命名为name2comment.vbs

```
'把pd中那么name想自动添加到comment里面
'如果comment为空,则填入name;如果不为空,则保留不变,这样可以避免已有的注释丢失.

Option   Explicit 
ValidationMode   =   True 
InteractiveMode   =   im_Batch 

Dim   mdl   '   the   current   model 

'   get   the   current   active   model 
Set   mdl   =   ActiveModel 
If   (mdl   Is   Nothing)   Then 
      MsgBox   "There   is   no   current   Model " 
ElseIf   Not   mdl.IsKindOf(PdPDM.cls_Model)   Then 
      MsgBox   "The   current   model   is   not   an   Physical   Data   model. " 
Else 
      ProcessFolder   mdl 
End   If 

'   This   routine   copy   name   into   comment   for   each   table,   each   column   and   each   view 
'   of   the   current   folder 
Private   sub   ProcessFolder(folder)    
      Dim   Tab   'running     table    
      for   each   Tab   in   folder.tables    
            if   not   tab.isShortcut then
                     if  trim(tab.comment)="" then'如果有表的注释,则不改变它.如果没有表注释.则把name添加到注释里面.
                        tab.comment   =   tab.name
                     end if  
                  Dim   col   '   running   column    
                  for   each   col   in   tab.columns   
                        if trim(col.comment)="" then '如果col的comment为空,则填入name,如果已有注释,则不添加;这样可以避免已有注释丢失.
                           col.comment=   col.name   
                        end if 
                  next    
            end   if    
      next    
  
      Dim   view   'running   view    
      for   each   view   in   folder.Views    
            if   not   view.isShortcut and trim(view.comment)=""  then    
                  view.comment   =   view.name    
            end   if    
      next    
  
      '   go   into   the   sub-packages    
      Dim   f   '   running   folder    
      For   Each   f   In   folder.Packages    
            if   not   f.IsShortcut   then    
                  ProcessFolder   f    
            end   if    
      Next    
end   sub
```

另外在使用REVERSE ENGINEER从数据库反向生成PDM的时候，PDM中的表的NAME和CODE事实上都是CODE，为了把NAME替换为数据库中Table或Column的中文Comment，可以使用以下脚本：

代码二：将Comment中的字符COPY至Name中，命名为comment2name.vbs

```
Option   Explicit
ValidationMode   =   True
InteractiveMode   =   im_Batch
 
Dim   mdl   '   the   current   model
 
'   get   the   current   active   model
Set   mdl   =   ActiveModel
If   (mdl   Is   Nothing)   Then
      MsgBox   "There   is   no   current   Model "
ElseIf   Not   mdl.IsKindOf(PdPDM.cls_Model)   Then
      MsgBox   "The   current   model   is   not   an   Physical   Data   model. "
Else
      ProcessFolder   mdl
End   If
 
Private   sub   ProcessFolder(folder)
On Error Resume Next
      Dim   Tab   'running     table
      for   each   Tab   in   folder.tables
            if   not   tab.isShortcut   then
                  tab.name   =   tab.comment
                  Dim   col   '   running   column
                  for   each   col   in   tab.columns
                  if col.comment="" then
                  else
                        col.name=   col.comment
                  end if
                  next
            end   if
      next
 
      Dim   view   'running   view
      for   each   view   in   folder.Views
            if   not   view.isShortcut   then
                  view.name   =   view.comment
            end   if
      next
 
      '   go   into   the   sub-packages
      Dim   f   '   running   folder
      For   Each   f   In   folder.Packages
            if   not   f.IsShortcut   then
                  ProcessFolder   f
            end   if
      Next
end   sub
```

以上两段代码都是VB脚本，在PowerDesigner中使用方法为：

Tools->Execute Commands->Edit/Run Scripts

将代码Copy进去执行就可以了，是对整个CDM或PDM进行操作





### 导出sql脚本

1. **首先打开powerdesigner，可以通过文件打开一个项目或者直接双击项目通过powerdesigner进行打开。**

   ![](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/1.jpg)

2. 

   **修改导出数据库类型。**点击工具栏上的“Database”，选择“Change Current DBMS”进行修改导出脚本类型，可以选择mysql、sql server/ oracle 、db2等主流的数据库。

   ![](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/2.jpg)

3. 

   **在DBMS中点击下拉菜单，选择需要导出的数据库脚本，对名字进行自定义，然后点击确定即可。**

   ![](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/3.jpg)

4. 

   **导出脚本**。点击工具栏是上的“Database”，选择“Generate Database”生成数据库选项。

   ![](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/4.jpg)

5. 

   **保存文件。选择导出脚本路径，输入脚本名称。点击确定即可进行对项目中的表进行导出操作。**

   ![](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/5.jpg)

6. 

   如果软件没有报错信息，说明导出成功，可以在下面的**控制台中看到导出的日志**，以及导出的文件，**点击Edit可以对文件进行编辑**。

   ![](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/6.jpg)

   

   ![](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/7.jpg)