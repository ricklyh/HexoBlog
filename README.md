# Markdown 简明语法手册

### 语法
   + #### 1.1 段落与换行
   
     - 段落的前后必须是空行，空行指的是行内什么都没有，或者只有空白符（空格或制表符）
        相邻两行文本，如果中间没有空行 会显示在一行中（换行符被转换为空格）
      
     - 如果需要在段落内加入换行（<br>）可以在前一行的末尾加入至少两个空格然后换行写其它的文字
      
     - Markdown 中的多数区块都需要在两个空行之间。
   + #### 1.2 标题
     -  ``` 
            #h1级标题
            ##h2级标题
            ###h3级标题
            ####h4级标题
            #####h5级标题
            ######h6级标题
        ```
   + #### 1.3 分割线
      三个以上的-或者*即可作为分割线
        ```
            ----
            ****
        ```
   + #### 1.4 超链接    
     ```
      超链接：[连接名称](网址 , 标题)
      [我是链接名](http://www.izhangbo.cn, "我是标题")
      [<i class="icon-refresh"></i> 点我刷新](/sonfilename/)
      
      另一种超链接写法：[链接名][链接代号]
      [here][3]
      然后在别的地方定义 3 这个详细链接信息，
      [3]: http://www.izhangbo.cn "聚牛团队"
      
      直接展示链接的写法：<http://www.izhangbo.cn>
      
      键盘键
      <kbd>Ctrl+[</kbd> and <kbd>Ctrl+]</kbd>
      
      code格式：反引号
      Use the `printf()` function.
      
      ``There is a literal backtick (`) here.针对在代码区段内插入反引号的情况`` 
      
      强调：
      *斜体强调*
      **粗体强调**
     ```
   + #### 1.5 图片
     ```
      ![Alt text](http://www.izhangbo.cn/wp-content/themes/minty/img/logo.png "Optional title")
      
      使用 icon 图标文字
      <i class="icon-cog"></i
     ```
   + #### 1.6 列表
      ```
      使用数字和点表示有序列表:
      
         1. 有序列表项 一
            1. 子有序列表项 一
            2. 子有序列表项 二
         2. 有序列表项 二
         3. 有序列表项 三
      
      使用 *，+，- 表示无序列表
         + 无序列表项 一
         	- 子无序列表 一
         	- 子无序列表 二
         		* 子无序列表 三
         + 无序列表项 二
         + 无序列表项 三
      ```
   + #### 1.7 代码快
      ```
         这是用 ``` some codes ```表示的代码块（左侧无空格）；
      ```

### 文章头格式
      
      ```
         front-matter 的基本格式示例如下：
         ---
         title: Hexo Markdown-it 简明语法手册
         date: 2016-01-15 20:19:32
         tags: [Markdown,LaTex,教程]
         categories: 学习
         toc: true
         mathjax: true
         ---
         
      ```
### 注释
      [//]: 我是注释
      [^_^]: 我是注释
      
      

